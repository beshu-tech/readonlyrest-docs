# Kibana Plugin overview

ReadonlyREST plugin for Kibana is not open source, and it's offered as part of the [ReadonlyREST PRO](https://readonlyrest.com/pro.html) and [ReadonlyREST ENTERPRISE](https://readonlyrest.com/enterprise.html) packages.
See product descriptions and a comparison chart in the official [ReadonlyREST website](https://readonlyrest.com)

ReadonlyREST plugins for Kibana **always require** ReadonlyREST Free to be installed in the Elasticsearch nodes your Kibana instance(s) will connect to.

It's not mandatory to install ReadonlyREST Free in all Elasticsearch nodes, but only in the ones in where you need the HTTP interface to be secured.

## After purchasing

You will receive a link to the plugin zip file in an email. Download your zip. 

You will be able to download it also in the future as long as your subscription is active.

## Version strings
All our plugins include in their file name a version string. For example the file `readonlyrest-1.16.26_es6.4.0.zip` has a version string `1.16.26_es6.4.0`.

### Reading version strings

Given the version string `1.16.26_es6.4.0`

* ReadonlyREST plugin code version `1.16.26`
* Works only with Elasticsearch/Kibana version `6.4.0`

The "es" stands for "Elastic stack" which used to mean the family of products made by Elastic which get released at the same time under the same version number. This was chosen **before** Elastic renamed their X-Pack commercial offer to Elastic Stack. 

To be clear, there is no affiliation between ReadonlyREST and Elastic, or their commercial products.

### Trial builds version strings

Trial builds are valid for 30 days after they were built, and they will stop working soon after the time is elapsed. Trial builds have a special version string which includes a build-time timestamp. 

I.e. `readonlyrest_kbn_pro-1.16.26-20180911_es6.0.0.zip`

* ReadonlyREST PRO plugin version 1.16.26
* Build date 11th September 2018, expiring on the 11th of October 2018.
* Works only with Kibana version 6.0.0


## When an update is out
You will receive another email notification that a new deliverable is available.

If the update contains a security fix, it is very important that you take action and **update the plugin immediately**.


# Installation
You can install this as a normal Kibana plugin using the `bin/kibana-plugin` utility. 

> Please note: the Kibana and Elasticsearch plugins version string need to match. That is, both plugin zip file names need to contain something like 1.16.26_es6.4.0 (or 1.16.26-XXXXXXXX_es6.4.0 in case it's a trial build). 

From your Kibana installation, launch the command:

```bash
$ bin/kibana-plugin install file:///home/user/downloads/readonlyrest_kbn-X.Y.Z_esW.Q.U.zip
```

Notice how we need to type in the format `file://` + absolute path (yes, with three slashes).

Kibana "optimization" process is long and buggy, please make sure that the plugin's css file file gets generated, otherwise create it empty (no css is loaded from there anyway).
To do so, run this command:

```bash
$ touch optimize/bundles/readonlyrest_kbn.style.css
```

Or on Windows:

```bash
type nul > optimize\bundles\readonlyrest_kbn.style.css
```

## Uninstall 

```bash
$ bin/kibana-plugin remove readonlyrest_kbn
```


## Upgrade

Just uninstall the old version and install the new version.

```bash
$ bin/kibana-plugin remove readonlyrest_kbn
```

Install the new version of ReadonlyREST into Kibana.

```bash
$ bin/kibana-plugin install file:///home/user/downloads/readonlyrest_kbn-*.zip

$ touch optimize/bundles/readonlyrest_kbn.style.css
```

Restart Kibana.



# Configuration

ReadonlyREST for Kibana is completely remote-controlled from the Elasticsearch configuration. 
Login credentials, hidden Kibana apps, etc. are all going to be configured from the Elasticearch side via the usual "rules".
This means the configuration will be kept all in one place and if you used ReadonlyREST before , it will be also very familiar.


> In this document, every time you will encounter references to "readonlyrest.yml" or "elasticsearch.yml", we will be referring **to the configuration files in the Elasticsearch plugin**. 
In general, by design, we tend to concentrate all configuration within the main plugin (the Elasticsearch one) as much as possible.


## Example: multiuser ELK

Make sure X-Pack is uninstalled or disabled from `elasticsearch.yml` (on the Elasticsearch side) and `kibana.yml` (on the Kibana side):
This is how you disable X-pack modules:

```yaml
# For X-Pack users: you may only leave monitoring on. 
# Don't add this if X-Pack is not installed at all
xpack.graph.enabled: false
xpack.ml.enabled: false
xpack.monitoring.enabled: true
xpack.security.enabled: false
xpack.watcher.enabled: false
```

This is a typical example of configuration snippet to add at the end of your `readonlyrest.yml` (the settings file of the Elasticsearch plugin), to support ReadonlyREST PRO.

```yaml

readonlyrest:

    access_control_rules:

    - name: "::LOGSTASH::"
      auth_key: logstash:logstash
      actions: ["indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
      indices: ["logstash-*"]

    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

    - name: "::RO::"
      auth_key: ro:dev
      kibana_access: ro
      indices: [ ".kibana", ".kibana-devnull", "logstash-*"]
      kibana_hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:management"]

    - name: "::RW::"
      auth_key: rw:dev
      kibana_access: rw
      indices: [".kibana", ".kibana-devnull", "logstash-*"]
      kibana_hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:management"]


    - name: "::ADMIN::"
      auth_key: admin:dev
      # KIBANA ADMIN ACCESS NEEDED TO EDIT SECURITY SETTINGS IN ROR KIBANA APP!
      kibana_access: admin

    - name: "::WEBSITE SEARCH BOX::"
      indices: ["public"]
      actions: ["indices:data/read/*"]
```


## Very important

Whatever your configuration ends up being, remember:

* The admin user has `kibana_access: admin` 
* Remember to use `kibana_hide_apps: ["readonlyrest_kbn"]` to hide the ReadonlyREST icon from who is not meant to use it (makes for a better UX).

### Rules ordering matters

> Blocks related to the authentication of the users should be at the top of the ACL

One of the most common mistakes is forgetting that the ACL blocks are evaluated in order from the first to the last.

So, some request with credentials can be let through from one of the first blocks and come back to Kibana with no user identity metadata associated.

Take this example of troublesome ACL:

```yml
    # PROBLEMATIC SETTINGS (EXAMPLE) ⚠️
    
    access_control_rules:

    - name: "::FIRST BLOCK::"
      hosts: ["127.0.0.1"]
      actions: [...]

    - name: "::ADMIN::"
      auth_key: admin:dev
      kibana_access: admin
```
The user will be able to login because the login request will be allowed by the first ACL block. But the ACL will not have resolved any metadata about the user identity (credentials checking was ignored)!

This means the response to the Kibana login request will contain no user identity metadata (username, hidden apps, etc) and ReadonlyREST for Kibana won't be able to function correctly.

The solution to this is to reorder the ACL blocks, so the ones that authenticate Kibana users are on the top.


```yml
    # SOLUTION: KIBANA USER AUTH RELATED BLOCKS GO FIRST! ✅👍
    
    access_control_rules:
    
    - name: "::ADMIN::"
      auth_key: admin:dev
      kibana_access: admin
      
    - name: "::FIRST BLOCK::"
      hosts: ["127.0.0.1"]
      actions: [...]

```



## Kibana configuration

Activate authentication for the Kibana server: let the Kibana daemon connect to Elasticsearch using a pair of credentials we just defined in `readonlyrest.yml` (see above, the ::KIBANA-SRV:: block).

Open up `conf/kibana.yml` and add the following:

```yaml
# This is kibana.yml, but copy the exact same in elasticsearch.yml if you have to use some X-pack features.
xpack.graph.enabled: false
xpack.ml.enabled: false
xpack.monitoring.enabled: true
xpack.security.enabled: false # this is fundamental!
xpack.watcher.enabled: false

# Kibana server use ::KIBANA-SRV:: credentials
elasticsearch.username: "kibana"
elasticsearch.password: "kibana"
```

And of course also make sure `elasticsearch.url` points to the designated Elasticsearch instance (check also the http or https)

## Proxy Auth
ROR for Elasticsearch can delegate authentication to a reverse proxy which will enforce some kind of authentication, and pass the successfully authenticated user's name inside a `X-Forwarded-For` header.

> Today, it's possible to skip the regular ROR login form and use the "delegated authentication" technique in ROR for Kibana as well. 

1. Configure ROR for ES to expect delegated authentication (see [`proxy_auth` rule](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#authentication)) in ROR for ES documentation.
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.proxy_auth_passthrough: true`

Now ROR for Kibana will **skip the login form entirely**, and will only require that all incoming requests must carry a `X-Forwarded-For` header containing the user's name. Based on this identity, ROR for Kibana will build an encrypted cookie and handle your session normally.

### Custom Logout link
Normally, when a user presses the logout button in ROR for Kibana, it deletes the encrytped cookie that represents the users identity and the login form is shown.

However, when the authentication is delegated to a proxy, the logout button needs to become a link to some URL capable to unregister the session a user initiated within the proxy.

For this, ROR for Kibana offers a way to customize the logout button's URL:

1. Find a link that will delete the reverse proxy's user session
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.custom_logout_link: https://..../logout`

Now users that gained a session through delegated auth, can also click on the logout button in ROR for kibana and actually exit their session.

### Caveat
Enabling proxy auth passthrough will relax the requirement to provide a password. Therefore, don't enable this option if you don't make sure Kibana can **only be accessed through the reverse proxy***.


## JWT Token Forwarding as URL Query Parameter
Alternatively to typing in credentials in the standard login form, it is possible to create an authenticated Kibana session by passing a JWT token as a query parameter in a URL.

### Configuration
To enable this feature in ReadonlyREST, you need to:

* Have JWT authentication configured in ReadonlyREST (modifying `readonlyrest.yml` or the cluster wide settings UI in the Kibana plugin). [See how](elasticsearch.md#json-web-token-jwt-auth).

* Specify the query parameter name in `kibana.yml` by adding the line `readonlyrest_kbn.jwt_query_param: "jwt"` as a string, in our case "jwt".

### In Action
Once Kibana is restarted, you will be able to navigate to a link like this:
```
http://kibana:5601/login?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

The following will happen:

1. The Kibana plugin will forward the JWT token found in the query parameter into the `Authrization` header in a request to Elasticsearch.

2. Elasticsearch will cryptographically authenticate and resolve the user's identity from the JWT claims.

3. Kibana will write an encrypted cookie in your browser and use that from now on for the length of the autenticated session. From here onwards, the session management will be identical to the normal login form flow.

4. When the user presses logout, Kibana will delete the cookie and redirect you to the login form, or whatever link you configured as `readonlyrest_kbn.custom_logout_link`.

#### Deep linking with JWT
Because the identity is embedded in the link, and ReadonlyREST is able to authenticate the call on the fly, the JWT authentication can be used in conjunction with `nextUrl` query parameter for sharing deep links inside Kibana apps, or embeddinig visualizations and dashboards inside I-Frames.

##### Anatomy of a JWT deep link
```
http://kibana:5601/login?jwt=<the-token>&nextUrl=urlEncode(<kibana-path>)
```

In Javascript one can compose a JWT deep link as follows:

```js
var absoluteKibanaPath = '/app/kibana#/visualize/edit/28dcde30-2258-11e8-82a3-af58d04b3c02?_g=()';

var url = 'http://kibana:5601/login?jwt=' + 
           jwtToken + 
           '&nextUrl=' + 
           encodeURI(absoluteKibanaPath);
           
console.log("Final JWT deep link: " + url)
```

The result may look something like this:
```
http://localhost:5601/login?nextUrl=%2Fapp%2Fkibana%23%2Fvisualize%2Fedit%2F28dcde30-2258-11e8-82a3-af58d04b3c02%3F_g%3D%28%29&jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

# SAML
ReadonlyREST Enterprise supports service provider initiated via SAML. This connector supports both SSO (single sign on) and SLO (single log out).
Here is how to configure it.

## Configure `ror_kbn_auth` bridge

In order for the user identity information to flow securely from Kibana to Elasticsearch, we need to set up the two plugin with a shared secret, that is: an arbitrarily long string.

## Elasticsearch side
Edit `readonlyrest.yml`

```yml
readonlyrest:
    access_control_rules:
    
    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana
      
    ... all usual blocks of rules...
        
    - name: "ReadonlyREST Enterprise instance #1"
      ror_kbn_auth:
        name: "kbn1"

    ror_kbn:
    - name: kbn1
      signature_key: "my_shared_secret_kibana1_(min 256 chars)" # <- use environmental variables for better security!
```

**⚠️IMPORTANT** the Basic HTTP auth credentials for the Kibana server are **still needed** for now, due to how Kibana works.

## Kibana side

Edit `kibana.yml` and append:

```yaml
readonlyrest_kbn.auth:
  signature_key: "my_shared_secret_kibana1(min 256 chars)"
  saml:
    enabled: true
    entryPoint: 'https://my-saml-idp/saml2/http-post/sso'
    kibanaExternalHost: 'my.public.hostname.com' # <-- public URL used by the Identity Provider to call back Kibana with the "assertion" message
    usernameParameter: 'nameID'
    groupsParameter: 'memberOf'
    logoutUrl: 'https://my-saml-idp/saml2/http-post/slo'
    
    # OPTIONAL advanced parameters
    # decryptionCert: /etc/ror/integration/certs/pub.crt
    # cert: /etc/ror/integration/certs/dag.crt
    # decryptionPvk: /etc/ror/integration/certs/decrypt_pvk.crt
    # issuer: saml_sso_idp
     
```
* `issuer`: issuer string to supply to identity provider.
* `disableRequestedAuthnContext`: if truthy, do not request a specific authentication context. This is known to help when authenticating against Active Directory (AD FS) servers.
* `decryptionPvk`: Service Provider Private Key. Private key that will be used to attempt to decrypt any encrypted assertions that are received.
* cert: The downloadable certificate in IDP Metadata (file, absolute path)

For advanced SAML options, see [passport-saml documentation](https://github.com/bergie/passport-saml).


## Identity provider side


1. Enter the settings of your identity provider, create a new app.
2. Configure it using the information found by connecting to `http://my.public.hostname.com/ror_kbn_sso/metadata.xml`

Example response:
```xml
<?xml version="1.0"?>
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" entityID="onelogin_saml" ID="onelogin_saml">
  <SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://my.public.hostname.com/ror_kbn_sso/notifylogout"/>
    <NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
    <AssertionConsumerService index="1" isDefault="true" Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://my.public.hostname.com/ror_kbn_sso/assert"/>
  </SPSSODescriptor>
</EntityDescriptor>
```

3. Create some users and some groups in the identity provider app
4. Check the user profile parameter names that the identity provider uses during the assertion callback ( **TIP**: set kibana in debug mode so ReadonlyREST will print the user profile).
5. Match the name of the parameter used by the identity provider to carry the unique user ID (in the assertion message) to the `usernameParameter` kibana YAML setting.
6. If you want to use SAML for authorization, take care of matching also the `groupsParameter` to the parameter name found in the assertion message to the kibana YAML setting.
 
# Load balancers

## Enable healthcheck endpoint 

Normally a load balancer needs a health check URL to see if the instance is still running, you can whitelist this Kibana path so the load balancer avoids a redirection to `/login`.

Edit `kibana.yml`

```yml
readonlyrest_kbn.whitelistedPaths: [".*/api/status$"]

```
## Common cookie encryption secret
**WARNING⚠️: this only works with Kibana 5.x. Use sticky sessions in your load balancer instead**
When you run multiple Kibana instances behind a load balancer, a user will have their identity cookie created and encrypted in one instance. 

A fresh cookie encryption key is generated at startup time on every Kibana node. This means that each Kibana instance behind the load balancer will have a different encryption key.

This is a problem because a cookie encrypted by one instance won't be recognised on the other instances.

> Today it's possible to share the cookie encryption key in all the Kibana instances

1. Come up with a string of 32 characters length or more
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.cookiePass: "12345678901234567890123456789012"` 
3. Do the above in all Kibana nodes behind the load balancer, and restart them. 


# Login screen tweaking
It is possible to customise the look of the login screen.

## Add your company logo
It's recommended to use a transparent PNG, negative logo. Ideally a white foreground, and transparent background.

Open `config/kibana.yml` and append the following:
```yml
readonlyrest_kbn.login_custom_logo: 'https://.../logo.png'
```
## Add custom CSS/JS
You have the opportunity to inject HTML code right before the closing head tag (`</head>`).

Open `config/kibana.yml` and append the following:
```yml
readonlyrest_kbn.login_html_head_inject: '<style> * { color:red; }</style>'
```
# Kibana UI tweaking
With ReadonlyREST Enterprise, it's possible to inject custom CSS and Javascript to achieve a customised user experience for your users/tenants.

## Inject custom CSS in Kibana
Open `config/kibana.yml` and append the following:
```yml
readonlyrest_kbn.kibana_custom_css_inject: '.global-nav { background-color: green }'
```
Alternatively, it's possible to load the CSS from a file in the filesystem:

```yml
readonlyrest_kbn.kibana_custom_css_inject_file: '/tmp/custom.css'
```

## Inject custom JS in Kibana
```yml
readonlyrest_kbn.kibana_custom_js_inject: '$(".global-nav__logo").hide(); alert("hello!")'
```

# Tenancy index templating
When a tenants logs in for the first time, ReadonlyREST Enterprise will create the ".kibana" index associated to the tenancy. For example, it will create and initialize the ".kibana_user1" index, where "user1" will store all the visualizations, dashboards, settings and index-patterns.

The issue is that "user1"'s user experience will be really raw as they will see a completely blank Kibana tenancy. Not even a default index pattern will be present. And this is particularly challenging if the tenant is supposed to be read-only (i.e. kibana_access: "ro") because they won't even have privileges to create their own index-pattern, let alone any dashboards.

To fix this, ReadonlyREST Enterprise offers the possibility for administrators to create a template kibana index from which all the Kibana objects will be copied over to the newly initialised tenancy.

## How to use tenancy templating

An administrator will need to create the template tenancy, populate it with the default Kibana objects (index-patterns, dashboards) and configure ReadonlyREST Enterprise to take the index template it in use. Let's see this step by step:

### Create the template tenancy

Let's start to add to our access control list (found in $ES_HOME/config/readonlyrest.yml, or ReadonlyREST App in Kibana) a local user "administrator" that will belong to two tenancies: the default one (stored in .kibana index), and the template one (stored in .kibana_template index).

```yml
readonlyrest:
  audit_collector: true

  access_control_rules:

  - name: "::KIBANA-SRV::"
    auth_key: kibana:kibana
    verbosity: error

  - name: "Admin Tenancy"
    groups: ["Admins"]
    verbosity: error
    kibana_access: admin
    kibana_index: ".kibana"
  
  - name: "Template Tenancy"
    groups: ["Template"]
    verbosity: error
    kibana_access: admin
    kibana_index: ".kibana_template"
      
 users:
 - name: administrator
   auth_key: administrator:dev
   groups: ["Admins", "Template"] # can hop between two tenancies with top-left drop-down menu
     
```
NB: If you know what you are doing, you can  add a tenancy with kibana_index: ".kibana_template" adding a LDAP/SAML group to your administrative user. 

## Configure the template tenancy
Now login as administrator in Kibana, hop into the "Template" tenancy, and start configuring the default UX for your future tenants. Add all the index patterns, create or import all the dashboards you want.

## Configure the template tenancy index in ReadonlyREST Enterprise
Open kibana.yml and add the following line:

```yml
readonlyrest_kbn.kibanaIndexTemplate: ".kibana_template"
```

Now, ReadonlyREST Enterprise will look for the ".kibana_template" index, and try to copy over all its documents every time a new kibana index is initialised to support a new tenancy.

## Try it out

Restart Kibana with the new setting. Add a new tenancy to the ACL:

```yml
readonlyrest:
  audit_collector: true

  access_control_rules:

  - name: "::KIBANA-SRV::"
    auth_key: kibana:kibana
    verbosity: error

  - name: "Admin Tenancy"
    groups: ["Admins"]
    verbosity: error
    kibana_access: admin
    kibana_index: ".kibana"
  
  - name: "Template Tenancy"
    groups: ["Template"]
    verbosity: error
    kibana_access: admin
    kibana_index: ".kibana_template"
    
  # Newly added tenant!
  - name: user1
    auth_key: user1:passwd
    kibana_access: rw
    kibana_index: ".kibana_user1"
      
 users:
 - name: administrator
   auth_key: administrator:dev
   groups: ["Admins", "Template"] # can hop between two tenancies with top-left drop-down menu
````

Now try to login as user1, and ReadonlyREST Enterprise should initialise the index ".kibana_user1" with all the index patterns and dashboards contained in the template tenancy.
