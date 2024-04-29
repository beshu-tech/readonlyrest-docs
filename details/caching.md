# Caching

ReadonlyREST can interact with external systems like LDAP or HTTP service. Calls to these external systems should be limited due to factors such as network latency, potential service outages, and the associated costs, necessitating efficient use of resources and minimizing dependencies on external entities. These problems can be mitigated by a cache. 

## Why caching is important in ReadonlyREST

To answer that question let's remind ourselves how ReadonlyREST's ACL works. 

```yaml
readonlyrest:

  access_control_rules:  
  - name: "Block 1"
    [...]
    ldap_authentication:
      name: ldap1

  - name: "Block 2"
    [...]
    ldap_authentication:
      name: ldap1
      
  - name: "Block 3"
    [...]
    ldap_authentication:
      name: ldap1

  ldaps:
  - name: ldap1
    [...]
```

When someone calls `GET /my_index/_search`, ReadonlyREST handles it by checking block by block if a block is allowed. The processing is stopped when the first block is matched or no block is matched. In the worst-case scenario, all blocks can be checked. As we see, each blocks has an `ldap_auth` rule. It means that each block generates a call to LDAP which can be described as: `Check using LDAP connector "ldap1" if the user XYZ exists and can be authenticated`. It's worth noticing that each call looks exactly the same. 

## Cache levels in ReadonlyREST

The solution to the problem described above is caching. Currently, in ReadonlyREST, there is implemented in-memory cache. It means that the cache entries exist in the memory of the node. The cache is invalidated using a time-based strategy (after TTL expires, entries are deleted). The cache is also invalidated after node restart.

We can define two levels of cache:
1. service client definition level cache
2. rule level cache

```yaml
readonlyrest:

  access_control_rules:  
  - name: "Block 1"
    [...]
    ldap_authentication:
      name: ldap1
      cache_ttl: 10 sec # rule level caching

  - name: "Block 2"
    [...]
    ldap_authentication:
      name: ldap1
      cache_ttl: 20 sec # rule level caching

  - name: "Block 3"
    [...]
    ldap_authentication:
      name: ldap1
      cache_ttl: 30 sec # rule level caching

  ldaps:
  - name: ldap1
    [...]
    cache_ttl: 60 sec # service client definition level cache
```

