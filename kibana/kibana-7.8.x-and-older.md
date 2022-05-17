---
description: User manual for ReadonlyREST Enterprise/PRO/Free plugins
---

# Kibana 7.8.x and older

## Kibana Plugin overview

ReadonlyREST plugin for Kibana is not open source, and it's offered as part of the [ReadonlyREST PRO](https://readonlyrest.com/pro.html) and [ReadonlyREST ENTERPRISE](https://readonlyrest.com/enterprise.html), and [ReadonlyREST Free](https://readonlyrest.com/free) packages. See product descriptions and a comparison chart in the official [ReadonlyREST website](https://readonlyrest.com)

ReadonlyREST plugins for Kibana **always require** ReadonlyREST Free to be installed in the Elasticsearch nodes your Kibana instance(s) will connect to.

It's not mandatory to install ReadonlyREST Free in all Elasticsearch nodes, but only in the ones in where you need the HTTP interface to be secured.

### After purchasing

You will receive a link to the plugin zip file in an email. Download your zip.

You will be able to download it also in the future as long as your subscription is active.

### Version strings

All our plugins include in their file name a version string. For example the file `readonlyrest-1.16.26_es6.4.0.zip` has a version string `1.16.26_es6.4.0`.

#### Reading version strings

Given the version string `1.16.26_es6.4.0`

* ReadonlyREST plugin code version `1.16.26`
* Works only with Elasticsearch/Kibana version `6.4.0`

The "es" stands for "Elastic stack" which used to mean the family of products made by Elastic which get released at the same time under the same version number. This was chosen **before** Elastic renamed their X-Pack commercial offer to Elastic Stack.

To be clear, there is no affiliation between ReadonlyREST and Elastic, or their commercial products.

#### Trial builds version strings

Trial builds are valid for 30 days after they were built, and they will stop working soon after the time is elapsed. Trial builds have a special version string which includes a build-time timestamp.

I.e. `readonlyrest_kbn_pro-1.16.26-20180911_es6.0.0.zip`

* ReadonlyREST PRO plugin version 1.16.26
* Build date 11th September 2018, expiring on the 11th of October 2018.
* Works only with Kibana version 6.0.0

### When an update is out

You will receive another email notification that a new deliverable is available.

If the update contains a security fix, it is very important that you take action and **update the plugin immediately**.

## Installation

You can install this as a normal Kibana plugin using the `bin/kibana-plugin` utility.

### Install via URL

This installation method is more practical if your Kibana server is connected to the internet.

According to what edition of ReadonlyREST you want to install, from your Kibana installation, launch one of the commands:

Please note that this will always download the latest version of Kibana plugin available for the current supported Elasticsearch version.

```bash
# ReadonlyREST Free edition
$ bin/kibana-plugin install "https://api.beshu.tech/download/kbn?edition=kbn_free&email=<your_email_address>"

# ReadonlyREST PRO (30 days trial) edition
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_pro&email=<your_email_address>"

# ReadonlyREST Enterprise (30 days trial) edition
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_enterprise&email=<your_email_address>"
```

If you want to download the latest version of plugin for a specific version of Elasticsearch, then use query parameter esVersion to specify your required Elasticsearch version.

```bash
# ReadonlyREST Free edition for Elasticsearch 7.6.1
$ bin/kibana-plugin install "https://api.beshu.tech/download/kbn?edition=kbn_free&esVersion=7.6.1&email=<your_email_address>"

# ReadonlyREST PRO (30 days trial) edition for Elasticsearch 7.6.1
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_pro&esVersion=7.6.1&email=<your_email_address>"

# ReadonlyREST Enterprise (30 days trial) edition for Elasticsearch 7.6.1
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_enterprise&esVersion=7.6.1&email=<your_email_address>"
```

If you want to download an older version of plugin for a specific version of Elasticsearch, then use query parameter pluginVersion along with esVersion.

```bash
# ReadonlyREST Free edition - version 1.22.0 for Elasticsearch 7.6.1
$ bin/kibana-plugin install "https://api.beshu.tech/download/kbn?edition=kbn_free&esVersion=7.6.1&pluginVersion=1.22.0&email=<your_email_address>"

# ReadonlyREST PRO (30 days trial) edition - version 1.22.0 for Elasticsearch 7.6.1
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_pro&esVersion=7.6.1&pluginVersion=1.22.0&email=<your_email_address>"

# ReadonlyREST Enterprise (30 days trial) edition - version 1.22.0 for Elasticsearch 7.6.1
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_enterprise&esVersion=7.6.1&pluginVersion=1.22.0&email=<your_email_address>"
```

