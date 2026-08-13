# Impersonation 
([Enterprise](https://readonlyrest.com/enterprise))

After describing what [the impersonation is](../kibana.md#impersonation), it's high time to see how ROR supports it and who and when could be interested in using this feature. Let's start with the latter.

## Use cases

The impersonation feature is intended for ROR administrators, rather than users. We can point out the two most obvious use cases when the admin could take advantage of the feature:

#### Debugging users' problems:

Let's imagine that some user has a problem with their ROR configuration (eg. the user doesn't have access to some feature that was blocked at ROR's level by you, the admin). And they are not able to clearly describe what the issue is (sounds familiar?). As an administrator, it would be extremely beneficial if you could see what the user sees. Thanks to the impersonation feature, an admin is allowed to impersonate the user and experience exactly what the user experiences.

#### Configuring a new user:

When an admin configures a new user in ROR settings, they face two problems:

1. `Will the updated configuration break the production cluster?`
2. `How do I know that the new user is correctly configured? Did I configure all their permissions correctly??`

Both of the problems can be solved using the ROR's impersonation. Thanks to the fact that the impersonation feature always uses its own Test Settings, that is completely independent from the main production settings, the admin can alter it without worries that their actions will break something and users won't be able to do their job.

Admin can add the new user configuration without worrying and then test it by impersonating the user. They can check if the user can log in without problems and if the user has access only to the Kibana features the admin wanted to grant. When the admin is sure that everything is configured correctly, they can promote the settings (test) to production.

## The impersonator/impersonated identity model

Before diving into the configuration, it's worth understanding *what an impersonating request actually contains* and *what ROR does with it*, since this is the part of the configuration that is most error-prone.

An impersonating request carries **two identities at once**:

* the **impersonator** - the real, credentialed admin sitting behind the keyboard (e.g. `admin1`), proven by whatever credentials travel with the request (typically an HTTP Basic Auth header),
* the **impersonated user** - the identity the admin wants to "borrow" for the duration of the request (e.g. `dev2`), carried in an internal header ROR/Kibana attaches to the request (`x-ror-impersonating`).

ROR has to answer two completely different questions before it lets the request through:

1. **"Is the real caller actually who they claim to be, and are they allowed to impersonate anyone at all?"** - this is a question about the *impersonator's* identity. It has nothing to do with `dev2`.
2. **"Given that we trust the caller, are they allowed to become `dev2` specifically, and does `dev2` even exist?"** - this is a question about the *impersonated user*, answered using the `impersonation` section and, where needed, [service mocks](#defining-mocks-of-the-external-services-optional).

This is why the `impersonation` section needs its own, explicit `authentication_rule`, separate from whatever rule authenticates users in `access_control_rules`. **It's not accidental duplication - the two rules answer different questions, for different identities, and they run at different times:**

* The rule in `access_control_rules` authenticates whoever is trying to act as `dev2` during `dev2`'s own, non-impersonating session. During impersonation, ROR deliberately does **not** execute that rule's normal authentication logic - that's the entire point of impersonation: it lets an admin experience `dev2`'s permissions without needing `dev2`'s actual password, and without ROR having to make a call to `dev2`'s LDAP/external backend (that's also why [mocks](#defining-mocks-of-the-external-services-optional) exist - the impersonated identity's data is simulated, not fetched from the backend).
* The `authentication_rule` inside the matching `impersonation` entry authenticates the **real caller** (`admin1`) using the credentials that are actually present on the wire. It has to be spelled out explicitly because there is no other rule anywhere in the ACL whose job is "verify this is really `admin1`, right now, for the purpose of impersonation" - the block rules are all written with the *impersonated* users in mind, not the impersonator.

In practice, admins usually configure the same credentials/mechanism for `admin1` in both places, because `admin1` authenticates the same way whether they're doing their own work or impersonating someone. That similarity is exactly what makes the two entries *look* like copy-pasted duplication - but they are checked by different execution paths, at different moments, and there's nothing stopping you from requiring a different (e.g. stronger) authentication method just for impersonation.

### What actually happens, step by step

1. The HTTP request arrives carrying the impersonator's own Basic Auth credentials (`Authorization: Basic ...`) plus the `x-ror-impersonating: <target-username>` header.
2. Before any ACL rule is evaluated, ROR decides which settings apply to the request. The mere presence of the `x-ror-impersonating` header makes ROR evaluate the request against **Test Settings** - never against Main Settings, even if Main Settings also happens to define an `impersonation` section of its own. If Test Settings aren't currently active (never applied, expired, or manually invalidated), the request is rejected immediately with `TEST_SETTINGS_NOT_CONFIGURED`, before any block gets a chance to run. See [Creating ROR's Test Settings](#creating-rors-test-settings) for how long Test Settings stay active.
3. ROR starts evaluating the Test Settings' `access_control_rules` blocks as usual, top to bottom.
4. The moment ROR reaches **any** authentication rule (`auth_key`, `ldap_authentication`, `external_authentication`, ...) inside a block, it notices the impersonation header and **does not run that rule's normal logic at all**. Instead, it switches into the impersonation flow below - the specific rule/type written in that block is irrelevant to this step; only the fact that it *is* an authentication rule matters.
5. ROR extracts the impersonator's username from the request's Basic Auth header, then searches the `impersonation` section, top to bottom, for the first entry where:
   * the `impersonator` pattern matches that username, **and**
   * the `users` pattern matches the target username from the `x-ror-impersonating` header.

   No match → the request is denied with `IMPERSONATION_NOT_ALLOWED`, regardless of whether `admin1` is a perfectly valid, authenticated user elsewhere in the ACL.
6. ROR authenticates the request's Basic Auth credentials against **that entry's own `authentication_rule`** - a fresh, independent check, unrelated to the block ROR happened to be evaluating. Failure → `IMPERSONATION_NOT_ALLOWED`.
7. ROR checks that the impersonator and the impersonated user aren't the same username (self-impersonation is rejected), and that the impersonated user actually exists - either because it's a statically configured user, or because a [service mock](#defining-mocks-of-the-external-services-optional) confirms it. If the existence check can't be performed at all (e.g. the rule doesn't support impersonation, or the required mock is missing), the request is denied with `IMPERSONATION_NOT_SUPPORTED`.
8. If everything checks out, ROR treats the impersonated user as logged in and continues evaluating the rest of the ACL (groups, indices, Kibana rules, etc.) exactly as it would for that user's real session - using mocked data wherever an external system would normally be consulted.

One consequence worth calling out: steps 5-7 are re-run independently every time ROR reaches an authentication rule while trying to match a block, regardless of which authentication rule is written in that particular block. If your ACL has three blocks with three different auth rules (say `auth_key`, `ldap_authentication`, `external_authentication`), all three behave identically under impersonation - they all defer to the *same* `impersonation` section entry. Only the rules matching the impersonator's username/target-user pattern decide the outcome, not whichever rule happens to sit in the block.

## Impersonation configuration

Before an admin will be able to impersonate a user, they have to configure ROR properly. The configuration consists of several parts:

1. creating ROR's Test Settings,
2. defining mocks of the external services (like [LDAP](../elasticsearch.md#ldap-connector), [External Basic Auth](../elasticsearch.md#external-basic-auth) or [Custom groups provider](../elasticsearch.md#custom-groups-providers)),
3. impersonating a chosen user.

#### Creating ROR's Test Settings

When you call Elasticsearch directly or through ROR Kibana, ROR ACL is defined by Settings (we can assume they are Main Settings). The Test Settings define another ACL, that is taken into consideration by ROR ES only when a proper impersonation header is passed. The header is managed by ROR internally. The Test Settings are active only for a strictly defined amount of time (by default it's _30 minutes_, but the admin can change it before applying Test Settings). After the time has expired, they are automatically invalidated (for security reasons). Obviously, the admin is allowed to invalidate the configured Test Settings in any time. There is no way to have more than one Test Settings configured at time.

ROR Kibana plugin provides a dedicated Test Settings UI. See our [Test Settings management guide](../examples/impersonation/test-settings-ui.md) for more information.

**This TTL gates impersonation directly.** As described in [step 2 above](#what-actually-happens-step-by-step), every impersonating request is evaluated exclusively against Test Settings - never against Main Settings. Once Test Settings expire or are invalidated, there are no Test Settings left to evaluate the request against, so impersonation stops working immediately with `TEST_SETTINGS_NOT_CONFIGURED`, regardless of how the `impersonation` section itself is configured. This is a distinct failure mode from a misconfigured `impersonation` entry, and it's worth ruling out first: re-apply Test Settings and retry before troubleshooting anything else.

But copying Main Settings as Test Settings is not enough. We also have to instruct ROR which users can be considered as impersonators (the ones, who are allowed to impersonate other users). Concretely, three things have to line up, matching the three questions/checks described [above](#what-actually-happens-step-by-step):

1. In the `access_control_rules` section in ROR Settings, there must be a rule that authenticates the impersonator user *for their own, normal (non-impersonating) session* - this is what grants `admin1` their everyday permissions, and it's a prerequisite for `admin1` being a legitimate ES/Kibana user at all.
2. The impersonator user must be defined in the `impersonation` section in ROR Settings, together with the list/pattern of users they're allowed to impersonate. **Being a valid, authenticated user in `access_control_rules` is not enough** - without a matching `impersonation` entry, ROR will refuse impersonation with `IMPERSONATION_NOT_ALLOWED`, even for a perfectly legitimate admin.
3. The impersonator's credentials must satisfy the `authentication_rule` configured *inside that `impersonation` entry* - this rule is evaluated completely independently of rule #1, using whatever credentials the impersonating request actually carries. It commonly mirrors rule #1 (same rule type, same credentials), but ROR never assumes that - it always re-checks.

```yaml
readonlyrest:
  access_control_rules:
    - name: "Authenticate admin1"
      auth_key: admin1:pass
    - name: "Authenticate admin2"
      ldap_authentication: "ldap1"

  impersonation:
    - impersonator: admin1      // Who can impersonate? (user name or pattern)
      users: ["*"]              // Who can be impersonated? (user names or patterns)
      auth_key: admin1:pass     // Authentication rule required to impersonate (any authentication rule can be used here)
    - impersonator: admin2
      users: ["dev2"]
      ldap_authentication: "ldap1"
```

In the example above, we see that we have two impersonators: `admin1` and `admin2`. The first one can impersonate any user (`*`) and they are able to authenticate using basic auth (`admin1:pass`). The second impersonator can impersonate only `dev2` user. They will be authenticated using `ldap1` connector.

When an impersonator passes wrong credentials ROR will tell Kibana that impersonation is not allowed.

A few structural rules ROR enforces when it loads this section (and that are worth knowing, since they explain some of the config-load errors you might see):

* Exactly one authentication rule is allowed per `impersonation` entry - you can't stack several auth methods for a single impersonator.
* Only rules that are genuinely authentication rules (`auth_key*`, `ldap_authentication`, `external_authentication`, `proxy_auth`, ...) can be used here - authorization-only rules (like `ldap_authorization`) are rejected.
* The same exact username (no wildcards) cannot appear as both an `impersonator` and a member of `users` in the same entry - a user can't be declared as being able to impersonate themselves.
* If the `authentication_rule` has a statically known, fixed username (e.g. `auth_key: someone:pass`), ROR checks at load time that this username actually matches the `impersonator` pattern, and refuses to start otherwise. This check can't be done for dynamic identities (LDAP, external auth), since the username isn't known until request time.

#### Defining mocks of the external services (optional)

ROR has many sophisticated authentication & authorization methods. Some of them are based on external systems like LDAP. The problem with such systems, in regard to to the impersonation feature, is that those systems either don't support it by default or don't support it at all and even if they do - the configuration is complex.

That's why we decided to solve it totally differently - using mocks. [Wikipedia](https://en.wiktionary.org/wiki/mock) defines `mock` as `an imitation, usually of lesser quality.` And in the case of external authentication systems we are going provide an imitation of it that will tell ACL which users should be successfully authenticated by it. When we consider an authorization service, a mock of it will return the ACL users with their roles in the service. And this is enough for ROR to support impersonation.

How does ROR use the mocks? Let's suppose we have an `ldap_auth` rule. When ROR processes the rule, it:

* asks the given LDAP service if the username can be authenticated with a given password, and if they can ...
* asks LDAP to list what groups the user belongs to

In the impersonation case, it looks pretty much the same. The difference being that ROR won't call any LDAP server - the mock will provide the required information instead (no password required). During impersonating, when ROR processes an LDAP rule, it:

* asks the mock if the username exists, and if it does ...
* asks the mock to tell what groups the user belongs to

**⚠️ IMPORTANT:** If one or more of the external services are not mocked, ROR might inform Kibana that the impersonation is not supported. It's better to always define all mocks, to avoid the "Impersonation not supported" Elasticsearch response.

ROR Kibana plugin helps administrators to visually create and edit service mocks with a dedicated graphical UI. Follow our [service mock configuration guide](../examples/impersonation/external-services-mocks-ui.md) for more.

#### Which rules support impersonation

Impersonation support isn't the same for every rule that can appear in an `access_control_rules` block. ROR checks this per rule and, when it applies Test Settings, reports a warning for each rule/block combination that won't work correctly during impersonation - these warnings surface through the Test Settings API and are shown by the ROR Kibana Test Settings UI.

| Rule                                                                                                                                      | Impersonation support    | Notes                                                                                                                                                                                                                                                     |
|-------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `auth_key`, `auth_key_unix`, `proxy_auth`, `token_authentication`                                                                         | Full                     | Work as-is, no extra configuration needed                                                                                                                                                                                                                 |
| Group-membership rules (`groups_any_of`, `groups_all_of`, and other [groups logic](authorization-rules-details.md#checking-groups-logic)) | Full                     | Groups are supplied directly in settings, or by an authorization rule that's itself impersonation-aware; no external call is involved                                                                                                                     |
| `auth_key_sha1`, `auth_key_sha256`, `auth_key_sha512`, `auth_key_pbkdf2_hmac_sha512`                                                      | Full, with one condition | Only works when the rule is written in the `USER_NAME:hash(PASSWORD)` form. A fully hashed `hash(USER_NAME:PASSWORD)` blob can't be reversed back to a username, so it never matches during impersonation - see [limitations](#impersonation-limitations) |
| `ldap_authentication`, `ldap_authorization`, `ldap_auth`                                                                                  | Requires a mock          | Needs a matching LDAP service mock (see [above](#defining-mocks-of-the-external-services-optional)); without it, denied with `IMPERSONATION_NOT_SUPPORTED`                                                                                                |
| `external_authentication`                                                                                                                 | Requires a mock          | Needs an external authentication service mock                                                                                                                                                                                                             |
| `external_authorization`                                                                                                                  | Requires a mock          | Needs an external authorization service mock                                                                                                                                                                                                              |
| `jwt_auth`, `jwt_authentication`, `jwt_authorization`                                                                                     | Not supported            | Always denied with `IMPERSONATION_NOT_SUPPORTED`; there is no mock or workaround for this today                                                                                                                                                           |
| `ror_kbn_auth`, `ror_kbn_authentication`, `ror_kbn_authorization`                                                                         | Not supported            | Always denied with `IMPERSONATION_NOT_SUPPORTED`; there is no mock or workaround for this today                                                                                                                                                           |
| Everything else (`indices`, `actions`, `kibana_*`, `fields`, `filter`, `hosts`, `uri_re`, ...)                                            | Not applicable           | These rules don't authenticate or authorize an identity - they evaluate normally against whichever user, real or impersonated, is already logged in                                                                                                       |

A block only needs to be fully impersonation-capable if you intend to impersonate the users it applies to. A block built entirely around a "not supported" rule simply can't be exercised through impersonation - traffic that would otherwise match it falls through to later blocks, exactly as it would if the block rejected the request for any other reason.

#### Impersonating a chosen user

Now that we have configured Test Settings and External Services Mocks, we can try to impersonate a user. In Elasticsearch ROR Settings, user can be:

* provided statically (defined in the settings),
* provided dynamically:
  * from external, dependant systems (like LDAP) - we mock them
  * from upstream systems (eg. through headers) - they are not known upfront

It means that we pick the users defined in Settings or Mocks, but also we can enter the username and try to impersonate such user.

Follow the instructions on how to [impersonate a user using the ROR Kibana plugin UI](../examples/impersonation/impersonate-user-ui.md).

## Full end-to-end examples

### Example 1: local admin impersonating any local user

Everything is defined statically, no mocks needed - the simplest possible setup.

```yaml
readonlyrest:
  access_control_rules:

    - name: "Admins"
      auth_key: admin1:pass
      groups_any_of: ["admins"]

    - name: "Devs"
      auth_key: dev2:devpass
      groups_any_of: ["devs"]
      indices: ["dev-*"]

  users:
    - username: admin1
      auth_key: admin1:pass
      groups: ["admins"]

    - username: dev2
      auth_key: dev2:devpass
      groups: ["devs"]

  impersonation:
    - impersonator: admin1
      users: ["*"]
      auth_key: admin1:pass   # re-checks the SAME credentials admin1 used to authenticate - but independently
```

What happens when Kibana sends a request with `Authorization: Basic YWRtaW4xOnBhc3M=` (i.e. `admin1:pass`) and `x-ror-impersonating: dev2`:

1. ROR reaches the "Admins" block first. Its `auth_key` rule notices the impersonation header and defers to the `impersonation` section instead of comparing `admin1:pass` against its own settings.
2. `admin1` matches the `impersonator` pattern, `dev2` matches `users: ["*"]`.
3. The `impersonation` entry's own `auth_key: admin1:pass` is checked against the request's credentials - it matches, so the caller is confirmed to really be `admin1`.
4. `dev2` is a statically defined local user, so its existence is confirmed without needing a mock.
5. ROR now evaluates the rest of the ACL as `dev2`: the "Admins" block's `groups_any_of: ["admins"]` doesn't match `dev2`'s groups, so it's skipped; the "Devs" block matches, and the response is scoped to `dev-*` indices - exactly what `dev2` would see in their own session.

### Example 2: LDAP-authenticated admin impersonating an LDAP-authorized user

Here the impersonated user's group membership comes from LDAP, so it needs a mock.

```yaml
readonlyrest:
  access_control_rules:

    - name: "LDAP admins can do everything"
      ldap_authentication: "ldap1"
      ldap_authorization:
        name: "ldap1"
        groups_any_of: ["admins"]

    - name: "LDAP devs see only their indices"
      ldap_auth:
        name: "ldap1"
        groups_any_of: ["devs"]
      indices: ["@{acl:user}_*"]

  ldaps:
    - name: ldap1
      host: ldap.example.com
      port: 389
      # ... rest of the connector settings

  impersonation:
    - impersonator: admin1
      users: ["dev2"]
      ldap_authentication: "ldap1"   # admin1's own LDAP credentials, checked independently
```

For this to work during impersonation, a **Test Settings mock** for `ldap1` must define `dev2` as an existing user belonging to the `devs` group - see [Defining mocks of the external services](#defining-mocks-of-the-external-services-optional). Without it, both LDAP rules will refuse to evaluate `dev2` and the request is denied with `IMPERSONATION_NOT_SUPPORTED`, even though `admin1`'s own impersonator authentication succeeded.

## Common misconfigurations

Most support tickets about "impersonation isn't working" trace back to one of these. The middle column is what you'll typically see in the ES response/ROR logs (`USR` field, or the `causes`/reason returned to Kibana).

| Symptom                                                                                                                     | What ROR reports                                                 | Most common root cause                                                                                                                                                                                                                                                                                                                                                 | Fix                                                                                                                                            |
|-----------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| Impersonation worked earlier in the session but every impersonating request now fails, though nothing in the config changed | `TEST_SETTINGS_NOT_CONFIGURED`                                   | Test Settings expired (default TTL is 30 minutes) or were manually invalidated. Every impersonating request is evaluated exclusively against Test Settings (see [step 2](#what-actually-happens-step-by-step)), so once they're gone, impersonation stops working regardless of the `impersonation` section                                                            | Re-apply Test Settings (optionally with a longer TTL) and retry                                                                                |
| Admin can log into Kibana fine, but impersonation is refused outright                                                       | `IMPERSONATION_NOT_ALLOWED`                                      | There's no `impersonation` entry at all for this admin - being authenticated in `access_control_rules` does **not** automatically grant impersonation rights                                                                                                                                                                                                           | Add an `impersonation` entry with an `impersonator` pattern matching the admin                                                                 |
| Impersonation works for some target users but not others                                                                    | `IMPERSONATION_NOT_ALLOWED`                                      | The `users` pattern in the matching `impersonation` entry doesn't include the requested target username                                                                                                                                                                                                                                                                | Broaden the `users` pattern, or add a dedicated entry                                                                                          |
| Admin's password was recently changed and impersonation broke, even though normal login still works                         | `IMPERSONATION_NOT_ALLOWED`                                      | The `authentication_rule` inside `impersonation` is checked completely independently of the rule in `access_control_rules` - updating one does not update the other                                                                                                                                                                                                    | Keep both in sync, or point both at the same external identity source (LDAP/external auth) instead of hardcoding credentials twice             |
| Config fails to load at startup, mentioning "should be either impersonator or a user to be impersonated"                    | Config validation error                                          | The exact same username (no wildcards) appears in both `impersonator` and `users` in one entry                                                                                                                                                                                                                                                                         | Remove the self-reference - a user can't be declared as able to impersonate themselves                                                         |
| Config fails to load at startup, mentioning "it's used in a context of user patterns"                                       | Config validation error                                          | The `impersonation` entry's `authentication_rule` has a fixed, statically known username that doesn't match the `impersonator` pattern (e.g. `impersonator: admin1` but `auth_key: someone_else:pass`)                                                                                                                                                                 | Make the rule's username match the `impersonator` pattern                                                                                      |
| Impersonator authenticates fine, but the request is still denied, mentioning the impersonated user doesn't exist            | Denied, logged as `AUTH_FAIL (Impersonated user does not exist)` | The target username isn't a statically configured user and isn't present in the relevant service mock                                                                                                                                                                                                                                                                  | Add the user to the mock, or confirm the username matches exactly                                                                              |
| Request denied with "impersonation not supported", even though the `impersonation` section looks correct                    | `IMPERSONATION_NOT_SUPPORTED`                                    | An ACL block relies on a rule that either doesn't support impersonation at all (`jwt_auth`, `ror_kbn_auth` and their authentication/authorization variants), or needs a service mock (LDAP/external auth/external authz) that hasn't been configured yet, or uses `auth_key_sha*` with a fully-hashed `user:pass` blob (see [limitations](#impersonation-limitations)) | Add the missing mock, switch to the `USER_NAME:hash(PASSWORD)` form for hashed auth rules, or accept that the rule type isn't impersonable yet |
| Impersonation UI can't find/list the user you want to impersonate                                                           | N/A (UI limitation)                                              | The target username is only reachable through a wildcard `users` pattern in the ACL, so ROR can't enumerate it upfront                                                                                                                                                                                                                                                 | Type the username manually in the impersonation UI, as described in [limitations](#impersonation-limitations)                                  |
| Nothing happens / impersonation is silently ignored even though credentials and patterns look correct                       | Request is treated as a normal (non-impersonating) request       | The impersonating client didn't send the impersonator's credentials as an HTTP Basic Auth header - ROR always identifies the impersonator from Basic Auth, regardless of which rule type is configured as the `authentication_rule`                                                                                                                                    | Make sure the client authenticates with Basic Auth (this is what the ROR Kibana Test Settings UI does under the hood)                          |

## Logs & audit

In Elasticsearch logs, in `USR` field, if an admin user finds something like this: `admin1 (as user1)` - it means that `admin1` was authenticated, and they are the impersonator who is impersonating `user1`.

All logs of impersonated user in Kibana will have this format `[<log level>][plugins][ReadonlyREST][<filename>][impersonating <impersonated user username>]`

When auditing is enabled, the audit document is going to contain an `impersonated_by` field.

## Impersonation limitations

Impersonation mode has some limitations. Please check if they have an impact on your use cases:

* Not all features available in the ROR configuration are testable with impersonation mode. Some rules used in ROR ACL do not support impersonation. For example, auth rule with hashed credentials (e.g. `auth_key_sha512`) can be used in impersonation mode only when credentials follow the format `USER_NAME: HASH(PASSWORD)`; A fully hashed username and password don't allow fetching a username. The auth rule in such a format won't match during impersonation. In the [rules description](../elasticsearch.md#rules) section you can find information about each rules impersonation support.
* Test Settings are stored in the memory of the node that handled the saving request sent by ROR Kibana plugin. Impersonation support will be limited to this node. We are going to improve it in the future, but for now your Kibana should only communicate with one Elasticsearch node.
*   Sometimes it is impossible to fetch usernames defined in the Test Settings. If a `users` rule contains a username pattern with a wildcard, to impersonate a user matching the pattern, you need to enter the username manually.

    ```yaml
    readonlyrest:

      access_control_rules:
        - name: "LDAP group g1"
          type: allow
          groups_any_of: ["g1"]
        
      users:
        - username: "admin*"  // To impersonate a user with a username matching 'admin*' you need to enter the username manually, like 'admin123'
          groups:
            - g1: group1
          ldap_auth:
            name: "ldap1"
            groups_any_of: ["group1"]
          
      ldaps:
        - name: ldap1
          [..]
          
      impersonation:
        [...]
    ```

## Glossary

* **Impersonator** - someone who imitates or copies the behavior or actions of another,
* **Impersonation** - imitating behaviors or actions of a given user,
* **Impersonated user** - the identity being borrowed for the duration of an impersonation session; their permissions/data determine what the impersonator sees, but their own credentials are never needed or checked,
* **Main Settings** - the ROR's settings that apply to ACL that handles requests during regular sessions (not the impersonation ones),
* **Test Settings** - the ROR's settings that apply to ACL that handles impersonating requests (the ones during impersonation session),
* **External Service Mock** - an imitation of an external service (the supported ones: LDAP, an external authentication service, an external authorization service).
