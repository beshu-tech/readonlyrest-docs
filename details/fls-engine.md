# FLS engine

Applicable in the context of the [`fields` rule](elasticsearch.md#fields) 
 
FLS engine specifies how ROR handles field-level security internally. Previously FLS was based entirely on [Lucene](https://en.wikipedia.org/wiki/Apache_Lucene) - that's why ROR needed to be installed on all nodes to make the `fields` rule work properly.
Now the `fields` rule is more flexible and part of FLS responsibilities is handled solely by ES. Increasing ES usage and reducing Lucene exploitation in FLS implementation makes the rule more efficient.

Unfortunately, a few FLS functionalities still have to be handled at the Lucene level, and cannot benefit of the new ES level implementation (see supported at ES level [requests](#ES-limitations) )
Lucene is still used by `fields` rule when ES is not able to handle a request properly (as kind of a fallback).

## Configuration 

FLS engine can be configured with global, optional property `fls_engine` set under the `readonlyrest.global_settings` section. 

There are two engines available:   

* **es_with_lucene** (default)

 **⚠️IMPORTANT** As Lucene is part of this engine, the ReadonlyREST plugin still needs to be installed in all the cluster nodes that contain data.

Default hybrid approach - the major part of FLS is handled by ES. Corner cases are passed to Lucene. 
This solution handles all requests properly being more performant than the old full Lucene-based approach.

* **es**

FLS is handled only by ES, without fallback to Lucene. When ES is not able to handle FLS properly, the `fields` rule is not matched. 
In the `es` engine, FLS is not available for some types of requests (requirements listed below). The major advantage of this approach is to not rely on Lucene, so **ROR doesn't need to be installed on all nodes**.

If a lack of full FLS support is unacceptable and all type of requests needs to be handled properly (rule matching, no rejection) it's advised to use a more reliable `es_with_lucene` engine.

## ES limitations
Supported by `es` FLS engine requests are: 

* all Get/MGet API requests
* Search/MSearch/AsyncSearch API requests with the following restrictions:
    * not using script fields
    * the used query is one of
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
    * the defined query doesn't use wildcards in field names
    * defined compound queries using only listed above supported queries as inner queries 
    * the Search request doesn't use scroll 

If the request doesn't meet above requirements (e.g. it's using `query_string` or script fields), the `es` engine will reject it.

Example configuration (ROR using `es` FLS engine):

 ```yaml
readonlyrest:
  
  global_settings:
    fls_engine: "es"
  
  access_control_rules:

    - name: "user_using_fields"
      auth_key: user:pass
      fields: ["~someNotAllowedField"]
 ```

Property `fls_engine` can be omitted, then by default, ROR uses `es_with_lucene` FLS engine. 