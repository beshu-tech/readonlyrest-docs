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

The audit outputs are disabled by default. To enable them, add `audit.enabled: true` and optionally configure `audit.outputs`.

**Note**: Even when `audit.enabled` is `false` or not set, the built-in ACL log is a special case — it writes a human-readable decision line to Elasticsearch logs for every request by default. See [The default ACL log](#the-default-acl-log) below.

The following is the explicit equivalent of the default behaviour when no `audit` section is configured at all:

```yaml
readonlyrest:
  audit:
    enabled: false                 # audit outputs disabled
    default_acl_log_enabled: true  # ACL log still fires for every request
```

When `audit.enabled: true` and no `outputs` are specified, ROR defaults to storing events in a local Elasticsearch index.

### Global audit settings

```yaml
readonlyrest:
  audit:
    enabled: true                   # enable/disable the entire audit subsystem
    default_acl_log_enabled: true   # enable/disable the built-in ACL log (default: true)
    outputs:
    - type: index
      name: my-index-output         # optional name, used for per-block output routing
```

| Setting | Default | Description |
|---|---|---|
| `enabled` | `false` | Master switch for the audit subsystem |
| `default_acl_log_enabled` | `true` | Controls the built-in ACL log output (see below) |
| `outputs` | default `index` output | List of audit outputs |

Each entry in `outputs` accepts an optional `name` field. Names are only needed for per-block routing: when you want a specific block to send events to only a subset of outputs, you reference them by name using `enabled_audit_outputs` or `disabled_audit_outputs` (see [Block-level audit control](#block-level-audit-control)). If you do not need per-block routing, you can omit `name` from all outputs.

### The default ACL log

When `default_acl_log_enabled: true` (the default), ROR writes a human-readable ACL decision line to Elasticsearch logs for every request, using the logger named `tech.beshu.ror.accesscontrol.logging.AccessControlListLoggingDecorator`. This happens regardless of whether any `outputs` are configured.

The default ACL log is exposed as a named output with the reserved name `default_acl_log`. You can use this name in block-level `enabled_audit_outputs` and `disabled_audit_outputs` to include or exclude it from per-block routing:

```yaml
readonlyrest:
  audit:
    enabled: true
    default_acl_log_enabled: true
    outputs:
    - type: index
      name: my-index-output

  access_control_rules:

  - name: High-volume service account
    auth_key: svc:secret
    audit:
      # send events only to the index, skip the ACL log line for this noisy block
      enabled_audit_outputs: [my-index-output]

  - name: Admin users
    auth_key: admin:admin
    # No audit section — all outputs active with default settings
```

To replace the default ACL log with a custom one — for example to send it to a different file — disable it globally and add a `log` output with the `acl` serializer:

```yaml
readonlyrest:
  audit:
    enabled: true
    default_acl_log_enabled: false   # turn off the built-in ACL log
    outputs:
    - type: log
      name: custom-acl-log
      logger_name: my.custom.acl.logger
      serializer:
        type: acl                    # reproduces the built-in ACL log format
```

`logger_name` is the log4j2 logger name that ROR uses when writing to this output (default: `readonlyrest_audit`). Setting a custom value lets you route these log lines to a dedicated appender in `log4j2.properties` — for example to write them to a separate file. See [Custom logging settings via log4j2](#custom-logging-settings-via-log4j2) for an example appender configuration.

The `acl` serializer produces exactly the same single-line human-readable format as the built-in ACL log: a concise decision summary that includes the request identity, matched block, final state, and key request details. The format is unchanged compared to previous versions.

### Block-level audit control

The `audit` section inside each `access_control_rules` block lets you tune audit behaviour per block.

```yaml
access_control_rules:
- name: Example block
  auth_key: user:pass
  audit:
    enabled: true                             # default: true — set to false to suppress all audit for this block
    log_allowed_events: true                  # default: true — set to false to suppress allowed-request events
    enabled_audit_outputs: [my-index-output]  # whitelist: only these named outputs receive events from this block
    # disabled_audit_outputs: [ops-log]       # blacklist: all outputs except these. Use one key or the other, never both
```

| Setting | Default | Description |
|---|---|---|
| `enabled` | `true` | When `false`, no audit events are emitted when this block is matched, regardless of global settings |
| `log_allowed_events` | `true` | When `false`, allowed requests matched by this block are not written to audit. Denied requests, errors, and index-not-found responses are always written |
| `enabled_audit_outputs` | (all outputs) | Whitelist of output names. Only the listed outputs receive events from this block. Use output `name` values from `audit.outputs`, plus `default_acl_log` for the built-in ACL log. The list must name at least one output |
| `disabled_audit_outputs` | (none) | Blacklist of output names. All outputs except the listed ones receive events from this block. The list must name at least one output |

`enabled_audit_outputs` and `disabled_audit_outputs` are mutually exclusive — you cannot specify both on the same block.

Neither list can be empty. ROR refuses to load settings that contain `enabled_audit_outputs: []` or `disabled_audit_outputs: []`. Omit the key to send events to every output. To send no events at all from a block, use `audit: {enabled: false}`.

**⚠️IMPORTANT**: When `audit.enabled: false` for a specific block, there will be no audit events at all when that block is matched — this suppresses both custom outputs and the default ACL log. **This is a change in behaviour from previous versions**, where block-level `audit: {enabled: false}` only suppressed the ES audit outputs while the ACL log continued to write.

#### Per-output routing example

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index
      name: security-index
    - type: log
      name: ops-log

  access_control_rules:

  - name: Security-sensitive block
    auth_key: admin:admin
    audit:
      # Only write to the security index, skip the ops log and default ACL log
      enabled_audit_outputs: [security-index]

  - name: Noisy read-only block
    auth_key: reader:pass
    audit:
      # Skip allowed events entirely, errors still get written
      log_allowed_events: false
      # Write to ops log only, skip security index for this block
      enabled_audit_outputs: [ops-log]

  - name: Regular block
    auth_key: user:pass
    # No audit section = all outputs active with default settings
```

### Multiple outputs

You can configure multiple audit outputs, including mixing output types:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index
      name: audit-index
    - type: log
      name: audit-log
    - type: data_stream
      name: audit-stream
      enabled: false   # this output is defined but currently disabled
```

Each output can be individually toggled with `enabled: true/false` (default: `true`).

### Backward compatibility

The `verbosity: error` and `verbosity: info` block-level settings from earlier versions are still accepted. They are treated as aliases for `audit: {log_allowed_events: false}` and `audit: {log_allowed_events: true}` respectively.

All other global audit settings (`audit.enabled`, `audit.outputs` and their sub-settings) are unchanged. Existing configurations that do not use the new settings will continue to work without modification. The `default_acl_log_enabled` setting defaults to `true`, so the ACL log continues to fire exactly as before for configurations that do not set it explicitly.

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

**⚠️IMPORTANT**: All audit nodes must belong to the same Elasticsearch cluster. Otherwise, each audit cluster will contain only a subset of audit events.
If you intend to send audit events to multiple clusters, define one output per Elasticsearch cluster.

The `cluster` setting is optional. It accepts two syntaxes.

**Simple syntax**: a non-empty list of audit cluster node URIs. ROR sends the audit events in the `round-robin` mode and does not check the connectivity of the cluster.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index
      cluster: ["https://user1:password@auditNode1:9200", "https://user1:password@auditNode2:9200"]
  ...
```

**Extended syntax**: an object which also sets the cluster mode, the credentials and the connectivity check.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index
      cluster:
        nodes: ["https://auditNode1:9200", "https://auditNode2:9200"]
        mode: failover
        username: "audit_user"
        password: "audit_password"
        connectivity_check: required
  ...
```

| Setting | Required | Default | Description |
| --- | --- | --- | --- |
| `nodes` | yes | - | Non-empty list of audit cluster node URIs. |
| `mode` | yes | - | How ROR picks the node for an audit event: `round-robin` or `failover`. |
| `username`, `password` | no | credentials from the node URIs | Credentials used for every audit node. Set both fields or none of them. |
| `connectivity_check` | no | `disabled` | How a connectivity problem of the audit cluster influences the settings loading: `required`, `best_effort` or `disabled`. |

##### Cluster modes

* `round-robin` - ROR spreads the audit events over all the audit nodes. One client knows all the nodes. When a node fails, the client marks it as dead, sends the request to another node and retries the dead node later.

* `failover` - ROR sends the audit events to the first audit node which is available, in the order of the `nodes` list. Each node has its own client and its own circuit breaker. After a failure, the node is skipped for one second. The skip time grows with each subsequent failure, up to 30 minutes. ROR tries the next node when the request ends with a connection error or with the `502`, `503` or `504` status code. Any other error status stops the request, because another node answers it in the same way.

Use `round-robin` when all the audit nodes are equal. Use `failover` when they are not, for example when the first node is the local one and the others are a remote backup.

##### Credentials

All the audit nodes must use the same credentials. ROR does not accept a configuration where the node URIs hold different credentials, because different credentials usually mean that the nodes belong to different clusters.

Put the credentials in the `username` and `password` fields, or in each node URI (`https://user:password@auditNode1:9200`). The `username` and `password` fields win when both places are used.

##### Connectivity check

ROR can verify the audit cluster when it loads the settings. That happens at the start of the node and at every settings reload. The check calls `GET /` on every audit node in parallel and compares the answers.

| Value | Behaviour |
| --- | --- |
| `disabled` | ROR does not check the audit cluster. |
| `required` | ROR rejects the settings when no audit node answers. |
| `best_effort` | ROR logs the connectivity problem and starts the auditing anyway. |

A node which does not answer is retried three times, with a first delay of 500 milliseconds which doubles after each attempt. Only a connection error and the `408`, `429`, `502`, `503` and `504` status codes are retried. The whole check ends after 30 seconds. The check does not validate the TLS certificates of the audit nodes.

A node which rejects the credentials of the check with the `401` or `403` status code counts as a node which did not answer.

When some nodes answer and the others do not, the check passes and ROR logs a warning with the details of the unreachable nodes.

When the nodes which answer report different `cluster_uuid` values, ROR rejects the settings in the `required` mode and in the `best_effort` mode. This is a configuration error, not a connectivity problem: one audit output can use the nodes of one cluster only.

When ROR rejects the settings, the node does not start the auditing with them.

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

**⚠️IMPORTANT**: All audit nodes must belong to the same Elasticsearch cluster. Otherwise, each audit cluster will contain only a subset of audit events.
If you intend to send audit events to multiple clusters, define one output per Elasticsearch cluster.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: data_stream
      cluster:
        nodes: ["https://auditNode1:9200", "https://auditNode2:9200"]
        mode: failover
        username: "audit_user"
        password: "audit_password"
        connectivity_check: required
  ...
```

The `cluster` setting accepts the same simple and extended syntax as the `index` output. See [Custom audit cluster](#custom-audit-cluster) for the description of `nodes`, `mode`, `username`, `password` and `connectivity_check`.

**⚠️IMPORTANT**: The `data_stream` output verifies the audit data stream when it loads the settings, and creates the data stream when it does not exist. Therefore an unreachable audit cluster makes the settings loading fail, also when `connectivity_check` is `best_effort` or `disabled`. The `index` output creates the audit index with the first audit event, so it does not have this restriction.

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
     * Edit the template's settings or mappings as needed.
     * Save the updated template; new indices created for the data stream will use these settings.

5. **Verify Changes**

   * After updating policies or templates, monitor your data stream to ensure rollover and retention behave as expected.
   * You can also query audit events via Kibana's Discover tab or using the Elasticsearch API.

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

### Sending audit events through an ingest pipeline

You can send the audit events of an `index` or `data_stream` output through an Elasticsearch [ingest pipeline](https://www.elastic.co/docs/manage-data/ingest/transform-enrich/ingest-pipelines). Elasticsearch runs the pipeline on each audit event, before it stores the event. Use a pipeline to change the audit events without a custom serializer. For example, a pipeline can:

* add a field, such as the name of the environment
* remove or mask a field that contains sensitive data
* add data from other indices, with the `enrich` processor

The `pipeline` setting is optional. If you do not set it, ROR sends the audit events to Elasticsearch without a pipeline, the same as before this setting existed. The `log` output does not support pipelines.

#### Step 1: Create the pipeline

Create the pipeline in the cluster that stores the audit events:

* If the output does not set `cluster`, create the pipeline in the local cluster.
* If the output sets `cluster`, create the pipeline in that audit cluster.

This example pipeline adds the field `environment` with the value `production` to each audit event:

```
PUT _ingest/pipeline/audit_add_environment
{
  "processors": [
    { "set": { "field": "environment", "value": "production" } }
  ]
}
```

To test the pipeline before you use it, run it on a sample document:

```
POST _ingest/pipeline/audit_add_environment/_simulate
{
  "docs": [
    { "_source": { "user": "admin", "action": "indices:data/read/search" } }
  ]
}
```

#### Step 2: Set the pipeline in the audit output

Put the ID of the pipeline in the `pipeline` setting of the output:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: index
      pipeline: "audit_add_environment"
    - type: data_stream
      pipeline: "audit_add_geoip"
```

Each output has its own optional `pipeline` setting. Two outputs can use the same pipeline or different pipelines. An output without the `pipeline` setting stores the audit events without changes.

ROR rejects the settings in these cases:

* The `pipeline` value is empty.
* The `pipeline` value is `_none`. To store audit events without a pipeline, remove the `pipeline` setting.
* A `log` output has the `pipeline` setting.

When you change the pipeline in Elasticsearch, the change applies to the next audit event. You do not have to reload the ROR settings.

#### When Elasticsearch cannot store an audit event

ROR does not check that the pipeline exists when it starts or when it loads new settings. If the pipeline does not exist, this is what happens:

1. ROR starts and loads the settings without an error.
2. Users send requests to Elasticsearch. ROR allows or forbids the requests as usual. The missing pipeline has no effect on the requests or on their responses.
3. ROR sends the audit event of each request to Elasticsearch, with the ID of the pipeline.
4. Elasticsearch cannot find the pipeline, and it returns an error for the audit event. Elasticsearch does not store the event.
5. ROR writes the error to the Elasticsearch log (see the messages below). ROR does not try again, and does not store the event without the pipeline.

**⚠️IMPORTANT**: The audit events that Elasticsearch rejects are lost. To prevent this, create the pipeline before you add it to the ROR settings. If you create the pipeline later, Elasticsearch stores the audit events from that time. The events from before are not recovered.

The same thing occurs when the pipeline exists but fails on an audit event, for example when a processor cannot read a field.

Elasticsearch also rejects all audit events that use a pipeline when the cluster has no node with the `ingest` role.

When Elasticsearch rejects audit events, ROR writes these errors to the Elasticsearch log:

* For the local cluster: `Some failures flushing the BulkProcessor:`, and then `<number of events>x: [<index>] <error from Elasticsearch>`. ROR groups the same errors into one line.
* For an audit cluster: `Cannot submit audit event [index: <index>, doc: <document id>]`, with the request and the response from Elasticsearch.

A `data_stream` output can use the data stream [failure store](https://www.elastic.co/docs/manage-data/data-store/data-streams/failure-store). If the failure store of the audit data stream is enabled, Elasticsearch puts the rejected events into the failure store. The events are not lost, and ROR does not write an error to the log.

#### Things to know

* **Default and final pipelines of the audit index.** The `pipeline` setting replaces the `index.default_pipeline` of the audit index or data stream. Thus, Elasticsearch does not run the default pipeline. If you also need the default pipeline, call it from your pipeline with the [`pipeline` processor](https://www.elastic.co/docs/reference/enrich-processor/pipeline-processor). The `index.final_pipeline` still runs, after your pipeline.
* **The `@timestamp` field.** A data stream accepts only documents with the `@timestamp` field. A pipeline for a `data_stream` output must not remove this field.
* **The ROR audit views in Kibana.** The ROR Kibana plugin reads the fields that the ROR serializer creates. If your pipeline removes or renames these fields, the audit views in Kibana can show wrong or empty data. It is safe to add new fields.

### The 'log' output specific configurations

The `log` output writes audit events to Elasticsearch log at INFO level using a dedicated logger.

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: log
  ...
```

#### Built-in rolling file appender

For a self-contained rolling file output — without editing `log4j2.properties` — use the `file_appender` sub-section:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: log
      name: rolling-audit
      file_appender:
        file_path: /var/log/elasticsearch/ror_audit.log   # absolute path; directory must be writable
        max_file_size: "100MB"                            # accepted units: B, KB, MB, GB, TB
        max_files: 10                                     # number of rotated files to keep
```

When `file_appender` is present, ROR creates and manages the rolling appender internally, bypassing the default Elasticsearch log routing for this output. The `logger_name` setting is still accepted and used as the appender name.

#### Custom logger name

If you want to route log output through a specific log4j2 logger:

```yaml
readonlyrest:
  audit:
    enabled: true
    outputs:
    - type: log
      logger_name: custom-logger-name
  ...
```

The default logger name is `readonlyrest_audit`.

#### Custom logging settings via log4j2

For advanced log configuration — custom patterns, external syslog appenders, etc. — configure the logger in `$ES_PATH_CONF/config/log4j2.properties`. The logger name must match the `logger_name` value in the output config (or the default `readonlyrest_audit`):

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

# Logger name must match the one configured in readonlyrest.yml (or the default "readonlyrest_audit")
logger.readonlyrest_audit.name = readonlyrest_audit
logger.readonlyrest_audit.appenderRef.readonlyrest_audit_rolling.ref = readonlyrest_audit_rolling
# set to false to use only desired appenders
logger.readonlyrest_audit.additivity = false
```

#### ACL serializer

The `log` output type supports a special `acl` serializer that reproduces the human-readable format written by the default ACL log. This is useful when you want to disable the built-in ACL log (`default_acl_log_enabled: false`) and replace it with a custom log output that you can route per-block:

```yaml
readonlyrest:
  audit:
    enabled: true
    default_acl_log_enabled: false
    outputs:
    - type: log
      name: custom-acl-log
      logger_name: my.acl.logger
      serializer:
        type: acl
```

The `acl` serializer type is only valid for `log` outputs. Attempting to use it with `index` or `data_stream` outputs will produce a configuration error.

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
* use ECS ([Elastic Common Schema](https://www.elastic.co/docs/reference/ecs)) serializer (no implementation required, [learn more about it](#using-ecs-serializer))
* implement and use your own serializer ([see how to implement a custom serializer](#custom-audit-event-serializer))

To add or change fields without a custom serializer, you can also send the audit events of an `index` or `data_stream` output through an ingest pipeline ([see how to do it](#sending-audit-events-through-an-ingest-pipeline)).


### Predefined serializers:
* `tech.beshu.ror.audit.instances.BlockVerbosityAwareAuditLogSerializer`
  * Serializes all non-`Allowed` events.
  * Serializes `Allowed` events only when the matched block has `log_allowed_events: true` (the default).
  * Recommended for standard audit logging, where full request body capture is not required.
  * Fields included:
    ```
     match — whether the request matched a rule (boolean)  
     matched_block_names - list of names of the blocks, that were matched (both forbidden and allowed) (array of strings)
     id — audit event identifier (string)  
     final_state — final processing state (ALLOWED/FORBIDDEN/ERRORED/INDEX NOT EXIST) (string)  
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
  * Serializes all events of all types, including all `Allowed` events, regardless of the block's `log_allowed_events` setting.
  * Included fields are the same as for `BlockVerbosityAwareAuditLogSerializer`
  * Use this serializer, when you need complete coverage of all events.
* `tech.beshu.ror.audit.instances.FullAuditLogWithQuerySerializer`
  * Serializes all events of all types, including all `Allowed` events, regardless of the block's `log_allowed_events` setting.
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
            allowed_events_serialization_mode: "based_on_block_settings" # serialize Allowed events only from blocks with log_allowed_events: true (default); use "always" to serialize all Allowed events
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

You can also define nested structure of fields, and use fixed text, number and boolean values:
```yaml
  fields:
    tid: "{TASK_ID}"
    es_details:
      node_name: "{ES_NODE_NAME}"
      cluster_name: "{ES_CLUSTER_NAME}"
    event_details:
      custom_system_id: 12345 # example of hardcoded number value, can also be a decimal number; in this example represents some hardcoded system id
      is_dev_environment: false # example of hardcoded boolean value; in this example represents flag marking the events from development environment
      http:
        request_description: "HTTP request: {HTTP_METHOD} {HTTP_PATH}"
        request_details:
          method: "{HTTP_METHOD}"
          path: "{HTTP_PATH}"
```

The configuration above corresponds to serialized event looking like that:

```json
{
  "tid": 0,
  "es_details": {
    "node_name": "esNode01",
    "cluster_name": "mainEsCluster"
  },
  "event_details": {
    "custom_system_id": 12345,
    "is_dev_environment": false,
    "http": {
      "request_description": "HTTP request: GET /_cat",
      "request_details": {
        "method": "GET",
        "path": "/_cat"
      }
    }
  }
}
```

Available placeholders:
```
  {IS_MATCHED} — whether the request matched a rule (boolean)
  {MATCHED_BLOCK_NAMES} — list of names of the blocks that were matched, both allowed and forbidden (array of strings)
  {ID} — audit event identifier (string)
  {FINAL_STATE} — final processing state (string)
  {ECS_EVENT_OUTCOME} - final processing state, mapped to ECS-compliant values: success/failure/unknown (string)
  {TIMESTAMP} — event timestamp (ISO-8601 string)
  {CORRELATION_ID} — correlation identifier for tracing (string)
  {PROCESSING_DURATION_MILLIS} — request processing duration in milliseconds (number)
  {PROCESSING_DURATION_NANOS} — request processing duration in nanoseconds (number)
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

### Using ECS serializer:

Configuration should look like that:
```yaml
    readonlyrest:
      audit:
        enabled: true
        outputs:
        - type: index
          serializer:
            type: "ecs"
            allowed_events_serialization_mode: "based_on_block_settings" # serialize Allowed events only from blocks with log_allowed_events: true (default); use "always" to serialize all Allowed events
            include_full_request_content: false # controls whether the full HTTP request body is included in the ECS audit log (http.request.body field), disabled by default
```

The configuration above corresponds to serialized event, compatible with ECS 1.6 schema, looking like that:
```json
{
  "@timestamp": "2017-06-30T09:41:58Z",
  "trace" : {
    "id" : "correlation_id_123"
  },
  "ecs" : {
    "version" : "1.6.0"
  },
  "source" : {
    "address" : "192.168.0.123"
  },
  "destination" : {
    "address" : "192.168.100.100"
  },
  "http" : {
    "request" : {
      "method" : "GET",
      "body" : {
        "bytes" : 123,
        "content" : "Full content of the request"
      }
    }
  },
  "event" : {
    "duration" : 5000000000,
    "reason" : "RRTestConfigRequest",
    "action" : "cluster:internal_ror/user_metadata/get",
    "id" : "trace_id_123",
    "outcome" : "failure"
  },
  "error" : {},
  "user" : {
    "effective" : {
      "name" : "impersonated_by_user"
    },
    "name" : "logged_user"
  },
  "url" : {
    "path" : "/path/to/resource"
  },
  "labels" : {
    "es_cluster_name" : "testEsCluster",
    "es_task_id" : 123,
    "es_node_name" : "testEsNode",
    "ror_acl_history" : "historyEntry1, historyEntry2",
    "ror_detailed_reason" : "default",
    "ror_involved_indices" : [],
    "ror_final_state" : "FORBIDDEN"
  }
}
```

The ECS schema is highly permissive and ambiguous. The ROR audit events can be mapped to ECS fields in multiple ways. 
If the provided ECS implementation does not suit your needs, you can define your own ECS-compliant serializer as `configurable` serializer.

The provided ECS implementation is equivalent to `configurable` serializer shown below. You can [use and adjust it as needed](#using-configurable-serializer) in the configuration.
```yaml
    readonlyrest:
      audit:
        enabled: true
        outputs:
          - type: index
            serializer:
              type: "configurable"
              allowed_events_serialization_mode: "based_on_block_settings"
              fields: 
                ecs:
                  version: "1.6.0"
                trace:
                  id: "{CORRELATION_ID}"
                url:
                  path: "{HTTP_PATH}"
                source:
                  address: "{REMOTE_ADDRESS}"
                destination:
                  address: "{LOCAL_ADDRESS}"
                http:
                  request:
                    method: "{HTTP_METHOD}"
                    body:
                      # Warning: Enabling logging of the full HTTP request body is not recommended when requests 
                      # may contain sensitive data. It can also significantly increase the size of audit log entries.
                      content: "{CONTENT}"
                      bytes: "{CONTENT_LENGTH_IN_BYTES}"
                user:
                  name: "{LOGGED_USER}"
                  effective:
                    name: "{IMPERSONATED_BY_USER}"
                event:
                  id: "{ID}"
                  duration: "{PROCESSING_DURATION_NANOS}"
                  action: "{ACTION}"
                  reason: "{TYPE}"
                  outcome: "{ECS_EVENT_OUTCOME}"
                error:
                  type: "{ERROR_TYPE}"
                  message: "{ERROR_MESSAGE}"
                labels:
                  x_forwarded_for: "{X_FORWARDED_FOR_HTTP_HEADER}"
                  es_cluster_name: "{ES_CLUSTER_NAME}"
                  es_node_name: "{ES_NODE_NAME}"
                  es_task_id: "{TASK_ID}"
                  ror_involved_indices: "{INVOLVED_INDICES}"
                  ror_acl_history: "{ACL_HISTORY}"
                  ror_final_state: "{FINAL_STATE}"
                  ror_matched_block_names: "{MATCHED_BLOCK_NAMES}"
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

## Protecting the audit index

To prevent users from modifying or deleting audit data, add a `forbid` block to your `readonlyrest.yml` that blocks write and delete actions on the audit indices.

```yaml
- name: "Protect audit index"
  type: forbid
  indices: ["readonlyrest_audit-*"]
  actions:
    - "indices:data/write/*"      # index, update, delete, bulk, *_by_query, reindex-into
    - "indices:admin/delete"      # delete the index
    - "indices:admin/close"       # close (then could be re-opened writable)
    - "indices:admin/open"
    - "indices:admin/settings/*"  # change settings (e.g. flip read-only / replicas)
    - "indices:admin/mapping/*"   # mapping/put + mapping/auto_put (NOT mappings/get — that's read)
    - "indices:admin/aliases"     # re-point / drop the audit aliases
    - "indices:admin/rollover"
    - "indices:admin/resize"      # shrink / split / clone
    - "indices:admin/forcemerge"
    - "indices:admin/freeze"
```

**Placement:** ACL blocks are evaluated top-to-bottom and the first matching block wins.

- If no user or service should be able to write to the audit index via the Elasticsearch API, place this block at the **very beginning** of the ACL. Audit events are written internally by the ReadonlyREST plugin and bypass the ACL entirely, so this does not affect audit collection.
- If some identities (e.g. a dedicated audit reader service) require access that would conflict with this rule, place the `forbid` block after their `allow` blocks but before all other allow rules.

The wildcard pattern `readonlyrest_audit-*` matches the default index name template. If you configured a custom `index_template` prefix, adjust the pattern accordingly.
