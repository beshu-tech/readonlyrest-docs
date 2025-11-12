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
    "acl_history": "[[::LOGSTASH::->[auth_key->false]], [kibana->[auth_key->false]], [::RO::->[auth_key->false]], [::RW::->[kibana->true, indices->true, auth_key->true]]]",
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
    "logged_user": "simone",
    "presented_identity": "simone"
  }
```

## Configuration

The audit collecting by default is disabled. To enable it, you need to add `audit.enabled: true` and optionally configure the `audit.outputs`.
In the `outputs` array, you can define i.a. where the audit events should be sent.
The currently supported output types are:
* `index` - similarly to Logstash it writes audit events in the documents stored in the ReadonlyREST audit index
* `data_stream` - similar to index type, but the audit events are stored in the ES data stream
* `log` - it allows you to collect audit events using the Elasticsearch logs and format them with the help of features that `log4j2` enables.

You can configure multiple outputs for audit events. When the audit is enabled, at least one output has to be defined.
If you omit `outputs` definition, the default `index` output will be used.

Audit can also be controlled at the block level:
- if audit is globally enabled, it is applied to all `access_control_rules` blocks by default — audit events will be generated for events pertaining to all blocks
- the audit can be optionally disabled for individual blocks, as shown in the example below for the `Kibana` block
- if audit is globally disabled, then it is disabled for all blocks, regardless of individual block settings

**⚠️IMPORTANT**: When audit is disabled for a specific block, then there will be no audit events when that block is matched.

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
     audit: # the `audit` section is optional, by default audit is enabled for each block
       enabled: false  

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
    outputs: [ index, data_stream, log ]

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

By default, the ReadonlyREST audit index name template is `readonlyrest_audit-YYYY-MM-DD`.
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
**⚠️IMPORTANT**: Notice the single quotes inside the double-quoted expression. This is the same syntax used for [Java's SimpleDateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html).

#### Custom audit cluster

It's possible to set up a custom audit cluster responsible for storing audit events. When a custom cluster is specified, items will be sent to defined cluster nodes instead of the local one.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs: 
    - type: index
      cluster: ["https://user1:password@auditNode1:9200", "https://user2:password@auditNode2:9200"]
  ...
```

Setting `audit.cluster` is optional, it accepts a non-empty list of audit cluster nodes URIs.

### The 'data_stream' output specific configurations
#### Custom audit data stream name
To change the default data stream name `readonlyrest_audit`, add the following configuration to your `readonlyrest.yml` config:

```yaml

readonlyrest:
  audit:
    enabled: true
    outputs:
      - type: data_stream
        data_stream: "custom_audit_data_stream"
```

Here, `custom_audit_data_stream` is the Elasticsearch data stream where audit events will be stored.

If the specified data stream does not exist, it will be automatically created by the ReadonlyREST plugin.
This creation process includes setting up the following components, each dedicated specifically to the configured data stream:

* A dedicated Index Lifecycle Policy `({{data-stream-name}}-lifecycle-policy)`.

* Necessary index settings and mappings (component templates: `{{data-stream-name}}-mappings` and `{{data-stream-name}}-settings`).

* A customized Index Template (`{{data-stream-name}}-template`).

#### Custom audit cluster

It's possible to set a custom audit cluster responsible for audit events storage. When a custom cluster is specified, items will be sent to defined cluster nodes instead of the local one.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs: 
    - type: data_stream
      cluster: ["https://user1:password@auditNode1:9200", "https://user2:password@auditNode2:9200"]
  ...
