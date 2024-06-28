# Authorization rules in ROR

In ROR we have plenty of authorization rules:
1. `groups` and `groups_or`
2. `ldap_authorization` (and `ldap_auth` which is an authentication and an authorization rule at the same time)
3. `groups_provider_authorization`
4. `jwt_auth` (and `ror_kbn_auth`)

They are very similar. The main difference is the source of the user groups. 

In a nutshell, a ROR authorization rule logic consists of the following steps:
1. obtaining users' groups 
2. checking groups logic

### Obtaining users' groups 

Each authorization rule has a different method of obtaining the users' groups:
* `groups` and `groups_or` - groups are locally defined
* `ldap_authorization` (and `ldap_auth`) - groups are fetched from LDAP
* `groups_provider_authorization` - groups come from an external HTTP service
* `jwt_auth` (and `ror_kbn_auth`) - groups are extracted from JWT claims
   
### Checking groups logic

In each authorization rule, we need to define what groups are eligible. You can use full group IDs (eg. `management_depA`) or group IDs with wildcards (eg. `management_*`). In the same place, you are defining the checking logic. At the moment ROR supports two of them:
* any of group IDs (or/and group ID patterns) should match (boolean OR logic - defined as `groups` or `groups_or`)
* all of group IDs (or/and group ID patterns) should match (boolean AND logic - defined as `groups_and`)

Example: 

```yaml
ldap_authorization:
  name: "LDAP_1"
  groups: ["group1", "group2", "group12*"] # match if in the obtained user's groups list there is one that equals `group1` OR equals `group2` OR matches the `group12*` pattern
```

```yaml
ldap_authorization:
  name: "LDAP_1"
  groups_and: ["group*", "*depA"] # match if in the obtained user's groups list there is one that matches the `group*` pattern AND matches the `*depA` pattern at the same time
```