---
description: Configuring impersonation
---

# Configuring impersonation
([Enterprise](https://readonlyrest.com/enterprise))

This page describes the settings that the [Kibana workflow](README.md#impersonating-a-user-in-kibana) depends on: Test Settings, the `impersonation` section, which defines who can impersonate whom and without which ROR refuses impersonation, and mocks of external services such as [LDAP](../../elasticsearch.md#ldap-connector), [External Basic Auth](../../elasticsearch.md#external-basic-auth) or [Custom groups provider](../../elasticsearch.md#custom-groups-providers). How ROR processes an impersonation request is explained in the [Impersonation guide](README.md#how-ror-processes-an-impersonation-request).

## Creating ROR's Test Settings

When you call Elasticsearch directly or through Kibana, ROR uses the ACL from Main Settings. Test Settings define a second ACL, which ROR for Elasticsearch uses only for requests with impersonation context. ROR adds that context internally.

Test Settings are active only for a limited time: 30 minutes by default, but you can set a different time to live (TTL) before you apply them. When the time runs out, ROR invalidates the Test Settings automatically, for security reasons. You can also invalidate them yourself at any time. Only one set of Test Settings can be active at a time.

When Test Settings expire or are invalidated, impersonation stops working immediately, whatever the `impersonation` section contains. If impersonation suddenly stops working, check this first: apply the Test Settings again and retry.

### Creating Test Settings in Kibana

1. Open the ROR menu.
1. Click the **Edit security settings** button.

    ![Test settings ror menu](<../../.gitbook/assets/test_settings_ror_menu.png>)

1. Go to the **Test settings** tab, where you can:

    * load the current Main Settings as Test Settings, and edit them,
    * set the TTL, after which the Test Settings are deactivated and the impersonation session ends,
    * deactivate the Test Settings yourself,
    * save the Test Settings as Main Settings.

    ![test settings tab](<../../.gitbook/assets/test_settings_tab.png>)

## The impersonation section

Test Settings alone are not enough. ROR also needs to know which users can impersonate others, and whom they can impersonate. This is defined in the `impersonation` section (see [The impersonator and the impersonated user are authorized differently](README.md#the-impersonator-and-the-impersonated-user-are-authorized-differently)):

1. Every impersonator must have an entry in the `impersonation` section, with the usernames or username patterns of the users they can impersonate. Being authenticated in `access_control_rules` is not enough. Without a matching entry, ROR refuses impersonation, whatever access the user has otherwise.
2. The impersonator's credentials must pass the authentication rule defined in that entry. ROR checks this rule separately from `access_control_rules`, using the credentials sent with the impersonation request.
3. The impersonator must log into Kibana before they can impersonate anyone, so `access_control_rules` also needs a block that authenticates them as a regular user. That block isn't used for impersonation.

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

## Defining mocks of the external services (optional)

Some ROR authentication and authorization methods rely on external systems such as LDAP. During impersonation, ROR can't use them:

* ROR doesn't have the impersonated user's password, and it shouldn't need one - that is the point of impersonation. Without it, ROR can't authenticate the user against the service.
* ROR can't ask the service who its users are, either. LDAP and the HTTP services behind `external_authentication` and `groups_provider_authorization` answer questions about one user at a time, and have no endpoint that returns everyone. This is also why Kibana can't always list the users you can impersonate.

ROR solves this with mocks. [Wiktionary](https://en.wiktionary.org/wiki/mock) defines a mock as "an imitation, usually of lesser quality". A mock of an external service is a list you write yourself: which users the service would return and, for an authorization service, which groups each of them belongs to. That is all ROR needs during impersonation.

For example, when ROR evaluates an `ldap_auth` rule for a regular request, it:

* asks the LDAP server whether the user can log in with the given password, and if so,
* asks the LDAP server which groups the user belongs to.

During impersonation, ROR doesn't contact the LDAP server, and no password is needed. Instead, it:

* asks the mock whether the user exists, and if so,
* asks the mock which groups the user belongs to.

A mock that makes `bob` impersonable as a member of the `devs` group of the `ldap1` connector contains just that:

| Service | User | Groups |
|---------|------|--------|
| `ldap1` | `bob` | `devs` |

**⚠️ IMPORTANT:** If an external service used in the ACL has no mock, ROR may report that impersonation is not supported. To avoid this, define mocks for all external services.

### Defining mocks in Kibana

Mocks are defined in the ROR menu, under **Edit security settings**, for the services listed in the Test Settings. You never create an account in the real service - you only list the users, and their groups, that the service would normally return.

![Auth mock](<../../.gitbook/assets/auth_mock.png>)

After clicking the add/edit user buttons (1), you see a dialog where you can add (2) or remove (3) a user of an external service mock.

![Add/edit external mock service](<../../.gitbook/assets/add_edit_auth_mock_service.png>)

## Which rules support impersonation

All rules that don't authenticate or authorize users - `indices`, `actions`, `kibana_*`, `fields`, `filter`, `hosts`, `uri_re` and so on - support impersonation. They work the same way for an impersonated user as for a regular one.

Authentication and authorization rules (auth rules) are different: some support impersonation without extra configuration, some need a mock of an external service, and some don't support it at all. The tables below list the auth rules in both groups. When Test Settings are applied, ROR checks every rule and reports a warning for each block with a rule that won't work during impersonation. The ROR Kibana plugin shows these warnings in the Test Settings UI.

### Auth rules that support impersonation

| Rule | What it needs | Notes |
|------|---------------|-------|
| `auth_key`, `auth_key_unix`, `proxy_auth`, `token_authentication` | Nothing | The usernames are written in the rule itself, or come with the request |
| Group rules (`groups_any_of`, `groups_all_of` and other [groups logic](../../details/authorization-rules-details.md#checking-groups-logic)) | Depends on the `users` section | The rules in the matching `users` entry decide. Static groups work without extra configuration. Groups from an external system, for example through `ldap_auth`, need a mock of that system |
| `ldap_authentication`, `ldap_authorization`, `ldap_auth` | An LDAP mock | See [Defining mocks of the external services](#defining-mocks-of-the-external-services-optional) |
| `external_authentication` | A mock of the external authentication service | As above |
| `groups_provider_authorization` | A mock of the external authorization service | As above |
| `auth_key_sha1`, `auth_key_sha256`, `auth_key_sha512`, `auth_key_pbkdf2` | The `USER_NAME:hash(PASSWORD)` form: the username in plain text and only the password hashed, for example `alice:280ac6f...94bf9` | In the `hash(USER_NAME:PASSWORD)` form, where the whole `username:password` string is hashed, the rule doesn't support impersonation. See the table below |

### Auth rules that don't support impersonation

The reason differs from rule to rule.

| Rule | Why | What you see |
|------|-----|--------------|
| `jwt_auth`, `jwt_authentication`, `jwt_authorization` | The rule takes the username from a JWT. An impersonation request carries the impersonator's Basic Auth credentials instead, and the rule has no other way to learn who is impersonated | The rule fails as it would for any request without a token. The block doesn't match, and ROR moves on to the next one. ROR shows a Test Settings warning for the block |
| `ror_kbn_auth`, `ror_kbn_authentication`, `ror_kbn_authorization` | The same, with the token issued by the ROR Kibana plugin | The same as above |
| `auth_key_sha1`, `auth_key_sha256`, `auth_key_sha512`, `auth_key_pbkdf2` in the `hash(USER_NAME:PASSWORD)` form | The username is part of the hash, so ROR can't read it and can't tell whether the rule knows the impersonated user | The block doesn't match. If no other block matches, ROR reports that impersonation is not supported. ROR shows a Test Settings warning |
| A rule from the table above that needs a mock, when the mock is missing | Without the mock, the rule has no way to check whether the impersonated user exists | The block doesn't match. If no other block matches, ROR reports that impersonation is not supported. ROR shows a Test Settings warning naming the service |

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

For impersonation to work, an LDAP mock for `ldap1` must define `bob` as a user in the `devs` group, exactly like the [mock example above](#defining-mocks-of-the-external-services-optional). Without the mock, neither LDAP rule can check whether `bob` exists, and the request is refused as not supported, even though `alice` was authenticated as an impersonator.

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
| The request is refused as not supported, although the `impersonation` section looks correct | Impersonation not supported | A block uses LDAP, `external_authentication` or `groups_provider_authorization` without a mock, or an `auth_key_sha*` rule in the `hash(USER_NAME:PASSWORD)` form (see [Auth rules that don't support impersonation](#auth-rules-that-dont-support-impersonation)) | Add the missing mock, or use the `USER_NAME:hash(PASSWORD)` form |
| A block that uses `jwt_auth` or `ror_kbn_auth` never matches during impersonation | No impersonation error, the block just doesn't match | These rules don't support impersonation. They look for a JWT or ROR Kibana token, don't find one, and fail. ROR shows a Test Settings warning for such blocks | Test these blocks in a real user session, or use a rule that supports impersonation |
| The user you want to impersonate isn't on the list in Kibana | None (UI limitation) | The user matches only a wildcard pattern in the `users` section, so ROR can't list them | Type the username manually (see [limitations](README.md#impersonation-limitations)) |
| Impersonation is refused for every user, although the `impersonation` entry looks correct and the credentials work elsewhere | Impersonation not allowed | ROR identifies the impersonator only by HTTP Basic Auth credentials, whatever rule the `impersonation` entry uses. Credentials sent in any other form don't match any entry | Make sure the impersonator authenticates with a username and password (HTTP Basic Auth) |
