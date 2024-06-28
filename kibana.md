---
description: User manual for ReadonlyREST Enterprise/PRO/Free
---

# For Kibana

🧙 **Are you using Kibana version 7.8.x or older? Go to the** [**old platform manual page**](details/kibana-7.8.x-and-older.md)**.**

### Kibana Plugin overview

ReadonlyREST plugin for Kibana is not open source, and it's offered as part of the [ReadonlyREST PRO](https://readonlyrest.com/pro.html) and [ReadonlyREST ENTERPRISE](https://readonlyrest.com/enterprise.html), and [ReadonlyREST Free](https://readonlyrest.com/free) packages. See product descriptions and a comparison chart on the official [ReadonlyREST website](https://readonlyrest.com)

ReadonlyREST plugins for Kibana **always require** the ReadonlyREST open-source plugin to be installed in the Elasticsearch nodes your Kibana instance(s) will connect to.

Installation of ReadonlyREST is not required on all Elasticsearch nodes. It's mandatory to be installed only on the nodes where you intend to secure the HTTP interface.

### After purchasing

If you haven't installed it yet, download the latest [universal build](https://docs.readonlyrest.com/universal-builds) from our [download page](https://readonlyrest.com/download/) and install it manually. Alternatively, see below if you want to install it directly via the command line without downloading it from the browser.

