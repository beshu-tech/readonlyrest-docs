# Caching

ReadonlyREST can interact with external systems such as LDAP or HTTP services. Calls to these external systems should be limited due to factors such as network latency, potential service outages, and associated costs, requiring efficient use of resources and minimizing dependencies on external entities. These issues can be mitigated by using a cache.

## Why caching is important in ReadonlyREST

To answer this question, let's remember how the ReadonlyREST ACL works.

```yaml
# Example: LDAP authentication rules used in 3 blocks without cache configuration

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

When someone calls `GET /my_index/_search`, ReadonlyREST handles it by checking block by block if a block is allowed. Processing stops when the first block matches, or when no block matches. In the worst case, all blocks can be checked. As we can see, each block has the `ldap_auth' rule. This means that each block generates a call to LDAP which can be described as `Check using LDAP connector "ldap1" if user XYZ exists and can be authenticated`. It's worth noting that every call looks exactly the same. 

## Cache levels in ReadonlyREST

The solution to the problem described above is caching. Currently, there is an in-memory cache implemented in ReadonlyREST. This means that the cache entries exist in the node's memory. The cache is invalidated using a time-based strategy (after the TTL expires, the entries are deleted). The cache is also invalidated after a node restart.

We can define two levels of cache:
1. rule-level cache
2. service client definition level cache

### Rule-level cache

You can think of it as a cache defined for a rule. The cached values are not visible in other rules or blocks. 

```yaml
# Example: LDAP authentication rules used in 3 blocks with cache configured at each LDAP rule level

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
```

How does it work? Let's take the previous example and the `GET /my_index/_search` request. But this time there are two successive requests (for the same user). In the case of the first request, we should see 3 LDAP calls. No caching. Yes, correct, but let's see what happens with the second request. No LDAP call. Yes, this is correct. In the first run, each `ldap_authentication` rule cached the response from LDAP. As mentioned, these are 3 separate caches, so there were 3 LDAP calls. But for the second request, the cached values were used, so there was no further call to LDAP. Caching works.

In the example above, we can see that the first rule has a cache TTL of `10 sec', the second rule has `20 sec' and the last rule has `30 sec'. What will happen if we make another request 10 seconds after the first one? We will see ONE request to LDAP. Why is that? Because the `ldap_authentication` cache entry from block `Block 1` has expired. The corresponding cache entries in the other two rules (from blocks `Block 2` and `Block 3`) are still valid, so the cached values are used in their cases. 

### Service client definition level cache

This cache is defined at the service client level. In our example, it's the LDAP connector definition "ldap1". This cache is visible to all rules that use the service on which it's defined. 

```yaml
# Example: LDAP authentication rules used in 3 blocks with cache configured at LDAP connector level (service definition level)

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
    cache_ttl: 60 sec # service client definition level cache
```

How does it work? Let's call a familiar query: `GET /my_index/_search`. What do we see? Just an LDAP call. The `ldap_authentication' from the `Block 1' block generated the request to LDAP. The other two `ldap_authentication` rules did not. 

The curious might ask: "This is great! Why bother with the rule-level cache at all? Yes, right - this type of cache would be sufficient in most scenarios. But remember that you also have the other one. Maybe it will be useful in your specific case! Also, you can use both levels at the same time.

## What do we cache in ReadonlyREST?

We cache the following information:
1. Can the user be authenticated?
2. What are the user's groups?

The first information only can be cached by the following rules:
* `ldap_authentication`
* `external_authentication`
  
The second information only can be cached by the following rules:
* `ldap_authorization`
* `groups_provider_authorization`

Both information can be cached by:
* `ldap_auth` rule

### Group caching

Group caching looks simple. In general, it is - we get the user's groups from e.g. LDAP and store the information in the cache. But with server-side group filtering* enabled, it's not so simple. Why is that? Let's look at the following example:

```yaml
# Example: LDAP auth rules used in 3 blocks with cache configured at LDAP connector level (service definition level) and the LDAP has configured the server-side group filtering

readonlyrest:

  access_control_rules:  
  - name: "Block 1"
    [...]
    ldap_auth:
      name: ldap1
      groups: [group1_*]

  - name: "Block 2"
    [...]
    ldap_auth:
      name: ldap1
      groups: [group2_*]

  - name: "Block 3"
    [...]
    ldap_auth:
      name: ldap1
      groups: [group3_*]

  ldaps:
  - name: ldap1
    [...]
    sever_side_groups_filtering: true
    cache_ttl: 60 sec # service client definition level cache
```

This example is a slightly modified version of the previous ones:
* we use the `ldap_auth` rule (authentication + authorization) instead of the `ldap_authentication` rule:
  * the block `Block 1` matches only when the user can be authenticated in LDAP and they have groups that match `group1_*` wildcard
  * the block `Block 2` matches only when the user can be authenticated in LDAP and they have groups that match `group2_*` wildcard
  * the block `Block 3` matches only when the user can be authenticated in LDAP and they have groups that match `group3_*` wildcard
* the LDAP connector "ldap1" has the server-side group filtering feature enabled

Let's call `GET /my_index/_search` and see what happens. We see 3 calls to LDAP. Why is that? Because the service level cache was configured?! Yes, but this is because of the server-side group filtering. Let's describe what each block LDAP group call looked like:

* the block `Block 1` - "find all groups for the user that start with `group1_`"
* the block `Block 2` - "find all groups for the user that start with `group2_`"
* the block `Block 3` - "find all groups for the user that start with `group3_`"

Oh yes. It should be obvious now. We don't get all user groups, just a subset of all existing groups. We do cache the information, but each block can't find the information it's looking for in the cache. So we call LDAP in each block. 

Is it bad or not? It depends. On the business case, of course. In some cases it's better to enable server-side rendering (e.g. when the LDAP groups are a really long list), but in other cases it's better to reduce LDAP calls (so more data can be sent over the network). In ROR, server-side group filtering is disabled by default. You should consider whether it should be enabled in your case. But be aware of the caching implications described above.

_\* Currently, only the LDAP connector supports server-side group filtering. Interested in this feature in other services? Let us know!_

## Summary

ReadonlyREST doesn't enable caching by default. We leave that decision up to you. But in most cases, caching should be enabled to reduce external service calls. It's very important when you look at it from the perspective of how the ROR's ACL handles the request. And the rule of thumb is that caching at the service client definition level would be sufficient. 