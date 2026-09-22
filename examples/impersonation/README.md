---
description: Impersonation
---

# Impersonation 
([Enterprise](https://readonlyrest.com/enterprise))

According to [Wikipedia](https://en.wikipedia.org/wiki/Impersonator):

> An impersonator is someone who imitates or copies the behavior or actions of another.

In ReadonlyREST, impersonation means that one user acts as another user. For example, an admin who has just configured access for a new user can impersonate that user and see what the user would see after logging in.

Impersonation is a Kibana feature, and this page describes it from the Kibana point of view. The ROR Kibana plugin handles the whole workflow in the ROR menu: it prepares a copy of the settings for testing, lets you mock external services, and starts and ends impersonation sessions. The ACL that decides what an impersonated user can see is evaluated by ROR for Elasticsearch, which is what the rest of this page explains.

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

1. **Create Test Settings.** Impersonation never uses your production settings. It uses Test Settings: a separate ACL that ROR applies to impersonation requests only. Regular users keep working with the production settings, so you can edit the Test Settings freely and promote them to Main Settings when they are ready. See [Creating ROR's Test Settings](configuring-impersonation.md#creating-rors-test-settings).
2. **Define mocks of external services (optional).** This step is needed only if the users you want to impersonate come from LDAP or another external service. A mock is a stand-in for such a service: a list of users, and the groups they belong to, that you write yourself. During impersonation, ROR asks the mock instead of the real service, so no real account or password is needed. See [Defining mocks of the external services](configuring-impersonation.md#defining-mocks-of-the-external-services-optional).
3. **Start impersonating.** Pick a user from the list, or type the username if it isn't listed. Kibana reloads as that user. See [Impersonating users](impersonate-user-ui.md).

    The list contains users defined statically in the settings, and users defined in the mocks of external services. Some users aren't known in advance and never appear on the list: with `proxy_auth`, for example, the username comes from a header set by a reverse proxy, so the settings only say which header to read, not who can appear in it. To impersonate such a user, type their username.

## How ROR processes an impersonation request

At this point you are logged into Kibana as yourself and impersonating another user. Kibana doesn't stop talking to Elasticsearch: it keeps sending requests on your behalf, now marked as impersonation requests.

Elasticsearch with the ReadonlyREST plugin handles such a request almost the same way as a request sent by the impersonated user. It checks the request against an ACL, [block by block](../../elasticsearch.md#acl-basics), in the same order. Rules such as `indices`, `kibana_*`, `fields`, `filter` or `hosts` work the same way as in that user's own session. This is what makes impersonation useful for testing the ReadonlyREST ACL.

There are three differences:

* The ACL is not the production one.
* The credentials in the request belong to the impersonator, for example `alice`. Which user is impersonated, for example `bob`, travels with the request as impersonation context that ROR and the ROR Kibana plugin set internally.
* Authentication and authorization rules behave differently.

The sections below explain how.

### Impersonating requests use Test Settings

ROR has two separate sets of settings:

* **Main Settings**: the production settings, with the ACL used for regular requests.
* **Test Settings**: a separate ACL used only for impersonation. You can change Test Settings without affecting users who work with Main Settings, and promote them to Main Settings when you're done.

ROR always checks impersonation requests against Test Settings, never against Main Settings. Test Settings are active only for a limited time and can be invalidated at any moment (see [Creating ROR's Test Settings](configuring-impersonation.md#creating-rors-test-settings)). When no Test Settings are active, impersonation is not available in Kibana.

### The impersonator and the impersonated user are authorized differently

`alice` can appear in the ROR settings in two independent roles:

* **As a regular user.** She is authenticated by the authentication rule of one of the ACL blocks, like any other user. This lets `alice` log into Kibana and do her own work.
* **As an impersonator.** She is listed in the `impersonation` section, a separate part of the settings that defines who can impersonate whom and how each impersonator is authenticated (see [The impersonation section](configuring-impersonation.md#the-impersonation-section)).

ROR checks the two roles separately, and one doesn't grant the other:

* An ACL block that authenticates `alice` doesn't give her the right to impersonate anyone. Only an entry in the `impersonation` section does.
* An entry in the `impersonation` section doesn't give `alice` any access of her own, and doesn't let her log into Kibana.

The Kibana workflow starts with `alice` logging into Kibana as herself, so an impersonator who uses it needs both: an ACL block that authenticates them, and an `impersonation` entry.

During an impersonation session, the authentication rules in the ACL blocks no longer check who the caller is. The `impersonation` section does that instead. This is why each entry in the `impersonation` section needs its own authentication rule: it is the only place in the settings that verifies the impersonator. When `alice` works as herself, none of this applies, and the ACL handles her requests like any other.

### Not every auth rule supports impersonation

When an authentication rule processes an impersonation request, what happens next depends on whether the rule supports impersonation.

**Rules that don't support impersonation** fail, and their block doesn't match. ROR moves on to the next block, and it shows a warning about such blocks when Test Settings are applied. They can't be tested with impersonation. Why a rule doesn't support impersonation, and what you see as a result, differs from rule to rule. See [Auth rules that don't support impersonation](configuring-impersonation.md#auth-rules-that-dont-support-impersonation).

**Rules that support impersonation** skip their normal authentication. Instead, ROR answers three questions, in this order:

1. **Can `alice` impersonate `bob`?** ROR takes the impersonator's username from the Basic Auth credentials of the request and looks for a matching entry in the `impersonation` section. If no entry allows `alice` to impersonate `bob`, the block doesn't match.
2. **Is the caller really `alice`?** ROR checks the credentials with the authentication rule of the matching `impersonation` entry. This check doesn't depend on the current block, or on the rule that authenticates `alice` as a regular user. Users who try to impersonate themselves are rejected at this step.
3. **Does `bob` exist?** The rule that ROR is currently evaluating answers this, based on what it knows. A rule with static credentials knows only the usernames written in it. A rule that uses an external system, such as LDAP or an external authentication or authorization service, doesn't contact that system during impersonation. It asks a [mock](configuring-impersonation.md#defining-mocks-of-the-external-services-optional) of the system instead, so no real account, password or connection to the production service is needed.

The answer to the third question decides what happens to the block:

* **The rule knows `bob`.** ROR treats the request as sent by `bob` and evaluates the rest of the block (groups, indices, Kibana rules and so on) as `bob`. Where the block would normally call an external system, ROR uses data from the mocks.
* **The rule doesn't know `bob`.** The block doesn't match, and ROR moves on to the next block. This is normal when the ACL has many blocks, because only the blocks that define `bob` can match. Blocks that don't define `bob` log `AUTH_FAIL (Impersonated user does not exist)`.
* **The rule can't check.** This happens when a mock is missing, or when an `auth_key_sha*` rule hashes the whole `user:pass` pair, so the username can't be read from it. The block doesn't match, and ROR moves on to the next block. If no block matches `bob`, ROR reports that impersonation is not supported.

A negative answer to any of the questions affects only the current block. ROR moves on to the next block, as it does for a regular request, and asks the questions again. The answers to the first two questions depend only on the `impersonation` section, so they are the same in every block. If one of them is negative, every block whose authentication rule supports impersonation fails, and the request is refused as not allowed, unless a block with no authentication rule matches it. The answer to the third question depends on the rule in each block, and it decides which block matches first.

## Impersonation configuration

The settings that impersonation depends on - Test Settings, the `impersonation` section and mocks of external services - are described in [Configuring impersonation](configuring-impersonation.md), together with the list of rules that support impersonation, end-to-end examples and common misconfigurations.

## Logs & audit

In the Elasticsearch logs, a `USR` field value such as `alice (as bob)` means that `alice` was authenticated and is impersonating `bob`.

In Kibana, all logs of the impersonated user have this format: `[<log level>][plugins][ReadonlyREST][<filename>][impersonating <impersonated user username>]`

When audit is enabled, the audit document contains an `impersonated_by` field.

## Impersonation limitations

Check whether these limitations affect your use cases:

* Not everything in the ROR settings can be tested with impersonation, because some rules don't support it. See [Auth rules that don't support impersonation](configuring-impersonation.md#auth-rules-that-dont-support-impersonation).
* Test Settings and mocks are stored in the ROR settings index. The node that receives them from Kibana applies them at once, and the other nodes pick them up the next time they poll the index for settings changes: every 5 seconds by default, as set by [`poll_interval`](../../elasticsearch.md#index-loading-strategy). Until then, those nodes still use the previous Test Settings, so an impersonation session can behave inconsistently for a moment after you apply or invalidate them. Nodes that don't poll the index, because `poll_interval` is `0s` or because they [load their settings from a file](../../elasticsearch.md#force-loading-from-file), don't pick up Test Settings saved through another node.
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