If you are a PRO or Enterprise subscriber, the link will include an extra parameter "token" which can only be used in association with the provided email address.

You can append required plugin version and Elasticsearch version query parameters to download specific version as described above.

**NB: This URL is personal, and should be handled as a secret.**

```bash
# ReadonlyREST PRO (Official) edition
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_pro&email=<your_email_address>&token=<your_secret_token>"

# ReadonlyREST Enterprise (30 days trial) edition
$ bin/kibana-plugin install "https://api.beshu.tech/download/trial?edition=kbn_enterprise&email=<your_email_address>&token=<your_secret_token>"
```

You can obtain official links with personal secret tokens using our self service [download form](https://readonlyrest.com/download/), once your email address has been recognized as active subscriber.

### Install from zip file

```bash
$ bin/kibana-plugin install file:///home/user/downloads/readonlyrest_kbn-X.Y.Z_esW.Q.U.zip
```

Notice how we need to type in the format `file://` + absolute path (yes, with three slashes).

### Uninstall

```bash
$ bin/kibana-plugin remove readonlyrest_kbn
```

### Upgrade

Just uninstall the old version and install the new version.

```bash
$ bin/kibana-plugin remove readonlyrest_kbn
```

Install the new version of ReadonlyREST into Kibana.

```bash
$ bin/kibana-plugin install file:///home/user/downloads/readonlyrest_kbn-*.zip

# Only for older versions (until Kibana early 6.x)
$ touch optimize/bundles/readonlyrest_kbn.style.css
```

Restart Kibana.

### Using ROR with a reverse proxy

ROR - just like Kibana itself - is meant to be used either with a proxy or without one, but not both simultaneously. If you decide to set the `server.basePath` property in `kibana.yml` be sure to access RoR via a proxy, as it will not work properly when accessed directly.

## Configuration

ReadonlyREST for Kibana is completely remote-controlled from the Elasticsearch configuration. Login credentials, hidden Kibana apps, etc. are all going to be configured from the Elasticearch side via the usual "rules". This means the configuration will be kept all in one place and if you used ReadonlyREST before , it will be also very familiar.

> In this document, every time you will encounter references to "readonlyrest.yml" or "elasticsearch.yml", we will be referring to the configuration files **in the Elasticsearch plugin** (our Kibana plugins do not need a "readonlyrest.yml").

In general, by design, we tend to concentrate all configuration within the main plugin (the Elasticsearch one) as much as possible.

### Clusterwide Settings vs readonlyrest.yml

Our Kibana plugins introduce a "ReadonlyREST" Kibana app. From here, you can edit the security settings of the whole Elasticsearch cluster, and they will take effect within 10 seconds in all Elasticsearch cluster nodes without the need to restart them.

When you change the security settings from the Kibana app, they will be saved in a special index called ".readonlyrest", so all the Elasticsearch nodes will pick them up. You can customize a name of the index by setting `readonlyrest.settings_index: .my_custom_readonlyrest` in `elasticsearch.yml` file (remember to set the same value for all your ES nodes).

When an Elasticsearch node restarts, the order of settings evaluation is the following: 1. Attempt to find valid settings in readonlyrest.yml 2. If none is found, look inside elasticsearch.yml 3. Once successfully bootstrapped using file-based settings, attempt to read ".readonlyrest" index 4. If the index exists and contains valid settings, override file based settings with the ones from the index. 5. Pressing "save" in the cluster wide settings app, will **not overwrite the readonlyrest.yml** file.

Best practices:

* Build and update your production security settings from the Kibana app (will be saved in index)
* Protect the ".readonlyrest" Kibana index with an ACL rule

#### Loading settings: order of precedence

As you read, there are two possible places where the settings can be read from:

* `readonlyrest.yml` a file the user needs to create in the same directory where `elasticsearch.yml` is found.
* `.readonlyrest` index. Our Kibana plugins' GUI (PRO/Enterprise) is programmed to write this index.

When the ES plugin boots up, it follows some logic to evaluate where to read the YAML settings from. The following diagram shows how that works.

![config loading diagram](<../.gitbook/assets/ror\_config\_loading\_diagram (1).png>)

#### Malformed in-index settings

If for some reason the in-index settings get corrupted and ROR can't parse them, then neither settings from file or in-index settings can be loaded, so ES can't start. In this case ES would print message like:

```
Loading ReadonlyREST settings from index failed: Settings config content is malformed. Details: while scanning a quoted scalar
 in 'reader', line 9, column 17:
          auth_key: "admin:container
                    ^
```

To recover from this state, set `readonlyrest.force_load_from_file: true` in `elasticsearch.yaml` on one node `es1`.

Example recovery settings:

elasticsearch.yaml

```yaml
[...]
readonlyrest:
  force_load_from_file: true
```

readonlyrest.yaml

```yaml
readonlyrest:

  access_control_rules:
  - name: "::ADMIN recover::"
    auth_key: admin:dev
    indices: ["*"]
```

Then remove in-index settings index manually.

```bash
curl -X DELETE "admin:dev@es1:9200/.readonlyrest?pretty"
```

Now you can restore your settings to `readonlyrest.yml`, remove `readonlyrest.force_load_from_file: true` `from elasticsearch.yaml` and restart node.

### Example: multiuser ELK

Make sure X-Pack is uninstalled or disabled from `elasticsearch.yml` (on the Elasticsearch side) and `kibana.yml` (on the Kibana side): This is how you disable X-pack modules:

```yaml
# For X-Pack users: you may only leave monitoring on. 
# Don't add this if X-Pack is not installed at all, or Kibana won't start.
xpack.monitoring.enabled: true
xpack.security.enabled: false
xpack.watcher.enabled: false
xpack.telemetry.enabled: false
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
      indices: [ ".kibana", "logstash-*"]
      kibana_hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:stack_management"]

    - name: "::RW::"
      auth_key: rw:dev
      kibana_access: rw
      indices: [".kibana", "logstash-*"]
      kibana_hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:stack_management"]


    - name: "::ADMIN::"
      auth_key: admin:dev
      # KIBANA ADMIN ACCESS NEEDED TO EDIT SECURITY SETTINGS IN ROR KIBANA APP!
      kibana_access: admin

    - name: "::WEBSITE SEARCH BOX::"
      indices: ["public"]
      actions: ["indices:data/read/*"]
```

### Very important

Whatever your configuration ends up being, remember:

* The admin user has `kibana_access: admin`&#x20;
* Remember to use `kibana_hide_apps: ["readonlyrest_kbn"]` to hide the ReadonlyREST icon from who is not meant to use it (makes for a better UX).

#### Rules ordering matters

> Blocks related to the authentication of the users should be at the top of the ACL

One of the most common mistakes is forgetting that the ACL blocks are evaluated in order from the first to the last.

So, some request with credentials can be let through from one of the first blocks and come back to Kibana with no user identity metadata associated.

Take this example of troublesome ACL:

```
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

```
    # SOLUTION: KIBANA USER AUTH RELATED BLOCKS GO FIRST! ✅👍

    access_control_rules:

    - name: "::ADMIN::"
      auth_key: admin:dev
      kibana_access: admin

    - name: "::FIRST BLOCK::"
      hosts: ["127.0.0.1"]
      actions: [...]
```

#### Session cookie expiration

When a user logs in, ReadonlyREST will write an encrypted cookie in the browser. This cookie has an time to live that can be tweaked with the following configuration key in `kibana.yml`.

```
readonlyrest_kbn.session_timeout_minutes: 600 # defaults to 4320 (3 days)
```

#### Clearing Session History

By default, all the session data like search history, dev tool commands history, etc, will be wiped out from the browser whenever a new user is logged in, or a user changes tenancy. To override this behaviour, use this setting:

```
readonlyrest_kbn.clearSessionOnEvents: ["never"]
```

Possible values: `"login", "tenancyHop", "never"`.

#### Kibana App strings

Examples of valid arguments for the `kibana_hide_apps: [...]` rule (readonlyrest.yml)

| hide-app key                     | App name         | App url                                                                                                                                                                                                                     |
| -------------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| kibana:discover                  | Discover         | [http://kibana-url:5601/app/kibana#/discover](http://kibana-url:5601/app/kibana#/discover)                                                                                                                                  |
| kibana:visualize                 | Visualize        | [http://kibana-url:5601/app/kibana#/visualize](http://kibana-url:5601/app/kibana#/visualize)                                                                                                                                |
| kibana:dashboard                 | Dashboard        | [http://kibana-url:5601/app/kibana#/dashboards](http://kibana-url:5601/app/kibana#/dashboards)                                                                                                                              |
| timelion                         | Timelion         | [http://kibana-url:5601/app/timelion](http://kibana-url:5601/app/timelion)                                                                                                                                                  |
| canvas                           | Canvas           | [http://kibana-url:5601/app/canvas](http://kibana-url:5601/app/canvas)                                                                                                                                                      |
| maps                             | Maps             | [http://kibana-url:5601/app/maps](http://kibana-url:5601/app/maps)                                                                                                                                                          |
| code                             | Code (Beta)      | [http://kibana-url:5601/app/code](http://kibana-url:5601/app/code)                                                                                                                                                          |
| ~~readonlyrest\_kbn~~ (obsolete) | ~~ReadonlyREST~~ | ~~\~\~\[~~[http://kibana-url:5601/app/readonlyrest\_kbn~~\](http://kibana-url:5601/app/readonlyrest\_kbn)~~\~\~](http://kibana-url:5601/app/readonlyrest\_kbn\~\~]\(http://kibana-url:5601/app/readonlyrest\_kbn\)\~\~\~\~) |
| ml                               | Machine Learning | [http://kibana-url:5601/app/ml](http://kibana-url:5601/app/ml)                                                                                                                                                              |
| infra:home                       | Infrastructure   | [http://kibana-url:5601/app/infra#/infrastructure/inventory?\_g=(](http://kibana-url:5601/app/infra#/infrastructure/inventory?\_g=%28))                                                                                     |
| infra:logs                       | Logs             | [http://kibana-url:5601/app/infra#/logs?\_g=(](http://kibana-url:5601/app/infra#/logs?\_g=%28))                                                                                                                             |
| apm                              | APM              | [http://kibana-url:5601/app/apm](http://kibana-url:5601/app/apm)                                                                                                                                                            |
| uptime                           | Uptime           | [http://kibana-url:5601/app/uptime#/](http://kibana-url:5601/app/uptime#/)                                                                                                                                                  |
| siem                             | SIEM             | [http://kibana-url:5601/app/siem](http://kibana-url:5601/app/siem)                                                                                                                                                          |
| graph                            | Graph            | [http://kibana-url:5601/app/graph](http://kibana-url:5601/app/graph)                                                                                                                                                        |
| kibana:dev\_tools                | Dev Tools        | [http://kibana-url:5601/app/kibana#/dev\_tools](http://kibana-url:5601/app/kibana#/dev\_tools)                                                                                                                              |
| monitoring                       | Stack Monitoring | [http://kibana-url:5601/app/monitoring](http://kibana-url:5601/app/monitoring)                                                                                                                                              |
| kibana:stack\_management         | Stack Management | [http://kibana-url:5601/app/kibana#/management](http://kibana-url:5601/app/kibana#/management)                                                                                                                              |

### Kibana configuration

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

### Proxy Auth

ROR for Elasticsearch can delegate authentication to a reverse proxy which will enforce some kind of authentication, and pass the successfully authenticated user's name inside a `X-Forwarded-User` header.

> Today, it's possible to skip the regular ROR login form and use the "delegated authentication" technique in ROR for Kibana as well.

1. Configure ROR for ES to expect delegated authentication (see [`proxy_auth` rule](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#proxy\_auth-)) in ROR for ES documentation.
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.proxy_auth_passthrough: true`

Now ROR for Kibana will **skip the login form entirely**, and will only require that all incoming requests must carry a `X-Forwarded-User` header containing the user's name. Based on this identity, ROR for Kibana will build an encrypted cookie and handle your session normally.

#### Custom Logout link

Normally, when a user presses the logout button in ROR for Kibana, it deletes the encrypted cookie that represents the users identity and the login form is shown.

However, when the authentication is delegated to a proxy, the logout button needs to become a link to some URL capable to unregister the session a user initiated within the proxy.

For this, ROR for Kibana offers a way to customize the logout button's URL:

1. Find a link that will delete the reverse proxy's user session
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.custom_logout_link: https://..../logout`

Now users that gained a session through delegated auth, can also click on the logout button in ROR for kibana and actually exit their session.

#### Custom Login link

When you delegate authentication to an external service, you can tell ReadonlyREST to skip the classic login form entirely and redirect users to your proxy or identity provider's login screen.

To enable this:

1. Find your authentication proxy or identity provider login URL for the ROR app
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.custom_login_link: "https://../login"`

The advantage of this approach is a streamlined user experience for users that login with an external IdP. The disadvantage is that you give up the possibility to login as a local user in ROR, as the login form will be always skipped.

#### Caveat

Enabling proxy auth passthrough will relax the requirement to provide a password. Therefore, don't enable this option if you don't make sure Kibana can **only be accessed through the reverse proxy\***.

### JWT Token Forwarding as URL Query Parameter

Alternatively to typing in credentials in the standard login form, it is possible to create an authenticated Kibana session by passing a JWT token as a query parameter in a URL.

#### Configuration

To enable this feature in ReadonlyREST, you need to:

* Have JWT authentication configured in ReadonlyREST (modifying `readonlyrest.yml` or the cluster wide settings UI in the Kibana plugin). [See how](../elasticsearch.md#json-web-token-jwt-auth).
* Specify the query parameter name in `kibana.yml` by adding the line `readonlyrest_kbn.jwt_query_param: "jwt"` as a string, in our case "jwt".

#### In Action

Once Kibana is restarted, you will be able to navigate to a link like this:

```
http://kibana:5601/login?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

The following will happen:

1. The Kibana plugin will forward the JWT token found in the query parameter into the `Authorization` header in a request to Elasticsearch.
2. Elasticsearch will cryptographically authenticate and resolve the user's identity from the JWT claims.
3. Kibana will write an encrypted cookie in your browser and use that from now on for the length of the authenticated session. From here onwards, the session management will be identical to the normal login form flow.
4. When the user presses logout, Kibana will delete the cookie and redirect you to the login form, or whatever link you configured as `readonlyrest_kbn.custom_logout_link`.

**Deep linking with JWT**

Because the identity is embedded in the link, and ReadonlyREST is able to authenticate the call on the fly, the JWT authentication can be used in conjunction with `nextUrl` query parameter for sharing deep links inside Kibana apps, or embedding visualizations and dashboards inside I-Frames.

**Anatomy of a JWT deep link**

```
http://kibana:5601/login?jwt=<the-token>&nextUrl=urlEncode(<kibana-path>)
```

In Javascript one can compose a JWT deep link as follows:

```javascript
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

## Audit log

The audit log feature is widely described in [📖docs for Elasticsearch plugin](../elasticsearch.md#audit-logs). Kibana plugin has predefined dashboard representing collected audit data.

### Loading visualization

In the _Audit_ tab of the ReadonlyREST Kibana app, there is a button that automatically creates a dashboard with some audit log specific visualizations.

![audit log tab](<../.gitbook/assets/audit\_tab (1).png>)

Click the _Load_ button to load the dashboard and visualizations. An _Override_ checkbox allows to reload the default dashboard and visualizations. It will override any previously loaded audit log dashboard.

![loading visualization](https://github.com/beshu-tech/readonlyrest-docs/tree/64130b8d9cafefb344b40743cf6df4dc99d60169/kibana/.gitbook/assets/load\_audit\_dashboard%20\(1\)%20\(1\)%20\(1\)%20\(1\)%20\(4\).png)

In detail, this feature creates three Kibana "saved objects":

* an index pattern for `readonlyrest_audit-*`
* a dashboard called `ReadonlyREST Audit Log`
* some visualizations

### Dashboard

The audit log dashboard, by default, has only a few basic visualizations. They cover security, access logs, and performance metrics.

## SAML

ReadonlyREST Enterprise supports service provider initiated via SAML. This connector supports both SSO (single sign on) and SLO (single log out). Here is how to configure it.

### Configure `ror_kbn_auth` bridge

In order for the user identity information to flow securely from Kibana to Elasticsearch, we need to set up the two plugin with a shared secret, that is: an arbitrarily long string.

### Elasticsearch side

Edit `readonlyrest.yml`

```
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

### Kibana side

Edit `kibana.yml` and append:

```yaml
readonlyrest_kbn.auth:
  signature_key: "my_shared_secret_kibana1(min 256 chars)"
  saml_serv1:
    enabled: true
    type: saml
    issuer: ror
    buttonName: "Partner's SSO Login"
    entryPoint: 'https://my-saml-idp/saml2/http-post/sso' # <-- identity Provider's URL, to request to sign on
    kibanaExternalHost: 'my.public.hostname.com' # <-- public URL used by the Identity Provider to call back Kibana with the "assertion" message
    protocol: http # <-- is the Kibana server listening for "http" "https" connections? Default: http
    usernameParameter: 'nameID'
    groupsParameter: 'memberOf'
    logoutUrl: 'https://my-saml-idp/saml2/http-post/slo'

    # OPTIONAL, advanced parameters
    # decryptionCert: /etc/ror/integration/certs/pub.crt
    # cert: /etc/ror/integration/certs/dag.crt
    # decryptionPvk: /etc/ror/integration/certs/decrypt_pvk.crt
    # issuer: saml_sso_idp
```

* `issuer`: issuer string to supply to identity provider during sign on request. Defaults to 'ror'
* `disableRequestedAuthnContext`: if truthy, do not request a specific authentication context. This is known to help when authenticating against Active Directory (AD FS) servers.
* `decryptionPvk`: Service Provider Private Key. Private key that will be used to attempt to decrypt any encrypted assertions that are received.
* cert: The downloadable certificate in IDP Metadata (file, absolute path)

For advanced SAML options, see [passport-saml documentation](https://github.com/bergie/passport-saml).

### Identity provider side

1. Enter the settings of your identity provider, create a new app.
2. Configure it using the information found by connecting to `http://my.public.hostname.com/ror_kbn_sso_saml_serv1/metadata.xml`

Example response:

```markup
<?xml version="1.0"?>
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" entityID="onelogin_saml" ID="onelogin_saml">
  <SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://my.public.hostname.com/ror_kbn_sso/notifylogout"/>
    <NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
    <AssertionConsumerService index="1" isDefault="true" Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://my.public.hostname.com/ror_kbn_sso/assert"/>
  </SPSSODescriptor>
</EntityDescriptor>
```

1. Create some users and some groups in the identity provider app
2. Check the user profile parameter names that the identity provider uses during the assertion callback ( **TIP**: set kibana in debug mode so ReadonlyREST will print the user profile).
3. Match the name of the parameter used by the identity provider to carry the unique user ID (in the assertion message) to the `usernameParameter` kibana YAML setting.
4. If you want to use SAML for authorization, take care of matching also the `groupsParameter` to the parameter name found in the assertion message to the kibana YAML setting.

## OpenID Connect (OIDC)

ReadonlyREST Enterprise support OpenID Connect for authentication and authorization.

> soon we will create a specific guide only for OpenID, like the ones we have for SAML

Here is how to configure it.

### Configure `ror_kbn_auth` bridge

This part is identical as seen in SAML connectors. In order for the user identity information to flow securely from Kibana to Elasticsearch, we need to set up the two plugin with a shared secret, that is: an arbitrarily long string.

### Elasticsearch side

Edit `readonlyrest.yml`

```
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

If you have configured OIDC with the `groupsParameter` ( _See below_ ), you can also restrict ACL to specific groups:

```
readonlyrest:
    access_control_rules:

    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

    ... all usual blocks of rules...

    - name: "ReadonlyREST Enterprise instance #1 for group 1"
      ror_kbn_auth:
        name: "kbn1"
        groups: ["group1"]

    - name: "ReadonlyREST Enterprise instance #1 for group 2"
      ror_kbn_auth:
        name: "kbn1"
        groups: ["group2"]

    ror_kbn:
    - name: kbn1
      signature_key: "my_shared_secret_kibana1_(min 256 chars)" # <- use environmental variables for better security!
```

You may also use any custom claim from the OIDC `userinfo` token in ACL rules by using `{{jwt:assertion.<path_to_your_claim>}}` syntax. See the [dedicated section ](../elasticsearch.md#Dynamic%20variables%20from%20JWT%20claims) for more information. ( **TIP** : Do not forget the `assertion` prefix in front of you jsonpath. )

### Kibana side

We will assume the OpenID identity provider responds to port 8080 of localhost. In our example, we used Keycloak, an open source implementation of OpenID Connect identity provide.

Edit `kibana.yml` and append:

```yaml
readonlyrest_kbn.auth:
  signature_key: "my_shared_secret_kibana1(min 256 chars)"
  oidc_kc: 
            buttonName: "KeyCloak OpenID"
            type: "oidc"
            issuer: 'http://localhost:8080/auth/realms/ror'
            authorizationURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/auth'
            tokenURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/token'
            userInfoURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/userinfo'
            clientID: 'ror_oidc'
            clientSecret: '9f1d39c8-a211-460a-84b6-0a4a1499c455'
            scope: 'openid profile roles role_list email'
            usernameParameter: 'preferred_username'
            groupsParameter: 'groups'
            kibanaExternalHost: 'localhost:8080'
            logoutUrl: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/logout'
```

### Identity provider side

1. Enter the settings interface of your identity provider, and create a new OpenID app .
2. The redirect URL should be configured as `http://localhost:5601/*`  assuming kibana is listening on localhost and on the default port.
3. Create some users and some groups in the identity provider if not present.
4. Check the user profile parameter names that the identity provider uses during the assertion callback ( **TIP**: set readonlyrest\_kbn.logLevel: debug\` in kibana.yml, so you will see the user profile how it's received from the identity provider right in the logs).
5. Match the name of the parameter used by the identity provider to carry the unique user ID (in the assertion message) to the `usernameParameter` kibana YAML setting.
6. If you want to use OpenID for authorization, take care of matching also the `groupsParameter` to the parameter name found in the assertion message to the kibana YAML setting. ( **TIP**: the `groupsParameter`  must be present in the `userinfo` token of your OIDC provider.)
7. If kibana is accessed through a reverse proxy, kibanaExternalHost should be configured with the external hostname. if omitted, the default value is equals to `server.host:server.port` defined in kibana.yml. ( This parameter can be used also when kibana is bound to 0.0.0.0, for example, if using docker.)&#x20;

## Load balancers

### Enable health check endpoint

Normally a load balancer needs a health check URL to see if the instance is still running, you can whitelist this Kibana path so the load balancer avoids a redirection to `/login`.

Edit `kibana.yml`

```
readonlyrest_kbn.whitelistedPaths: [".*/api/status$"]
```

### Session management with multiple Kibana instances

Each Kibana node stores user sessions in-memory. This will cause problems when using multiple Kibana instances behind a load balancer (especially without sticky sessions), as there would be no synchronization between nodes' sessions cache. To avoid this, session synchronization via an Elasticsearch index should be enabled. Follow these steps:

1. Come up with a string of at least 32 characters length or more to be used as the shared cookie encryption key, called `cookiePass`.
2. Open up `conf/kibana.yml` and add:
   * `readonlyrest_kbn.cookiePass: "generatedStringIn1step"` (example: "12345678901234567890123456789012")
   * `readonlyrest_kbn.cookieName` (custom cookie name - this property is optional, if not specified default cookie name would be `rorCookie`)
   * `readonlyrest_kbn.store_sessions_in_index: true` (enable session storage in index)
   * `readonlyrest_kbn.sessions_index_name: "someCustomIndexName"` (index name - this property is optional, if not specified default index would be `.readonlyrest_kbn_sessions`)
   * `readonlyrest_kbn.sessions_refresh_after: 1000` (time in milliseconds, describes how often sessions should be fetched from ES and refreshed for each node - optional, by default 2 seconds)
   * `readonlyrest_kbn.sessions_probe_interval_seconds: 15` (default 10s) how often should the browser poll Kibana to check if their session is still valid. Raise this value if you connect to Kibana through slow networks (i.e. VPN), or have very slow loading dashboards.
3. Add the above config in all Kibana nodes behind the load balancer, and restart them.

## Login screen tweaking

It is possible to customize the look of the login screen.

### Two column layout

By default,the login form appears in a single column view. ![one column](blob:https://imgur.com/f7514ca2-7f8f-4f96-aecd-09e7ea636b62)

But once title and subtitle are configured, it will switch to two columns for making room to the new text.

```
readonlyrest_kbn.login_title: "Some Title"
readonlyrest_kbn.login_subtitle: "Longer text <b>any HTML is supported<b/> including ifrmaes"
```

![two columns](https://i.imgur.com/Sqf1GIL.png)

### Add your company logo

It's recommended to use a transparent PNG, negative logo. Ideally a white foreground, and transparent background.

Open `config/kibana.yml` and append the following:

```
readonlyrest_kbn.login_custom_logo: 'https://.../logo.png'
```

### Add custom CSS/JS

You have the opportunity to inject HTML code right before the closing head tag (`</head>`).

Open `config/kibana.yml` and append the following:

```
readonlyrest_kbn.login_html_head_inject: '<style> * { color:red; }</style>'
```

## Kibana UI tweaking

With ReadonlyREST Enterprise, it's possible to inject custom CSS and Javascript to achieve a customised user experience for your users/tenants.

### Inject custom CSS in Kibana

Open `config/kibana.yml` and append the following:

```
readonlyrest_kbn.kibana_custom_css_inject: '.global-nav, kbnGlobalNav { background-color: green }'
```

Alternatively, it's possible to load the CSS from a file in the filesystem:

```
readonlyrest_kbn.kibana_custom_css_inject_file: '/tmp/custom.css'
```

### Inject custom JS in Kibana

```
readonlyrest_kbn.kibana_custom_js_inject: '$(".global-nav__logo").hide(); alert("hello!")'
```

### Map groups to aliases

You can provide a function, mapping group names to aliases of your choosing. To do so, add the following line to `config/kibana.yml`:

```
readonlyrest_kbn.groupsMapping: '(group) => group.toLowerCase()'
```

**⚠️IMPORTANT** The mapping function has to return a string. Otherwise, an error will be printed in kibana logs and the original group name will be used as fallback. Also, if the mapping function is not specified, the original group name value will be used.

## Tenancy index templating

When a tenants logs in for the first time, ReadonlyREST Enterprise will create the ".kibana" index associated to the tenancy. For example, it will create and initialize the ".kibana\_user1" index, where "user1" will store all the visualizations, dashboards, settings and index-patterns.

The issue is that "user1"'s user experience will be really raw as they will see a completely blank Kibana tenancy. Not even a default index pattern will be present. And this is particularly challenging if the tenant is supposed to be read-only (i.e. kibana\_access: "ro") because they won't even have privileges to create their own index-pattern, let alone any dashboards.

To fix this, ReadonlyREST Enterprise offers the possibility for administrators to create a template kibana index from which all the Kibana objects will be copied over to the newly initialised tenancy.

### How to use tenancy templating

An administrator will need to create the template tenancy, populate it with the default Kibana objects (index-patterns, dashboards) and configure ReadonlyREST Enterprise to take the index template it in use. Let's see this step by step:

#### Create the template tenancy

Let's start to add to our access control list (found in $ES\_PATH\_CONF/config/readonlyrest.yml, or ReadonlyREST App in Kibana) a local user "administrator" that will belong to two tenancies: the default one (stored in .kibana index), and the template one (stored in .kibana\_template index).

```
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
 - username: administrator
   auth_key: administrator:dev
   groups: ["Admins", "Template"] # can hop between two tenancies with top-left drop-down menu
```

NB: If you know what you are doing, you can add a tenancy with kibana\_index: ".kibana\_template" adding a LDAP/SAML group to your administrative user.

### Configure the template tenancy

Now login as administrator in Kibana, hop into the "Template" tenancy, and start configuring the default UX for your future tenants. Add all the index patterns, create or import all the dashboards you want.

### Configure the template tenancy index in ReadonlyREST Enterprise

Open kibana.yml and add the following line:

```
readonlyrest_kbn.kibanaIndexTemplate: ".kibana_template"
```

Now, ReadonlyREST Enterprise will look for the ".kibana\_template" index, and try to copy over all its documents every time a new kibana index is initialised to support a new tenancy.

### Try it out

Restart Kibana with the new setting. Add a new tenancy to the ACL:

```
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
 - username: administrator
   auth_key: administrator:dev
   groups: ["Admins", "Template"] # can hop between two tenancies with top-left drop-down menu
`
```

Now try to login as user1, and ReadonlyREST Enterprise should initialise the index ".kibana\_user1" with all the index patterns and dashboards contained in the template tenancy.
