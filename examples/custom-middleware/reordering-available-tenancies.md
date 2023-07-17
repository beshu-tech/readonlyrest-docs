---
description: Reordering available tenancies
---

# Reordering available tenancies

We can change the default tenancy, and the display ordering of the tenancies in the ROR menu by providing the `defaultGroup` query parameter in the HTTP request submitted by the login form, and change the order of `availableGroups`
thanks to the `enrichIdentitySessionMetadata` method.

1. Declare `readonlyrest_kbn.custom_middleware_inject_file: 'path/to/custom_middleware_inject_file.js'` in the kibana.yml and declare `custom_middleware_inject_file.js`

```js
async function customMiddleware(req, res, next) {
    const rorRequest = req.rorRequest;
    const metadata =
        req.rorRequest && req.rorRequest.getIdentitySession() && req.rorRequest.getIdentitySession().metadata;
    const defaultGroup = 'infosec';
    const X_FORWARDED_USER = 'x-forwarded-user';
    
    if (rorRequest.getPath() === '/login' && rorRequest.getMethod() === 'post') {
        // For the login form
        if (rorRequest.getBody().username === 'admin') {
            rorRequest.setQuery('defaultGroup', defaultGroup);
        }

        // For the SAML/OIDC login
        const token = rorRequest.getBody().conn_svc_transient_jwt;
        if (token) {
            const parsedJWT = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString());

            if (parsedJWT.user === 'admin') {
                rorRequest.setQuery('defaultGroup', defaultGroup);
            }
        }
    }

    // For the Proxy authorization
    if (!metadata && req.headers[X_FORWARDED_USER]) {
        if (req.headers[X_FORWARDED_USER] === 'admin') {
            rorRequest.setQuery('defaultGroup', defaultGroup);
        }
    }

    if (metadata && rorRequest.getPath() === '/pkp/api/info') {
        const availableGroups = metadata.availableGroups;
        if (availableGroups.some(availableGroup => availableGroup === defaultGroup)) {
            const index = availableGroups.indexOf(defaultGroup);
            const groupAvailable = index !== -1;
            if (groupAvailable) {
                availableGroups.splice(index, 1);
                availableGroups.unshift(defaultGroup);
            }

            rorRequest.enrichIdentitySessionMetadata({ availableGroups });
        }
    }

    return next();
}
```

In this example, before the login to the Kibana, when the username is equal 'admin', we add default tenant `rorRequest.setQuery('defaultGroup', defaultGroup);` which means,
that it will be the first tenant opened after the login. During the active Kibana session, we will also change the order of tenants displayed in the ROR menu and our default tenant will be the first on the list.

**⚠️IMPORTANT** Custom middleware must return `next()` function, to not block the request
