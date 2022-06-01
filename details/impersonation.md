# Impersonation 

After describing what [the impersonation is](../kibana.md#impersonate), it's high time to see how does ROR support it and who and when could be interested in using this feature. Let's start with the latter one.

## Use cases

The impersonation feature is intended for ROR administrators, rather than users. We can point out two the most obvious use cases when the admin could take advantage of the feature:

#### Debugging users' problems:

Let's imagine that some user has a problem with its ROR configuration (eg. the user doesn't have an access to some feature that was blocked at ROR's level by you, the admin). And they are not able to clearly describe what the issue is (sounds familiar?). As an administrator, it would be extremely beneficial if you could see what he user sees. Thanks to impersonation feature, an admin is allowed to impersonate the user and feel and see the same the user sees. 

#### Configuring a new user:

When an admin configures a new user in ROR settings, they face two problems:

1. `Will the updated configuration break the production cluster?`
1. `How do I know that the new user is correctly configured? Did I configure all their permission correctly??`

Both of the problems can be solved using the ROR's impersonation. Thanks to the fact that the impersonation feature always uses its own test settings, that is completely independent from the main production settings, the admin can alter it without worries that their actions will break something and users won't be able to do their job. 

Admin can add the new user configuration without worries and then test it by impersonate the user. They can check if the user can log in without problems and if the user has access only to the Kibana's features the admin wanted to grant. When the admin is sure that everything is configured correctly, they can promote the settings (test) to production. 

## Impersonation configuration

Before an admin will be able to impersonate the user, they have to configure ROR properly. The configuration consists of several parts:

1. creating ROR's Test Settings,
2. defining mocks of the external services,
3. impersonating a chosen user.

#### Creating ROR's Test Settings

When you call Elasticsearch directly or through ROR Kibana, ROR ACL is defined by Settings (we can assume they are main settings). The Test Settings define another ACL, that is taken into consideration by ROR ES only then a proper impersonation header is passed. The Test Settings are active only for a strictly defined amount of time. After the time is expired, they are automatically invalidated (for a security reasons). 

Detailed instruction how to define Test Settings can be find [here](../examples/impersonation/test-settings-ui.md).

Copying Main Settings as Test Settings is not enough. We have to instruct ROR which users can be considered as impersonators (the ones, who are allowed to impersonate any other user). The list of allowed impersonators and their permissions are defined in the `impersonation` section in ROR Settings:

```yaml
readonlyrest:
  
  access_control_rules:

    [...]

  impersonation:

    - impersonator: admin1      // Who can impersonate? (user name or pattern)
      users: ["*"]              // Who can be impersonated? (User names or patterns)
      auth_key: admin1:pass     // Authentication rule required to impersonate (any authentication rule can be used here)

    - impersonator: admin2
      users: ["dev2"]
      ldap_authentication: "ldap1"
```

In the example above, we see that we have two impersonators: `admin1` and `admin2`. The first one can impersonate any user (`*`) and they are able to authenticate using basic auth (`admin1:pass`). The second impersonator can impersonate only `dev2` user. They will be authenticated using `ldap1` connector.

When an impersonator passes wrong credentials ROR will tell Kibana that impersonation is not allowed.

#### Defining mocks of the external services (optional)

ROR has many sophisticated authentication & authorization methods. Some of them are based on external systems like LDAP. A problem with such systems according to the impersonation feature is that, the systems don't support it by default, or the configuration is difficult, or they don't support it at all. 

That's why we decided to solve it totally differently - using mocks. [Wikipedia](https://en.wiktionary.org/wiki/mock) defines `mock` as `an imitation, usually of lesser quality.` And in case of external authentication system we are going provide an imitation of it that tell ACL what users can be authenticated by it. When we consider an authorization service, a mock of it will return the ACL users with their roles in the service. And this is enough for ROR to support impersonation. 

How does ROR use the mocks? Let's suppose we have `ldap_auth` rule. When ROR processes the rule, it:
* asks a given LDAP if the username can be authenticated with a given password, and if they can ...
* asks LDAP to list what groups the user belongs to

In the impersonation case, it looks pretty much the same. Just we won't call any LDAP server, rather, the mock will provide the required information (no passwords required):
* asks the mock if the username exists, and if it does ...
* asks the mock to tell what groups the user belongs to

**⚠️ IMPORTANT:** When some external service is not mocked, ROR might return to Kibana information that the impersonation is not supported. It's better to always define all mocks, to avoid the "not supported impersonation" ES response.

How to define mocks in Kibana is described [here](../examples/impersonation/external-services-mocks-ui.md).

#### Impersonating a chosen user

Now, when we have configured Test Settings and External Services Mocks, we can try to impersonate a user. In ROR Settings, user can be:
* provided statically (defined in the settings),
* provided dynamically:
   * from external, dependant systems (like LDAP) - we mock them
   * from upstream systems (eg. through headers) - they are unknown

It means that we pick the users defined in Settings or Mocks, but also we can enter the username and try to impersonate such user.

[Here](../examples/impersonation/impersonate-user-ui.md) you find a description how to impersonate a user in ROR Kibana.

## Logs & audit

In ES logs in `USR` admin is going to find sth like this: `admin1 as (user1)` - it means that `admin1` was authenticated and they are the impersonator who is impersonating `user1`. 

All logs of impersonated user in Kibana will have this format `[<log level>][plugins][ReadonlyREST][<filename>][impersonating <impersonated user username>]`

When auditing is enabled, the audit document is going to contain an `impersonated_by` field.

## Impersonation limitations

Impersonation mode has some limitations. Please check if they have an impact on your use cases:

* Not all features available in the ROR configuration are testable with impersonation mode.
Some rules used in ROR ACL do not support impersonation. For example, auth rule with hashed credentials(e.g. `auth_key_sha512`) can be used in impersonation mode only when credentials follow the format `USER_NAME: HASH(PASSWORD)`; A fully hashed username and password don't allow fetching a username. The auth rule in such a format won't match during impersonation.
A list of the rules with info about impersonation support can be found [here](../elasticsearch.md#rules).

* Test settings are stored in the memory of the node that handled the POST tests settings request.
Impersonation support will be limited to this node. We are going to improve it in the future, but for now your kibana should talk only to one ES node.

* Sometimes is impossible to fetch usernames defined in the test settings.
If a `users` rule contains a username pattern with a wildcard, to impersonate a user matching a given pattern, you need to enter the username manually.

  ```yaml
  readonlyrest:

    access_control_rules:
      - name: "LDAP group g1"
        type: allow
        groups: ["g1"]
      
    users:
      - username: "admin*"  // To impersonate a user with a username matching 'admin*' you need to enter the username manually, like 'admin123'
        groups:
          - g1: group1
        ldap_auth:
          name: "ldap1"
          groups: ["group1"]
        
    ldaps:
      - name: ldap1
        [..]
        
    impersonation:
      [...]
  ```

## Glossary

* **Impersonator** - someone who imitates or copies the behavior or actions of another,
* **Impersonation** - imitating behaviors or actions of a given user,
* **Main Settings** - the ROR's settings that apply to ACL that handles request during regular sessions (not the impersonation ones),
* **Test Settings** - the ROR's settings that apply to ACL that handles impersonating requests (the ones during impersonation session),
* **External Service Mock** - an imitation of an external service (the supported ones: LDAP, an external authentication service, an external authorization service).