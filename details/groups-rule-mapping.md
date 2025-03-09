# External to local groups mapping 

The `groups` ACL rule accepts a list of group IDs. This rule will match on a requests in which the resolved username belongs at least to one of the listed groups. The association between usernames and groups is explicitly declared in the users section of the ACL. This is a list of usernames, and today, full wildcard patterns are also supported.

In the `users` section, each entry requires:
* an authentication rule: (I.e. one of the `auth_key_*` rules for local credentials, or `ldap_authentication`, `external_authentication`, etc)
* a list of groups within the ones precendently inserted in the groups rules of the ACL blocks
* optionally, an authorization rule (`ldap_authorization`, `groups_provider_authorization`, etc.)
  
When the users section's `groups` rule and the authorization rule are used together, we obtain "group mapping". That is: we are effectively mapping remote groups to local groups. There are two types of mapping available: **common** and **detailed** group mappings.

*Note:* the rule `ldap_auth` is the composition of `ldap_authentication` and `ldap_authorization`. So it can be used as a shortcut for both.
## Example

```yaml
readonlyrest:

  access_control_rules:
  - name: "Viewer block"
    indices: ["logstash-viewers*"]
    groups_any_of: ["viewers"]

  - name: "DevOps block"
    indices: ["logstash-devops*"]
    groups_any_of: ["devops"]

  [...]

  users:
  # PLAIN LOCAL GROUPS EXAMPLE
  # Local user "joe" is associated to local group "editors"
  - username: "joe"
    groups: ["editors"]
    auth_key: joe:password
    
  # COMMON GROUP MAPPING EXAMPLE
  # Externally authenticated user + authorization via external groups provider + groups common mapping
  # Users belonging to "external_group1" OR "external_group2" are authorized as "viewers" AND "editors" in the ACL.
  - username: "*"
    groups: ["viewers", "editors"]
    external_authentication: "ext1"
    groups_provider_authorization:
      user_groups_provider: "ext2"
      groups_any_of: ["external_group1", "external_group2"]
  
  # DETAILED GROUP MAPPING EXAMPLE
  # LDAP authenticated user + authorization via LDAP + groups detailed mapping (any LDAP user is valid; groups from `ldap1` are mapped to local groups) 
  # Users belonging to LDAP role `ldap_role_ops`, or any other LDAP role that matched `ldap_*_devops` pattern, will be mapped to "devops" local group 
  # AND 
  # Users belonging to LDAP `ldap_role_dev` are mapped to "developers" local group
  - username: "*"
    groups: 
      - devops: ["ldap_role_ops", "ldap_*_devops"]
      - developers: ["ldap_role_dev"]
    ldap_auth:
      name: "ldap1"
      groups_any_of: ["ldap_*_devops", "ldap_role_ops", "ldap_role_dev"]


  # DETAILED GROUP MAPPING EXAMPLE (STRUCTURED GROUPS)
  # LDAP authenticated user + authorization via LDAP + groups detailed mapping (any LDAP user is valid; groups from `ldap1` are mapped to local groups) 
  # Users belonging to LDAP role `ldap_role_ops`, or any other LDAP role that matched `ldap_*_devops` pattern, will be mapped to "devops" local group 
  # AND 
  # Users belonging to LDAP `ldap_role_dev` are mapped to "developers" local group
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
      groups_any_of: ["ldap_*_devops", "ldap_role_ops", "ldap_role_dev"]

  external_authentication_service_configs:
  - name: "ext1"
    [...]

  user_groups_providers:
  - name: ext2
    [...]

  ldaps:
  - name: ldap1
    [...]
```

As we can see, there are two blocks in our ACL:

1. `Viewer block` allows all users, which belong to `viewers` group, to access indices matching pattern `logstash-viewers*`
1. `DevOps block` allows all users, which belong to `devops` group, to access indices matching pattern `logstash-devops*`

### Common mapping example
```yml
  - username: "*"
    groups: ["viewers", "editors"]
    external_authentication: "ext1"
    groups_provider_authorization:
      user_groups_provider: "ext2"
      groups_any_of: ["external_group1", "external_group2"]
```

`viewers`, `devops`, (unused in the ACL example), `editors` and `developers` are local groups. That is, they exist only at ROR's configuration level. But ROR can also integrate with external authorization systems like an LDAP or some REST service, where we can find similar concepts to ROR groups (eg. users in LDAP can have roles assigned).

And sometimes we'd like to fulfil a requirement such as:

> Users having usernames defined by a given pattern, and having a given set of roles, should have certain given ROR internal groups assigned.

You can think about it as a mapping external groups to local ones. 

Let's go back to our example. In the second element of the `users` array, we declare that:

* any user can be taken into consideration by this user definition
* a user should be authenticated by an `external_authentication` rule which uses the `ext1` service
* a user should be authorized by a `groups_provider_authorization` rule which uses the `ext2` service and such user belongs to at least one of `external_group1`, `external_group2` external groups.
* if all the above conditions are true, we can assign `viewers`, `editors` groups to the user

We have just "mapped" the external groups `external_group1`, `external_group2` returned by service `ext2` to the local ROR groups `viewers`, `editors`.

### Detailed mapping example
```yml
  - username: "*"
    groups: 
      - devops: ["ldap_*_devops", "ldap_role_ops"]
      - developers: ["ldap_role_dev"]
    ldap_auth:
      name: "ldap1"
      groups_any_of: ["ldap_role_devops", "ldap_role_ops", "ldap_role_dev"]
```
The third element of `users` array (in the example above) is similar, but we use one rule which is authentication and authorization rule at the same time (it can authenticate a user and then authorize him). And that's how we defined the following mappings: 
* `ldap_role_ops` LDAP role, and any other LDAP role matching `ldap_*_devops` pattern, are mapped to `devops` ROR's local group
* LDAP role `ldap_role_dev` is mapped to `developers` ROR's local group

The "detailed" mapping offers a bit more structured approach to group mapping, and although less intuitive at first sight, it's more powerful and concise.
