# Objectives 

The `kibana_access` rule is not mandatory to support a Kibana session.

```yml
- name: "Kibana user"
  auth_key: bob:pass
```
The above ACL block does a perfect job at allowing "bob" to access Kibana without any restrictions.

However, ReadonlyREST aims to provide more guarantees than wildly unrestricted access to any indices, and cluster configuration. 

So we came up with 5 distinct access levels:

## Level 0, unrestricted
This level is completely equivalent to the example above, where the `kibana_access` rule was omitted.

```yml
- name: "Kibana user"
  auth_key: bob:pass
  kibana_access: unrestricted # <- Commenting this line out? same effect!
```

Why does `unrestricted` level exist? To let us override template values:

```
helpers:
    cr: &common-rules-tpl
        verbosity: error
        kibana_access: rw
        kibana_hide_apps: [ "Enterprise Search|Overview", "Observability" ]
        kibana_index: ".kibana_@{acl:current_group}"

access_control_rules:
  - name: ADMIN_GRP
    groups: [ administrators ]
    <<: *common-rules-tpl
    kibana_access: unrestricted # <--- Overriding the "rw" default
```

## Other "restricted" levels: admin, rw, ro, ro_strict
The constant idea across all restricted levels is: removing the possibility of accidental data loss, or tampering of the "data indices".

The rw, ro, and ro_strict levels have progressive restrictions to the alteration of the Kibana session, and Elasticsearch stack. And this is reflected in the removal of whole UI components (through CSS). For example, ro/ro_strict can't see "edit", "delete", "new" buttons all across Kibana.

The only difference between "rw" and "admin" is the ability of interaction with ROR settings, activation keys APIs.

In detail:

### 1. Kibana index regulated access
Any RW action is allowed towards ".kibana" index, or any index specified in `kibana_index` rule, in the same ACL block. 
This is necessary to allow for the creation of saved objects: visualizations, dashboards, index patterns, etc.

### 2. Data indices protection
We define "data indices" all indices except the designated kibana index.
We want to avoid accidental deletion or modification of them, therefore they are **read only** for any "restricted" level.

### 3. Special cases
Writing into some other indices is allowed in certain cases (i.e. `.kibana_reporting`)


Overview of levels and what a user is allowed to access with it:

| level\permission | kibana_index | other "data" indices | reporting index | cluster mgmt | ROR settings API |
|------------------|--------------|----------------------|-----------------|--------------|------------------|
| admin            | full         | read only            | full            | read only    | full             |
| rw               | full         | read only            | full            | read only    | none             |
| ro               | full         | read only            | full            | none         | none             |
| ro_strict        | read only    | read only            | none            | none         | none             |


## The logic of the kibana_access rule in plain English

We identify groups of action patterns: CLUSTER_ACTIONS, ADMIN_ACTIONS, RW_ACTIONS, RO_ACTIONS. 

### Action groups definitions (from [Constants.java](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/blob/develop/core/src/main/scala/tech/beshu/ror/Constants.java))

<details>
  
```
CLUSTER_ACTIONS = [
      "cluster:monitor/nodes/info",
      "cluster:monitor/main",
      "cluster:monitor/health",
      "cluster:monitor/state",
      "cluster:monitor/ccr/follow_info",
      "cluster:*/xpack/*",
      "indices:admin/template/get*",
      "cluster:*/info",
      "cluster:*/get"
]
ADMIN_ACTIONS = [
      "cluster:ror/*",
      "indices:data/write/*", // <-- DEPRECATED!
      "indices:admin/create",
      "indices:admin/create_index",
      "indices:admin/ilm/*",
      "indices:monitor/*"
]
RW_ACTIONS = [
      "indices:admin/create",
      "indices:admin/create_index",
      "indices:admin/mapping/put",
      "indices:data/write/delete*",
      "indices:data/write/index",
      "indices:data/write/update*",
      "indices:data/write/bulk*",
      "indices:admin/template/*",
      "cluster:admin/settings/*",
      "indices:admin/aliases/*"
]
                                     
RO_ACTIONS = [
      "indices:admin/exists",
      "indices:admin/mappings/fields/get*",
      "indices:admin/mappings/get*",
      "indices:admin/validate/query",
      "indices:admin/get",
      "indices:admin/refresh*",
      "indices:data/read/*",
      "indices:admin/resolve/*",
      "indices:admin/aliases/get",
      "indices:admin/*/explain",
      "indices:monitor/settings/get",
      "indices:data/read/xpack/rollup/get/*",
      "indices:monitor/stats"
]                                     
```
</details>
  
## Decision tree (from [KibanaAccessRule.scala](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin/blob/master/core/src/main/scala/tech/beshu/ror/accesscontrol/blocks/rules/KibanaAccessRule.scala))
* If value is unrestricted: ALLOW
* If user metadata API was requested (login attempt): ALLOW
* If action in RO_ACTIONS: ALLOW
* If action in CLUSTER_ACTIONS: ALLOW
* If request involves no indices: ALLOW
* If request creates Kibana's sample data: ALLOW
* If request index is kibana_index and value is **not** `ro_strict`: ALLOW
* If action in RO_ACTION, RW_ACTION targeting kibana_index: ALLOW
* If action starts with `"indices:data/write*"`: REJECT
* If value is `admin` and action in ADMIN_ACTIONS: ALLOW
* anything else: REJECT


  

  
