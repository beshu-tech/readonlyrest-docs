# Authorization rules in ROR

In ROR we have plenty of authorization rules:
1. [Groups rules](../elasticsearch.md#groups-rules)
2. `ldap_authorization` (and `ldap_auth` which is an authentication and an authorization rule at the same time)
3. `groups_provider_authorization`
4. `jwt_auth` 
5. `ror_kbn_authorization` (and `ror_kbn_auth` which is an authentication and an authorization rule at the same time)

They are very similar. The main difference is the source of the user groups. 

In a nutshell, a ROR authorization rule logic consists of the following steps:
1. obtaining users' groups 
2. checking groups logic

### Obtaining users' groups 

Each authorization rule has a different method of obtaining the users' groups:
* [Groups rules](../elasticsearch.md#groups-rules) - groups are locally defined
* `ldap_authorization` (and `ldap_auth`) - groups are fetched from LDAP
* `groups_provider_authorization` - groups come from an external HTTP service
* `jwt_auth` - groups are extracted from JWT claims
* `ror_kbn_authorization` (and `ror_kbn_auth`) - groups are extracted from JWT claims

### Checking groups logic

In each authorization rule, we need to define what groups are eligible. You can use full group IDs (eg. `management_depA`) or group IDs with wildcards (eg. `management_*`). In the same place, you are defining the checking logic. At the moment ROR supports:
* [any of group IDs](../elasticsearch.md#groups_any_of) (or/and group ID patterns) should match (boolean OR logic - defined as `groups_any_of`)
* [all of group IDs](../elasticsearch.md#groups_all_of) (or/and group ID patterns) should match (boolean AND logic - defined as `groups_all_of`)
* [not any of group IDs](../elasticsearch.md#groups_not_any_of) (or/and group ID patterns) should match (defined as `groups_not_any_of`)
* [not all of group IDs](../elasticsearch.md#groups_not_all_of) (or/and group ID patterns) should match (defined as `groups_not_all_of`)
* [combination of positive logic with negative logic](../elasticsearch.md#groups_combined)

**⚠️IMPORTANT** the negative groups logic (`groups_not_any_of`/`groups_not_all_of`) cannot be used, when `server_side_groups_filtering` is enabled for LDAP authorization rules. 
- in that case please use the combined logic, for example with `any_of` positive logic.
- Check the [LDAP connector section](../elasticsearch.md#ldap-connector) to see how to configure the connector

Example: 

```yaml
ldap_authorization:
  name: "LDAP_1"
  groups_any_of: ["group1", "group2", "group12*"] # match if in the obtained user's groups list there is one that equals `group1` OR equals `group2` OR matches the `group12*` pattern
```

```yaml
ldap_authorization:
  name: "LDAP_1"
  groups_all_of: ["group*", "*depA"] # match if in the obtained user's groups list there is one that matches the `group*` pattern AND matches the `*depA` pattern at the same time
```

```yaml
ldap_authorization:
  name: "LDAP_1"
  groups: # match if group1 present or group2 present, but not when both are present
    any_of: ["group1", "group2"]
    not_all_of: ["group1", "group2"]
```
