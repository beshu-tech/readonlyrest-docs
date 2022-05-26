# Impersonation 

According to [Wikipedia](https://en.wikipedia.org/wiki/Impersonator):

> An impersonator is someone who imitates or copies the behavior or actions of another.

So, an impersonation can be understood as imitating behaviors or actions.
In the context of ReadonlyREST: one user could imitate an action 
of another user. Why would we want it? Let's suppose the first user is 
an admin, who has just configured access for a new user. They would like 
to know if the rule(s) are configured correctly. And here it comes the impersonation feature. The admin can send an ES request on behalf of the 
tested user and authenticate using their credentials. 

## Impersonators configuration

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
curl -vk -u admin1:pass -H "impersonate_as: user1" "https://localhost:9200/twitter/_search?pretty"
```

As we can see the request looks pretty much the same. The main difference is the `impersonate_as` header that contains `user1`, and authorization header value has the admin's credentials now. 

Responses of both requests should be the same.

Should the admin pass wrong credentials, or should they not be alowed to impersonate the user provided in the `impersonate_as` header, 
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

After clicking add/edit user buttons(5), you will see a dialog with an option to add(6) or remove(7) user from external auth mock

![Add/edit auth mock service](<../.gitbook/assets/add_edit_auth_mock_service.png>)


## Impersonate a user

1. When an impersonation session is started correctly, the "impersonating" will be visible in the ROR menu a shown in the picture.
2. Click the Finish impersonation button to stop impersonation and go back into Impersonate tab

All logs of impersonated user will have this format `[<log level>][plugins][ReadonlyREST][<filename>][impersonating <impersonated user username>]`

![Impersonate user](<../.gitbook/assets/impersonate_impersonated_menu.png>)