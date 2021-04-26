# External to local groups mapping 

The `groups` accepts a list of group names. This rule will match requests in which the resolved username belongs at least to one of the listed groups. The association between usernames and groups is explicitly declared in the users section of the ACL. This is a list of usernames (full wildcard patterns are supported).

Each entry in the association rule requires:
* an authentication rule: (I.e. auth_key_* for local credentials, or ldap_authentication, external_authentication, etc)
* a list of groups within the ones precendently inserted in the groups rules of the ACL blocks
* optional: a remote authorization rule (ldap_authorization, groups_provider_authorization, etc.)
  
When the groups rule and the authorization rule are used together, we obtain "group mapping". That is: we are effectively mapping remote groups to local groups.

*Note:* the rule ldap_auth is the composition of ldap_authentication and ldap_authorization. So it can be used as a shortcut for both.
## Example

```yaml
readonlyrest:

  access_control_rules:
  - name: "Viewer block"
    indices: ["logstash-viewers*"]
    groups: ["viewers"]

  - name: "DevOps block"
    indices: ["logstash-devops*"]
    groups: ["devops"]

  [...]

  users:
  # Local user "joe" is associated to local group "editors"
  - username: "joe"
    groups: ["editors"]
    auth_key: joe:password

  # Externally authenticated user + authorization via group mapping
  # Users belonging to "external_group1" OR "external_group2" are authorized as "viewers" and "editors" in the ACL.
  - username: "*"
    groups: ["viewers", "editors"]
    external_authentication: "ext1"
    groups_provider_authorization:
      user_groups_provider: "ext2"
      groups: ["external_group1", "external_group2"]

  # LDAP Authentication (any LDAP user is valid)
  # LDAP group mapping: users belonging to ANY of these LDAP groups, are mapped to "devops" local group
  - username: "*"
    groups: ["devops"]
    ldap_auth:
      name: "ldap1"
      groups: ["ldap_role_devops", "ldap_role_ops", "ldap_role_dev"]

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

`viewers`, `devops` and (unused in the ACL example) `editors` are local groups. They may exist only at ROR's configuration level. But ROR can also integrate with external systems like an LDAP or some REST service, where we can find a similar concept to ROR groups (eg. users in LDAP can have roles assigned).

And sometimes we'd like to set up a requirement like this:

> Users defined by a given pattern, which can be authorized in external system using given roles, in ROR, should have given groups assigned.

You can think about it as a mapping external groups to local ones. 

Let's go back to our example. In the second element of `users` array, it states that:

* any user can be taken into consideration by this user definition
* a user should be authenticated by an `external_authentication` rule which uses the `ext1` service
* a user should be authorized by a `groups_provider_authorization` rule which uses the `ext2` service and matches only if the service returns at least one of `external_group1`, `external_group2` groups
* if all of the requirements defined above are fulfilled, we can assign `viewers`, `editors` groups to the user 

It means that we mapped external `external_group1`, `external_group2` returned by service `ext2` to local ROR group `viewers`, `editors`. 

The third element of `users` array (in the example above) is similar, but we use one rule which is authentication and authorization rule at the same time (it can authenticate a user and then authorize him). And that's how we defined a following mapping: LDAP rules `ldap_role_devops`, `ldap_role_ops`, `ldap_role_dev` are mapped to `devops` ROR's local group.

The feature described above can be useful, because in `users` section we can define all external to local groups mappings and after that, in our ACL, we can use only local groups. This makes our ACL much more concise, readable and easier to maintain. 