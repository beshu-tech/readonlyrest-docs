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

---

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

> FORBIDDEN by default req={ ID:193441275-173645661#8, TYP:PutComposableIndexTemplateAction$Request, CGR:N/A, USR:dev1 (attempted), BRS:true, KDX:null, ACT:indices:admin/index_template/put, OA:127.0.0.1/32, XFF:null, DA:127.0.0.1/32, `IDX:index*,dev2_index,dev1_index`, MET:PUT,  `PTH:/_index_template/test`, CNT:<OMITTED, LENGTH=157.0 B> , HDR:Accept=*/*, Authorization=<OMITTED>, Content-Length=157, Content-Type=application/json, Host=127.0.0.1:9200, User-Agent=curl/7.64.1, HIS:[CONTAINER ADMIN-> RULES:[auth_key->false] RESOLVED:[indices=index*,dev2_index,dev1_index;template=ADD(test:index*:dev2_index,dev1_index)]], `[dev1 block-> RULES:[auth_key->true, indices->false]` RESOLVED:[user=dev1;indices=index*,dev2_index,dev1_index;template=ADD(test:index*:dev2_index,dev1_index)]], [dev2 block-> RULES:[auth_key->false] RESOLVED:[indices=index*,dev2_index,dev1_index;template=ADD(test:index*:dev2_index,dev1_index)]], }

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

  {
    "acknowledged" : true
  }
```

Hooray! The index template was added. This time ROR allowed us to do so. It's because `dev1` user has an access to index pattern `idev1_test*`, because it is contained in `idev1_*`. Used aliases are also allowed. 

</details>

---
### Modify an index template

The request will be allowed when all of following conditions are met:
* a template with requested name does exist,
* all index patterns of the existing template are allowed,
* all aliases of the existing template are allowed,
* all index patterns of the requested template are allowed,
* all aliases of the requested template are allowed.

<details>
  <summary>Example (click to expand)</summary>

Let's assume the user `dev1` would like to modify the previously created template, because the index pattern is too detailed:

```text
$ curl -vk -u dev1:test -XPUT "http://127.0.0.1:9200/_index_template/test?pretty" -H "Content-Type: application/json" -d \
  '{
     "index_patterns":["idev*"],
     "template": {
       "aliases": {
         "idev1": {},
         "idev1_test": {}
       }
     }
   }'

  HTTP/1.1 403 Forbidden
  content-type: application/json; charset=UTF-8

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

Ups! Something is wrong. Let's check the ROR forbidden log:

> FORBIDDEN by default req={ ID:918326057-1726421783#75, TYP:PutComposableIndexTemplateAction$Request, CGR:N/A, USR:dev1 (attempted), BRS:true, KDX:null, ACT:indices:admin/index_template/put, OA:127.0.0.1/32, XFF:null, DA:127.0.0.1/32, `IDX:idev*,idev1,idev1_test`, MET:PUT, PTH:/_index_template/test, CNT:<OMITTED, LENGTH=151.0 B> , HDR:Accept=*/*, Authorization=<OMITTED>, Content-Length=151, Content-Type=application/json, Host=127.0.0.1:9200, User-Agent=curl/7.64.1, HIS:[CONTAINER ADMIN-> RULES:[auth_key->false] RESOLVED:[indices=idev*,idev1,idev1_test;template=ADD(test:idev*:idev1,idev1_test)]], `[dev1 block-> RULES:[auth_key->true, indices->false]` RESOLVED:[user=dev1;indices=idev*,idev1,idev1_test;template=ADD(test:idev*:idev1,idev1_test)]], [dev2 block-> RULES:[auth_key->false] RESOLVED:[indices=idev*,idev1,idev1_test;template=ADD(test:idev*:idev1,idev1_test)]], }

We can see that `indices` rule hasn't not been matched. Looking at the IDX section, we can figure out that the index pattern we requested `idev*`, cannot be allowed. `idev*` is too generic, because in the `indices` list we have `["idev1", "idev1_*"]`. Let's try to fix that:

```text
$ curl -vk -u dev1:test -XPUT "http://127.0.0.1:9200/_index_template/test?pretty" -H "Content-Type: application/json" -d \
  '{
     "index_patterns":["idev1_*"],
     "template": {
       "aliases": {
         "idev1": {},
         "idev1_test": {}
       }
     }
   }'

  HTTP/1.1 200 OK
  content-type: application/json; charset=UTF-8

  {
    "acknowledged" : true
  }
```

Yeah, now it works. Let's check if the template is modified (we will use `admin` user to do so):

```text
$ curl -vk -u admin:admin "http://127.0.0.1:9200/_index_template?pretty"

  HTTP/1.1 200 OK
  content-type: application/json; charset=UTF-8

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

All is good. We have only one template and the modifications was applied.

So far, so good. But we can wonder what happens if `dev2` will try to modify (or override) template `temp`? Let's check:

```text
$ curl -vk -u dev2:test -XPUT "http://127.0.0.1:9200/_index_template/test?pretty" -H "Content-Type: application/json" -d \
  '{
     "index_patterns":["idev2_*"],
     "template": {
       "aliases": {
         "idev2": {},
         "idev2_test": {}
       }
     }
   }'

  HTTP/1.1 403 Forbidden
  content-type: application/json; charset=UTF-8

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

