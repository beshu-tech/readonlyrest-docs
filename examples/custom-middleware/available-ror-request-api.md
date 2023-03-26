---
description: Available rorRequest API
---

# Available rorRequest API

You can access the rorRequest API via `req.rorRequest` in your custom middleware. The available options are:

| Property name                                                          | Return value type            | Example return value                                                         | Description                                                                    |
|:-----------------------------------------------------------------------|------------------------------|------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| getAuthorizationHeaders()                                              | Map<string, string>          | Map(2) {'authorization' => 'Basic BWRtaW46ZGV2', 'cookie' => 'cookie value'} | Get headers using in the authorization                                         |
| isAuthenticated()                                                      | boolean                      | true                                                                         | Check if the session is authenticated                                          |
| isCookiePresent(cookieName: string)                                    | boolean or undefined         | true                                                                         | Check if the specific cookie is presented in the request                       |
| getIdentitySession()                                                   | IdentitySession or undefined | Check User Session identity section                                          | Get the session identity (check the information below, for the exact response) |
| setIdentitySession(identitySession: IdentitySession or undefined)      | void                         | -                                                                            | Set the new session                                                            |
| enrichIdentitySessionMetadata(customMetadata: Record<string, unknown>) | void                         | -                                                                            | Enrich existing user session by the additional custom metadata                 |
| lastSessionActivityDate                                                | Date or undefined            | 2023-03-23T19:50:37.932Z                                                     | Date of the last session activity. Using in the context of a session timeout   |
| extractHiddenAppsNames                                                 | string[]                     | [ 'Enterprise Search, Overview', 'Observability' ]                           | List of all hidden apps for specific users                                     |

You also have access to the standard [Express.js](https://expressjs.com) [request](https://expressjs.com/en/api.html#req) and [response](https://expressjs.com/en/api.html#res) objects
