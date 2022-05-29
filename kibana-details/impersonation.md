# Impersonation 

After describing what [the impersonation is](../kibana.md), it's high time to see how does ROR support it and who and when could be interested in using this feature. Let's start with the latter one.

## Use cases

The impersonation feature is prepared for a ROR admin. It's not rather useful for users (at least not directly). We can point out two the most obvious use cases when an admin could take advantage of the feature:

#### Debugging users' problems:

Let's imagine that some user problem with its ROR configuration (eg. the user doesn't has an access to some feature that was blocked at ROR's level by you, the admin). They are not able to clearly describe what the issue is. For the admin, it'd be better if they can see what the user sees. Thanks to impersonation feature, an admin is allowed to impersonate the user and feel and see the same the user sees. 

#### Configuring a new user:

When an admin configures a new user in ROR settings, they face two problems:

1. `Won't the updated configuration break the production cluster?`
1. `How do I know that the new user is correctly configured? Do they have correct permissions?`

Both of the problems can be solved using the ROR's impersonation. Thanks to the fact that the impersonation operates on different than production (main) settings, the admin can alter it without worries that their actions will break something and users won't be able to do their job. 

Admin can add the new user configuration without worries and then test it by impersonate the user. They can check if user can log in without problems and if the user has access only to the Kibana's features the admin wanted to grant. When the admin is sure that everything is configured correctly, they can promote the settings (test) to production. 

## Impersonation configuration

### In ROR ES Settings

### In Kibana UI

Not every user can be the impersonator. The list of allowed impersonators and their permissions are defined in the `impersonation` section:

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

We can read it as follows:

1. The impersonator "admin1" can impersonate users whose names match pattern `*` (any user) and the impersonator should be authenticated using `auth_key` rule.

2. The impersonator "admin2" can impersonate only `dev2` and the impersonator should be authenticated using LDAP by `ldap1` connector.

## Request with user impersonation

Let's suppose our `user1` wants to search `twitter` index. He probably will call:

```bash
curl -vk -u user1:user_secret "https://localhost:9200/twitter/_search?pretty"
```

Now, how the same request would look like if our `admin1` wants to impersonate `user1` and search `twitter` on their behalf?

```bash
curl -vk -u admin1:pass -H "x-ror-impersonating: user1" "https://localhost:9200/twitter/_search?pretty"
```

As we can see the request looks pretty much the same. The main difference is the `x-ror-impersonating` header that contains `user1`, and authorization header value has the admin's credentials now. 

Responses of both requests should be the same.

Should the admin pass wrong credentials, or should they not be allowed to impersonate the user provided in the `x-ror-impersonating` header, 
ROR is going to return a response like the following:

> ```text
> HTTP/1.1 403 Forbidden
> content-type: application/json; charset=UTF-8
> 
>{
>  "error":{
>    "root_cause":[
>      {
>        "type":"forbidden_response",
>        "reason":"forbidden",
>        "due_to":["OPERATION_NOT_ALLOWED", "IMPERSONATION_NOT_ALLOWED"]
>      }
>    ],
>    "type":"forbidden_response",
>    "reason":"forbidden",
>    "due_to":["OPERATION_NOT_ALLOWED", "IMPERSONATION_NOT_ALLOWED"]
>  },
>  "status":403
>}
>```

Some authentication or/and authorization rules (like LDAP ones or related to external systems) don't support impersonation by default (the support will be possible with cooperation with Kibana plugin soon). 

When a request with impersonation is rejected because it triggers a rule that doesn't support impersonation, the caller will see this kind of response:

> ```text
> HTTP/1.1 403 Forbidden
> content-type: application/json; charset=UTF-8
> 
>{
>  "error":{
>    "root_cause":[
>      {
>        "type":"forbidden_response",
>        "reason":"forbidden",
>        "due_to":["OPERATION_NOT_ALLOWED", "IMPERSONATION_NOT_SUPPORTED"]
>      }
>    ],
>    "type":"forbidden_response",
>    "reason":"forbidden",
>    "due_to":["OPERATION_NOT_ALLOWED", "IMPERSONATION_NOT_SUPPORTED"]
>  },
>  "status":403
>}
>```

## Writing test settings

For impersonation to work, some valid test settings should be created and saved. It's important that the main ROR setting will be unaffected, so you, as an admin/user don't need to worry that you will break something. Here is how to write test settings:

1. Open the ROR menu
2. Click the Edit security settings button

![Test settings ror menu](<../.gitbook/assets/test_settings_ror_menu.png>)

3. Go into the Test settings tab
4. You can set the "time to live" (TTL), that is a time interval after which the test settings will be automatically deactivated and impersonation session will also abruptly exit
5. You can load current settings as test settings
6. You can deactivate settings manually
7. You can save test settings as settings

![test settings tab](<../.gitbook/assets/test_settings_tab.png>)

## Local and auth mock services users configuration

1. Open the ROR menu
2. Click the Edit security settings button

![Impersonate ror menu](<../.gitbook/assets/test_settings_ror_menu.png>)

3. Go into the Impersonate tab
4. You can free type impersonate. This button is available only in the situation when in some cases, the system is not able to receive all usernames. In this case, to impersonate, you need to type impersonating username manually.
5. You can add/edit user in a specific external auth mock service
6. You can impersonate a user and imitate his behavior and actions

![Impersonate tab](<../.gitbook/assets/impersonate_tab.png>)

## Add/edit auth mock user
External services mock is used to simulate the response of existing authentication or authorization service like LDAP.
You don't need to create a user account for configuration testing.
You will only need to define users (and their associated groups) that would normally be returned by the external services, listed in test settings.

After clicking add/edit user buttons(5), you will see a dialog with an option to add(6) or remove(7) user from external auth mock

![Add/edit auth mock service](<../.gitbook/assets/add_edit_auth_mock_service.png>)


## Impersonate a user

1. When an impersonation session is started correctly, the "impersonating" will be visible in the ROR menu a shown in the picture.
2. Click the Finish impersonation button to stop impersonation and go back into Impersonate tab

All logs of impersonated user will have this format `[<log level>][plugins][ReadonlyREST][<filename>][impersonating <impersonated user username>]`

![Impersonate user](<../.gitbook/assets/impersonate_impersonated_menu.png>)

## Impersonation limitations

Impersonation mode has some limitations. Please check if they have an impact on your use cases:

* Not all features available in the ROR configuration are testable with impersonation mode.
Some rules used in ROR ACL do not support impersonation. For example, auth rule with hashed credentials(e.g. `auth_key_sha512`) can be used in impersonation mode only when credentials follow the format `USER_NAME: HASH(PASSWORD)`; A fully hashed username and password don't allow fetching a username. The auth rule in such a format won't match during impersonation.
A list of the rules with info about impersonation support can be found [here](../elasticsearch.md#rules).

* Test settings are stored in the memory of the node that handled the POST tests settings request.
Impersonation support will be limited to this node.

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
