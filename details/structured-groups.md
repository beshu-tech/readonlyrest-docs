# Structured groups

**TIP** Structured groups are a useful concept in the context of ReadonlyREST Enterprise Kibana plugin. You can skip this configuration if you have a Pro or Free plugin version.

Group ID represents a group/roles concept in the ROR ACL. You can define a neat, human-readable ID of the group when you introduce groups on the configuration level.
However, when group IDs come from external integrations, the group IDs may have an unfriendly format.
That's why we introduced a structured group concept, which consists of:
* Group ID - a group identifier, used in the `groups`, `groups_or`, and `groups_and` ACL rules. Group IDs are the only values that ROR operates on (if you use variables in the `groups` rule, like `acl:current_group` and `acl:available_groups`, only group IDs will be involved)
* Group Name - a human-readable name of the group. If not defined, it's equal to the group ID. You can use group names in the Kibana Enterprise Plugin and map group IDs to friendly names for the end users in the Tenancy Selector.

Thanks to the structured groups, you can configure neat group names and hide the internals of external integrations from end users.

### Configuration examples

The section below presents configuration examples for different connectors. For more details check the configuration of each connector

#### LDAP

```yaml
readonlyrest:
  ldaps:
  - name: ldap1
    [...]
    groups:
      # structured groups configuration
      group_id_attribute: "cn"  # optional, default `cn`
      group_name_attribute: "o" # optional, default - the 'group_id_attribute'
```

#### HTTP Service

```yaml
readonlyrest:
  user_groups_providers:
    - name: GroupsService
      # structured groups configuration
      response_groups_ids_json_path: "$..groups[?(@.id)].id"         # JSON-path style, see https://github.com/json-path/JsonPath
      response_groups_names_json_path: "$..groups[?(@.name)].name"   # JSON-path style, see https://github.com/json-path/JsonPath
      [...]
```

#### JWT claims

```yaml
readonlyrest:
  jwt:
    - name: jwt_provider_1
      # structured groups configuration
      group_ids_claim: resource_access.client_app.group_ids # JSON-path style, see https://github.com/json-path/JsonPath
      group_names_claim: resource_access.client_app.group_names # JSON-path style, see https://github.com/json-path/JsonPath
      [...]
```

#### Locally defined groups

```yaml
readonlyrest:
  users:

    # structured group defined locally 
  - username: "bob"
    groups:
    - id: "admins"
      name: "Admins Group"
    - id: "support"
      name: "Support Group"
    auth_key: bob:s3cr37

    # structured group with group mapping
  - username: "*"
    groups:
    - local_group:
        id: "devops"
        name: "DevOps Group"
      external_group_ids:  ["ldap_role_ops", "ldap_*_devops"]
    - local_group:
        id: "developers"
        name: "Developers Group"
      external_group_ids: ["ldap_role_dev"]
    ldap_auth:
      name: "ldap1"
      groups: ["ldap_*_devops", "ldap_role_ops", "ldap_role_dev"]

  ldaps:
    - name: ldap1
      [...]
```
