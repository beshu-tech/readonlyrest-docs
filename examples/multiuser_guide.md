# Multi-user Elastic Stack 
([PRO](https://readonlyrest.com/pro))

This document will guide you through setting up your Elasticsearch and Kibana stack with ReadonlyREST such that:

* 3 users will be able to login into Kibana using their own set of credentials
* All users will see the same Kibana dashboards, but may be seeing different subsets of the whole data contained in Elasticsearch. 

### Users and capabilities

For this tutorials, we want to have three users, each of them has a distinct access level to a shared Kibana tenancy \(set of dashboards and settings\).

|  | "admin" | "rw\_usr" | "ro\_usr" |
| :--- | :--- | :--- | :--- |
| Can create, edit, delete dashboards | ✅ | ✅ |  |
| Can change Kibana settings | ✅ | ✅ |  |
| Only sees logstash data from 2019 |  |  | ✅ |
| Can see "add", "delete", "edit" buttons | ✅ | ✅ |  |
| "dev-tools" Kibana App is hidden | ✅ | ✅ |  |
| "readonlyrest" Kibana App is hidden | ✅ |  |  |

NB: ReadonlyREST for Elastisearch and ReadonlyREST PRO for Kibana have an great amount of features like groups, connector for external systems like LDAP, etc. Don't forget to visit the full documentation and the forum to know more about it. NB: This guide works with ROR Enterprise as well.

## Before you start

For the scope of this guide, we will assume:

* You will have a functioning installation of Elasticsearch and Kibana
* You have [installed the ROR plugin for Elasticsearch](https://github.com/beshu-tech/readonlyrest-docs/blob/master/elasticsearch.md#installing)
* You have [installed the ROR PRO/Enterprise plugin for Kibana](https://github.com/beshu-tech/readonlyrest-docs/blob/master/kibana.md#installation)

If you don't have the ROR [PRO](https://readonlyrest.com/pro) \(or [Enterprise](https://readonlyrest.com/enterprise)\) plugin for Kibana, get yourself a two weeks free trial build

## Setup: the Elasticsearch side

On the same directory with your `elasticsearch.yml` \(default: `config/`, create a file called `readonlyrest.yml` and write the following settings into it.

```yaml
readonlyrest:

    access_control_rules:

    #########################################################
    # These credentials shall be used by the logstash daemon.
    #########################################################  
    - name: "::LOGSTASH::"
      auth_key: logstash:logstash
      actions: ["indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
      indices: ["*logstash-*"]


    #####################################################################################
    # These credentials have no limitations, and shall be used only by the Kibana deamon.
    #####################################################################################
    - name: "::KIBANA-SRV::"
      auth_key: kibana:kibana

    #######################
    # Actual human users...
    #######################
    - name: "::RO::"
      auth_key: ro_usr:dev
      indices: ["logstash-2019*"] # <--- can see only data from 2019
      kibana:
        access: ro
        hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:management"]

    - name: "::RW::"
      auth_key: rw_usr:dev
      indices: ["logstash-*"]
      kibana:
        access: rw
        hide_apps: ["readonlyrest_kbn", "timelion", "kibana:dev_tools", "kibana:management"]

    - name: "::ADMIN::"
      auth_key: admin_usr:dev
      indices: ["logstash-*"]
      kibana:
        access: admin
```

## Setup: the Kibana side

With ROR, we try as much as possible to keep all the settings withing the Elasticsearch domain. Therefore, you'll notice how few settings are needed on the Kibana side, apart from actually installing the plugin.

Open up `config/kibana.yml` and add/edit the following settings:

```yaml
# Kibana server use ::KIBANA-SRV:: credentials
elasticsearch.username: "kibana"
elasticsearch.password: "kibana"

# ReadonlyREST required properties
readonlyrest_kbn.cookiePass: '12312313123213123213123abcdefghijklm'
```

## Running

Fire up Elasticsearch

```bash
$ bin/elasticsearch
```

And then Kibana

```bash
$ bin/kibana
```

## Logging in

Now you are ready to point your browser to the Kibana server IP \(defaulting on port 5601\) and you should see a login prompt. You can login as any user i.e. "rw\_usr", or "admin" and the password is always "dev".

Just remember to login with a RW user first, so Kibana can create its own default settings.

