# "Index not found" scenario

Examples:

> Let's assume that our ES cluster has 2 indices: `index_a` and `index_b`. At the same time we have two users: `userA` and `userB`. We'd like to give `userA` access to index `index_a`, and `userB` to `index_b`. `userA` should not see or be even aware of `index_b` and vice versa. We'd like to give each of them a feeling that they are alone on the cluster.
>
> ROR `readonlyrest.yml` configuration may look like this:
>
> ```yaml
> readonlyrest:
>   enable: true
>   access_control_rules:
>
>      - name: "user A indices"
>        indices: ["index_a"]
>        auth_key: userA:secret
>
>      - name: "user B indices"
>        indices: ["indexB"]
>        auth_key: userB:secret
> ```
>
> We can test if `userA` is able to reach `index_a`:
>
> ```text
> $ curl -v -u userA:secret "http://127.0.0.1:9200/index_a?pretty"
>   HTTP/1.1 200 OK
>   content-type: application/json; charset=UTF-8
>   content-length: 611
>    
>   {
>     "index_a" : {
>       "aliases" : { },
>       "mappings" : { ... }
>       "settings" : { ... }
>     }
>   }
> ```
>
> It looks like he is. So far, so good. Let's try to access nonexistent index \(we know, that index with name `nonexistent` for sure doesn't exist on our cluster\):
>
> ```text
> $ curl -i -u userA:secret "http://127.0.0.1:9200/nonexistent?pretty"                                                                                             18:15:28
>   HTTP/1.1 404 Not Found
>   content-type: application/json; charset=UTF-8
>   content-length: 634
>
>   {
>     "error" : {
>       "root_cause" : [ ... ],
>       "type" : "index_not_found_exception",
>       "reason" : "no such index [nonexistent_ROR_ZA1FXDsR7M]",
>       "resource.type" : "index_or_alias",
>       "resource.id" : "nonexistent_ROR_ZA1FXDsR7M",
>       "index_uuid" : "_na_",
>       "index" : "nonexistent_ROR_ZA1FXDsR7M"
>     },
>     "status" : 404
>  }
> ```
>
> The response is pretty straight forward - the index doesn't exist. But, let's see what happens, when the same user, `userA`, will try to get `index_b`:
>
> ```text
> $ curl -v -u userA:secret "http://127.0.0.1:9200/index_b?pretty"
>   HTTP/1.1 404 Not Found
>   content-type: application/json; charset=UTF-8
>   content-length: 610
>  
>   {
>     "error" : {
>       "root_cause" : [ ... ],
>       "type" : "index_not_found_exception",
>       "reason" : "no such index [index_b_ROR_QcskliAl8A]",
>       "resource.type" : "index_or_alias",
>       "resource.id" : "index_b_ROR_QcskliAl8A",
>       "index_uuid" : "_na_",
>       "index" : "index_b_ROR_QcskliAl8A"
>     },
>     "status" : 404
>   }
> ```
>
> As we can see `userA` is not able to get `index_b`. But the response is HTTP 404 Not Found - it means that the index doesn't exist.
>
> So, the response is the same as we get if the called index really doesn't exist. Thanks to the described behaviour, `userA` is not aware that on the cluster there are any other indices but the ones he was given access to.
>
> > note:
> >
> > Careful reader may notice that, in example above, `userA` was getting `index_b`, but the response says that there is no `index_b_ROR_QcskliAl8Aindex_b_ROR_QcskliAl8A` index. It's the trick ROR does to fool ES and be sure that asking index, which the user should not be allowed to see, won't be reached by him.
>
> But we should also consider the other case - using an index name with wildcard. So, `userA` will try to get all indices which names match `index*` pattern:
>
> ```text
> $ curl -i -u userA:secret "http://127.0.0.1:9200/index*?pretty"                                                                                                  19:58:29
>   HTTP/1.1 200 OK
>   content-type: application/json; charset=UTF-8
>   content-length: 611
>   
>   {
>     "index_a" : {
>       "aliases" : { },
>       "mappings" : { ... },
>        "settings" : { ... }
>      }
>    }
> ```
>
> Response is exactly like we'd expect - only `index_a` was returned. But what if nothing matches our index name pattern?
>
> ```text
> $ curl -i -u userA:secret "http://127.0.0.1:9200/index_userA*?pretty"                                                                                                20:05:10
>   HTTP/1.1 200 OK
>   content-type: application/json; charset=UTF-8
>   content-length: 4
>
>   { }
> ```
>
> Response is empty list. Now, let's see what happens when an index name pattern matches an index which is not authorized for a user who asks about it.
>
> ```text
> $ curl -i -u userA:secret "http://127.0.0.1:9200/index_b*?pretty"                                                                                                20:14:34
>   HTTP/1.1 200 OK
>   content-type: application/json; charset=UTF-8
>   content-length: 4
>
>   { }
> ```
>
> As we see, response is the same as we have experienced when there was really no index matching the pattern. Also here a user has a feeling that only his indices are present on a cluster.