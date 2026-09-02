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

## How ROR processes an impersonation request

An impersonating request is almost identical to the request the impersonated user would send themselves. It reaches the same cluster and is evaluated by the same ACL, block by block, in the same order. Rules like `indices`, `kibana_*`, `fields`, `filter` or `hosts` see exactly what they would see in that user's own session - which is what makes impersonation useful for testing a configuration in the first place.

Two things are different:

* **the authentication data the request carries** - the credentials on the wire belong to the *impersonator* (say, `alice`), while the identity to be evaluated - the *impersonated user* (say, `bob`) - travels separately, in an internal header managed by ROR and its Kibana plugin,
* **the way authentication and authorization rules behave** - and that is where the whole feature lives.

The rest of this section is about that second difference.

### Impersonating requests use Test Settings

ROR keeps two independent sets of settings:

* **Main Settings** - the ACL that handles regular traffic,
* **Test Settings** - a separate ACL, used only for impersonation. They are a scratchpad: the admin can edit them freely, without any risk to the users working against Main Settings, and promote them to Main Settings once the result is satisfying.

An impersonating request is always evaluated against Test Settings, never against Main Settings. Test Settings stay active only for a limited time and can be invalidated at any moment (see [Creating ROR's Test Settings](#creating-rors-test-settings)); when none are active, impersonation simply doesn't work, no matter how the rest of the configuration looks.

### The impersonator and the impersonated user are authorized differently

`alice` can appear in ROR settings in two completely independent roles:

* **as a regular user** - authenticated by an authentication rule, which is a part of a block, which is one of many in the ACL, exactly like anybody else. This is what lets `alice` log into Kibana and do her own work.
* **as an impersonator** - declared in the `impersonation` section, a separate part of ROR settings saying who may impersonate whom, and how such an impersonator is authenticated (see [Impersonation configuration](#impersonation-configuration)).

Neither role implies the other, and that is deliberate:

* An impersonator who never logs into Kibana needs only an `impersonation` entry. They can impersonate users through the Elasticsearch REST API without being a regular ACL user at all.
* If `alice` should also log into Kibana as herself, some ACL block has to authenticate her as a regular user. The `impersonation` section grants no everyday access whatsoever.

So, when a request comes in as `alice` herself, it is authorized by the ACL like any other request. When the same `alice` sends a request on behalf of `bob`, the authentication rules in the ACL blocks no longer answer the question "who is the caller?" - the `impersonation` section does. This is why the impersonator's authentication has to be spelled out there explicitly: it is the only place in the settings whose job is to verify that the caller really is `alice`, right now, for the purpose of impersonation.

### Not every authentication rule supports impersonation

The first thing an authentication rule does with an impersonating request is to check whether it supports impersonation at all (the full list is in [Which rules support impersonation](#which-rules-support-impersonation)). The two paths are completely different.

**When the rule doesn't support impersonation** (`jwt_*`, `ror_kbn_*`), it takes no part in the impersonation flow. It evaluates the request the way it always does: it looks for a JWT or a ROR Kibana token, finds none (the request carries the impersonator's Basic Auth credentials instead), and doesn't match - so its block doesn't match either, and ROR moves on to the next block. A block built around such a rule cannot be exercised through impersonation at all; ROR points this out with a warning when Test Settings are applied.

**When the rule does support impersonation**, it skips its normal authentication logic entirely and hands the request over to the impersonation flow, which answers three questions, in order:

1. **Is `alice` allowed to impersonate `bob`?** The impersonator is identified by the Basic Auth credentials carried by the request, and looked up in the `impersonation` section. If nothing there allows this particular pair, the request is denied.
2. **Is the caller really `alice`?** Her credentials are verified against the authentication rule of the matching `impersonation` entry - independently of the block ROR happens to be evaluating, and independently of whatever rule authenticates `alice` as a regular user. Trying to impersonate oneself is rejected here as well.
3. **Does `bob` exist?** This one is answered by the very rule ROR is currently evaluating, using only what that rule knows. A rule holding static credentials knows the usernames written next to it. A rule backed by an external system (LDAP, an external authentication or authorization service) would normally have to ask that system - during impersonation it asks a [mock](#defining-mocks-of-the-external-services-optional) of it instead, so that no real account, no password and no connection to the production service are needed.

The answer to the last question decides the fate of the block:

* the rule knows `bob` → ROR treats the request as logged in as `bob`, and the rest of the block (groups, indices, Kibana rules, ...) is evaluated as `bob`, using mocked data wherever an external system would normally be consulted,
* the rule can answer, and the answer is "I don't know this user" → this block doesn't match, and ROR moves on to the next one. In a multi-block ACL this is perfectly normal: only the block that actually defines `bob` can match him, and the others log `AUTH_FAIL (Impersonated user does not exist)` along the way,
* the rule cannot answer at all - a missing service mock, or an `auth_key_sha*` rule whose whole `user:pass` pair is hashed and can't be reversed back to a username → the request is denied, and ROR reports that the impersonation is not supported.

These three questions are asked from scratch in every block ROR tries. The first two always get the same answer - they depend only on the `impersonation` section, never on the block. The third one is what differs: each rule answers it with its own knowledge, and that is what makes exactly one block match while the others fall through.

## Impersonation configuration

Before an admin will be able to impersonate a user, they have to configure ROR properly. The configuration consists of several parts:

1. creating ROR's Test Settings,
2. defining mocks of the external services (like [LDAP](../elasticsearch.md#ldap-connector), [External Basic Auth](../elasticsearch.md#external-basic-auth) or [Custom groups provider](../elasticsearch.md#custom-groups-providers)),
3. impersonating a chosen user.

#### Creating ROR's Test Settings

When you call Elasticsearch directly or through ROR Kibana, ROR ACL is defined by Settings (we can assume they are Main Settings). The Test Settings define another ACL, that is taken into consideration by ROR ES only when a proper impersonation header is passed. The header is managed by ROR internally. The Test Settings are active only for a strictly defined amount of time (by default it's _30 minutes_, but the admin can change it before applying Test Settings). After the time has expired, they are automatically invalidated (for security reasons). Obviously, the admin is allowed to invalidate the configured Test Settings in any time. There is no way to have more than one Test Settings configured at time.

ROR Kibana plugin provides a dedicated Test Settings UI. See our [Test Settings management guide](../examples/impersonation/test-settings-ui.md) for more information.

**This TTL gates impersonation directly.** As described [above](#impersonating-requests-use-test-settings), every impersonating request is evaluated exclusively against Test Settings. Once they expire or are invalidated, there is nothing left to evaluate such a request against, so impersonation stops working immediately, regardless of how the `impersonation` section itself is configured. This is a distinct failure mode from a misconfigured `impersonation` entry, and it's worth ruling out first: re-apply Test Settings and retry before troubleshooting anything else.

But copying Main Settings as Test Settings is not enough. We also have to instruct ROR which users can be considered as impersonators (the ones, who are allowed to impersonate other users). As described [above](#the-impersonator-and-the-impersonated-user-are-authorized-differently), this is what the `impersonation` section is for:

1. The impersonator must be declared in the `impersonation` section of ROR Settings, together with the list/pattern of users they are allowed to impersonate. **Being a valid, authenticated user in `access_control_rules` is not enough** - without a matching entry here, ROR refuses the impersonation, even for a perfectly legitimate admin.
2. The impersonator's credentials must satisfy the authentication rule configured *inside that entry*. It is evaluated completely independently of anything in `access_control_rules`, using whatever credentials the impersonating request actually carries.
3. Only if the impersonator is also supposed to work with Kibana as themselves, `access_control_rules` needs a block authenticating them as a regular user. Such a block plays no part in impersonation - it's what grants them their everyday access.

```yaml
readonlyrest:
  access_control_rules:
    - name: "Authenticate alice"
      auth_key: alice:pass
    - name: "Authenticate carol"
      ldap_authentication: "ldap1"

  impersonation:
    - impersonator: alice      // Who can impersonate? (user name or pattern)
      users: ["*"]              // Who can be impersonated? (user names or patterns)
      auth_key: alice:pass     // Authentication rule required to impersonate (any authentication rule can be used here)
    - impersonator: carol
      users: ["bob"]
      ldap_authentication: "ldap1"
```

In the example above, we see that we have two impersonators: `alice` and `carol`. The first one can impersonate any user (`*`) and they are able to authenticate using basic auth (`alice:pass`). The second impersonator can impersonate only `bob` user. They will be authenticated using `ldap1` connector.

When an impersonator passes wrong credentials ROR will tell Kibana that impersonation is not allowed.

The order of the entries matters. ROR uses the **first** entry whose `impersonator` pattern matches the caller, and only that one - it never falls through to a later entry, even if that one would allow the requested impersonated user. With overlapping patterns (e.g. `admin*` before `alice`), the `users` list of the later entry is dead configuration. Prefer one entry per impersonator, and put the most specific patterns first.

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
| `ldap_authentication`, `ldap_authorization`, `ldap_auth`                                                                                  | Requires a mock          | Needs a matching LDAP service mock (see [above](#defining-mocks-of-the-external-services-optional)); without it, ROR reports that impersonation is not supported                                                                                                |
| `external_authentication`                                                                                                                 | Requires a mock          | Needs an external authentication service mock                                                                                                                                                                                                             |
| `external_authorization`                                                                                                                  | Requires a mock          | Needs an external authorization service mock                                                                                                                                                                                                              |
| `jwt_auth`, `jwt_authentication`, `jwt_authorization`                                                                                     | Not supported            | These rules ignore impersonation entirely: they evaluate the real request, find no JWT in it, fail authentication and their block doesn't match. No mock or workaround today - ROR reports a Test Settings warning for such blocks                        |
| `ror_kbn_auth`, `ror_kbn_authentication`, `ror_kbn_authorization`                                                                         | Not supported            | Same as above, with the ROR Kibana token: the block can't be exercised through impersonation, and ROR reports a Test Settings warning for it                                                                                                              |
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
      auth_key: alice:pass
      groups_any_of: ["admins"]

    - name: "Devs"
      auth_key: bob:bobpass
      groups_any_of: ["devs"]
      indices: ["dev-*"]

  users:
    - username: alice
      auth_key: alice:pass
      groups: ["admins"]

    - username: bob
      auth_key: bob:bobpass
      groups: ["devs"]

  impersonation:
    - impersonator: alice
      users: ["*"]
      auth_key: alice:pass   # re-checks the SAME credentials alice used to authenticate - but independently
```

What happens when Kibana sends a request with `Authorization: Basic YWxpY2U6cGFzcw==` (i.e. `alice:pass`) and `x-ror-impersonating: bob`:

1. ROR reaches the "Admins" block first. Its `auth_key` rule notices the impersonation header and defers to the `impersonation` section instead of comparing `alice:pass` against its own settings.
2. `alice` matches the `impersonator` pattern, `bob` matches `users: ["*"]`.
3. The `impersonation` entry's own `auth_key: alice:pass` is checked against the request's credentials - it matches, so the caller is confirmed to really be `alice`.
4. Finally, ROR asks the rule it is currently evaluating - the "Admins" block's `auth_key: alice:pass` - whether `bob` exists. That rule knows only `alice`, so the answer is no: the **"Admins" block is rejected** (logged as `AUTH_FAIL (Impersonated user does not exist)`) and ROR moves on to the next block. That log line is expected here, not a misconfiguration - the "Admins" block is simply not the block that defines `bob`.
5. In the "Devs" block, steps 1-3 repeat identically, and this time the block's own `auth_key: bob:bobpass` rule confirms that `bob` exists (a statically defined local user, so no mock is needed). ROR marks the request as logged in as `bob` and evaluates the remaining rules as `bob`: `groups_any_of: ["devs"]` matches and the response is scoped to `dev-*` indices - exactly what `bob` would see in their own session.

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
    - impersonator: alice
      users: ["bob"]
      ldap_authentication: "ldap1"   # alice's own LDAP credentials, checked independently
```

For this to work during impersonation, a **Test Settings mock** for `ldap1` must define `bob` as an existing user belonging to the `devs` group - see [Defining mocks of the external services](#defining-mocks-of-the-external-services-optional). Without it, both LDAP rules will refuse to evaluate `bob` and the request is denied as not supported, even though `alice`'s own impersonator authentication succeeded.

## Common misconfigurations

Most support tickets about "impersonation isn't working" trace back to one of these. The middle column is what you'll typically see in Kibana, in the ES response, or in the ROR logs.

| Symptom                                                                                                                                  | What ROR reports                                                   | Most common root cause                                                                                                                                                                                                                                                                                                                                   | Fix                                                                                                                                |
|------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| Impersonation worked earlier in the session but every impersonating request now fails, though nothing in the config changed              | Kibana reports that no Test Settings are configured                | Test Settings expired (default TTL is 30 minutes) or were manually invalidated. Every impersonating request is evaluated exclusively against Test Settings (see [above](#impersonating-requests-use-test-settings)), so once they're gone, impersonation stops working regardless of the `impersonation` section                                              | Re-apply Test Settings (optionally with a longer TTL) and retry                                                                    |
| Admin can log into Kibana fine, but impersonation is refused outright                                                                    | Impersonation not allowed                                          | There's no `impersonation` entry at all for this admin - being authenticated in `access_control_rules` does **not** automatically grant impersonation rights                                                                                                                                                                                             | Add an `impersonation` entry with an `impersonator` pattern matching the admin                                                     |
| Impersonation works for some target users but not others                                                                                 | Impersonation not allowed                                          | The `users` pattern of the **first** `impersonation` entry matching this impersonator doesn't include the requested target username. ROR uses only that first entry and never falls through to a later one, so a second entry added for the same admin is never consulted                                                                                | Broaden the `users` pattern *in that first matching entry* - appending another entry below it won't help                           |
| Admin's password was recently changed and impersonation broke, even though normal login still works                                      | Impersonation not allowed                                          | The `authentication_rule` inside `impersonation` is checked completely independently of the rule in `access_control_rules` - updating one does not update the other                                                                                                                                                                                      | Keep both in sync, or point both at the same external identity source (LDAP/external auth) instead of hardcoding credentials twice |
| Config fails to load at startup, mentioning "should be either impersonator or a user to be impersonated"                                 | Config validation error                                            | The exact same username (no wildcards) appears in both `impersonator` and `users` in one entry                                                                                                                                                                                                                                                           | Remove the self-reference - a user can't be declared as able to impersonate themselves                                             |
| Config fails to load at startup, mentioning "it's used in a context of user patterns"                                                    | Config validation error                                            | The `impersonation` entry's `authentication_rule` has a fixed, statically known username that doesn't match the `impersonator` pattern (e.g. `impersonator: alice` but `auth_key: someone_else:pass`)                                                                                                                                                   | Make the rule's username match the `impersonator` pattern                                                                          |
| Impersonator authenticates fine, but the request is still denied, mentioning the impersonated user doesn't exist                         | Denied; `AUTH_FAIL (Impersonated user does not exist)` in the logs | **No** block could confirm the target user: they aren't statically configured in any block and aren't present in the relevant service mock. Note that this message is logged by every block whose authentication rule doesn't know the impersonated user, so seeing it in a healthy setup is normal - it's a problem only when no block ends up matching | Add the user to the mock, or confirm the username matches exactly                                                                  |
| Request denied with "impersonation not supported", even though the `impersonation` section looks correct                                 | Impersonation not supported                                        | An ACL block needs a service mock (LDAP / external authentication / external authorization) that hasn't been configured yet, or uses `auth_key_sha*` with a fully-hashed `user:pass` blob (see [limitations](#impersonation-limitations))                                                                                                                | Add the missing mock, or switch to the `USER_NAME:hash(PASSWORD)` form for hashed auth rules                                       |
| The block you wanted to test is never matched during impersonation, and it uses `jwt_auth` or `ror_kbn_auth`                             | No impersonation-specific error - the block just doesn't match     | These rules don't take part in the impersonation flow: they look for a real JWT / ROR Kibana token in the request, don't find one, and reject the block. ROR reports it as a Test Settings warning                                                                                                                                                       | Not impersonable today - test such blocks with a real session, or authenticate the users with an impersonation-aware rule          |
| Impersonation UI can't find/list the user you want to impersonate                                                                        | N/A (UI limitation)                                                | The target username is only reachable through a wildcard `users` pattern in the ACL, so ROR can't enumerate it upfront                                                                                                                                                                                                                                   | Type the username manually in the impersonation UI, as described in [limitations](#impersonation-limitations)                      |
| Impersonation is refused for every target user, although the `impersonation` entry looks correct and the same credentials work elsewhere | Impersonation not allowed                                          | The impersonating client didn't send the impersonator's credentials as an HTTP Basic Auth header - ROR always identifies the impersonator from Basic Auth, regardless of which rule type is configured as the `authentication_rule`                                                                                                                      | Make sure the client authenticates with Basic Auth (this is what the ROR Kibana Test Settings UI does under the hood)              |

## Logs & audit

In Elasticsearch logs, in `USR` field, if an admin user finds something like this: `alice (as bob)` - it means that `alice` was authenticated, and they are the impersonator who is impersonating `bob`.

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
