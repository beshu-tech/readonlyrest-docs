# Kibana Plugin overview

ReadonlyREST plugin for Kibana is not open source, and it's offered as part of the [ReadonlyREST PRO](https://readonlyrest.com/pro.html) and [ReadonlyREST ENTERPRISE](https://readonlyrest.com/enterprise.html) packages.
See product descriptions and a comparison chart in the official [ReadonlyREST website](https://readonlyrest.com)

## After purchasing

You will receive a link to the plugin zip file in an email. Download your zip. 

You will be able to download it also in the future as long as your subscription is active.

## When an update is out
You will receive another email notification that a new deliverable is available.

If the update contains a security fix, it is very important that you take action and **update the plugin immediately**.


# Installation
You can install this as a normal Kibana plugin using the `bin/kibana-plugin` utility. 

> Please note: ReadonlyREST for Kibana requires ReadonlyREST for Elasticsearch version 1.16.2 or greater.

From your Kibana installation, launch the command:

```bash
$ bin/kibana-plugin install file:///home/user/downloads/readonlyrest_kbn-*.zip
```

Kibana "optimization" process is long and buggy, please make sure that the plugin's css file file gets generated, otherwise create it empty (no css is loaded from there anyway).
To do so, run this command:

```bash
$ touch optimize/bundles/readonlyrest_kbn.style.css
```

## Uninstall 

```bash
$ bin/kibana-plugin remove readonlyrest
```


## Upgrade

Just uninstall the old version and install the new version.

```bash
$ bin/kibana-plugin remove readonlyrest
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


> Again, make sure you have an installed and running  ReadonlyREST for Elasticsearch **1.16.2 or greater**. 



## Example: multiuser ELK

Make sure X-Pack is uninstalled or disabled from `elasticsearch.yml`:

```yaml
# For xpack users: you may only leave monitoring on.
# xpack.graph.enabled: false
# xpack.ml.enabled: false
# xpack.monitoring.enabled: true
# xpack.security.enabled: false
# xpack.watcher.enabled: false
```

This is a typical example of configuration snippet to add at the end of your `readonlyrest.yml` file, to support ReadonlyREST PRO.

```yaml

readonlyrest:

     # IMPORTANT FOR LOGIN/LOGOUT TO WORK
    prompt_for_basic_auth: false

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
* ALWAYS add this line when using the Kibana plugin : `prompt_for_basic_auth: false`
* Remember to use `kibana_hide_apps: ["readonlyrest_kbn"]` to hide the ReadonlyREST icon  from who is not meant to use it (makes better UX).



## Kibana configuration

Activate authentication for the Kibana server: let the Kibana daemon connect to Elasticsearch using a pair of credentials we just defined in `readonlyrest.yml` (see above, the ::KIBANA-SRV:: block).

Open up `conf/kibana.yml` and add the following:

```yaml
# Same as ES, xpack users: only leave monitoring on.
# xpack.graph.enabled: false
# xpack.ml.enabled: false
# xpack.monitoring.enabled: true
# xpack.security.enabled: false
# xpack.watcher.enabled: false

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

### Logging out
Normally, when a user presses the logout button in ROR for Kibana, it deletes the encrytped cookie that represents the users identity and the login form is shown.

However, when the authentication is delegated to a proxy, the logout button needs to become a link to some URL capable to unregister the session a user initiated within the proxy.

For this, ROR for Kibana offers a way to customize the logout button's URL:

1. Find a link that will delete the reverse proxy's user session
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.custom_logout_link: https://..../logout`

Now users that gained a session through delegated auth, can also click on the logout button in ROR for kibana and actually exit their session.

### Caveat
Enabling proxy auth passthrough will relax the requirement to provide a password. Therefore, don't enable this option if you don't make sure Kibana can **only be accessed through the reverse proxy***.

## Load balancers
When you run multiple Kibana instances behind a load balancer, a user will have their identity cookie created and encrypted in one instance. 

A fresh cookie encryption key is generated at startup time on every Kibana node. This means that each Kibana instance behind the load balancer will have a different encryption key.

This is a problem because a cookie encrypted by one instance won't be recognised on the other instances.

> Today it's possible to share the cookie encryption key in all the Kibana instances

1. Come up with a string of 32 characters length or more
2. Open up `conf/kibana.yml` and add `readonlyrest_kbn.cookiePass: 12345678901234567890123456789012` 
3. Do the above in all Kibana nodes behind the load balancer, and restart them. 


# Login screen
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