```

Setting `audit.cluster` is optional, it accepts a non-empty list of audit cluster nodes URIs.

#### Data stream settings

Here are the default settings set for the audit data stream created by the ReadonlyREST plugin:

![Audit data stream](../.gitbook/assets/ror_audit_datastream.png)![Audit data stream template](../.gitbook/assets/ror_audit_datastream_template.png)
![Index lifecycle policy defaults](../.gitbook/assets/ror_audit_datastream_lifecycle_policy.png)

Managing Elasticsearch data streams, such as the ReadonlyREST audit data stream, should be customized based on your specific use case. Aspects like:

* data retention policies (how long to keep and when to delete data),

* migrating old indices into the new data stream,

* handling transitions between different index lifecycle phases (e.g., hot, warm, cold, delete),

depend on your business requirements, data volume and characteristics, and how the data is analyzed and used.

Therefore, we encourage you to configure these settings yourself to best fit your needs. Elasticsearch provides flexible tools, like Index Lifecycle Management (ILM), that allow automating data management based on user-defined rules.
Customizing your configuration helps optimize storage costs and search performance.

You can manage and update settings related to your audit data stream directly from Kibana's **Index Management** UI.

##### Steps to Change Data Stream Settings using Kibana

1. **Open Kibana and Navigate to Index Management**

   * In Kibana, go to **Management** > **Stack Management** > **Index Management**.
   * Select the **Data Streams** tab to see the list of available data streams.

2. **Select Your Audit Data Stream**

   * Find your audit data stream (e.g., `custom_audit_data_stream`) in the list.
   * Click on it to view details such as indices backing the data stream, mappings, and lifecycle policies.

3. **Edit Index Lifecycle Policy (ILM)**

   * If you want to update rollover criteria, retention period, or other lifecycle actions:
     * Navigate to **Index Lifecycle Policies** under **Stack Management**.
     * Select the ILM policy associated with your audit data stream.
     * Modify phases such as `hot`, `warm`, or `delete` to adjust settings like maximum size, max age, or deletion timing.
     * Save your changes — they will be applied automatically to the indices backing the data stream.

4. **Update Index Template**

   * To change index settings or mappings for new backing indices:
     * Go to **Index Templates** in Stack Management.
     * Locate the template associated with your audit data stream (usually matching the data stream name or pattern).
     * Edit the template’s settings or mappings as needed.
     * Save the updated template; new indices created for the data stream will use these settings.

5. **Verify Changes**

   * After updating policies or templates, monitor your data stream to ensure rollover and retention behave as expected.
   * You can also query audit events via Kibana’s Discover tab or using the Elasticsearch API.

###### Important Notes

* Changes to lifecycle policies and index templates affect **new indices** created after the update; existing indices are not modified retroactively.
* To apply mapping changes to existing indices, you may need to reindex data.
* Ensure you carefully test ILM and template changes in a staging environment before applying to production audit streams.

#### Rolling Migration from `index` to `data_stream`

To migrate ReadonlyREST audit logging from the `index` output type to `data_stream` in a **rolling update**, follow this safe, zero-downtime approach:

1. **Add `data_stream` as an Additional Output**

Temporarily configure both `index` and `data_stream` outputs so that audit events are sent to both destinations:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
      - type: index
      - type: data_stream # add data stream output type to your config
        data_stream: "custom_audit_data_stream" 
```

> ✅ This ensures no audit logs are lost during the transition.

2. **Verify Data Stream Creation**

```
GET _data_stream/custom_audit_data_stream
```

Ensure the data stream is being created and audit events are flowing in.

3. **Monitor for Consistency**

Use Kibana or the `_search` API to confirm that events are present in both audit indices and `custom_audit_data_stream`.

4. **(Optional) Backfill Historical Data**

If you wish to migrate historical audit data from the old audit index, you can reindex it manually:

```json
POST _reindex
{
  "conflicts": "proceed",
  "source": {
    "index": "readonlyrest_audit-2025-06-07"
  },
  "dest": {
    "index": "custom_audit_data_stream",
    "op_type": "create"
  }
}
```

> ⚠️ Ensure both audit outputs have the same serializer for data consistency.

> ⚠️ Data streams are append-only — use `"op_type": "create"` to avoid overwrites.

> ⚠️ If the source index contains documents already present in the destination data stream, `"conflicts": "proceed"` will skip duplicates.

5. **Remove the `index` Output**

After confirming successful logging to the data stream from all nodes, update your config to remove the `index` output:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
      - type: data_stream
        data_stream: "custom_audit_data_stream"
