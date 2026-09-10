---
description: Impersonation
---

# Impersonation 
([Enterprise](https://readonlyrest.com/enterprise))

According to [Wikipedia](https://en.wikipedia.org/wiki/Impersonator):

> An impersonator is someone who imitates or copies the behavior or actions of another.

In ReadonlyREST, impersonation means that one user acts as another user. For example, an admin who has just configured access for a new user can impersonate that user and see what the user would see after logging in.

Impersonation is mainly a Kibana feature, and this page describes it from the Kibana point of view. The ROR Kibana plugin handles the whole workflow in the ROR menu: it prepares a copy of the settings for testing, lets you mock external services, and starts and ends impersonation sessions. The plugin relies on ROR for Elasticsearch, so you can also impersonate users with the Elasticsearch REST API alone. That requires calling ROR's internal APIs directly and is not described here.

## Use cases

Impersonation is a tool for ROR administrators rather than for regular users. The two most common use cases are described below.

### Debugging users' problems

A user reports a problem with their access, for example a Kibana feature they can't use because of a ROR rule, but they can't clearly describe what is wrong. If you impersonate that user, you see what they see, which makes the problem much easier to find.

### Configuring a new user

When you add a new user to the ROR settings, you usually have two questions:

1. Will the updated settings break anything on the production cluster?
2. Is the new user configured correctly, with all the permissions they need and no more?

Impersonation answers both. It always uses its own Test Settings, which are separate from the production settings, so you can change them without affecting other users. You add the new user to the Test Settings, impersonate them, and check that they can log in and see only the Kibana features you meant to give them. When everything is correct, you promote the Test Settings to production.

## Impersonating a user in Kibana

The workflow is available in the ROR menu, under **Edit security settings**. It has three steps:

1. **Create Test Settings.** Impersonation doesn't use your production settings. It uses Test Settings: a separate, temporary ACL that you can edit freely and later promote to Main Settings. See [Creating Test Settings](test-settings-ui.md) for the UI, and [Creating ROR's Test Settings](#creating-rors-test-settings) for how Test Settings work.
2. **Define mocks of external services (optional).** This step is needed only if the users you want to impersonate come from LDAP or another external service. During impersonation, ROR doesn't connect to that service. Instead, it asks a mock which users exist and which groups they belong to, so you don't need a real account or password. See [Defining external services mock configurations](external-services-mocks-ui.md) and [Defining mocks of the external services](#defining-mocks-of-the-external-services-optional).
3. **Start impersonating.** Pick a user from the list, or type the username if it isn't listed. Kibana reloads as that user. See [Impersonating users](impersonate-user-ui.md).

The list of users contains:

* users defined statically in the settings,
* users defined in the mocks of external services, such as LDAP.

Users that come from upstream systems, for example through HTTP headers, aren't known in advance. To impersonate such a user, type their username.

The settings must also define who can impersonate whom. See [The impersonation section](#the-impersonation-section).

## How ROR processes an impersonation request

ROR handles an impersonation request almost the same way as a request sent by the impersonated user. It checks the request against the same ACL, block by block, in the same order. Rules such as `indices`, `kibana_*`, `fields`, `filter` or `hosts` work the same way as in that user's own session. This is what makes impersonation useful for testing settings.

There are two differences:

* The credentials in the request belong to the impersonator, for example `alice`. The name of the impersonated user, for example `bob`, is sent separately, in an internal header set by ROR and the ROR Kibana plugin.
* Authentication and authorization rules behave differently.

The sections below explain how.

### Impersonating requests use Test Settings

ROR has two separate sets of settings:

* **Main Settings**: the ACL used for regular requests.
* **Test Settings**: a separate ACL used only for impersonation. You can change Test Settings without affecting users who work with Main Settings, and promote them to Main Settings when you're done.

ROR always checks impersonation requests against Test Settings, never against Main Settings. Test Settings are active only for a limited time and can be invalidated at any moment (see [Creating ROR's Test Settings](#creating-rors-test-settings)). When no Test Settings are active, impersonation doesn't work.

### The impersonator and the impersonated user are authorized differently

`alice` can appear in the ROR settings in two independent roles:

* **As a regular user.** She is authenticated by the authentication rule of one of the ACL blocks, like any other user. This lets `alice` log into Kibana and do her own work.
* **As an impersonator.** She is listed in the `impersonation` section, a separate part of the settings that defines who can impersonate whom and how each impersonator is authenticated (see [The impersonation section](#the-impersonation-section)).

One role doesn't require the other:

* An impersonator who never logs into Kibana needs only an entry in the `impersonation` section. They can still impersonate users through the Elasticsearch REST API.
* If `alice` should also be able to log into Kibana as herself, an ACL block must authenticate her as a regular user. The `impersonation` section doesn't give her any access of her own.

When `alice` sends a request as herself, the ACL handles it like any other request. When she sends a request as `bob`, the authentication rules in the ACL blocks no longer check who the caller is. The `impersonation` section does that instead. This is why each entry in the `impersonation` section needs its own authentication rule: it is the only place in the settings that verifies the impersonator.

### Not every authentication rule supports impersonation

When an authentication rule receives an impersonation request, what happens next depends on whether the rule supports impersonation. The full list is in [Which rules support impersonation](#which-rules-support-impersonation).

**Rules that don't support impersonation** (`jwt_*`, `ror_kbn_*`) handle the request as usual. They look for a JWT or a ROR Kibana token, don't find one (the request contains the impersonator's Basic Auth credentials instead), and fail. The block doesn't match, and ROR moves on to the next block. Such blocks can't be tested with impersonation, and ROR shows a warning about them when Test Settings are applied.

**Rules that support impersonation** skip their normal authentication. Instead, ROR answers three questions, in this order:

1. **Can `alice` impersonate `bob`?** ROR takes the impersonator's username from the Basic Auth credentials of the request and looks for a matching entry in the `impersonation` section. If no entry allows `alice` to impersonate `bob`, the block doesn't match.
2. **Is the caller really `alice`?** ROR checks the credentials with the authentication rule of the matching `impersonation` entry. This check doesn't depend on the current block, or on the rule that authenticates `alice` as a regular user. Users who try to impersonate themselves are rejected at this step.
3. **Does `bob` exist?** The rule that ROR is currently evaluating answers this, based on what it knows. A rule with static credentials knows only the usernames written in it. A rule that uses an external system, such as LDAP or an external authentication or authorization service, doesn't contact that system during impersonation. It asks a [mock](#defining-mocks-of-the-external-services-optional) of the system instead, so no real account, password or connection to the production service is needed.

The answer to the third question decides what happens to the block:

* **The rule knows `bob`.** ROR treats the request as sent by `bob` and evaluates the rest of the block (groups, indices, Kibana rules and so on) as `bob`. Where the block would normally call an external system, ROR uses data from the mocks.
* **The rule doesn't know `bob`.** The block doesn't match, and ROR moves on to the next block. This is normal when the ACL has many blocks, because only the blocks that define `bob` can match. Blocks that don't define `bob` log `AUTH_FAIL (Impersonated user does not exist)`.
* **The rule can't check.** This happens when a mock is missing, or when an `auth_key_sha*` rule hashes the whole `user:pass` pair, so the username can't be read from it. The block doesn't match, and ROR moves on to the next block. If no block matches `bob`, ROR reports that impersonation is not supported.

A negative answer to any of the questions affects only the current block. ROR moves on to the next block, as it does for a regular request, and asks the questions again. The answers to the first two questions depend only on the `impersonation` section, so they are the same in every block. If one of them is negative, every block whose authentication rule supports impersonation fails, and the request is refused as not allowed, unless a block with no authentication rule matches it. The answer to the third question depends on the rule in each block, and it decides which block matches first.

## Impersonation configuration

This part describes the settings that the [Kibana workflow](#impersonating-a-user-in-kibana) depends on: Test Settings, the `impersonation` section, and mocks of external services such as [LDAP](../../elasticsearch.md#ldap-connector), [External Basic Auth](../../elasticsearch.md#external-basic-auth) or [Custom groups provider](../../elasticsearch.md#custom-groups-providers).

### Creating ROR's Test Settings

When you call Elasticsearch directly or through Kibana, ROR uses the ACL from Main Settings. Test Settings define a second ACL, which ROR for Elasticsearch uses only for requests that carry the impersonation header. ROR manages this header internally.

Test Settings are active only for a limited time: 30 minutes by default, but you can set a different time before you apply them. When the time runs out, ROR invalidates the Test Settings automatically, for security reasons. You can also invalidate them yourself at any time. Only one set of Test Settings can be active at a time.

The ROR Kibana plugin has a dedicated UI for Test Settings. See the [Test Settings management guide](test-settings-ui.md).

When Test Settings expire or are invalidated, impersonation stops working immediately, whatever the `impersonation` section contains. If impersonation suddenly stops working, check this first: apply the Test Settings again and retry.

### The impersonation section

Test Settings alone are not enough. ROR also needs to know which users can impersonate others, and whom they can impersonate. This is defined in the `impersonation` section (see [The impersonator and the impersonated user are authorized differently](#the-impersonator-and-the-impersonated-user-are-authorized-differently)):

1. Every impersonator must have an entry in the `impersonation` section, with the usernames or username patterns of the users they can impersonate. Being authenticated in `access_control_rules` is not enough. Without a matching entry, ROR refuses impersonation, whatever access the user has otherwise.
2. The impersonator's credentials must pass the authentication rule defined in that entry. ROR checks this rule separately from `access_control_rules`, using the credentials sent with the impersonation request.
3. If the impersonator should also use Kibana as themselves, `access_control_rules` needs a block that authenticates them as a regular user. That block isn't used for impersonation.

```yaml
readonlyrest:
  access_control_rules:
    - name: "Authenticate alice"
      auth_key: alice:pass
    - name: "Authenticate carol"
      ldap_authentication: "ldap1"

  impersonation:
    - impersonator: alice      # who can impersonate (a username or pattern)
      users: ["*"]             # who can be impersonated (usernames or patterns)
      auth_key: alice:pass     # how the impersonator is authenticated (any authentication rule)
    - impersonator: carol
      users: ["bob"]
      ldap_authentication: "ldap1"
```

In this example there are two impersonators. `alice` can impersonate any user (`*`) and is authenticated with Basic Auth (`alice:pass`). `carol` can impersonate only `bob` and is authenticated with the `ldap1` LDAP connector.

If an impersonator sends wrong credentials, ROR tells Kibana that impersonation is not allowed.

The order of the entries matters. ROR uses only the first entry whose `impersonator` pattern matches the caller. It doesn't check later entries, even if one of them would allow the requested user. For example, if an entry for `a*` comes before an entry for `alice`, the `users` list of the `alice` entry is never used. Use one entry per impersonator where possible, and put the most specific patterns first.

When ROR loads this section, it checks that:

* each entry has exactly one authentication rule,
* the rule is an authentication rule (`auth_key*`, `ldap_authentication`, `external_authentication`, `proxy_auth` and so on), not an authorization-only rule such as `ldap_authorization`,
* no username (without wildcards) appears in both `impersonator` and `users` of the same entry, because users can't impersonate themselves,
* a fixed username in the rule (for example `auth_key: someone:pass`) matches the `impersonator` pattern. ROR can't check this for LDAP or external authentication, because the username is known only when a request arrives.

If any of these checks fails, ROR doesn't load the settings.

### Defining mocks of the external services (optional)

Some ROR authentication and authorization methods rely on external systems such as LDAP. These systems usually don't support impersonation, and when they do, it's complex to configure.

ROR solves this with mocks. [Wiktionary](https://en.wiktionary.org/wiki/mock) defines a mock as "an imitation, usually of lesser quality". A mock of an external authentication service tells ROR which users exist. A mock of an authorization service also tells ROR which groups each user belongs to. This is enough for ROR to support impersonation.

For example, when ROR evaluates an `ldap_auth` rule for a regular request, it:

* asks the LDAP server whether the user can log in with the given password, and if so,
* asks the LDAP server which groups the user belongs to.

During impersonation, ROR doesn't contact the LDAP server, and no password is needed. Instead, it:

* asks the mock whether the user exists, and if so,
* asks the mock which groups the user belongs to.

**⚠️ IMPORTANT:** If an external service used in the ACL has no mock, ROR may report that impersonation is not supported. To avoid this, define mocks for all external services.

The ROR Kibana plugin has a UI for creating and editing mocks. See the [service mock configuration guide](external-services-mocks-ui.md).

### Which rules support impersonation

Rules differ in how they support impersonation. When Test Settings are applied, ROR checks every rule and reports a warning for each block with a rule that won't work during impersonation. The ROR Kibana plugin shows these warnings in the Test Settings UI.

| Rule | Impersonation support | Notes |
|------|-----------------------|-------|
| `auth_key`, `auth_key_unix`, `proxy_auth`, `token_authentication` | Full | No extra configuration needed |
| Group rules (`groups_any_of`, `groups_all_of` and other [groups logic](../../details/authorization-rules-details.md#checking-groups-logic)) | Depends on the `users` section | The rules in the matching `users` entry decide. Static groups work without extra configuration. Groups from an external system, for example through `ldap_auth`, need a mock of that system |
| `auth_key_sha1`, `auth_key_sha256`, `auth_key_sha512`, `auth_key_pbkdf2_hmac_sha512` | Partial | Work only in the `USER_NAME:hash(PASSWORD)` form. In the `hash(USER_NAME:PASSWORD)` form, ROR can't read the username, so the rule never matches during impersonation. See [limitations](#impersonation-limitations) |
| `ldap_authentication`, `ldap_authorization`, `ldap_auth` | Requires a mock | Need an LDAP mock (see [Defining mocks of the external services](#defining-mocks-of-the-external-services-optional)). Without it, ROR reports that impersonation is not supported |
| `external_authentication` | Requires a mock | Needs a mock of the external authentication service |
| `external_authorization` | Requires a mock | Needs a mock of the external authorization service |
| `jwt_auth`, `jwt_authentication`, `jwt_authorization` | Not supported | These rules look for a JWT in the request, don't find one, and fail, so the block doesn't match. ROR shows a Test Settings warning for such blocks |
| `ror_kbn_auth`, `ror_kbn_authentication`, `ror_kbn_authorization` | Not supported | Same as the JWT rules, but with the ROR Kibana token |
| All other rules (`indices`, `actions`, `kibana_*`, `fields`, `filter`, `hosts`, `uri_re` and so on) | Not applicable | These rules don't identify users. They work the same way for an impersonated user as for a regular user |

Only blocks for users you want to impersonate need to support impersonation. A block that relies on an unsupported rule can't be tested with impersonation. Requests that would match it go on to the next blocks, as they would if the block didn't match for any other reason.

## Full end-to-end examples

### Example 1: a local user impersonating other local users

All users are defined in the settings, so no mocks are needed. This is the simplest setup.

```yaml
readonlyrest:
  access_control_rules:

    - name: "Alice"
      auth_key: alice:pass

    - name: "Developers"
      auth_key: bob:bobpass
      indices: ["dev-*"]

  impersonation:
    - impersonator: alice
      users: ["*"]
      auth_key: alice:pass   # checked separately from the "Alice" block
```

When `alice` impersonates `bob` in Kibana, the first two questions have the same answers in both blocks: `alice` can impersonate any user, and her credentials match the `impersonation` entry. The answers to the third question differ:

* The `auth_key` rule in the "Alice" block knows only `alice`. For this block, `bob` doesn't exist, so the block doesn't match and logs `AUTH_FAIL (Impersonated user does not exist)`. This is expected.
* The `auth_key` rule in the "Developers" block knows `bob`, so this block matches. ROR evaluates the request as `bob` and limits it to the `dev-*` indices, as in `bob`'s own session.

### Example 2: an LDAP user impersonating another LDAP user

In this example, the impersonated user's groups come from LDAP, so a mock is needed.

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
      ldap_authentication: "ldap1"   # alice's own LDAP credentials, checked separately
```

For impersonation to work, an LDAP mock for `ldap1` must define `bob` as a user in the `devs` group (see [Defining mocks of the external services](#defining-mocks-of-the-external-services-optional)). Without the mock, neither LDAP rule can check whether `bob` exists, and the request is refused as not supported, even though `alice` was authenticated as an impersonator.

## Common misconfigurations

Most impersonation problems are caused by one of the misconfigurations below. The "What ROR reports" column shows what you typically see in Kibana, in the Elasticsearch response or in the ROR logs.

| Symptom | What ROR reports | Cause | Fix |
|---------|------------------|-------|-----|
| Impersonation worked earlier, but now every impersonation request fails, and the settings haven't changed | Kibana reports that no Test Settings are configured | The Test Settings expired (after 30 minutes by default) or were invalidated | Apply the Test Settings again, optionally with a longer expiration time |
| The impersonator can log into Kibana, but impersonation is refused | Impersonation not allowed | There is no `impersonation` entry for this user. Being authenticated in `access_control_rules` doesn't give the right to impersonate | Add an `impersonation` entry whose `impersonator` pattern matches the user |
| Impersonation works for some users but not for others | Impersonation not allowed | The first `impersonation` entry that matches the impersonator doesn't include the requested user in `users`. ROR ignores later entries for the same impersonator | Add the user to `users` in that first matching entry. Adding a new entry below it won't help |
| Impersonation stopped working after the impersonator's password changed, but normal login still works | Impersonation not allowed | The authentication rule in the `impersonation` entry is separate from the one in `access_control_rules`. Changing one doesn't change the other | Update both, or use the same external source (LDAP or external authentication) in both instead of hardcoded credentials |
| The settings fail to load with "should be either impersonator or a user to be impersonated" | Settings validation error | The same username (without wildcards) is in both `impersonator` and `users` of one entry | Remove the username from one of them. Users can't impersonate themselves |
| The settings fail to load with "it's used in a context of user patterns" | Settings validation error | The authentication rule of an `impersonation` entry has a fixed username that doesn't match the `impersonator` pattern, for example `impersonator: alice` with `auth_key: someone_else:pass` | Change the username in the rule so that it matches the `impersonator` pattern |
| The impersonator is authenticated, but the request is refused because the impersonated user doesn't exist | `AUTH_FAIL (Impersonated user does not exist)` in the logs | No block knows the user: they aren't defined in any block and aren't in the mock. Blocks that don't know the user always log this message, so it's a problem only if no block matches | Add the user to the mock, or check the spelling of the username |
| The request is refused as not supported, although the `impersonation` section looks correct | Impersonation not supported | A block uses LDAP, external authentication or external authorization without a mock, or an `auth_key_sha*` rule in the `hash(USER_NAME:PASSWORD)` form (see [limitations](#impersonation-limitations)) | Add the missing mock, or use the `USER_NAME:hash(PASSWORD)` form |
| A block that uses `jwt_auth` or `ror_kbn_auth` never matches during impersonation | No impersonation error, the block just doesn't match | These rules don't support impersonation. They look for a JWT or ROR Kibana token, don't find one, and fail. ROR shows a Test Settings warning for such blocks | Test these blocks in a real user session, or use a rule that supports impersonation |
| The user you want to impersonate isn't on the list in Kibana | None (UI limitation) | The user matches only a wildcard pattern in the `users` section, so ROR can't list them | Type the username manually (see [limitations](#impersonation-limitations)) |
| Impersonation is refused for every user, although the `impersonation` entry looks correct and the credentials work elsewhere | Impersonation not allowed | ROR identifies the impersonator only by HTTP Basic Auth credentials, whatever rule the `impersonation` entry uses. Credentials sent in any other form don't match any entry | Make sure the impersonator authenticates with a username and password (HTTP Basic Auth) |

## Logs & audit

In the Elasticsearch logs, a `USR` field value such as `alice (as bob)` means that `alice` was authenticated and is impersonating `bob`.

In Kibana, all logs of the impersonated user have this format: `[<log level>][plugins][ReadonlyREST][<filename>][impersonating <impersonated user username>]`

When audit is enabled, the audit document contains an `impersonated_by` field.

## Impersonation limitations

Check whether these limitations affect your use cases:

* Not everything in the ROR settings can be tested with impersonation, because some rules don't support it. For example, a rule with hashed credentials (such as `auth_key_sha512`) works during impersonation only in the `USER_NAME:HASH(PASSWORD)` form. If the username and password are hashed together, ROR can't read the username, and the rule won't match during impersonation. The [rules description](../../elasticsearch.md#rules) says which rules support impersonation.
* Test Settings are stored in the memory of the Elasticsearch node that received the save request from the ROR Kibana plugin, so impersonation works only on that node. For now, Kibana should communicate with only one Elasticsearch node. We plan to improve this in the future.
*   ROR can't always list the users defined in Test Settings. If the `users` section contains a username pattern with a wildcard, you have to type the username of a user that matches the pattern manually.

    ```yaml
    readonlyrest:

      access_control_rules:
        - name: "LDAP group g1"
          type: allow
          groups_any_of: ["g1"]
        
      users:
        - username: "admin*"  # to impersonate a user matching 'admin*', type the username manually, for example 'admin123'
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

* **Impersonator** - a user who acts as another user.
* **Impersonation** - acting as another user.
* **Impersonated user** - the user whose identity is used during an impersonation session. Their permissions decide what the impersonator sees, but their credentials are never needed.
* **Main Settings** - the ROR settings with the ACL that handles regular requests.
* **Test Settings** - the ROR settings with the ACL that handles impersonation requests.
* **External Service Mock** - an imitation of an external service. Mocks are supported for LDAP, external authentication services and external authorization services.