Yes! This is something what we wanted like to see. Even if the request was correct and the user `dev2` has an access to the requested index pattern and aliases, the request was forbidden. Obviously, there is already existed template `temp` which has the index pattern and aliases, which are not allowed for `dev2`. ROR deduces that `dev2` cannot be considered as an owner of the template, so it forbade to modify/overwrite it. 

Pretty awesome. Won't `dev2` also be able to remove it? We'll see in next section ...

</details>

---
### Delete an index template

The request will be allowed when template does not exist OR all of the following conditions are met:
* a template with requested name does exist,
* all index patterns of the existing template are allowed,
* all aliases of the existing template are allowed.

<details>
  <summary>Example (click to expand)</summary>

In the last section we wondered, if ROR will be able to block removing the template `temp` by the user `dev2`. Let's recall, that we proved that the user is not able to modify this template, because ROR considers that he isn't an owner of the template. 

```text
$ curl -vk -u dev2:test -XDELETE "http://127.0.0.1:9200/_index_template/test?pretty"

  HTTP/1.1 403 Forbidden
  content-type: application/json; charset=UTF-8

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

Perfect! OK, but we also would like to know if `user1` will be able to remove his template. Let's check it:

```text
$ curl -vk -u dev1:test -XDELETE "http://127.0.0.1:9200/_index_template/test?pretty"

  HTTP/1.1 200 OK
  content-type: application/json; charset=UTF-8

  {
    "acknowledged" : true
  }
```

Great! Everything works.

</details>

---
### Get index templates

At the moment ROR doesn't have `templates` rule (similar to
`snapshots` or `repositories`), which allows to restrict which
templates can be visible to the user (it is going to change in the
future). But the `indices` rule is enough to filter templates based
on index patterns in their definitions. An index template is
considered to be visible for a user, when the user has access to 
AT LEAST ONE index pattern of the template's index pattern list. 
ROR is going to show the template but to hide the information about not allowed index patterns and not allowed aliases.


<details>
  <summary>Example (click to expand)</summary>

</details>

In previous sections we proved that ROR gets along with index templates adding, modifying and removing pretty well. Now, we'd like check what index templates are supposed to be visible for users. Let's assume we have 4 index templates:

```text
$ curl -vk -u admin:admin "http://127.0.0.1:9200/_index_template?pretty"

  HTTP/1.1 200 OK
  content-type: application/json; charset=UTF-8

  {
    "index_templates" : [
      {
        "name" : "t1",
        "index_template" : {
          "index_patterns" : ["i*"],
          "template" : {
            "aliases" : {
              "idev2" : { },
              "idev3" : { },
              "idev1" : { }
            }
          },
          "composed_of" : [ ]
        }
      },
      {
        "name" : "t2",
        "index_template" : {
          "index_patterns" : ["idev1_*"],
          "template" : {
            "aliases" : {
              "admin_idev" : { },
              "idev1" : { }
            }
          },
          "composed_of" : [ ],
          "priority" : 1
        }
      },
      {
        "name" : "t3",
        "index_template" : {
          "index_patterns" : ["idev2_*"],
          "template" : {
            "aliases" : {
              "idev2" : { },
              "admin_idev" : { }
            }
          },
          "composed_of" : [ ],
          "priority" : 1
        }
      },
      {
        "name" : "t4",
        "index_template" : {
          "index_patterns" : ["idev1_*", "idev2_*"],
          "template" : {
            "aliases" : {
              "idev2" : { },
              "admin_idev" : { },
              "idev1" : { }
            }
          },
          "composed_of" : [ ],
          "priority" : 2
        }
      }
    ]
  }
