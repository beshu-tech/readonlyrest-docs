---
description: Reject machine-to-machine traffic using custom metadata ACL rules
---

# Reject machine-to-machine traffic using custom metadata ACL rules

We can also reject the specific request for example based on the custom metadata

1. Define ACL in your `readonlyrest.yml` file

```yaml
  - name: ADMIN_GRP
    groups: [ administrators ]
    kibana:
       access: admin
       kibana_index: '.kibana_@{acl:current_group}'
       metadata:
          rejectBasicAuth: true
```

2. Declare custom Kibana JS file `readonlyrest_kbn.kibana_custom_js_inject_file: '/path/to/custom_kibana.js'`

```js
async function customMiddleware(req, res, next) {
   const metadata =
           req.rorRequest && req.rorRequest.getIdentitySession() && req.rorRequest.getIdentitySession().metadata;

   const headerAuth = req.rorRequest && req.rorRequest.getAuthorizationHeaders && req.rorRequest.getHeaders().getAuthorizationHeaders().get('authorization');
   const isBasicAuth = headerAuth && headerAuth.includes('Basic')
   
  if (metadata.customMetadata && metadata.customMetadata.rejectBasicAuth && isBasicAuth) {
     return res.status(401).json({ message: 'Machine to machine communication is not allowed' });
  }

  return next()
}
```
You can pass any custom metadata and based on it accepts or reject the specific request

**⚠️IMPORTANT** Custom middleware must return `next()` function, to not block the request