```

6. **Final Verification**

Use Kibana dashboards, metrics, or direct queries to confirm that new audit events are flowing into the configured data stream.

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

The audit events are JSON documents describing incoming requests and how the system has handled them. To create such events, we use a `serializer`, which is responsible for the event's serialization and filtering.
The example [event](#audit) is in default format and was produced by the default serializer (`tech.beshu.ror.audit.instances.BlockVerbosityAwareAuditLogSerializer`).

You can:
* skip serializer configuration - in that case the default is `tech.beshu.ror.audit.instances.BlockVerbosityAwareAuditLogSerializer`
    ```yaml
    readonlyrest:
      audit:
        enabled: true
        outputs:
        - type: index
    ```
* use any of the predefined serializers ([see the list of predefined serializers](#predefined-serializers))
    ```yaml
    readonlyrest:
      audit:
        enabled: true
        outputs:
        - type: index
          serializer:
            type: "static"
            class_name: "tech.beshu.ror.audit.instances.QueryAuditLogSerializer" # or any other serializer class
    ```
* use dynamic, configurable serializer - define JSON fields in ReadonlyREST settings (no implementation required, [see how to do it](#using-configurable-serializer))
* implement and use your own serializer ([see how to implement a custom serializer](#custom-audit-event-serializer))


### Predefined serializers:
* `tech.beshu.ror.audit.instances.BlockVerbosityAwareAuditLogSerializer`
  * Serializes all non-`Allowed` events.
  * Serializes `Allowed` events only when the corresponding rule specifies, that they should be logged at `INFO` verbosity level.
  * Recommended for standard audit logging, where full request body capture is not required.
  * Fields included:
    ```
     match — whether the request matched a rule (boolean)  
     block — reason for blocking, if blocked (string)  
     id — audit event identifier (string)  
     final_state — final processing state (string)  
     @timestamp — event timestamp (ISO-8601 string)  
     correlation_id — correlation identifier for tracing (string)  
     processingMillis — request processing duration in milliseconds (number)  
     error_type — type of error, if any (string)  
     error_message — error message, if any (string)  
     content_len — request body size in bytes (number)  
     content_len_kb — request body size in kilobytes (number)  
     type — request type (string)  
     origin — client (remote) address (string)  
     destination — server (local) address (string)  
     xff — X-Forwarded-For HTTP header value (string)  
     task_id — Elasticsearch task ID (number)  
     req_method — HTTP request method (string)  
     headers — HTTP header names (array of strings)  
     path — HTTP request path (string)  
     user — authenticated user (string) - deprecated field, please use `logged_user` or `presented_identity`
     logged_user — human-readable username (string)
     presented_identity — user identity that was presented with the request, e.g. basic auth username (string)
     impersonated_by — impersonating user, if applicable (string)  
     action — Elasticsearch action name (string)  
     indices — indices involved in the request (array of strings)  
     acl_history — access control evaluation history (string)  
     es_node_name — Elasticsearch node name (string)  
     es_cluster_name — Elasticsearch cluster name (string)  
     ```
* `tech.beshu.ror.audit.instances.QueryAuditLogSerializer`
  * Similar to the `BlockVerbosityAwareAuditLogSerializer` regarding `Allowed` event handling and included JSON fields.
  * Additionally, captures the full request body (`content` field)
  * Recommended for standard audit logging, where full request body capture is required.
* `tech.beshu.ror.audit.instances.FullAuditLogSerializer`
  * Serializes all events of all types, including all `Allowed` events, regardless of the rule verbosity.
  * Included fields are the same as for `BlockVerbosityAwareAuditLogSerializer`
  * Use this serializer, when you need complete coverage of all events.
* `tech.beshu.ror.audit.instances.FullAuditLogWithQuerySerializer`
  * Serializes all events of all types, including all `Allowed` events, regardless of the rule verbosity.
  * Included fields are the same as for `QueryAuditLogSerializer` (includes `content` field - full request body)
  * Use this serializer, when you need complete coverage of all events with full request body.

### Using configurable serializer:

Configuration should look like that:
```yaml
    readonlyrest:
      audit:
        enabled: true
        outputs:
        - type: index
          serializer:
            type: "configurable"
            verbosity_level_serialization_mode: [INFO, ERROR] # define which Allowed events will be serialized based on the rule verbosity level
            fields: # list of fields in the resulting JSON; placeholders (like {ES_NODE_NAME}) will be replaced with their corresponding values
              node_details: "{ES_CLUSTER_NAME}/{ES_NODE_NAME}"
              http_request: "{HTTP_METHOD} {HTTP_PATH}"
              tid: "{TASK_ID}"
              bytes: "{CONTENT_LENGTH_IN_BYTES}"
```

The configuration above corresponds to serialized event looking like that:
```json
  {
    "node_details": "mainEsCluster/esNode01",
    "http_request": "GET /_cat",
    "tid": 0,
    "bytes": 123
  }
```

Available placeholders:
```
  {IS_MATCHED} — whether the request matched a rule (boolean)
  {REASON} — reason for blocking, if blocked (string)
  {ID} — audit event identifier (string)
  {FINAL_STATE} — final processing state (string)
  {TIMESTAMP} — event timestamp (ISO-8601 string)
  {CORRELATION_ID} — correlation identifier for tracing (string)
  {PROCESSING_DURATION_MILLIS} — request processing duration in milliseconds (number)
  {ERROR_TYPE} — type of error, if any (string)
  {ERROR_MESSAGE} — error message, if any (string)
  {CONTENT_LENGTH_IN_BYTES} — request body size in bytes (number)
  {CONTENT_LENGTH_IN_KB} — request body size in kilobytes (number)
  {TYPE} — request type (string)
  {REMOTE_ADDRESS} — client (remote) address (string)
  {LOCAL_ADDRESS} — server (local) address (string)
  {X_FORWARDED_FOR_HTTP_HEADER} — `X-Forwarded-For` HTTP header value (string)
  {TASK_ID} — Elasticsearch task ID (number)
  {HTTP_METHOD} — HTTP request method (string)
  {HTTP_HEADER_NAMES} — HTTP header names (array of strings)
  {HTTP_PATH} — HTTP request path (string)
  {LOGGED_USER} — human-readable username (string)
  {PRESENTED_IDENTITY} — user identity that was presented with the request, e.g. basic auth username (string)
  {IMPERSONATED_BY_USER} — impersonating user, if applicable (string)
  {ACTION} — Elasticsearch action name (string)
  {INVOLVED_INDICES} — indices involved in the request (array of strings)
  {ACL_HISTORY} — access control evaluation history (string)
  {CONTENT} — request body content (string or object)
  {ES_NODE_NAME} — Elasticsearch node name (string)
  {ES_CLUSTER_NAME} — Elasticsearch cluster name (string)
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
            serializer:
              type: "static"
              class_name: "JavaCustomAuditLogSerializer" # when your serializer class is not in default package, you should use full class name here (eg. "tech.beshu.ror.audit.instances.QueryAuditLogSerializer")
   ```

3. Start elasticsearch \(with ROR installed\) and grep for:

   ```text
    [2023-03-26T16:28:40,471][INFO ][t.b.r.a.f.d.AuditingSettingsDecoder$] Using custom serializer: JavaCustomAuditLogSerializer
   ```
