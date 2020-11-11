## FLS engine

Applicable in context of `fields` [rule](elasticsearch.md#fields) 
 
FLS engine specifies how ROR handles field level security internally. Previously FLS was based entirely on [Lucene](https://en.wikipedia.org/wiki/Apache_Lucene) - that's why ROR needed to be installed on all nodes to make `fields` rule work properly.
Now `fields` rule is more flexible and part of FLS responsibilities is handled solely by ES. Increasing ES usage and reducing Lucene exploitation in FLS implementation makes rule more efficient.

Unfortunately, a few FLS functionalities still have to be handled at Lucene level, and cannot benefit of the new ES level implementation (see supported at ES level [requests](#ES limitations) )
Lucene is still used by `fields` rule when ES is not able to handle request properly (as kind of a fallback).

### Configuration 

FLS engine can be configured with global, optional property `fls_engine` set under `readonlyrest:` section. 

There are two engines available:   

* **es_with_lucene** (default)

Default hybrid approach - major part of FLS is handled by ES. Corner cases are passed to Lucene. 
This solution handles all requests properly being more performant than old full Lucene based approach.

 **⚠️IMPORTANT** As Lucene is part of this engine, ReadonlyREST plugin still needs to be installed  in all the cluster nodes that contain data.

* **es**

FLS is handled only by ES, without fallback to Lucene. When ES is not able to handle FLS properly, `fields` rule is not matched. 
In `es` engine FLS is not available for some type of requests (requirements listed below). Major advantage of this approach is not relying on Lucene, so ROR doesn't need to be installed on all nodes.
If lack of full FLS support is unacceptable and all type of requests needs to be handled properly (rule matching, no rejection) it's advised to use more reliable `es_with_lucene` engine.

### ES limitations
Supported by `es` fls engine requests are: 

* all Get/MGet API requests
* Search/MSearch/AsyncSearch API requests with following restrictions:
    * not using script fields
    * used query is one of:
        * common terms
        * match bool
        * match
        * match phrase
        * match phrase prefix
        * exists
        * fuzzy 
        * prefix
        * range
        * regexp
        * term
        * wildcard
        * terms set
        * bool
        * boosting
        * constant score
        * dis max
    * defined query is not using wildcards in field names
    * defined compound queries using only listed above supported queries as inner queries    

If the request doesn't meet above requirements (e.g. it's using `query_string` or script fields), the `es` engine would reject it.

Example configuration (ROR using `es` fls engine):

 ```yaml
readonlyrest:
  
  fls_engine: "es"
  
  access_control_rules:

    - name: "user_using_fields"
      auth_key: user:pass
      fields: ["~someNotAllowedField"]
 ```

Property `fls_engine` can be omitted, then by default ROR uses `es_with_lucene` fls engine. 