Once the universal build plugin for Kibana is installed, you can activate it using an **activation key**. You can get one of these in the [ReadonlyREST customer portal](https://readonlyrest.com/customer) if you are a subscriber, otherwise, use the same portal to get a trial activation key (for PRO or Enterprise) for 30 days evaluation.

### Version strings

All our plugins include in their file name a version string. For example, the file `readonlyrest-1.46.0_es8.6.0.zip` has a version string `1.46.0_es8.6.0`.

#### Reading version strings

Given the version string `1.46.0_es8.6.0`

* ReadonlyREST plugin code version `1.46.0`
* Works only with Elasticsearch/Kibana version `8.6.0`

The "es" stands for "Elastic stack" which used to mean the family of products made by Elastic which get released at the same time under the same version number. This was chosen **before** Elastic renamed their X-Pack commercial offer to Elastic Stack.

To be clear, there is no affiliation between ReadonlyREST and Elastic, or their commercial products.

#### Universal Kibana plugin version strings

Our Kibana plugin file naming follows very similar rules:

I.e. `readonlyrest_kbn_universal-1.46.0_es8.6.0.zip`

* ReadonlyREST PRO plugin version 1.46.0
* Works only with Kibana version 8.6.0

### When an update is out

You will receive another email notification that a new deliverable is available.

If the update contains a security fix, it is very important that you take action and **update the plugin immediately**.

## Installation

You can install this as a normal Kibana plugin using the `bin/kibana-plugin` utility. Let's see the two ways to use this utility with ReadonlyREST.

{% hint style="warning" %}
**Don't forget**

After Kibana 7.9.x, it's necessary to [patch](./#patching-kibana) Kibana after you install, otherwise ReadonlyREST will NOT work.
{% endhint %}

### Installing via URL

This installation method is more practical if your Kibana server is connected to the internet.

Please note that this will always download the latest version of Kibana plugin available for the current supported Elasticsearch version.

```bash
# ReadonlyREST Universal Kibana plugin
$ bin/kibana-plugin install "https://api.beshu.tech/download/kbn?edition=kbn_universal&email=<your_email_address>"
```

If you want to download the latest version of the plugin for a specific version of Kibana, then use the query parameter `esVersion` to specify your required Kibana version.

```bash
$ bin/kibana-plugin install "https://api.beshu.tech/download/kbn?edition=kbn_universal&esVersion=7.6.1&email=<your_email_address>"

```

If you want to download an older version of the plugin for a specific version of Elasticsearch, then use the query parameter `pluginVersion` along with `esVersion`. Please note that you can only go so far back with plugin versions. [Let us know](https://readonlyrest.com/contact) if you can't download a specific one.

```bash
$ bin/kibana-plugin install "https://api.beshu.tech/download/kbn?edition=kbn_universal&esVersion=8.6.0&pluginVersion=1.46.0&email=<your_email_address>"
```

It's possible to add an extra query parameter (`checksum=true`) to any download URL to obtain a `sha1` checksum of the corresponding deliverable. For example:

```bash
$curl -vvv  "https://api.beshu.tech/download/kbn?esVersion=8.6.0&pluginVersion=1.46.0&email=your@emailaddress.com&edition=kbn_universal&checksum=true" 
[...]
$ curl -vvv  "https://api.beshu.tech/download/es?esVersion=8.6.0&pluginVersion=1.46.0&checksum=true" 
[...]
```

Now you are ready to [patch Kibana](./#patching-kibana).

### Installing from a zip file

```bash
$ bin/kibana-plugin install file:///home/user/downloads/readonlyrest_kbn-X.Y.Z_esW.Q.U.zip
```

Notice how we need to type in the format `file://` + absolute path (yes, with three slashes).

### Patching Kibana

If you are using Kibana 7.9.x or newer, you need **an extra post-installation step**. This will slightly modify some core Kibana files.

```bash
# Patch Kibana core files 
$ node/bin/node plugins/readonlyrestkbn/ror-tools.js patch
```

### Unpatching Kibana

If you are using Kibana 7.9.x or newer, you need **an extra pre-uninstallation step**. This will restore the core Kibana files to the original state.

```bash
# Unpatch Kibana core files 
$ node/bin/node plugins/readonlyrestkbn/ror-tools.js unpatch
```

### Configuring Kibana

For the activation key persistence after upgrading the license to PRO or Enterprise edition, From readonlyREST version 1.51.0 `readonlyrest_kbn.cookiePass` is a required `kibana.yml` config parameter. It needs to be configured also in case of a free license.

### Uninstalling

{% hint style="info" %}
To uninstall, you should unpatch Kibana first, then uninstall the ReadonlyREST plugin. However, **the Kibana plugin system uninstallation process is highly unreliable**.

So we highly recommend throwing away the entire Kibana directory and starting from scratch. Ideally, use ephemeral docker containers.

Need inspiration? Try the [ROR Docker demo](https://github.com/sscarduzio/ror-docker-demo)!
{% endhint %}

To bring Kibana to its pre-patching original state, it's possible to unpatch.

```bash
# Un-patch Kibana core files 
$ node/bin/node plugins/readonlyrestkbn/ror-tools.js unpatch

# Uninstall normally
$ bin/kibana-plugin remove readonlyrestkbn
```

And the classic uninstall command...

```bash
$ bin/kibana-plugin remove readonlyrest_kbn
```

### Upgrading

To upgrade to a new version of ReadonlyREST plugin for Kibana, you should:

* [Unpatch Kibana](./#unpatching-kibana)
* [Uninstall](kibana.md#uninstalling) the old plugin
* [Install](./#installation) the new one
* [Patch Kibana](./#patching-kibana)
* Restart Kibana

### Major version upgrades when using multi-tenancy

If you use multi-tenancy (Enterprise only), you will have one or more tenancy-specific Kibana indices beyond the main `.kibana` (e.g. `.kibana_tenant1`, `.kibana_tenant2`, etc.).

The first time you run Kibana after a major version upgrade (e.g. upgrading from Kibana 7.17.7 to Kibana 8.0.0), Kibana will run a [saved objects migration](https://www.elastic.co/guide/en/kibana/current/saved-object-migrations.html) on the default `.kibana` index, or whatever it finds configured as `kibana.index` in `kibana.yml`.

Now, because you may have multiple Kibana indices containing saved objects, you should apply the "saved object migration" to those indices as well.

ReadonlyREST Enterprise will automatically make sure a tenancy index is migrated to satisfy the current Kibana version **right before every time it's being used.**

For example, after a tenant logs in, before the Kibana session is started, or when a user changes tenancy with the tenancy switcher, the tenancy index gets created if absent, checked and migrated if necessary. These logs mean that migration started correctly:

```
  [savedobjects-service] Waiting until all Elasticsearch nodes are compatible with Kibana before starting saved objects migrations...
  [savedobjects-service] Starting saved objects migrations
  [savedobjects-service] [.kibana] INIT -> CREATE_NEW_TARGET. took: 27ms.
  [savedobjects-service] [.kibana_task_manager] INIT -> CREATE_NEW_TARGET. took: 29ms.
  [savedobjects-service] [.kibana_task_manager] CREATE_NEW_TARGET -> MARK_VERSION_INDEX_READY. took: 82ms.
  [savedobjects-service] [.kibana] CREATE_NEW_TARGET -> MARK_VERSION_INDEX_READY. took: 95ms.
  [savedobjects-service] [.kibana_task_manager] MARK_VERSION_INDEX_READY -> DONE. took: 23ms.
  [savedobjects-service] [.kibana_task_manager] Migration completed after 135ms
  [savedobjects-service] [.kibana] MARK_VERSION_INDEX_READY -> DONE. took: 20ms.
  [savedobjects-service] [.kibana] Migration completed after 143ms
```

Now Kibana will have migrated the tenancy index, like it did with the main `.kibana` index.

### Using RoR with a reverse proxy

RoR - just like Kibana itself - is meant to be used either with a proxy or without one.

* If you decide to set the `server.basePath` property in `kibana.yml` and set `server.rewriteBasePath` into a `true`, RoR will be accessed directly and via a reverse proxy,
* If you decide to rewrite the base path manually by your reverse proxy and set the `server.rewriteBasePath` property in `kibana.yml` into a `false`, be sure to access RoR via a proxy, as it will not work properly when accessed directly.

## Configuration

ReadonlyREST for Kibana is almost entirely remote-controlled from the Elasticsearch configuration. Login credentials, hidden Kibana apps, etc. are all going to be configured from the Elasticearch side via the usual "rules". This means the configuration will be kept all in one place and if you used ReadonlyREST before, it will be also very familiar.

### ROR Settings in kibana.yml

* `readonlyrest_kbn.logLevel: <trace|debug|info>`: for extra visibility set debug or (rarely) trace. Keep in mind `trace` could leak secrets into logs, so be careful.
* [session configuration](#session-management-with-multiple-kibana-instances)
* [UI customisation](#login-screen-tweaking)
* [custom middleware](#custom-middleware)

> In this document, every time you will encounter references to "readonlyrest.yml" or "elasticsearch.yml", we will be referring to the configuration files **in the Elasticsearch plugin** (our Kibana plugins do not need a "readonlyrest.yml").

In general, by design, we tend to concentrate all configuration within the main plugin (the Elasticsearch one) as much as possible.

### Cluster-wide Settings VS readonlyrest.yml

This feature is available in Free and PRO editions

Our Kibana plugins introduce a "ReadonlyREST" Kibana app. From here, you can edit the security settings of the whole Elasticsearch cluster, and they will take effect within 10 seconds in all Elasticsearch cluster nodes without the need to restart them.

When you change the security settings from the Kibana app, they will be saved in a special index called ".readonlyrest", so all the Elasticsearch nodes will pick them up. You can customize the name of the index by setting `readonlyrest.settings_index: .my_custom_readonlyrest` in the `elasticsearch.yml` file (remember to set the same value for all your ES nodes).

When an Elasticsearch node restarts, the order of settings evaluation is the following: 1. Attempt to find valid settings in readonlyrest.yml 2. If none is found, look inside elasticsearch.yml 3. Once successfully bootstrapped using file-based settings, attempt to read ".readonlyrest" index 4. If the index exists and contains valid settings, override file-based settings with the ones from the index. 5. Pressing "save" in the cluster-wide settings app, will **not overwrite the readonlyrest.yml** file.

Best practices:

* Build and update your production security settings from the Kibana app (will be saved in index)
* Protect the ".readonlyrest" Kibana index with an ACL rule

#### Loading settings: order of precedence

As you read, there are two possible places where the settings can be read from:

* `readonlyrest.yml` a file the user needs to create in the same directory where `elasticsearch.yml` is found.
* `.readonlyrest` index. Our Kibana plugins' GUI (PRO/Enterprise) is programmed to write this index.

When the ES plugin boots up, it follows some logic to evaluate where to read the YAML settings from. The following diagram shows how that works.

![config loading diagram](<.gitbook/assets/ror\_config\_loading\_diagram (1) (1) (1).png>)

#### Malformed in-index settings

If for some reason the in-index settings get corrupted and ROR can't parse them, then neither settings from file or in-index settings can be loaded, so ES can't start. In this case, ES would print a message like:

```
Loading ReadonlyREST settings from index failed: Settings config content is malformed. Details: while scanning a quoted scalar
 in 'reader', line 9, column 17:
          auth_key: "admin:container
                    ^
```

To recover from this state, set `readonlyrest.force_load_from_file: true` in `elasticsearch.yml` on one node `es1`.

Example recovery settings:

elasticsearch.yml

```yaml
[...]
readonlyrest:
  force_load_from_file: true
```

readonlyrest.yml

```yaml
readonlyrest:

  access_control_rules:
  - name: "::ADMIN recover::"
    auth_key: admin:dev
    indices: ["*"]
```

Then remove the in-index settings index manually.

```bash
curl -X DELETE "admin:dev@es1:9200/.readonlyrest?pretty"
```

Now you can restore your settings to `readonlyrest.yml`, remove `readonlyrest.force_load_from_file: true` `from elasticsearch.yml` and restart the node.

### Example: multiuser ELK

This configuration will work in PRO and Enterprise editions

Make sure X-Pack is uninstalled or disabled from `elasticsearch.yml` (on the Elasticsearch side) and `kibana.yml` (on the Kibana side): This is how you disable X-pack modules:

```yaml
# For X-Pack users: you may only leave monitoring on. 
# Don't add this if X-Pack is not installed at all, or Kibana won't start.
xpack.monitoring.enabled: true
xpack.security.enabled: false
xpack.watcher.enabled: false
xpack.telemetry.enabled: false
```

This is a typical example of a configuration snippet to add at the end of your `readonlyrest.yml` (the settings file of the Elasticsearch plugin), to support ReadonlyREST PRO.

```yaml
readonlyrest:

    access_control_rules:

    - name: "::LOGSTASH::"
      auth_key: logstash:logstash
      actions: ["indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
      indices: ["logstash-*"]

    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

   #  use the following block instead of the `::KIBANA-SRV::` block if you use service account tokens (see https://www.elastic.co/guide/en/elasticsearch/reference/current/service-accounts.html)
   #
   #- name: "::KIBANA-SRV-TOKEN::"  
   #  token_authentication:
   #    token: "Bearer AAEAAWVsYXN0aWMva2liYW5hL3Rva2VuXzE6MVhQUXRubWhRd3FxUmlzNmhFVVZQdw" # generated token for Kibana
   #    username: kibana

    - name: "::RO::"
      auth_key: ro:dev
      indices: ["logstash-*"]
      kibana:
        access: ro
        hide_apps: [ "Security", "Enterprise Search"]

    - name: "::RW::"
      auth_key: rw:dev
      indices: ["logstash-*"]
      kibana:
        access: rw
        hide_apps: [ "Security", "Enterprise Search"]


    - name: "::ADMIN::"
      auth_key: admin:dev
      # KIBANA ADMIN ACCESS NEEDED TO EDIT SECURITY SETTINGS IN ROR KIBANA APP!
      kibana:
        access: admin

    - name: "::WEBSITE SEARCH BOX::"
      indices: ["public"]
      actions: ["indices:data/read/*"]
```

### Very important

#### ACL blocks ordering matters

> Blocks related to the authentication of the users should be at the top of the ACL

One of the most common mistakes is forgetting that the ACL blocks are evaluated in order from the first to the last.

So, some requests with credentials can be let through from one of the first blocks and come back to Kibana with no user identity metadata associated.

Take this example of a troublesome ACL:

```yaml
    # PROBLEMATIC SETTINGS (EXAMPLE) ⚠️

    access_control_rules:

    - name: "::FIRST BLOCK::"
      hosts: ["127.0.0.1"]
      actions: [...]

    - name: "::ADMIN::"
      auth_key: admin:dev
      kibana:
        access: admin
```

The user will be able to login because the login request will be allowed by the first ACL block. But the ACL will not have resolved any metadata about the user identity (credentials checking was ignored)!

This means the response to the Kibana login request will contain no user identity metadata (username, hidden apps, etc) and ReadonlyREST for Kibana won't be able to function correctly.

The solution to this is to reorder the ACL blocks, so the ones that authenticate Kibana users are on the top.

```yaml
    # SOLUTION: KIBANA USER AUTH RELATED BLOCKS GO FIRST! ✅👍

    access_control_rules:

    - name: "::ADMIN::"
      auth_key: admin:dev
      kibana:
        access: admin

    - name: "::FIRST BLOCK::"
      hosts: ["127.0.0.1"]
      actions: [...]
```

### Multi-tenancy Kibana

ReadonlyREST Enterprise is capable of going beyond multi-user. Users or groups can be isolated into tenancies, so their dashboards and configurations won't mix. Behind each tenancy, there is a kibana index.

#### What is a kibana index?

In the vanilla Kibana, all the configuration objects are stored under an Elasticsearch index called `.kibana`, but with ReadonlyREST Enterprise installed, you can dynamically route Kibana into reading and writing to other indices entirely, for example `.kibana_tenancy1`. So when "tenancy1" is selected from the UI, Kibana hard reloads and all settings, dashboards, and visualizations are (potentially) different.

A user can be associated to multiple tenancies, and if so, will be presented with a tenancy switcher in the UI. ![image](https://github.com/beshu-tech/readonlyrest-docs/assets/1327189/b07d27d3-310c-4754-a5c5-21b0fe3f3d45)

Using this tool, they can hop between tenancies. Keep in mind that the ACL evaluation is slightly different when multi tenancy is activated: if a tenancy is selected, only blocks without `kibana.index` rule, or with the `kibana.index` [rule](https://docs.readonlyrest.com/elasticsearch#kibana) matching to the current teancy name will be evaluated.

In ReadonlyREST Enterprise, multi-tenancy is activated by default. But if you want it to behave as in PRO/Free editions, you can disable it by writing into `kibana.yml`:

```yml
readonlyrest_kbn.multiTenancyEnabled: false
```

### Configuring Multi-tenancy

You can configure an ACL in multi tenancy mode by adding a few ACL blocks containing the `kibana.index` [rule](https://docs.readonlyrest.com/elasticsearch#kibana). See examples and further explanation under our [multi-tenancy guide](examples/multitenancy\_guide.md).

#### Session cookie expiration

When a user logs in, ReadonlyREST will write an encrypted cookie in the browser. This cookie has an time to live that can be tweaked with the following configuration key in `kibana.yml`.

```yaml
readonlyrest_kbn.session_timeout_minutes: 600 # defaults to 4320 (3 days)
```

#### Automatic Session cleanup
All expired Index or In-memory sessions, determined by an `expiresAt` date that falls prior to the current time and date, will be systematically cleaned. The parameters for this automated session cleanup procedure can be adjusted within the `kibana.yml` configuration file.

```yaml
readonlyrest_kbn.sessions_cleanup_interval: '1h' # Default to 1d 
```
##### Automatic Session cleanup options

You can  defines interval as: 

| Value | Description | Example |
|-------|-------------|---------|
| s     | seconds     | "1s"    |
| m     | minutes     | "1m"    |
| h     | hours       | "1h"    |
| d     | days        | "1d"    |

#### Clearing Session History

By default, all the session data like search history, dev tool commands history, etc, will be wiped out from the browser whenever a new user is logged in, or a user changes tenancy. To override this behaviour, use this setting:

```yaml
readonlyrest_kbn.clearSessionOnEvents: ["never"]
```

Possible values: `"login", "tenancyHop", "never"`.

#### No authentication rule defined

The “problem with the configuration of authentication” error message is presented in ReadonlyREST Free/PRO/Enterprise when the login request is checked by the ACL and gets accepted by an ACL block with no authentication rule in it.

An example of this would be:

```yaml
readonlyrest:
   access_control_rules:
   - name: "LDAP Auth"
     ldap_authentication: ...
   
   - name: "Allow requests from localhost"
     hosts: ["127.0.0.1"]
```

Imagine you run Elasticsearch and Kibana on the same host:

* the Kibana user login request comes to Elasticsearch
* Credentials are wrong, and the first block does not match
* The second block is then evaluated, and the request is allowed because of its origin IP

As you can see, Elasticsearch has no user-related information (metadata) to return to Kibana, and the error “problem with the configuration of authentication ” is shown.

In general, we highly discourage implementing access control using origin IPs alone, users should set up SSL, Basic HTTP auth in their agents in any case, even on localhost. The `hosts` rule would then be an extra protection.

If this is not possible for very important reasons, then we would prevent any Kibana-originated request to match that rule by using the negated form of the [headers rule](elasticsearch.md#headers). I.e.

readonlyrest.yml

```yaml
- name: "Allow requests from localhost"
  hosts: ["127.0.0.1"]
  headers: [ "~x-from-kibana:true" ]
```

kibana.yml (append)

```yaml
elasticsearch.customHeaders:  {"x-from-kibana":"true"}
```

### Hiding Kibana Apps

This feature will work in ReadonlyREST PRO and Enterprise.

Previously we needed to keep track and document all Kibana app IDs, and you had to look them up all the time. Now we made it simpler by letting you type the apps and submenu titles exactly as you see them in the UI.

For example, this is how you hide the whole Enterprise Search submenu.

![kibana\_hide\_apps: \["Enterprise Search"\]](<.gitbook/assets/image (2).png>)

And this is how you hide only one app from the Enterprise Search menu:

![kibana\_hide\_apps: \["Enterprise Search|Workplace Search"\]](<.gitbook/assets/image (3).png>)

More generally, either of these two ways will work:

```yaml
kibana:
  hide_apps: [ "<submenu-title>" ]
```

```yaml
kibana:
  hide_apps: [ "<submenu-title|app-title>" ]
```

For example the following is a valid rule:

```yaml
kibana:
  hide_apps: [ "Security", "Management|Stack Management", "Enterprise Search" ]
```

There is also a way to use regular expression as a `kibana.hide_apps` value

for example, you can hide all submenus except for the specific app

```yaml
kibana:
  hide_apps: [ "/^Analytics\\|(?!(Maps)$).*$/"]
```

In this case, all analytics apps will be hidden except `Maps`

**⚠️IMPORTANT** Pipe operator needs to be escaped correctly when it's declared in the regular expression `\\|`. The regular expression must be declared between double quote `"/<regular-expression/"`

You can also hide all submenus except specified values

```yaml
kibana:
  hide_apps: ["/^(?!(Analytics|Management).*$).*$/"]
```

In this case everything except of `Analytics` and `Management` will submenus will be hidden

**⚠️IMPORTANT** In this case `|` is treated ad logical `or` operator, that's why it shouldn't be escaped

To check all regular expressions available options, check [regular expressions syntax cheatsheet](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular\_Expressions/Cheatsheet)

#### Hiding kibana management apps

There is an option to hide specific management apps. You can declare hide\_apps value like:

```yaml
hide_apps: [ "<submenu-title|app-title|management-submenu-title|management-app-title>" ]

```

* To hide a single management application, you can use:

```yaml
kibana_hide_apps: [ "Management|Stack Management|Kibana|Tags" ]
```

In this case, only the Stack Management Tags application will be hidden

* To hide all management Kibana section applications, you can use:

```yaml
kibana_hide_apps: [ "/^Management\\|Stack Management\\|(?!(Kibana)|$).*$/" ]:
```

In this case, all Stack Management Kibana sections will be hidden

* To hide all management Kibana section applications except selected, you can use

```yaml
kibana_hide_apps: ["/^Management\\|Stack Management\\|Kibana\\|(?!(Data Views|Tags)$).*$/"]
```

In this case, all Stack Management Kibana section apps except Data Views and Tags will be hidden

* To hide all management applications except selected, you can use

```yaml
kibana_hide_apps: ["/^Management\\|Stack Management\\|(?!(Kibana)|$).*$/", "/^Management\\|Stack Management\\|Kibana\\|(?!(Data Views|Tags)$).*$/"]
```

In this case, all Stack Management apps except Data Views and Tags will be hidden

### Hiding ReadonlyREST menu elements

This feature will work in ReadonlyREST PRO and Enterprise.

To hide the `Manage kibana` button for the specific user you need to provide `ROR Manage Kibana` value into a `kibana.hide_apps`

```yaml
kibana:
  hide_apps: [ "ROR Manage Kibana" ]
```

To hide the `Edit security settings` button for the specific user you need to provide `ROR Security Settings` or `readonlyrest_kbn` value into a `kibana.hide_apps`

```yaml
kibana:
  hide_apps: [ "ROR Security Settings" ]
```

![Hiding ReadonlyREST menu elements](.gitbook/assets/hiding\_readonlyrest\_menu\_elements.png)

### Kibana configuration

Activate authentication for the Kibana server: let the Kibana daemon connect to Elasticsearch using one of the following methods:
 * a pair of credentials defined in `readonlyrest.yml` (see above, the ::KIBANA-SRV:: block).
 * [a service account token](https://www.elastic.co/guide/en/elasticsearch/reference/current/service-accounts.html#service-accounts-tokens) generated for Kibana, defined in `readonlyrest.yml` (see above, the ::KIBANA-SRV-TOKEN:: block).
Open up `conf/kibana.yml` and add the following:

```yaml
# This is kibana.yml, but copy the exact same in elasticsearch.yml if you have to use some X-pack features.
xpack.graph.enabled: false
xpack.ml.enabled: false
xpack.monitoring.enabled: true
xpack.security.enabled: false # this is fundamental!
xpack.watcher.enabled: false

# Kibana server use the ::KIBANA-SRV:: basic auth credentials
elasticsearch.username: "kibana"
elasticsearch.password: "kibana"

# Kibana server use the ::KIBANA-SRV-TOKEN:: token value (without the bearer scheme)
# use the following setting instead of the 'elasticsearch.username' and the 'elasticsearch.password'
# elasticsearch.serviceAccountToken: AAEAAWVsYXN0aWMva2liYW5hL3Rva2VuXzE6MVhQUXRubWhRd3FxUmlzNmhFVVZQdw

# ReadonlyREST required properties
readonlyrest_kbn.cookiePass: '12312313123213123213123abcdefghijklm'
```

And of course, also make sure `elasticsearch.url` points to the designated Elasticsearch instance (check also the http or https)

### Proxy Auth

This feature will work in all ReadonlyREST editions.

ROR for Elasticsearch can delegate authentication to a reverse proxy which will enforce some kind of authentication, and pass the successfully authenticated user's name inside an `X-Forwarded-User` header.

> Today, it's possible to skip the regular ROR login form and use the "delegated authentication" technique in ROR for Kibana as well.

1. Configure ROR for ES to expect delegated authentication (see [`proxy_auth` rule](elasticsearch.md#proxy\_auth-)) in ROR for ES documentation.
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.proxy_auth_passthrough: true`

Now ROR for Kibana will **skip the login form entirely**, and will only require that all incoming requests must carry an `X-Forwarded-User` header containing the user's name. Based on this identity, ROR for Kibana will build an encrypted cookie and handle your session normally.

#### Custom Logout link

This feature will work in all ReadonlyREST editions.

Normally, when a user presses the logout button in ROR for Kibana, it deletes the encrypted cookie that represents the user's identity and the login form is shown.

However, when the authentication is delegated to a proxy, the logout button needs to become a link to some URL capable to unregister the session a user-initiated within the proxy.

For this, ROR for Kibana offers a way to customize the logout button's URL:

1. Find a link that will delete the reverse proxy's user session
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.custom_logout_link: https://..../logout`

Now users who gained a session through delegated auth can also click on the logout button in ROR for Kibana and actually exit their session.

#### Custom Login link

This feature will work in all ReadonlyREST editions.

When you delegate authentication to an external service, you can tell ReadonlyREST to skip the classic login form entirely and redirect users to your proxy or identity provider's login screen.

To enable this:

1. Find your authentication proxy or identity provider login URL for the ROR app
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.custom_login_link: "https://../login"`

The advantage of this approach is a streamlined user experience for users that login with an external IdP. The disadvantage is that you give up the possibility to login as a local user in ROR, as the login form will be always skipped.

#### Caveat

Enabling proxy auth passthrough will relax the requirement to provide a password. Therefore, don't enable this option if you don't make sure Kibana can **only be accessed through the reverse proxy\***.

### JWT Token Forwarding as URL Query Parameter

This feature will work in all ReadonlyREST editions.

As an alternative to typing in credentials in the standard login form, it is possible to create an authenticated Kibana session by passing a JWT token as a query parameter in a URL.

#### Configuration

To enable this feature in ReadonlyREST, you need to:

* Have JWT authentication configured in ReadonlyREST (modifying `readonlyrest.yml` or the cluster-wide settings UI in the Kibana plugin). [See how](elasticsearch.md#json-web-token-jwt-auth).
* Specify the query parameter name in `kibana.yml` by adding the line `readonlyrest_kbn.jwt_query_param: "jwt"` as a string, in our case "jwt".

#### In Action

Once Kibana is restarted, you will be able to navigate to a link like this:

```text
http://kibana:5601/login?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

The following will happen:

1. The Kibana plugin will forward the JWT token found in the query parameter into the `Authorization` header in a request to Elasticsearch.
2. Elasticsearch will cryptographically authenticate and resolve the user's identity from the JWT claims.
3. Kibana will write an encrypted cookie in your browser and use that from now on for the length of the authenticated session. From here onwards, the session management will be identical to the normal login form flow.
4. When the user presses logout, Kibana will delete the cookie and redirect you to the login form, or whatever link you configured as `readonlyrest_kbn.custom_logout_link`.

**Deep linking with JWT**

Because the identity is embedded in the link, and ReadonlyREST is able to authenticate the call on the fly, the JWT authentication can be used in conjunction with the `nextUrl` query parameter for sharing deep links inside Kibana apps.

**Anatomy of a JWT deep link**

```text
http://kibana:5601/login?jwt=<the-token>&nextUrl=urlEncode(<kibana-path>)
```

In JavaScript one can compose a JWT deep link as follows:

```javascript
var absoluteKibanaPath = '/app/kibana#/visualize/edit/28dcde30-2258-11e8-82a3-af58d04b3c02?_g=()';

var url = 'http://kibana:5601/login?jwt=' + 
           jwtToken + 
           '&nextUrl=' + 
           encodeURI(absoluteKibanaPath);

console.log("Final JWT deep link: " + url)
```

The result may look something like this:

```text
http://localhost:5601/login?nextUrl=%2Fapp%2Fkibana%23%2Fvisualize%2Fedit%2F28dcde30-2258-11e8-82a3-af58d04b3c02%3F_g%3D%28%29&jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

## Embedding Kibana Dashboard or Visualization with an iframe and JWT Authentication

You have the option to embed visualizations and dashboards inside iframes. For more information, refer to the [official Elastic documentation](https://www.elastic.co/guide/en/kibana/current/reporting-getting-started.html#embed-code).

To add JWT authentication, modify the iframe `src` attribute as follows:

Original iframe `src`:

```html
<iframe src="https://localhost:5601/s/default/app/dashboards#/view/722b74f0-b882-11e8-a6d9-e546fe2bba5f?embed=true&_g=()&_a=()" height="600" width="800"></iframe>
```

Modified iframe `src` with JWT:

```html
<iframe src="https://localhost:5601/s/default/app/dashboards?jwt=<the-token>#/view/722b74f0-b882-11e8-a6d9-e546fe2bba5f?embed=true&_g=()&_a=()" height="600" width="800"></iframe>
```

Replace <the-token> with your actual JWT token to enable authentication.

{% hint style="info" %}
For a cross-domain iframe, you need to set the cookie sameSite: none and secure: true. You can do this via the kibana.yml configuration file by setting `readonlyrest_kbn.cookies.secure: true` and `readonlyrest_kbn.cookies.sameSite: 'none'`.
{% endhint %}

## SSL/TLS server

You can configure Kibana with the ReadonlyREST plugin to accept SSL connection the same way you would with vanilla Kibana configuration. For example, in `kibana.yml`:

```yaml
server.ssl.enabled: true
server.ssl.keystore.path: "/usr/share/kibana/config/certificates/kibana-server.p12"
server.ssl.keystore.password: ""
server.ssl.supportedProtocols: ["TLSv1.2", "TLSv1.3"]
```

### Secure cookies

ReadonlyREST will set the "secure" flag to its Kibana session cookie ("ror-cookie") automatically when SSL is enabled in Kibana.\
\
This is because modern browsers like Chrome won't accept "secure"-flagged cookies if the website is not HTTPS.

However, a common situation is when SSL is configured in a reverse proxy (SSL termination): so the browser will interact with Kibana using HTTPS. But because ROR doesn't know it, it will still serve session cookies without the "secure" flag.\
\
In this case, you can force ReadonlyREST to create "secure"-flagged cookies by adding this line in `kibana.yml`:

```yaml
xpack.security.secureCookies: true 
```

## Audit log

This feature will work in all ReadonlyREST editions.

The audit log feature is widely described in [📖docs for the Elasticsearch plugin](elasticsearch.md#audit-logs). The Kibana plugin has a predefined dashboard representing collected audit data.

### Loading visualization

In the _Audit_ tab of the ReadonlyREST Kibana app, there is a button that automatically creates a dashboard with some audit log-specific visualizations.

![audit log tab](<.gitbook/assets/audit\_tab (1) (1).png>)

Click the _Load_ button to load the dashboard and visualizations. An _Override_ checkbox allows reloading the default dashboard and visualizations. It will override any previously loaded audit log dashboard.

![loading visualization](<.gitbook/assets/load\_audit\_dashboard (1) (1) (1) (1) (4) (6) (7) (10) (15).png>)

In detail, this feature creates three Kibana "saved objects":

* an index pattern for `readonlyrest_audit-*`
* a dashboard called `ReadonlyREST Audit Log`
* some visualizations

### Dashboard

The audit log dashboard, by default, has only a few basic visualizations. They cover security, access logs, and performance metrics.

## SAML

This feature will work in ReadonlyREST Enterprise.

ReadonlyREST Enterprise supports service provider-initiated via SAML. This connector supports both SSO (single sign-on) and SLO (single log out). Here is how to configure it.

### Configure `ror_kbn_auth` bridge

In order for the user identity information to flow securely from Kibana to Elasticsearch, we need to set up the two plugins with a shared secret, that is: an arbitrarily long string.

### Elasticsearch side

Edit `readonlyrest.yml`

```yaml
readonlyrest:
    access_control_rules:

    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

    # ... all usual blocks of rules...

    - name: "ReadonlyREST Enterprise instance #1"
      ror_kbn_auth:
        name: "kbn1"

    ror_kbn:
    - name: kbn1
      signature_key: "my_shared_secret_kibana1_(min 256 chars)" # <- use environmental variables for better security!
```

**⚠️IMPORTANT** Basic HTTP auth credentials for the Kibana server are **still needed** for now, due to how Kibana works.

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
    cert: /etc/ror/integration/certs/dag.crt # <-- It can be also provided a string value 
    
    # OPTIONAL, advanced parameters
    # decryptionCert: /etc/ror/integration/certs/pub.crt
    # decryptionPvk: /etc/ror/integration/certs/decrypt_pvk.crt
    # issuer: saml_sso_idp
```

* `issuer`: issuer string to supply to identity provider during sign-on request. Defaults to 'ror'
* `disableRequestedAuthnContext`: if truthy, do not request a specific authentication context. This is known to help when authenticating against Active Directory (AD FS) servers.
* `decryptionPvk`: Service Provider Private Key. A private key will be used to attempt to decrypt any encrypted assertions that are received.
* `cert`: The downloadable certificate in IDP Metadata (file, absolute path) or single line string value

For advanced SAML options, see [passport-saml documentation](https://github.com/bergie/passport-saml).

### Identity provider side

1. Enter the settings of your identity provider, and create a new app.
2. Configure it using the information found by connecting to `http://my.public.hostname.com/ror_kbn_saml_serv1/metadata.xml`

Example response:

```markup
<?xml version="1.0"?>
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" entityID="onelogin_saml" ID="onelogin_saml">
  <SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://my.public.hostname.com/ror_kbn/notifylogout"/>
    <NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
    <AssertionConsumerService index="1" isDefault="true" Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://my.public.hostname.com/ror_kbn/assert"/>
  </SPSSODescriptor>
</EntityDescriptor>
```

1. Create some users and some groups in the identity provider app
2. Check the user profile parameter names that the identity provider uses during the assertion callback ( **TIP**: set Kibana in debug mode so ReadonlyREST will print the user profile).
3. Match the name of the parameter used by the identity provider to carry the unique user ID (in the assertion message) to the `usernameParameter` kibana YAML setting.
4. If you want to use SAML for authorization, take care of matching also the `groupsParameter` to the parameter name found in the assertion message to the kibana YAML setting.

## OpenID Connect (OIDC)

This feature will work in ReadonlyREST Enterprise.

ReadonlyREST Enterprise supports OpenID Connect for both authentication and authorization.

Here is how to configure it.

### Configure `ror_kbn_auth` bridge

This part is identical as seen in SAML connectors. In order for the user identity information to flow securely from Kibana to Elasticsearch, we need to set up the two plugins with a shared secret, that is: an arbitrarily long string.

### Elasticsearch side

Edit `readonlyrest.yml`

```yaml
readonlyrest:
    access_control_rules:

    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

    # ... all usual blocks of rules...

    - name: "ReadonlyREST Enterprise instance #1"
      ror_kbn_auth:
        name: "kbn1"

    ror_kbn:
    - name: kbn1
      signature_key: "my_shared_secret_kibana1_(min 256 chars)" # <- use environmental variables for better security!
```

**⚠️IMPORTANT** the Basic HTTP auth credentials for the Kibana server are **still needed** for now, due to how Kibana works.

If you have configured OIDC with the `groupsParameter` ( _See below_ ), you can also restrict ACL to specific groups:

```yaml
readonlyrest:
    access_control_rules:

    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

    # ... all usual blocks of rules...

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

You may also use any custom claim from the OIDC `userinfo` token in ACL rules by using `{{jwt:assertion.<path_to_your_claim>}}` syntax. See the [dedicated section ](<elasticsearch.md#Dynamic variables from JWT claims>)for more information. ( **TIP** : Do not forget the `assertion` prefix in front of you jsonpath. )

### Kibana side

We will assume the OpenID identity provider responds to port 8080 of localhost. In our example, we used Keycloak, an open-source implementation of OpenID Connect identity provide.

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
    kibanaExternalHost: 'localhost:5601'
    logoutUrl: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/logout'
    jwksURL: 'http://localhost:8080/auth/realms/ror/protocol/openid-connect/certs'
```

### Identity provider side

1. Enter the settings interface of your identity provider, and create a new OpenID app.
2. The redirect URL should be configured as `http://localhost:5601/*` assuming Kibana is listening on localhost and on the default port.
3. Create some users and some groups in the identity provider if not present.
4. Check the user profile parameter names that the identity provider uses during the assertion callback ( **TIP**: set `readonlyrest_kbn.logLevel: debug` in kibana.yml, so you will see the user profile how it's received from the identity provider right in the logs).
5. Match the name of the parameter used by the identity provider to carry the unique user ID (in the assertion message) to the `usernameParameter` kibana YAML setting.
6. If you want to use OpenID for authorization, take care of matching also the `groupsParameter` to the parameter name found in the assertion message to the kibana YAML setting. ( **TIP**: the `groupsParameter` must be present in the `userinfo` token of your OIDC provider.)
7. If Kibana is accessed through a reverse proxy, kibanaExternalHost should be configured with the external hostname. if omitted, the default value is equal to `server.host:server.port` defined in kibana.yml. ( This parameter can be used also when Kibana is bound to 0.0.0.0, for example, if using docker.)

## Load balancers

These features will work with all ReadonlyREST Editions

### Enable health check endpoint

Normally a load balancer needs a health check URL to see if the instance is still running, you can whitelist this Kibana path so the load balancer avoids a redirection to `/login`.

Edit `kibana.yml`

```
readonlyrest_kbn.whitelistedPaths: [".*/api/status$"]
```

### Session management with multiple Kibana instances

Each Kibana node stores user sessions in memory. This will cause problems when using multiple Kibana instances behind a load balancer (especially without sticky sessions), as there would be no synchronization between nodes' session cache. To avoid this, session synchronization via an Elasticsearch index should be enabled. Follow these steps:

1. Come up with a string of at least 32 characters length or more to be used as the shared cookie encryption key, called `cookiePass`.
2. Open up `conf/kibana.yml` and add:
   * `readonlyrest_kbn.cookiePass: "generatedStringIn1step"` (example: "12345678901234567890123456789012")
   * `readonlyrest_kbn.cookieName` (custom cookie name - this property is optional; if not specified default cookie name would be `rorCookie`)
   * `readonlyrest_kbn.store_sessions_in_index: true` (enable session storage in index)
   * `readonlyrest_kbn.sessions_index_name: "someCustomIndexName"` (index name - this property is optional; if not specified default index would be `.readonlyrest_kbn_sessions`)
   * `readonlyrest_kbn.sessions_refresh_after: 5000` (time in milliseconds, describes how often sessions should be fetched from ES and refreshed for each node - optional, by default 2 seconds)
   * `readonlyrest_kbn.sessions_probe_interval_seconds: 120` (default 60s) how often should the browser poll Kibana to check if their session is still valid. Raise this value if you connect to Kibana through slow networks (i.e. VPN), or have very slow-loading dashboards.
3. Add the above config in all Kibana nodes behind the load balancer, and restart them.


{% hint style="warning" %}
From ReadonlyREST version 1.51.0 `readonlyrest_kbn.cookiePass` is a required `kibana.yml` config parameter.
{% endhint %}

## Login screen tweaking

These features will work with ReadonlyREST PRO and Enterprise.

It is possible to customize the look of the login screen.

### Two column layout

By default, the login form appears in a single-column view. ![one column](blob:https://imgur.com/f7514ca2-7f8f-4f96-aecd-09e7ea636b62)

But once the title and subtitle are configured, it will switch to two columns to make room for the new text.

```yaml
readonlyrest_kbn.login_title: "Some Title"
readonlyrest_kbn.login_subtitle: "Longer text <b>any HTML is supported<b/> including ifrmaes"
```

![two columns](https://i.imgur.com/Sqf1GIL.png)

### Add your company logo

It's recommended to use a transparent PNG, negative logo. Ideally a white foreground, and transparent background.

Open `config/kibana.yml` and append the following:

```yaml
readonlyrest_kbn.login_custom_logo: 'https://.../logo.png'
```

To incorporate your personalized logo into the login page, place your image file within the `<YOUR_ROOT_DIRECTORY>/kibana/plugins/readonlyrestkbn/public/assets directory`. Then, proceed by appending the following code snippet to `kibana.yml`:

```yaml
readonlyrest_kbn.login_custom_logo: '/pkp/legacy/web/assets/<YOUR_LOGO>'
```

Your personalized logo can be in any format [supported by web browsers](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types). The maximum file size varies depending on the browser you're using. We recommend keeping them smaller, with a maximum size of 500KB, to maintain optimal page load speed.

### Add custom CSS/JS

#### Inject via HTML code

You have the opportunity to inject HTML code right before the closing head tag (`</head>`).

Open `config/kibana.yml` and append the following:

```yaml
readonlyrest_kbn.login_html_head_inject: '<style> * { color:red; }</style>'
```

#### Inject via JS file

There is an option to inject JavaScript file before the login screen is rendered.

Open `config/kibana.yml` and append the following:

```yaml
readonlyrest_kbn.login_html_head_inject: '<ABSOLUTE_PATH_TO_CUSTOM_JS_FILE>'
```

#### Inject via CSS file

There is an option to inject CSS file before the login screen is rendered.

```yaml
readonlyrest_kbn.login_custom_css_inject_file: '<ABSOLUTE_PATH_TO_CUSTOM_CSS_FILE>'
```

## Kibana UI tweaking

This feature will work with ReadonlyREST Enterprise

It's possible to inject custom CSS and Javascript to achieve a customized user experience for your users/tenants.

### Inject custom CSS in Kibana

Open `config/kibana.yml` and append the following:

```yaml
readonlyrest_kbn.kibana_custom_css_inject: '.global-nav, kbnGlobalNav { background-color: green }'
```

Alternatively, it's possible to load the CSS from a file in the filesystem:

```yaml
readonlyrest_kbn.kibana_custom_css_inject_file: '/tmp/custom.css'
```

**⚠️IMPORTANT** If you use relative paths, you end up pointing to kibana home, i.e. `readonlyrest_kbn.kibana_custom_css_inject_file: 'config/custom.css'` will refer to `$KBN_HOME/config/custom.css` which is the same directory where `kibana.yml` can normally be found.

### Inject custom JS in Kibana

```yaml
readonlyrest_kbn.kibana_custom_js_inject: '$(".global-nav__logo").hide(); alert("hello!")'
```

Alternatively, it's possible to load the JS from a file in the filesystem:

```yaml
readonlyrest_kbn.kibana_custom_js_inject_file: '/tmp/custom.js'
```

**⚠️IMPORTANT** If you use relative paths, you end up pointing to kibana home, i.e. `readonlyrest_kbn.kibana_custom_js_inject: 'config/custom.js'` will refer to `$KBN_HOME/config/custom.js` which is the same directory where `kibana.yml` can normally be found.

### Map groups to aliases

You can provide a function, mapping group IDs to aliases of your choosing. To do so, add the following line to `config/kibana.yml`:

```yaml
readonlyrest_kbn.groupsMapping: '(group) => group.toLowerCase()'
```

**⚠️IMPORTANT** The mapping function has to return a string. Otherwise, an error will be printed in kibana logs and the original group ID will be used as fallback. Also, if the mapping function is not specified, the original group ID value will be used.

## Tenancy index templating

This feature will work only with ReadonlyREST Enterprise

When a tenant logs in for the first time, ReadonlyREST Enterprise will create the kibana index associated to the tenancy as per ACL. For example, it will create and initialize the ".kibana\_user1" index, where the tenant "user1" will store all the ["saved objects"](https://www.elastic.co/guide/en/kibana/current/managing-saved-objects.html), that is: visualizations, dashboards, spaces, settings, data views, etc.

The problem is that user1, and any other new users would login for the first time in to a completely blank Kibana. And this is particularly challenging if the tenant is supposed to be read-only (i.e. kibana.access: "ro") because they won't even have privileges to create their own index-pattern, let alone any dashboards.

To fix this, ReadonlyREST Enterprise offers the possibility for administrators to create and curate a template kibana index from which all the Kibana objects will be copied over to the newly initialised tenancy. The objects in the templating index will be copied every time the user logs in (or changes tenancy with the tenancy selector), and **if the objects were already present, they will be overwritten**.

The object overwrite is desirable because administrators would like to improve and enrich the content of the template tenancy over time, and these enhancements need to be propagated to the tenants.

If the tenants were not read-only, and created other objects of their own (e.g. another space, another dashboard), these won't be deleted.

### Reset tenancy to template

If you add `readonlyrest_kbn.resetKibanaIndexToTemplate: true` to `kibana.yml` your tenants will get their index deleted and reinitialized to the content in the kibana template index specified in `readonlyrest_kbn.kibanaIndexTemplate` every time they log in, or change tenancy using the tenancy selector.

The reset tenancy to template only works if a valid kibana index template is specified.

### How to use tenancy templating

An administrator will need to create the template tenancy, populate it with the default Kibana objects (index-patterns, dashboards) and configure ReadonlyREST Enterprise to take the index template it in use. Let's see this step by step:

#### Create the template tenancy

Let's start to add to our access control list (found in $ES\_PATH\_CONF/config/readonlyrest.yml, or ReadonlyREST App in Kibana) a local user "administrator" that will belong to two tenancies: the default one (stored in .kibana index), and the template one (stored in .kibana\_template index).

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index

  access_control_rules:

  - name: "::KIBANA-SRV::"
    auth_key: kibana:kibana
    verbosity: error

  - name: "Admin Tenancy"
    groups: ["Admins"]
    verbosity: error
    kibana:
      access: admin
      index: ".kibana"

  - name: "Template Tenancy"
    groups: ["Template"]
    verbosity: error
    kibana:
      access: admin
      index: ".kibana_template"

  users:
  - username: administrator
    auth_key: administrator:dev
    groups: ["Admins", "Template"] # can hop between two tenancies with top-left drop-down menu
```

NB: If you know what you are doing, you can add a tenancy with kibana\_index: ".kibana\_template" adding a LDAP/SAML group to your administrative user.

### Configure the template tenancy

Now login as administrator in Kibana, hop into the "Template" tenancy, and start configuring the default saved objects for your future tenants: add all the data views, create or import all the dashboards you want.

### Configure the template tenancy index in ReadonlyREST Enterprise

Open kibana.yml and add the following line:

```yaml
readonlyrest_kbn.kibanaIndexTemplate: ".kibana_template"
```

Now, ReadonlyREST Enterprise will look for the ".kibana\_template" index, and try to copy over all its documents every time a new kibana index is initialised to support a new tenancy.

### Try it out

Restart Kibana with the new setting. Add a new tenancy to the ACL:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index

  access_control_rules:

  - name: "::KIBANA-SRV::"
    auth_key: kibana:kibana
    verbosity: error

  - name: "Admin Tenancy"
    groups: ["Admins"]
    verbosity: error
    kibana:
      access: admin
      index: ".kibana"

  - name: "Template Tenancy"
    groups: ["Template"]
    verbosity: error
    kibana:
      access: admin
      index: ".kibana_template"

  # Newly added tenant!
  - name: user1
    auth_key: user1:passwd
    kibana:
      access: rw
      index: ".kibana_user1"

  users:
  - username: administrator
    auth_key: administrator:dev
    groups: ["Admins", "Template"] # can hop between two tenancies with top-left drop-down menu

```

Now try to login as user1, and ReadonlyREST Enterprise should initialize the index ".kibana\_user1" with all the index patterns and dashboards contained in the template tenancy.

## Tenant index configuration

You can configure the `number_of_shards` and `number_of_replicas` for the tenant index via the `kibana.yml` file, allowing you to override the default index settings. This can be particularly useful in a single-node environment.
```yaml
readonlyrest.tenantIndex.number_of_shards: 1
readonlyrest.tenantIndex.number_of_replicas: 0
```

{% hint style="warning" %}
These settings will overwrite the index template settings.
{% endhint %}

## Impersonation

According to [Wikipedia](https://en.wikipedia.org/wiki/Impersonator):

> An impersonator is someone who imitates or copies the behavior or actions of another.

So, an impersonation can be understood as imitating behaviors or actions. In the context of ReadonlyREST: one user could imitate an action of another user. Why would we want it? Let's suppose the first user is an admin, who has just configured access for a new user. They would like to know if the rule(s) are configured correctly. And here comes the impersonation feature. The admin can impersonate the given user in Kibana and see what the user would see if they logged in themselves.

ROR plugins support impersonation and provide UI for configuring a cluster before using it. Visit the [impersonation details page](details/impersonation.md) to know more.

## Custom middleware

Sometimes, Enterprise users might need more flexibility and customize the plugin behavior to adjust the product to the business needs. There are two options to declare the custom middleware:

* JS file: `readonlyrest_kbn.custom_middleware_inject_file: '/path/to/your/file.js'` // You can also use a relative path here. It's relative to the kibana root folder
* Inline: `readonlyrest_kbn.custom_middleware_inject: 'function test(req, res, next) {logger.debug("custom middleware called"); next()}'`

Visit the [Custom middleware](examples/custom-middleware/) to know more.
