# Templates handling in `indices` rule

<details>
  <summary>A ROR configuration for all examples below (click to expand)</summary>

  ```yaml
    readonlyrest:
      prompt_for_basic_auth: false

      access_control_rules:
        - name: "admin block"
          verbosity: error
          type: allow
          auth_key: admin:admin

        - name: "dev1 block"
          indices: ["idev1", "idev1_*"]
          auth_key: dev1:test

        - name: "dev2 block"
          indices: ["idev2", "idev2_*"]
          auth_key: dev2:test

  ```
</details>
<br>

## Index templates

An `indices` rule takes into consideration index patterns and aliases which are a part of a template definition. We should consider four types of template related requests:

### Create an index template

The request will be allowed when all of following conditions are met:
* a template with requested name does not exist (if it does, it's rather a template modification, than a creation),
* all index patterns of the new, requested template are allowed,
* all aliases of the new, requested template are allowed.

<details>
  <summary>Example (click to expand)</summary>

Let's try to add an index template. We can see, using `admin` account, that
there are no templates defined yet.

```text
$ curl -vk -u admin:admin "http://localhost:9200/_index_template?pretty"
  HTTP/1.1 200 OK
  content-type: application/json; charset=UTF-8
  content-length: 30

  {
    "index_templates" : [ ]
  }
```

Now, let's use `dev1` user account to create an index template `temp1`:

```text
$ curl -vk -u dev1:test -XPUT "http://127.0.0.1:9200/_index_template/test?pretty" -H "Content-Type: application/json" -d \
  '{
     "index_patterns":["index*"],
     "template": {
       "aliases": { 
         "dev1_index": {},
         "dev2_index": {}
       }
     }
  }'

  HTTP/1.1 403 Forbidden
  content-type: application/json; charset=UTF-8
  content-length: 264
  {
    "error" : {
      "root_cause" : [
        {
          "reason" : "forbidden",
          "due_to" : ["OPERATION_NOT_ALLOWED"]
        }
      ],
      "reason" : "forbidden",
      "due_to" : ["OPERATION_NOT_ALLOWED"],
      "status" : 403
    }
  }
```

Oh, something went wrong. It seems that, a user `dev1` is not allowed to add this template. Let's check ROR logs to figure out why:

> FORBIDDEN by default req={ ID:193441275-173645661#8, TYP:PutComposableIndexTemplateAction$Request, CGR:N/A, USR:dev1 (attempted), BRS:true, KDX:null, ACT:indices:admin/index_template/put, OA:127.0.0.1/32, XFF:null, DA:127.0.0.1/32, <span style="color:crimson">IDX:index*,dev2_index,dev1_index</span>, MET:PUT, <span style="color:crimson">PTH:/_index_template/test</span>, CNT:<OMITTED, LENGTH=157.0 B> , HDR:Accept=*/*, Authorization=<OMITTED>, Content-Length=157, Content-Type=application/json, Host=127.0.0.1:9200, User-Agent=curl/7.64.1, HIS:[CONTAINER ADMIN-> RULES:[auth_key->false] RESOLVED:[indices=index*,dev2_index,dev1_index;template=ADD(test:index*:dev2_index,dev1_index)]], <span style="color:crimson">[dev1 block-> RULES:[auth_key->true, indices->false]</span> RESOLVED:[user=dev1;indices=index*,dev2_index,dev1_index;template=ADD(test:index*:dev2_index,dev1_index)]], [dev2 block-> RULES:[auth_key->false] RESOLVED:[indices=index*,dev2_index,dev1_index;template=ADD(test:index*:dev2_index,dev1_index)]], }

We can see that our request was forbidden - credentials were OK, but `indices` rule was not matched in `dev1 block`. We can see also that ROR found 3 indices which are related to the request: 
* `index*` - an index pattern from our request
* `dev1_index` - a first alias from out request
* `dev2_index` - a second alias from out request

When we take a look at indices configured in `indices` rule for our user, we 
can see that, he has an access only to `idev1` and `idev1_*` indices. Now, 
it's pretty much obvious why the request was blocked - the user has no access to index pattern and aliases used in the request. Let's try to fix that:

```text
$ curl -vk -u dev1:test -XPUT "http://127.0.0.1:9200/_index_template/test?pretty" -H "Content-Type: application/json" -d \
  '{
     "index_patterns":["idev1_test*"],
     "template": {
       "aliases": { 
         "idev1": {},
         "idev1_test": {}
       }
     }
  }'

  HTTP/1.1 200 OK
  content-type: application/json; charset=UTF-8
  content-length: 28

  {
    "acknowledged" : true
  }
```

Hooray! The index template was added. This time ROR allowed us to do so. It's because `dev1` user has an access to index pattern `idev1_test*`, because it is contained in `idev1_*`. Used aliases are also allowed. 

</details>

### Modify an index template

The request will be allowed when all of following conditions are met:
* a template with requested name does exist,
* all index patterns of the existing template are allowed,
* all aliases of the existing template are allowed,
* all index patterns of the requested template are allowed,
* all aliases of the requested template are allowed.

### Delete an index template

The request will be allowed when template does not exist OR all of the following conditions are met:
* a template with requested name does exist,
* all index patterns of the existing template are allowed,
* all aliases of the existing template are allowed.

### Get index templates

At the moment ROR doesn't have `templates` rule (similar to
`snapshots` or `repositories`), which allows to restrict which
templates can be visible to the user (it is going to change in the
future). But the `indices` rule is enough to filter templates based
on index patterns in their definitions. An index template is
considered to be visible for a user, when the user has access to 
AT LEAST ONE index pattern of the template's index pattern list. 
ROR is going to show the template but to hide the information about not allowed index patterns and not allowed aliases.

## Component templates

Component templates doesn't have index patterns but could have aliases. So, in this case, we should also consider four  types of template related requests:

### Create a component template

The request will be allowed when all of following conditions are met:
* a template with requested name does not exist (if it does, it's rather a template modification, than a creation),
* all aliases of the new, requested template are allowed.

### Modify a component template

The request will be allowed when all of following conditions are met:
* a template with requested name does exist,
* all aliases of the existing template are allowed,
* all aliases of the requested template are allowed.
  
### Delete a component template

The request will be allowed when template does not exist OR all of the following conditions are met:
* a template with requested name does exist,
* all aliases of the existing template are allowed.
  
### Get component templates

At the moment there is no way to restrict which component templates can be visible to the user (it is going to change in the
future - see a corresponding index template section). But ROR is 
going to hide the information about not allowed aliases of returned
component templates.

## Troubleshooting 

To figure out why the template is not returned or/and cannot be altered, you should [enable a DEBUG log level](../elasticsearch.md#Troubleshooting) and check your logs. ROR logs each step of template request handling in the `indices` rule, so detailed description should explain the given template is not allowed.