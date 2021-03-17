# Templates handling in `indices` rule
<br>

## Index templates

An `indices` rule takes into consideration index patterns and aliases which are a part of a template definition. We should consider four types of template related requests:

### Create an index template

The request will be allowed when all of following conditions are met:
* a template with requested name does not exist (if it does, it's rather a template modification, than a creation),
* all index patterns of the new, requested template are allowed,
* all aliases of the new, requested template are allowed.

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