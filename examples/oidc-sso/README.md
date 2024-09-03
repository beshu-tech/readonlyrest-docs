---
description: External connectors integration
---

# OpenID Connect (OIDC) SSO

With ReadonlyREST Enterprise, you can integrate with OpenID Connect (OIDC) Single Sign-on identity providers for both authentication and authorization.

Follow the guides to know more.

## Client Authentication Methods

Here is a rephrased and reformatted version of the text:

---

There are two possible authentication methods:

1. **client_secret_basic** (default): The `client_id` and `client_secret` are sent using the Authorization header, as specified in [RFC 6749, Section 2.3.1](https://datatracker.ietf.org/doc/html/rfc6749#section-2.3.1). Before sending, the `client_id` and `client_secret` are encoded.
2. **client_secret_post**: The `client_id` and `client_secret` are included in the request body, following the guidelines in [RFC 6749, Section 2.3.1](https://datatracker.ietf.org/doc/html/rfc6749#section-2.3.1). In this method, the `client_id` and `client_secret` are not encoded before being sent. Choose this method when the OpenID Connect provider, such as [lemonLDAP::NG](https://lemonldap-ng.org/), cannot decode the encoded values.