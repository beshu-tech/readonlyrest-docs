
# Overview

ReadonlyREST is a light weight Elasticsearch plugin that adds encryption, authentication, authorization and access control capabilities to Elasticsearch embedded REST API.
The core of this plugin is an ACL engine that checks each incoming request through a sequence of **rules** a bit like a firewall.
There is a dozen rules that can be grouped in sequences of blocks and form a powerful representation of a logic chain.

**blocks of rules** and **rules** form a declarative access control list capable of powerful logic chains.

```yml
readonlyrest:
    access_control_rules:

    - name: "Block 1 - Allowing anything from localhost"
      hosts: [127.0.0.1]

    - name: "Block 2 - Other hosts can only read certain indices"
      actions: ["indices:data/read/*"]
      indices: ["logstash-*"] # aliases are taken in account!
```

*An Example of Access Control List (ACL) made of 2 blocks.*

The YAML snippet above, like all of this plugin's settings should be saved inside the `readonlyrest.yml` file. 
Create this file **on the same path where `elasticsearch.yml` is found**.

**PRO TIP**: If you are a subscriber of the [PRO](https://readonlyrest.com/pro.html) or [Enterprise](https://readonlyrest.com/pro.html) Kibana plugin, you can edit and refresh the settings through a GUI. For more on this, see the [documentation for ReadonlyREST plugin for Kibana](kibana.md).

## Installing
To install ReadonlyREST plugin for Elasticsearch:

1. **Obtain the build**: From the [official download page](https://readonlyrest.com/download.html). Select your Elasticsearch version and send yourself a link to the compatible ReadonlyREST zip file.

2. **Install the build**:
```bash
bin/elasticsearch-plugin install file://<download_path>/readonlyrest-*.zip
```

3. **Update settings**: Edit Elasticsearch configuration file and add any of the snippets you find in the documentation page
 ```bash
 vim $ES_HOME/conf/readonlyrest.yml
 
 ```
 
4. **Start Elasticsearch**: Now you should be able to see the logs and ReadonlyREST related lines.


### Upgrading

To upgrade ReadonlyREST for Elasticsearch:

1. Stop Elasticsearch.

2. Uninstall ReadonlyREST from Elasticsearch:

```
bin/elasticsearch-plugin remove readonlyrest
```


3. Install the new version of ReadonlyREST into Elasticsearch.

```
bin/elasticsearch-plugin install file://<download_dir>/readonlyrest-<ROR_VERSION>_es<ES_VERSION>.zip
```

e.g.
```
bin/elasticsearch-plugin install file:///tmp/readonlyrest-1.16.15_es6.1.1.zip
```

4. Restart Elasticsearch.


### Removing

1. Stop Elasticsearch.

2. Uninstall ReadonlyREST from Elasticsearch:

```
bin/elasticsearch-plugin remove readonlyrest
```

3. Start Elasticsearch.


## Encryption
An SSL encrypted connection is a prerequisite for secure exchange of credentials and data over the network.
ReadonlyREST can be configured to require that all REST requests come through HTTPS.

[Letsencrypt](https://letsencrypt.org/) certificates work just fine, once you [make create a JKS keystore](https://maximilian-boehm.com/hp2121/Create-a-Java-Keystore-JKS-from-Let-s-Encrypt-Certificates.htm).

**IMPORTANT:** to enable ReadonlyREST's SSL stack, open `elasticsearch.yml` and append this one line:

```yml
http.type: ssl_netty4
```

Now in `readonlyrest.yml` add the following settings:

```yml
readonlyrest:
    ssl:
      # put the keystore in the same dir with elasticsearch.yml 
      keystore_file: "keystore.jks"
      keystore_pass: readonlyrest
      key_pass: readonlyrest
```

### Restrict SSL protocols and ciphers
Optionally, it's possible to specify a list allowed SSL protocols and SSL ciphers. Connections from clients that don't support the listed protocols or ciphers will be dropped.

```yml
readonlyrest:
    ssl:
      # put the keystore in the same dir with elasticsearch.yml 
      keystore_file: "keystore.jks"
      keystore_pass: readonlyrest
      key_pass: readonlyrest
      allowed_protocols: [TLSv1.2]
      allowed_ciphers: [TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256]
```

ReadonlyREST will log a list of available ciphers and protocols supported by the current JVM at startup.

```
[2018-01-03T10:09:38,683][INFO ][t.b.r.e.SSLTransportNetty4] ROR SSL: Available ciphers: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA
[2018-01-03T10:09:38,684][INFO ][t.b.r.e.SSLTransportNetty4] ROR SSL: Restricting to ciphers: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
[2018-01-03T10:09:38,684][INFO ][t.b.r.e.SSLTransportNetty4] ROR SSL: Available SSL protocols: TLSv1,TLSv1.1,TLSv1.2
[2018-01-03T10:09:38,685][INFO ][t.b.r.e.SSLTransportNetty4] ROR SSL: Restricting to SSL protocols: TLSv1.2
[2018-0
```


## Blocks of rules
Every block **must** have at least the `name` field, and optionally a `type` field valued either "allow" or "forbid". If you omit the `type`, your block will be treated as `type: allow` by default. 

Keep in mind that ReadonlyREST ACL is a white list, so by default all request are blocked, unless you specify a block of rules that allowes all or some requests.

- `name` will appear in logs, so keep it short and distinctive.
- `type` can be either `allow` or `forbid`. Can be omitted, default is `allow`.

```yml
    - name: "Block 1 - Allowing anything from localhost"
      type: allow
      # In real life now you should increase the specificity by adding rules here (otherwise this block will allow all requests!)
```
*Example: the most simple allow block.*

> **IMPORTANT**: If no blocks are configured, ReadonlyREST rejects all requests.

 
# Rules

ReadonlyREST access control rules allow to take decisions on three levels: 
- Network level
- HTTP level
- Elasticsearch level

Please refrain from using HTTP level rules to protect certain indices or limit what people can do to an index. 
The level of control at this level is really coarse, especially because Elasticsearch REST API does not always respect RESTful principles. 
This makes of HTTP a bad abstraction level to write ACLs in Elasticsearch all together.

>The only **clean and exhaustive** way to implement access control is to reason about requests **AFTER ElasticSearch has parsed** them.
Only then, the list of affected **indices** and the **action** will be known for sure. See **Elasticsearch level** rules.

## Transport level
These are the most basic rules. It is possible to allow/forbid requests originating from a 
list of IP addresses, host names or IP networks (in slash notation).

| Rule and example argument                  | Description  |
| --------------------------------- | ---         |
| `hosts: [localhost, 10.0.0.0/24]` | a list of origin IP addresses or subnets|

## HTTP Level

| Rule and example argument                  | Description  |
| --------------------------------- | ---         |
| `accept_x-forwarded-for_header: false` | **⚠️DEPRECATED** modifier for `hosts` rule: if the origin IP won't match, check the `X-Forwarded-For` header|
| `x_forwarded_for: ["192.168.1.0/24"]` | exactly like `hosts`, but looks inside the `X-Forwarded-For` header only (useful when requests come through a load balancer like AWS ELB)|
| `methods: [GET, DELETE]` | match the HTTP method|
| `uri_re: ^/secret-index/.*` | **☠️HACKY** A regular expression to match the request URI. Hint: superseded by indices!|
| `maxBodyLength: 0` |**⚠️DEPRECATED**identify a maximum length for HTTP request body.|

The `x_forwarded_for` rule is the only HTTP level rule that is still recommendeable to use in production. This is because when Elasticsearch is behind a load balancer or reverse proxy, it gives an equivalent of the `hosts` rule (restrict access by origin IP or network).

### Load balancers
This is a nice tip if your Elasticsearch is behind a load balancer. If you want to match all the requests that come through the load balancer, use `x_forwarded_for: ["0.0.0.0/0"]`. 
This will match the requests with a valid IP address as a value of the `X-Forwarded-For` header.

## Elasticsearch level
| Rule and example argument                  | Description  |
| --------------------------------- | ---         |
| `indices: ["sales", "logstash-*"]` | Match if the request involves indices whose name is "sales", starts with "logstash-" or of both. |
| `actions: ["indices:data/read/*"]` | Match if the request action starts with "indices:data/read/" |
| `kibana_access: ro` | Enables the minimum set of actions necessary for browsers to use Kibana. See below. |
| `kibana_index: .kibana-user1` | **OPTIONAL: Defaults to `.kibana`** specify to what index we expect Kibana to attempt to read/write its settings (use this together with `kibana.index` setting in kibana.yml.)|

### Indices rule
If a **read request** asks for a some indices they have permissions for and some indices that they do NOT have permission for, the request is **rewritten** to involve only the subset of indices they have permission for.
This is behaviour is very useful in Kibana: **different** users can see the **same** dashboards with data from only their own indices.|

If a **write request** wants to write to indices they don't have permission for, the write request is rejected.  

### Action rule
Each request carries only one action. Here is a complete list of valid action strings as of Elasticsearch 5.1.2.
```
cluster:admin/ingest/pipeline/delete
cluster:admin/ingest/pipeline/get
cluster:admin/ingest/pipeline/put
cluster:admin/ingest/pipeline/simulate

cluster:admin/repository/delete
cluster:admin/repository/get
cluster:admin/repository/put
cluster:admin/repository/verify

cluster:admin/reroute

cluster:admin/script/delete
cluster:admin/script/get
cluster:admin/script/put

cluster:admin/settings/update

cluster:admin/snapshot/create
cluster:admin/snapshot/delete
cluster:admin/snapshot/get
cluster:admin/snapshot/restore
cluster:admin/snapshot/status

cluster:admin/tasks/cancel

cluster:monitor/allocation/explain
cluster:monitor/health
cluster:monitor/main
cluster:monitor/nodes/hot_threads
cluster:monitor/nodes/info
cluster:monitor/nodes/stats
cluster:monitor/state
cluster:monitor/stats
cluster:monitor/task
cluster:monitor/task/get
cluster:monitor/tasks/lists

indices:admin/aliases
indices:admin/aliases/exists
indices:admin/aliases/get

indices:admin/analyze
indices:admin/cache/clear
indices:admin/close
indices:admin/create
indices:admin/delete
indices:admin/exists
indices:admin/flush
indices:admin/forcemerge
indices:admin/get

indices:admin/mapping/put

indices:admin/mappings/fields/get
indices:admin/mappings/get

indices:admin/open
indices:admin/refresh
indices:admin/rollover
indices:admin/settings/update
indices:admin/shards/search_shards
indices:admin/shrink
indices:admin/synced_flush

indices:admin/template/delete
indices:admin/template/get
indices:admin/template/put

indices:admin/types/exists
indices:admin/upgrade
indices:admin/validate/query

indices:data/read/explain
indices:data/read/field_stats
indices:data/read/get
indices:data/read/mget
indices:data/read/msearch
indices:data/read/mtv
indices:data/read/scroll
indices:data/read/scroll/clear
indices:data/read/search
indices:data/read/tv

indices:data/write/bulk
indices:data/write/delete
indices:data/write/index
indices:data/write/update

indices:monitor/recovery
indices:monitor/segments
indices:monitor/settings/get
indices:monitor/shard_stores
indices:monitor/stats
indices:monitor/upgrade

internal:indices/admin/upgrade
```
### kibana_access rule
This rule allows the minimum set of actions necessary for a browser to use Kibana.
Possible values:

* `ro_strict`: the browser has a read-only view on Kibana dashboards and settings and all other indices. 
* `ro`: some write requests can go through to the `.kibana` index so that UI state in discover can be saved and short urls can be created. 
* `rw`: some more actions will be allowed towards the `.kibana` index only, so Kibana dashboards and settings can be modified. 
* `admin`: like above, but has additional permissions to use the ReadonlyREST PRO/Enterprise Kibana app.

This rule is often used with the `indices` rule, to limit the data a user is able to see represented on the dashboards.
## Authentication
Local ReadonlyREST users are authenticated via HTTP Basic Auth. This authentication method is secure only if SSL is enabled.
   
| Rule and example argument                  | Description  |
| --------------------------------- | ---         |
| `auth_key: sales:p455wd` | Accepts [HTTP Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication). Configure this value *in clear text*. Clients will need to provide the header e.g. ```Authorization: Basic c2FsZXM6cDQ1NXdk``` where "c2FsZXM6cDQ1NXdk" is Base64 for "sales:p455wd". |
| `auth_key_sha1: 6cef1...9145a0` |  **⚠️DEPRECATED**  Accepts [HTTP Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication). The value is a string like `username:password` *hashed in [SHA1](http://www.sha1-online.com/)* Clients will need to provide the usual Authorization header. |
| `auth_key_sha256: 280ac6f...94bf9` | Accepts [HTTP Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication). The value is a string like `username:password` *hashed in [SHA256](http://www.xorbin.com/tools/sha256-hash-calculator)*. Clients will need to provide the usual Authorization header. |
| `proxy_auth: "*"` | Delegated auth. Trust that a reverse proxy has taken care of authenticating the request and has written the resolved user name into the  `X-Forwarded-User` header. The value "*" in example, will let this rule match any value of `X-Forwarded-User` header. If you are using this technique for authentication in **Kibana**, don't forget to add this snippet to `conf/kibana.yml`: `elasticsearch.requestHeadersWhitelist: ['authorization', 'x-forwarded-user'] ` so Kibana forwards the right header to Elasticsearch. |
| `api_keys: [123456, abcdefg]` |a list of api keys expected in the header ```X-Api-Key``` |
| `groups: ["group1", "group2"]` | Limit access to members of specific user groups. See [User management](#user-management-with-groups)|
| `session_max_idle: 1h` | browser session timeout (via cookie). Example values 1w (one week), 10s (10 seconds), 7d (7 days), etc. NB: not available for Elasticsearch 2.x|
| `ldap_auth` | see below ldap section|
| `jwt_auth` | see below JSON Web Tokens section|

## Ancillary rules
| Rule and example argument                  | Description  |
| --------------------------------- | ---         |
| `verbosity: error` | Don't spam elasticsearch log file with successful requests (that match this block). Defaults to `info` |



## Audit & Troubleshooting
### Interpreting logs
ReadonlyREST prints a log line for each incoming request (this can be selectively avoided on ACL block level using the `verbosity` rule).

#### Allowed requests
This is an example of a request that matched an ACL block (allowed) and has been let through to Elasticsearch.

>ALLOWED by { name: '::PERSONAL_GRP::', policy: ALLOW} req={ ID:1667655475--1038482600#1312339, TYP:SearchRequest, CGR:N/A, USR:simone, BRS:true, ACT:indices:data/read/search, OA:127.0.0.1, IDX:, MET:GET, PTH:/_search, CNT:<N/A>, HDR:Accept,Authorization,content-length,Content-Type,Host,User-Agent,X-Forwarded-For, HIS:[::PERSONAL_GRP::->[kibana_access->true, kibana_hide_apps->true, auth_key->true, kibana_index->true]], [::Kafka::->[auth_key->false]], [::KIBANA-SRV::->[auth_key->false]], [guest lol->[auth_key->false]], [::LOGSTASH::->[auth_key->false]] }

#### Explanation
The log line immediately states that this request has been allowed by an ACL block called "::PERSONAL_GRP::".  Immediately follows a summary of the requests' anatomy.
The format is semi-structured, and it's intended for humans to read quickly, it's not JSON, or anything else.

Similar information gets logged in JSON format into Elasticsearch documents enabling the [audit logs](#audit-logs) feature described later.

Here is a glossary:

* `ID`: ReadonlyREST-level request id
* `TYP`: String, the name of the Java class that internally represent the request type (very useful for debug)
* `CGR`: String, the request carries a "current group" header (used for multi-tenancy).
* `USR`: String, the user name ReadonlyREST was able to extract from Basic Auth, JWT, LDAP, or other methods as specified in the ACL.
* `BRS`: Boolean, an heuristic attempt to tell if the request comes from a browser.
* `ACT`: String, the elasticsearch level action associated with the request. For a list of actions, see our [actions rule docs](#action-rule).
* `OA`: IP Address, originating address of the TCP connection underlying the http session.
* `IDX`: Strings array: the list of indices affected by this request.
* `MET`: String, HTTP Method
* `CNT`: String, HTTP body content. Comes as a summary of its lenght, full body of the request is available in debug mode.
* `HDR`: String array, list of HTTP headers, headers' content is available in debug mode.
* `HIS`: Chronologically ordered history of the ACL blocks and their rules being evaluated, This is super useful for knowing what ACL block/rule is forbidding/allowing this request.

In the example, the block `::PERSONAL_GRP::` is allowing the request because all the rules in this block evaluate to `true`.

#### Forbidden requests
This is an example of a request that gets forbidden by ReadonlyREST ACL.
```
FORBIDDEN by default req={ ID:747832602--1038482600#1312150, TYP:SearchRequest, CGR:N/A, USR:[no basic auth header], BRS:true, ACT:indices:data/read/search, OA:127.0.0.1, IDX:, MET:GET, PTH:/_search, CNT:<N/A>, HDR:Accept,content-length,Content-Type,Host,User-Agent,X-Forwarded-For, HIS:[::Infosec::->[groups->false]], [::KIBANA-SRV::->[auth_key->false]], [guest lol->[auth_key->false]], [::LOGSTASH::->[auth_key->false]], [::Infosec::->[groups->false]], [::ADMIN_GRP::->[groups->false]], [::Kafka::->[auth_key->false]], [::PERSONAL_GRP::->[groups->false]] }
```
The above rule gets forbidden "by default". This means that no ACL block has matched the request, so ReadonlyREST's default policy of rejection takes effect. 



### Audit logs
ReadonlyREST can write events very similarly to Logstash into to a series of indices: `readonlyrest_audit-YYYY-MM-DD`. Every event contains information about a request and how the system has handled it.
Here is an example of the data points contained in each audit event. We can leverage all this information to build interesting Kibana dashboards, or any other visualization.

```json
{
    "error_message": null,
    "headers": [
      "Accept",
      "Authorization",
      "content-length",
      "Host",
      "User-Agent"
    ],
    "acl_history": "[[::LOGSTASH::->[auth_key->false]], [::RW::->[kibana_access->true, indices->true, kibana_hide_apps->true, auth_key->true]], [kibana->[auth_key->false]], [::RO::->[auth_key->false]]]",
    "origin": "127.0.0.1",
    "final_state": "ALLOWED",
    "task_id": 1158,
    "type": "SearchRequest",
    "req_method": "GET",
    "path": "/readonlyrest_audit-2017-06-29/_search?pretty",
    "indices": [
      "readonlyrest_audit-2017-06-29"
    ],
    "@timestamp": "2017-06-30T09:41:58Z",
    "content_len_kb": 0,
    "error_type": null,
    "processingMillis": 0,
    "action": "indices:data/read/search",
    "matched_block": "::RW::",
    "id": "933409190-292622897#1158",
    "content_len": 0,
    "user": "simone"
  }
```

Here is a configuration example, you can see the `audit_collector: true` setting, which nomally defaults to false. 
Note how the successful requests matched by the first rule (Kibana) will not be written to the audit log, because the verbosity is set to error.
Audit log in facts, obey the verbosity setting the same way regular text logs do.

```yml
http.type: ssl_netty4
readonlyrest:
    
    audit_collector: true
    
    access_control_rules:

    - name: Kibana
      type: allow
      auth_key: kibana:kibana
      verbosity: error
    
    - name: "::RO::"
      auth_key: simone:ro
      kibana_access: ro
```
## Custom audit log serializer

You can write your own custom audit log serializer class, add it to the ROR plugin class path and configure it through the YAML settings.

### Implementation
1. Create a new Java project in your IDE
2. Create a class like this:
```java
import tech.beshu.ror.ResponseContext;
import tech.beshu.ror.requestcontext.AuditLogSerializer;

import java.util.HashMap;
import java.util.Map;

  public static class MySerializer implements AuditLogSerializer {

    @Override
    public Map<String, ?> createLoggableEntry(ResponseContext context) {
      Map<String, Object> theMap = new HashMap<>();
      theMap.put("indices", context.getRequestContext().getIndices());
      return theMap;
    }
  }

```
3. Satisfy the two tech.beshu.* imports above by copy-pasting the two classes from ROR core code base into your project (they have no other dependency).

4. Find the MyCustomSerializer.class somewhere in your build directory

5. ` jar cvf CUSTOMSERIALIZER.jar MyCustomSerializer.class`

6.  mv CUSTOMSERIALIZER.jar plugins/readonlyrest/

7. Your config/readonlyrest.yml should start like this
```yml
readonlyrest:
    audit_serializer: MyCustomSerializer
```

8. Start elasticsearch (with ROR installed) and grep for:
```
[2017-11-09T09:42:51,260][INFO ][t.b.r.r.SerializationTool] Using custom serializer: MyCustomSerializer
```

### Troubleshooting 
Follow these approaches until you find the solution to your problem

#### Scenario: you can't understand why your requests are being forbidden by ReadonlyREST (or viceversa)

**Step 1: see what block/rule is matching**
Take the Elasticsearch log file, and grep the logs for `ACT:`. 
This will show you the whole request context (including the action and indices fields) of the blocked requests. You can now tweak your ACL blocks to include that action.

**Step 2: enable debug logs**

Logs are good for auditing the activity on the REST API. You can configure them by editing `$ES_HOME/config/logging.yml` (Elasticsearch 2.x) or `$ES_HOME/config/l4j2.properties` [file](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html#logging) (Elasticsearch 5.x)

For example, you can **enable the debug log** globally by setting the `rootLogger`to `debug`.
This is really useful especially to debug the activity of LDAP and other external connectors.

#### Trick: log requests to different files
Here is a `l4j2.properties` snippet for ES 5.x that logs all the received requests as a new line in a separate file:

```
#Plugin readonly rest separate access logging file definition
appender.access_log_rolling.type = RollingFile
appender.access_log_rolling.name = access_log_rolling
appender.access_log_rolling.fileName = ${sys:es.logs}_access.log
appender.access_log_rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %marker%.-10000m%n
appender.access_log_rolling.layout.type = PatternLayout
appender.access_log_rolling.filePattern = ${sys:es.logs}_access-%d{yyyy-MM-dd}.log
appender.access_log_rolling.policies.type = Policies
appender.access_log_rolling.policies.time.type = TimeBasedTriggeringPolicy
appender.access_log_rolling.policies.time.interval = 1
appender.access_log_rolling.policies.time.modulate = true

logger.access_log_rolling.name = org.elasticsearch.plugin.readonlyrest.acl
logger.access_log_rolling.level = info
logger.access_log_rolling.appenderRef.access_log_rolling.ref = access_log_rolling
logger.access_log_rolling.additivity = false

# exclude kibana, beat and logstash users as they generate too much noise
logger.access_log_rolling.filter.regex.type = RegexFilter
logger.access_log_rolling.filter.regex.regex = .*USR:(kibana|beat|logstash),.*
logger.access_log_rolling.filter.regex.onMatch = DENY
logger.access_log_rolling.filter.regex.onMisMatch = ACCEPT

``` 


# Users and Groups
Sometimes we want to make allow/forbid decisions according to the username associated to a HTTP request. The extraction of the user identity (username) can be done via HTTP Basic Auth (Authorization header) or 
 delegated to a reverse proxy (see `proxy_auth` rule).
 
The validation of the said credentials can be carried on locally with hard coded credential hashes (see `auth_key_sha256` rule), 
via one or more LDAP server, or we can forward the Authorization header to an external web server and examine the HTTP status code (see `external_authentication`).

Optionally we can introduce the notion of groups (see them as bags of users). 
The aim of having groups is to write a very specific block once, and being able to allow multiple usernames that satisfy the block.

Groups can be declared and associated to users statically in the readonlyrest.yml file. 
Alternatively, groups for a given username can be retrieved from an LDAP server or from a LDAP server, or a custom JSON/XML service.
 
You can mix and match the techniques to satisfy your requirements. For example, you can configure ReadonlyREST to:
* Extract the username from X-Forwarded-User
* Resolve groups associated to said user through a JSON microservice

Another example:
* Extract the username from Authorization header (HTTP Basic Auth)
* Validate said username's password via LDAP server
* resolve groups associated to the user from groups defined in readonlyrest.yml

More examples are shown below together with a sample configuration.  

## Local users and groups
The `groups` rule accepts a list of group names. 
This rule will match if the resolved username (i.e. via `auth_key`) is associated to the given groups.
In this example, the usernames are statically associated to group names.

```yaml
 access_control_rules:

    - name: Accept requests from users in group team1 on index1
      type: allow  # Optional, defaults to "allow" will omit now on.
      groups: ["team1"]
      indices: ["index1"]

    - name: Accept requests from users in group team2 on index2
      groups: ["team2"]
      indices: ["index2"]

    - name: Accept requests from users in groups team1 or team2 on index3
      groups: ["team1", "team2"]
      indices: ["index3"]

    users:

    - username: alice
      auth_key: alice:p455phrase
      groups: ["team1"]

    - username: bob
      auth_key: bob:s3cr37
      groups: ["team2", "team4"]

    - username: claire
      auth_key_sha256: e0bba5fda92dbb0570fd2e729a3c8ed6b1d52b380581f32427a38e396ba28ec6 #claire:p455key
      groups: ["team1", "team5"]
```
*Example: rules are associated to groups (instead of users) and users-group association is declared separately later under `users:`*

## Environmental variables 
Anywhere in `readonlyrest.yml` you can use the espression `${MY_ENV_VAR}` to replace in place the environmental variables. This is very useful for injecting credentials like LDAP bind passwords, especially in Docker.

For example, here we declare an environment variable, and we write `${LDAP_PASSWORD}` in our settings:
```bash
$ export LDAP_PASSWORD=S3cr3tP4ss 
$ cat readonlyrest.yml
```

```yml
....
ldaps:
    
    - name: ldap1
      host: "ldap1.example.com"
      port: 389                                                     
      ssl_enabled: false                                            
      ssl_trust_all_certs: true                                     
      bind_dn: "cn=admin,dc=example,dc=com"                         
      bind_password: "${LDAP_PASSWORD}"                                     
      search_user_base_DN: "ou=People,dc=example,dc=com"
....
```
And ReadonlyREST ES will load "S3cr3tP4ss" as `bind_password`.

## Dynamic variables
One of the neatest feature in ReadonlyREST is that you can use dynamic variables inside most rules values.
The variables you can currently replace into rules values are these:

* `@{user}` gets replaced with the username of the successfully authenticated user
* `@{xyz}` gets replaced with any `xyz` HTTP header included in the incoming request (useful when reverse proxies handle authentication)
 
### Indices from user name
You can let users authenticate externally, i.e. via LDAP, and use their user name string inside the `indices` rule. 

```yml
readonlyrest:
    access_control_rules:

    - name: "Users can see only their logstash indices i.e. alice can see alice_logstash-20170922"
      ldap_authentication:
        name: "myLDAP" 
      indices: ["@{user}_logstash-*"]
    
    # LDAP connector settings omitted, see LDAP section below..
```

### Kibana index from headers
Imagine that we delegate authentication to a reverse proxy, so we know that only authenticated users will ever reach Elasticsearch.
We can tell the reverse proxy (i.e. Nginx) to inject a header called `x-nginx-user` containing the username. 

```yml
readonlyrest:
    access_control_rules:

    - name: "Identify a personal kibana index where each user is supposed to save their dashboards"
      kibana_access: rw
      kibana_index: ".kibana_@{x-nginx-user}" 
```

## LDAP connector

In this example, users credentials are validate via LDAP. The groups associated to each validated users are resolved using the same LDAP server.

**Simpler: authentication and authorization in one rule**
```yml
readonlyrest:
    
    access_control_rules:

    - name: Accept requests from users in group team1 on index1
      type: allow                                           # Optional, defaults to "allow", will omit from now on.
      ldap_auth:
        name: "ldap1"                                       # ldap name from below 'ldaps' section
        groups: ["g1", "g2"]                                # group within 'ou=Groups,dc=example,dc=com'
      indices: ["index1"]
      
    - name: Accept requests from users in group team2 on index2
      ldap_auth:
        name: "ldap2"
        groups: ["g3"]
        cache_ttl_in_sec: 60
      indices: ["index2"]

    ldaps:
    
    - name: ldap1
      host: "ldap1.example.com"
      port: 389                                                     # optional, default 389
      ssl_enabled: false                                            # optional, default true
      ssl_trust_all_certs: true                                     # optional, default false
      bind_dn: "cn=admin,dc=example,dc=com"                         # optional, skip for anonymous bind
      bind_password: "password"                                     # optional, skip for anonymous bind
      search_user_base_DN: "ou=People,dc=example,dc=com"
      user_id_attribute: "uid"                                      # optional, default "uid"
      search_groups_base_DN: "ou=Groups,dc=example,dc=com"
      unique_member_attribute: "uniqueMember"                       # optional, default "uniqueMember"
      connection_pool_size: 10                                      # optional, default 30
      connection_timeout_in_sec: 10                                 # optional, default 1
      request_timeout_in_sec: 10                                    # optional, default 1
      cache_ttl_in_sec: 60                                          # optional, default 0 - cache disabled
      group_search_filter: "(objectClass=group)(cn=application*)"   # optional, default (cn=*)
      group_name_attribute: "cn"                                    # optional, default "cn"
    
    - name: ldap2
      host: "ldap2.example2.com"
      port: 636
      search_user_base_DN: "ou=People,dc=example2,dc=com"
      search_groups_base_DN: "ou=Groups,dc=example2,dc=com"
```

**Advanced: authentication and authorization in separate rules**
```yml
readonlyrest:
    enable: true
    response_if_req_forbidden: Forbidden by ReadonlyREST ES plugin
    
    access_control_rules:

    - name: Accept requests to index1 from users with valid LDAP credentials, belonging to LDAP group 'team1' 
      ldap_authentication: "ldap1"  
      ldap_authorization:
        name: "ldap1"                                       # ldap name from 'ldaps' section
        groups: ["g1", "g2"]                                # group within 'ou=Groups,dc=example,dc=com'
      indices: ["index1"]
      
    - name: Accept requests to index2 from users with valid LDAP credentials, belonging to LDAP group 'team2'
      ldap_authentication:
        name: "ldap2"  
        cache_ttl_in_sec: 60
      ldap_authorization:
        name: "ldap2"
        groups: ["g3"]
        cache_ttl_in_sec: 60
      indices: ["index2"]

    ldaps:
    
    - name: ldap1
      host: "ldap1.example.com"
      port: 389                                                 # default 389
      ssl_enabled: false                                        # default true
      ssl_trust_all_certs: true                                 # default false
      bind_dn: "cn=admin,dc=example,dc=com"                     # skip for anonymous bind
      bind_password: "password"                                 # skip for anonymous bind
      search_user_base_DN: "ou=People,dc=example,dc=com"
      user_id_attribute: "uid"                                  # default "uid"
      search_groups_base_DN: "ou=Groups,dc=example,dc=com"
      unique_member_attribute: "uniqueMember"                   # default "uniqueMember"
      connection_pool_size: 10                                  # default 30
      connection_timeout_in_sec: 10                             # default 1
      request_timeout_in_sec: 10                                # default 1
      cache_ttl_in_sec: 60                                      # default 0 - cache disabled
    
    - name: ldap2
      host: "ldap2.example2.com"
      port: 636
      search_user_base_DN: "ou=People,dc=example2,dc=com"
      search_groups_base_DN: "ou=Groups,dc=example2,dc=com"
```

### LDAP configuration notes
- `search_user_base_DN` should refer to the base Distinguished Name of the users to be authenticated.
- `search_groups_base_DN` should refer to the base Distinguished Name of the groups to which these users may belong.
- By default, users in `search_user_base_DN` should contain a `uid` LDAP attribute referring to a unique ID for the user within the base DN. An alternative attribute name can be specified via the optional `user_id_attribute` configuration item.
- By default, groups in `search_groups_base_DN` should contain a `uniqueMember` LDAP attribute referring to the full DNs of the users that belong to the group. (There may be any number of occurrences of this attribute within a particular group, as any number of users may belong to the group.) An alternative attribute name can be specified via the optional `unique_member_atttribute` configuration item.
- `group_name_attribute` is the LDAP group object attribute that contains the names of the ROR groups
- `group_search_filter` is the LDAP search filter (or filters) to limit the user groups returned by LDAP. This filter will be joined (with `&`) with `unique_member_atttribute=user_dn` filter resulting in this LDAP search filter: (&YOUR_GROUP_SEARCH_FILTER(unique_member_attribute=user_dn)). Examples:
```
group_search_filter: "(objectClass=group)"
group_search_filter: "(objectClass=group)(cn=application*)"
group_search_filter: "(cn=*)" # basically no group filtering
```

(An example OpenLDAP configuration file can be found in our tests: /src/test/resources/test_example.ldif)

Caching can be configured per LDAP client (see `ldap1`) or per rule (see `Accept requests from users in group team2 on index2` rule)

## External Basic Auth
ReadonlyREST will forward the received `Authorization` header to a website of choice and evaluate the returned HTTP status code to verify the provided credentials.
This is useful if you already have a web server with all the credentials configured and the credentials are passed over the `Authorization` header.

```yml
readonlyrest:    
    access_control_rules:
    
    - name: "::Tweets::"
      methods: GET
      indices: ["twitter"]
      external_authentication: "ext1"

    - name: "::Facebook posts::"
      methods: GET
      indices: ["facebook"]
      external_authentication:
        service: "ext2"
        cache_ttl_in_sec: 60

    external_authentication_service_configs:

    - name: "ext1"
      authentication_endpoint: "http://external-website1:8080/auth1"
      success_status_code: 200
      cache_ttl_in_sec: 60
      validate: false # SSL certificate validation (default to true)

    - name: "ext2"
      authentication_endpoint: "http://external-website2:8080/auth2"
      success_status_code: 204
      cache_ttl_in_sec: 60
```

To define an external authentication service the user should specify: 
- `name` for service (then this name is used as id in `service` attribute of `external_authentication` rule)
- `authentication_endpoint` (GET request)
- `success_status_code` - authentication response success status code
- 

Cache can be defined at the service level or/and at the rule level. In the example, both are shown, but you might opt for setting up either.

## Custom groups providers
This external authorization connector makes it possible to resolve to what groups a users belong, using an external JSON or XML service.

```yml
readonlyrest:    
    access_control_rules:

    - name: "::Tweets::"
      methods: GET
      indices: ["twitter"]
      proxy_auth:
        proxy_auth_config: "proxy1"
        users: ["*"]
      groups_provider_authorization:
        user_groups_provider: "GroupsService"
        groups: ["group3"]

    - name: "::Facebook posts::"
      methods: GET
      indices: ["facebook"]
      proxy_auth:
        proxy_auth_config: "proxy1"
        users: ["*"]
      groups_provider_authorization:
        user_groups_provider: "GroupsService"
        groups: ["group1"]
        cache_ttl_in_sec: 60

    proxy_auth_configs:

    - name: "proxy1"
      user_id_header: "X-Auth-Token"                           # default X-Forwarded-User

    user_groups_providers:

    - name: GroupsService
      groups_endpoint: "http://localhost:8080/groups"
      auth_token_name: "token"
      auth_token_passed_as: QUERY_PARAM                        # HEADER OR QUERY_PARAM
      response_groups_json_path: "$..groups[?(@.name)].name"   # see: https://github.com/json-path/JsonPath
      cache_ttl_in_sec: 60
```

In example above, a user is authenticated by reverse proxy and then external service is asked for groups for that user. 
If groups returned by the service contain any group declared in `groups` list, user is authorized and rule matches. 

To define user groups provider you should specify:
- `name` for service (then this name is used as id in `user_groups_provider` attribute of `groups_provider_authorization` rule)
- `groups_endpoint` - service with groups endpoint (GET request)
- `auth_token_name` - user identifier will be passed with this name
- `auth_token_passed_as` - user identifier can be send using HEADER or QUERY_PARAM
- `response_groups_json_path` - response can be unrestricted, but you have to specify JSON Path for groups name list (see example in tests)

As usual, the cache behaviour can be defined at service level or/and at rule level.

## JSON Web Token (JWT) Auth
The information about the user name can be extracted from the "claims" inside a JSON Web Token. Here is an example.

```yml
readonlyrest:
    access_control_rules:
    - name: Valid JWT token with a viewer role
      jwt_auth:
        name: "jwt_provider_1"
        roles: ["viewer"]
    jwt: 
    - name: jwt_provider_1
      signature_algo: RSA
      signature_key: "your_signature"
      user_claim: email
      roles_claim: resource_access.client-app.roles # JSON-path style
      header_name: Authorization
```
The `user_claim` indicates which field in the JSON will be interpreted as the user name.
The `signature_key` is used shared secret between the issuer of the JWT and ReadonlyREST. It is used to verify the cryptographical "paternity" of the message.
The `header_name` is used if we expect the JWT Token in a custom header (i.e. [Google Cloud IAP signed headers](https://cloud.google.com/iap/docs/signed-headers-howto))

# GPLv3 License
ReadonlyREST Free (Elasticsearch plugin) is released under the GPLv3 license. For what this kind of software concerns, this is identical to GPLv2, that is, you can treat ReadonlyREST as you would treat Linux code.
The big difference from Linux is that here you can ask for a commercial license and stop thinking about legal implications.

Here is a practical summary of what dealing with GPLv3 means:

## You CAN
* Sell or give away an **unmodified** version of this code (along with its license and attributions).
* Use a modified version **internally to your company** without making your changes available under the GPLv3 license.
* Distribute for free or commercially a modified version (partial or total) of this software, provided that the source is contributed back as pull request to 
the [original project](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin) or publicly made available under the GPLv3 or compatible license.


## You CANNOT
* Sell or give away a modified version of the plugin (or parts of it, or any derived work) without publishing the modified source under GPLv3 compatible licenses.
* Modify the code for a paying client without immediately contributing your changes back to this project's GitHub as a pull request, or alternatively publicly release said fork under GPLv3 or compatible license.

## GPLv3 license FAQ
 **1. Q**: I sell a proprietary software solution that already includes many other OSS components (i.e. Elasticsearch). Can I bundle also ReadonlyREST into it?
> **A**: No, GPLv3 does not allow it. But hey, no problem, just go for the [Enterprise subscription](../enterprise.html).

**2. Q**: I have a SaaS and we want to use a version of ReadonlyREST for Elasticsearch (as is, or modified), do I need a commercial license?
> **A**: No, you don't. Go for it! However if you are using Kibana, consider the [Enterprise offer](../enterprise.html) which includes multi-tenancy. 

**3. Q**: I'm a consultant and I will charge my customer for modifying this software and they will not sell it as a product or part of their product.
> **A**: This is fine with GPLv3. 

# Dual-license
Please don't hesitate to [contact us](mailto:info@readonlyrest.com) for a re-licensed copy of this source. Your success is what makes this project worthwhile, don't let legal issues slow you down.

See [commercial license FAQ page](commercial.html) for more information.

# Installation
1. Download the binary release of the latest version of ReadonlyREST from the [download page](../download.html)
2. `cd` to the Elasticsearch home
3. Install the plugin 

**Elasticsearch 5.x**
```bash
 bin/elasticsearch-plugin install file:///download-folder/readonlyrest-1.13.2_es5.1.2.zip
```

**Elasticsearch 2.x**
 ```bash
 bin/plugin install file:///download-folder/readonlyrest-1.13.2_es5.1.2.zip
```

 4. Edit `config/readonlyrest.yml` and add your configuration as seen in examples.
 
## Build from Source
You need to have installed: git, maven, Java 8 JDK, zip. So use apt-get or brew to get them.

1. Clone the repo 
```bash
git clone https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin
```
2. `cd elasticsearch-readonlyrest-plugin`
3. Launch the build script `bin/build.sh`

You should find the plugin's zip files under `/target` (Elasticsearch 2.x) or `build/releases/` (Elasticsearch 5.x). 

# Examples

A small library of typical use cases.
 
## Secure Logstash

We have a Logstash agent installed somewhere and we want to ship the logs to our Elasticsearch cluster securely.

### Elasticsearch side
**Step 1**
Obtain a private key and a certificate chain and generate a java keystore in JKS format. You can either use real certs or - if we're testing locally - we can create a self signed certificate. 

For convenience, we'll do with a self-signed certificate. But if you deploy this to a server, use a real one!
```bash
keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass readonlyrest -validity 360 -keysize 2048
```
**Step 2**
Now copy the `keystore.jks` inside the plugin directory inside the Elasticsearch home. 

```bash
cp keystore.jks /elasticsearch/conifg/
```

**Step 3**
Now We need to create some credentials for logstash to login, let's say 
* user = logstash
* password = logstash


**Step 4**
Hash the credentials string `logstash:logstash` using SHA256.
The simplest way is to paste the string in an [online tool](http://www.xorbin.com/tools/sha256-hash-calculator)
You should have obtained "4338fa3ea95532196849ae27615e14dda95c77b1".


**Step 5**
Let's add some configuration to our Elasticsearch: edit `conf/readonlyrest.yml` and append the following lines:

 ```yaml
 readonlyrest:
    
     ssl:
       enable: true
       # keystore in the same dir with readonlyrest.yml
       keystore_file: "keystore.jks"
       keystore_pass: readonlyrest
       key_pass: readonlyrest
 
     response_if_req_forbidden: Forbidden by ReadonlyREST ES plugin
 
     access_control_rules:
 
     - name: "::LOGSTASH::"
       auth_key_sha256: "280ac6f756a64a80143447c980289e7e4c6918b92588c8095c7c3f049a13fbf9" #logstash:logstash
       actions: ["cluster:monitor/main","indices:admin/types/exists","indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
       indices: ["logstash-*"] 
 ```
 
 ### Logstash side
 Edit the logstash configuration file and fix the output block as follows:
 ```ruby
 output {
   elasticsearch {
     ssl => true
     ssl_certificate_verification => false
     hosts => ["YOUR_ELASTICSEARCH_HOST:9200"]
     user => logstash
     password => logstash
   }
 }
 ```
 The `ssl_certificate_verification` bit is necessary for accepting self-signed SSL certificates.

## Secure Metricbeats
Very similarly to Logstaash, here's a snippet of configuration for [Metricbeats](https://www.elastic.co/downloads/beats/metricbeat) logging agent 
configuration of metricbeat - elasticsearch section

### On the Metricbeats side

```
output.elasticsearch:
  output.elasticsearch:
  username: metricbeat
  password: hereyourpasswordformetricbeat
  protocol: https
  hosts: ["xx.xx.xx.xx:9x00"]
  worker: 1
  index: "log_metricbeat-%{+yyyy.MM}"
  template.enabled: false
  template.versions.2x.enabled: false
  ssl.enabled: true
  ssl.certificate_authorities: ["./certs/your-rootca_cert.pem"]
  ssl.certificate: "./certs/your_srv_cert.pem"
  ssl.key: "./certs/your_srv_key.pem"
```

Of course, if you do not use ssl, disable it.

### On the Elasticsearch side

```yml
 readonlyrest:
     ssl:
       enable: true
       # keystore in the same dir with elasticsearch.yml
       keystore_file: "keystore.jks"
       keystore_pass: readonlyrest
       key_pass: readonlyrest

    access_control_rules:
    - name: "metricbeat can write and create its own indices"
      auth_key_sha1: fd2e44724a234234454324253094080986e8fda
      actions: ["indices:data/read/*","indices:data/write/*","indices:admin/template/*","indices:admin/create"]
      indices: ["metricbeat-*",  "log_metricbeat*"]
```
