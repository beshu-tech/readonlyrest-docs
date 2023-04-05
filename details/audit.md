# Audit

ReadonlyREST can collect audit events containing information about a request and how the system has handled it and send them to configured outputs.
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

## Configuration

The audit collecting by default is disabled. To enable it, you need to add `audit.enabled: true` and optionally configure the `audit.outputs`.
In the `outputs` array, you can define i.a. where the audit events should be sent.
The currently supported output types are:
* `index` - similarly to Logstash it writes audit events in the documents stored in the ReadonlyREST audit index
* `log` - it allows you to collect audit events using the Elasticsearch logs and format them with the help of features that `log4j2` enables.
  You can configure multiple outputs for audit events. When the audit is enabled, at least one output has to to be defined.
  If you omit `outputs` definition, the default `index` output will be used.


Here is an example of how to enable audit events collecting with all defaults:
```yaml
readonlyrest:

  audit:
    enabled: true

  access_control_rules:

   - name: Kibana
     type: allow
     auth_key: kibana:kibana
     verbosity: error

   - name: "::RO::"
     auth_key: simone:ro
     kibaba:
       access: ro
```

You can also use multiple audit outputs, e.g.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs: [ index, log ]

    ...
```

When you want to have more control over the audit outputs, the extended `outputs` format is for you.
For example, you can disable given output by adding `enabled: false` to the output config:
```yaml
readonlyrest:
  audit:
    enabled: true
    outputs: 
    - type: index
    - type: log
      enabled: false # by default is true
    ...
```

The other settings, specific to the type of audit outputs, are mentioned in the next sections.

### The 'index' output specific configurations

#### Custom audit indices name and time granularity

By default ReadonlyREST audit index name template is `readonlyrest_audit-YYYY-MM-DD`.
You can customize the name template using the `index_template` settings.

Example: tell ROR to write on the monthly index.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs: 
    - type: index
      index_template: "'custom-prefix'-yyyy-MM"  # <--monthly pattern
  ...
```
**⚠️IMPORTANT**: notice the single quotes inside the double quoted expression. This is the same syntax used for [Java's SimpleDateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html).

#### Custom audit cluster

It's possible to set a custom audit cluster responsible for audit events storage. When a custom cluster is specified, items will be sent to defined cluster nodes instead of the local one.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs: 
    - type: index
      cluster: ["https://user1:password@auditNode1:9200", "https://user2:password@auditNode2:9200"]
  ...
```

Setting `audit.cluster` is optional, it accepts non empty list of audit cluster nodes URIs.

### The 'log' output specific configurations

The `log` output uses a dedicated logger to write the audit events to the Elasticsearch log at INFO level.

To make ReadonlyREST start adding the audit events to the Elasticsearch log, all you have to do is add "log" as one of the outputs, e.g:
```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:      # you can use also 'outputs: [ log ]'
    - type: log  
  ...
```

#### Custom logging settings
If you want to control the logging process of audit events, you can do it via the `$ES_PATH_CONF/config/log4j2.properties`.
Here is an example config with the default logger name, with a separate log file, and configured rolling:
```text
appender.readonlyrest_audit_rolling.type = RollingFile
appender.readonlyrest_audit_rolling.name = readonlyrest_audit_rolling
appender.readonlyrest_audit_rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}readonlyrest_audit.log
appender.readonlyrest_audit_rolling.layout.type = PatternLayout
appender.readonlyrest_audit_rolling.layout.pattern = [%d{ISO8601}] %m%n
appender.readonlyrest_audit_rolling.filePattern = readonlyrest_audit-%i.log.gz
appender.readonlyrest_audit_rolling.policies.type = Policies
appender.readonlyrest_audit_rolling.policies.size.type = SizeBasedTriggeringPolicy
appender.readonlyrest_audit_rolling.policies.size.size = 1GB
appender.readonlyrest_audit_rolling.strategy.type = DefaultRolloverStrategy
appender.readonlyrest_audit_rolling.strategy.max = 4


