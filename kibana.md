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
