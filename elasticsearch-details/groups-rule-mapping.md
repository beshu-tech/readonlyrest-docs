# External to local groups mapping 

`Groups` rule matches request which user is assigned to at least one of its groups. Groups are being assigned to a username (or username pattern) in `users` section of ROR's configuration. Each element of the section requires also an authentication method (like eg. `auth_key` or `ldap_authentication`). But we can also use an authorization rule in this section beside the mentioned authentication rule (or one rule which provides both - the authentication and authorization altogether). Thanks to this, we gain a possibility to map external groups to local ones. Let's see it on an example.

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
  - username: "joe"
    groups: ["editors"]
    auth_key: joe:password

  - username: "*"
    groups: ["viewers", "editors"]
    external_authentication: "ext1"
    groups_provider_authorization:
      user_groups_provider: "ext2"
      groups: ["external_group1", "external_group2"]

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

Let's back to our example. In the second element of `users` array, it states that:

* any user can be taken into consideration by this user definition
* a user should be authenticated by an `external_authentication` rule which uses the `ext1` service
* a user should be authorized by a `groups_provider_authorization` rule which uses the `ext2` service and matches only if the service returns at least one of `external_group1`, `external_group2` groups
* if all of the requirements defined above are fulfilled, we can assign `viewers`, `editors` groups to the user 

It means that we mapped external `external_group1`, `external_group2` returned by service `ext2` to local ROR group `viewers`, `editors`. 

The third element of `users` array (in the example above) is similar, but we use one rule which is authentication and authorization rule at the same time (it can authenticate a user and then authorize him). And that's how we defined a following mapping: LDAP rules `ldap_role_devops`, `ldap_role_ops`, `ldap_role_dev` are mapped to `devops` ROR's local group.

The feature described above can be useful, because in `users` section we can define all external to local groups mappings and after that, in our ACL, we can use only local groups. This makes our ACL much more concise, readable and easier to maintain. 