logger.readonlyrest_audit.name = readonlyrest_audit   # required logger name, must be the same as the one defined in `readonlyrest.yml` 
logger.readonlyrest_audit.appenderRef.ror_audit.ref = readonlyrest_audit_rolling    
logger.readonlyrest_audit.additivity = false          # set to false to use only desired appenders
```
All settings are up to you. The only required entry is the logger name `logger.{your-logger-name}.name = {your-logger-name}`.
The default logger name is the `readonlyrest_audit`.

If you want to set a custom logger name for the `log` output, add the `logger_name` setting for the given output:
```yaml
readonlyrest:
  audit:
    enabled: true
    outputs: 
    - type: log
      logger_name: custom-logger-name
  ...
```

## Extending audit events

The audit events are JSON documents describing incoming requests and how the system has handled them. To create such events, we use a  `serializer`, which is responsible for the event's serialization and filtering.
The example [event](#audit) is in default format and was produced by the default serializer (`tech.beshu.ror.audit.instances.DefaultAuditLogSerializer`).
You can use any of the predefined serializers or use a custom one.

For example, if you want to add the request content to the audit event then an additional serializer is provided.
This will add the entire user request within the content field of the audit event.
To enable, configure the `serializer` parameter as below.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index
      serializer: tech.beshu.ror.requestcontext.QueryAuditLogSerializer
      # in case when the `log` type is used
    - type: log
      serializer: tech.beshu.ror.requestcontext.QueryAuditLogSerializer
  ...
```

### Custom audit event serializer

You can write your own custom audit events serializer class, add it to the ROR plugin class path and configure it through the YAML settings.

We provided 2 project examples with custom serializers \(in Scala and Java\). You can use them as an example to write yours in one of those languages.

#### Create custom audit event serializer in Scala

1. Checkout [https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin)

   `git clone git@github.com:sscarduzio/elasticsearch-readonlyrest-plugin.git`

2. Install SBT

   `https://www.scala-sbt.org/download.html`

3. Find and go to: `elasticsearch-readonlyrest-plugin/custom-audit-examples/ror-custom-scala-serializer/`
4. Create own serializer:
    * from scratch \(example can be found in class `ScalaCustomAuditLogSerializer`\)
    * extending default one \(example can be found in class `ScalaCustomAuditLogSerializer`\)
5. Build serializer JAR:

   `sbt assembly`

6. Jar can be find in:

   `elasticsearch-readonlyrest-plugin/custom-audit-examples/ror-custom-scala-serializer/target/scala-2.13/ror-custom-scala-serializer-1.0.0.jar`

#### Create custom audit event serializer in Java

1. Checkout [https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin)

   `git clone git@github.com:sscarduzio/elasticsearch-readonlyrest-plugin.git`

2. Install Maven

   `https://maven.apache.org/install.html`

3. Find and go to: `elasticsearch-readonlyrest-plugin/custom-audit-examples/ror-custom-java-serializer/`
4. Create own serializer:
    * from scratch \(example can be found in class `JavaCustomAuditLogSerializer`\)
    * extending default one \(example can be found in class `JavaCustomAuditLogSerializer`\)
5. Build serializer JAR:

   `mvn package`

6. Jar can be find in:

   `elasticsearch-readonlyrest-plugin/custom-audit-examples/ror-custom-java-serializer/target/ror-custom-java-serializer-1.0.0.jar`

#### Configuration

1. mv ror-custom-java-serializer-1.0.0.jar plugins/readonlyrest/
2. Your config/readonlyrest.yml should start like this

   ```yaml
    readonlyrest:
        audit:
          enabled: true
          outputs:
          - type: index
            serializer: "JavaCustomAuditLogSerializer" # when your serializer class is not in default package, you should use full class name here (eg. "tech.beshu.ror.audit.instances.QueryAuditLogSerializer")
   ```

3. Start elasticsearch \(with ROR installed\) and grep for:

   ```text
    [2023-03-26T16:28:40,471][INFO ][t.b.r.a.f.d.AuditingSettingsDecoder$] Using custom serializer: JavaCustomAuditLogSerializer
   ```