```

`admin` has unrestricted access to all templates. Now, let's check which templates `dev` are supposed to see:

```text
$ curl -vk -u dev1:test "http://127.0.0.1:9200/_index_template?pretty"

  HTTP/1.1 200 OK
  content-type: application/json; charset=UTF-8

  {
    "index_templates" : [
      {
        "name" : "t1",
        "index_template" : {
          "index_patterns" : ["i*"],
          "template" : {
            "aliases" : {
              "idev1" : { }
            }
          },
          "composed_of" : [ ]
        }
      },
      {
        "name" : "t2",
        "index_template" : {
          "index_patterns" : ["idev1_*"],
          "template" : {
            "aliases" : {
              "idev1" : { }
            }
          },
          "composed_of" : [ ],
          "priority" : 1
        }
      },
      {
        "name" : "t4",
        "index_template" : {
          "index_patterns" : ["idev1_*"],
          "template" : {
            "aliases" : {
              "idev1" : { }
            }
          },
          "composed_of" : [ ],
          "priority" : 2
        }
      }
    ]
  }
```

Hmm, we can see many weird things here. Let's start with the simplest case: index template `t2` is allowed for the user, because the used index pattern is allowed by `indices` rule. But we can also see that user `dev1` is not aware of existence the `admin_idev` alias - it was filter out from the aliases list. The user has no access to the alias, so he should not be able to see it. 

What about the index template `t3`? `dev1` is not allowed to see it because the index pattern `idev2_*` is not allowed for him. It was also pretty much obvious!

The next is `t4`. When `admin` had listed index templates, we saw that template `t4` has 2 index patterns. But `dev1` can see only one. This is great, because he has an access to a part of that template, so he definitely should be able to see it. ROR behaviour here is pretty neat - it allows the user to see a template with filtered, not allowed parts of it, but at the same time, the user is not treated by ROR as an owner of the template - he won't be able to eg. remove it (Don't believe me? Go ahead and check!)   

And the last one to explain - `t1`. The index pattern of the template is `i*`. Obviously user `dev1` has no access to it, because his allowed indices are `idev1, idev1_*`. But if we imagine all possible values generated from pattern `i*` and all possible values generated from `idev1, idev1_*`, we can notice that the latter will be a subset of the first. It means that this template can be interesting for the user `dev1`, because it will ba applied to indices created by him. That's why ROR decides to show it. 

---
## Component templates

Component templates doesn't have index patterns but could have aliases. So, in this case, we should also consider four  types of template related requests:

---
### Create a component template

The request will be allowed when all of following conditions are met:
* a template with requested name does not exist (if it does, it's rather a template modification, than a creation),
* all aliases of the new, requested template are allowed.

---
### Modify a component template

The request will be allowed when all of following conditions are met:
* a template with requested name does exist,
* all aliases of the existing template are allowed,
* all aliases of the requested template are allowed.
  
---
### Delete a component template

The request will be allowed when template does not exist OR all of the following conditions are met:
* a template with requested name does exist,
* all aliases of the existing template are allowed.
  
---
### Get component templates

At the moment there is no way to restrict which component templates can be visible to the user (it is going to change in the
future - see a corresponding index template section). But ROR is 
going to hide the information about not allowed aliases of returned
component templates.

---
## Troubleshooting 

To figure out why the template is not returned or/and cannot be altered, you should [enable a DEBUG log level](../elasticsearch.md#Troubleshooting) and check your logs. ROR logs each step of template request handling in the `indices` rule, so detailed description should explain the given template is not allowed.