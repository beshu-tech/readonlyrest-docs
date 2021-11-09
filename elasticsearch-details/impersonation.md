# Impersonation 

According to [Wikipedia](https://en.wikipedia.org/wiki/Impersonator):

> An impersonator is someone who imitates or copies the behavior or actions of another.

So, an impersonation can be understood as imitating behaviors or actions.
In the context of ReadonlyREST eg. one user could imitate an action 
of another user. Why would we want it? Let's suppose the first user is 
an admin, who has just configured access for a new user. He would like 
to know if the rule or rules are configured correctly. And here it comes the impersonation feature. The admin can call ES request on behalf of the 
tested user and authenticate using his credentials. 

## Impersonators configuration

Not anyone can be the impersonator. The list of impersonators and their permissions are defined in `impersonation` section:

```yaml
readonlyrest:
  
  access_control_rules:

    [...]

  impersonation:

    - impersonator: admin1      // impersonator user name or pattern
      users: ["*"]              // user name patterns that can be impersonated by the impersonator
      auth_key: admin1:pass     // authentication rule (any authentication rule can be used here)

    - impersonator: admin2
      users: ["dev2"]
      ldap_authentication: "ldap1"
```

We can read it as follows:

1. The impersonator "admin1" can impersonate users which names match pattern `*` (any user) and the impersonator should be authenticated using `auth_key` rule.

2. The impersonator "admin2" can impersonate only `dev2` and the impersonator should be authenticated using LDAP by `ldap1` connector.

## Request with user impersonation

Let's suppose our `user1` wants to search `twitter` index. He probably will call:

```bash
curl -vk -u user1:user_secret "https://localhost:9200/twitter/_search?pretty"
```

Now, how the same request would look like it our `admin1` wants to impersonate `user1` and search `twitter` on this behalf?

```bash
curl -vk -u admin1:pass -H "impersonate_as: user1" "https://localhost:9200/twitter/_search?pretty"
```

As we can see the request looks pretty much the same. The main difference is the `impersonate_as` header that contains `user1` and authorization header value has the admin's credentials now. 

Responses of both requests should be the same.

When the admin will pass wrong credentials or cannot impersonate the passed user
ROR is going to return response like this:

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

Some authentication or/and authorization rules (like LDAP ones or related to external systems) don't support impersonation by default (the support will be possible with cooperation with Kibana plugin soon). When request with impersonation
will be rejected by rule that doesn't support impersonation, the caller should
see result like this:

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