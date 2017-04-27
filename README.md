# Opinionated MRS perspective

## Use-case Scenarios

Data Scientist working in Python/IPython/Notebooks:

## Goals

From Python/IPython/Notebooks:

- Rapidly and iteratively operationalize models 
- Easy to consume and test deployed models simplified workflow
- Ability to apply service management (list, get/discover, delete, update)
- Remove the learning cure of swagger

## API Overview

The Python client library is a light-weight progressive fluent API crafted for flexibility, readability, and a low learning curve. 

```python
from mldeploy import MlDeploy, AzureActiveDirectory 
from sklearn import datasets
import numpy as np

# Authentication LDAP  
ml = MlDeploy('url', auth=('username', "password')) 
 
# Authentication AAD 
auth = AzureActiveDirectory('username', 'tenantid', 'clientid', 'resource') 
# --- or authenticate via `username` and `apikey` :: auth = AzureActiveDirectory('username', 'apikey')
ml = MlDeploy('url', auth=auth) 

## -- Prepare for publishing --
local_obj = 'Local Object'

def init:
   # Any objects defined here are evaluated first and are accessible in `code`
   init_obj = 'Initialized Object'

def add_one(x):
   print init_obj
   print local_obj
   return x + 1

# --- Publish service via fluent API ---
ml.service('add-one')
   .version('1.0.1')
   .code_fn(add_one, init)
   .inputs({ 'x', 'float' }),
   .outputs({ 'answer', 'float' }),
   .objects([ local_obj ])
   .packages([ 'pandas==0.18.0', 'sklearn', np ])
   .description('The Description of the `add-one` service, accepts _markdown_.')
   .deploy()

# --- Publish service via `kwargs` dictionary (alternative equivalent) ---
kwargs = { 
    'version': '1.0.1', 
    'code_fn': [ add_one, init ]    
    'objects': [ local_obj ], 
    'inputs': { 'x', 'float' },
    'outputs': { 'answer', 'float' },
    'packages': [ 'pandas==0.18.0', 'sklearn', np ],
    'description': 'The Description of the `add-one` service, accepts _markdown_.' 
}
ml.service('add-one', **kargs).deploy()
# or perhaps: `ml.deploy_service('add-one', **kargs)

# --- Discover service by `name` or `name` and optional `version` ---
service = ml.get('add-one')

# consume/test the service (raw response)
res = service.add_one(5)

print res.output('answer')

# --- service management ---
services = ml.list('add-one') # all versions
service = ml.get('add-one', version = '1.0.1') # get `v1.0.1` otherwise latest
ml.delete('add-one')

# --- service update and re-deploy ---
ml.service('add-one', '1.0.1')
   .description('Update the description field.')
   .deploy()
```

## Highlevel API to Swagger mapping (MRS)

How **optional** properties map to a service's dynamic swagger definition:

```python
ml.service('add-one')
  .version('1.0.1')
  .inputs({ 'x', 'float' }),
  .outputs({ 'answer', 'float' }),
  .description('The Description of the `add-one` service, accepts _markdown_.')
  .deploy()
```

### Produces a service:

- API: `/api/add-one/1.0.1`
- swagger.json: `/api/add-one/1.0.1/swagger.json`

### `name`

`swagger.info.title`

### `version`

`swagger.info.version`  

### `description`

`swagger.info.description`

### `code_fn` or `alias`

The function name or `alias` map to the swagger's `operationId`

### `inputs` and `outputs`

```json
"definitions":{
   "InputParameters":{
       "type":"object",
          "properties":{
              "x":{
                  "type":"float"
               }
            }
   },
   "OutputParameters":{
      "type":"object",
      "properties":{
           "answer":{
              "type":"float"
            }
       }
   }
}
```

## Full swagger.json upon publishing

> Note: things left out for brevity

```json
{
   "swagger":"2.0",
   "info":{
      "description":"The service description in text or `markdown`",
      "title":"add-one",
      "version":"1.0.0"
   },
   "schemes":[
      "http",
      "https"
   ],
   "securityDefinitions":{
      "Bearer":{
         "type":"apiKey",
         "name":"Authorization",
         "in":"header"
      }
   },
   "tags":[
      {
         "name":"add-one"         
      }
   ],
   "consumes":[
      "application/json"
   ],
   "produces":[
      "application/json"
   ],
   "/api/add-one/1.0.0":{
      "post":{
         "tags":[
            "add-one"
         ],
         "description":"Consume the add-one web service.",
         "operationId":"add_one",
         "parameters":[
            {
               "name":"WebServiceParameters",
               "in":"body",
               "required":true,
               "description":"Input parameters to the web service.",
               "schema":{
                  "$ref":"#/definitions/InputParameters"
               }
            }
         ],
         "responses":{
            "200":{
               "description":"OK",
               "schema":{
                  "$ref":"#/definitions/WebServiceResult"
               }
            }
         }
      },
      "definitions":{
         "InputParameters":{
            "type":"object",
            "properties":{
               "x":{
                  "type":"float"
               }
            }
         },
         "OutputParameters":{
            "type":"object",
            "properties":{
               "answer":{
                  "type":"float"
               }
            }
         }
      }
   }
```
# Opinionated MRS perspective

## Use-case Scenarios

Data Scientist working in Python/IPython/Notebooks:

## Goals

From Python/IPython/Notebooks:

- Rapidly and iteratively operationalize models 
- Easy to consume and test deployed models simplified workflow
- Ability to apply service management (list, get/discover, delete, update)
- Remove the learning cure of swagger

## API Overview

The Python client library is a light-weight progressive fluent API crafted for flexibility, readability, and a low learning curve. 

```python
from mldeploy import MlDeploy, AzureActiveDirectory 
from sklearn import datasets
import numpy as np

# Authentication LDAP  
ml = MlDeploy('url', auth=('username', "password')) 
 
# Authentication AAD 
auth = AzureActiveDirectory('username', 'tenantid', 'clientid', 'resource') 
# --- or authenticate via `username` and `apikey` :: auth = AzureActiveDirectory('username', 'apikey')
ml = MlDeploy('url', auth=auth) 

## -- Prepare for publishing --
local_obj = 'Local Object'

def init:
   # Any objects defined here are evaluated first and are accessible in `code`
   init_obj = 'Initialized Object'

def add_one(x):
   print init_obj
   print local_obj
   return x + 1

# --- Publish service via fluent API ---
ml.service('add-one')
   .version('1.0.1')
   .code_fn(add_one, init)
   .inputs({ 'x', 'float' }),
   .outputs({ 'answer', 'float' }),
   .objects([ local_obj ])
   .packages([ 'pandas==0.18.0', 'sklearn', np ])
   .description('The Description of the `add-one` service, accepts _markdown_.')
   .deploy()

# --- Publish service via `kwargs` dictionary (alternative equivalent) ---
kwargs = { 
    'version': '1.0.1', 
    'code_fn': [ add_one, init ]    
    'objects': [ local_obj ], 
    'inputs': { 'x', 'float' },
    'outputs': { 'answer', 'float' },
    'packages': [ 'pandas==0.18.0', 'sklearn', np ],
    'description': 'The Description of the `add-one` service, accepts _markdown_.' 
}
ml.service('add-one', **kargs).deploy()
# or perhaps: `ml.deploy_service('add-one', **kargs)

# --- Discover service by `name` or `name` and optional `version` ---
service = ml.get('add-one')

# consume/test the service (raw response)
res = service.add_one(5)

print res.output('answer')

# --- service management ---
services = ml.list('add-one') # all versions
service = ml.get('add-one', version = '1.0.1') # get `v1.0.1` otherwise latest
ml.delete('add-one')

# --- service update and re-deploy ---
ml.service('add-one', '1.0.1')
   .description('Update the description field.')
   .deploy()
```

## Highlevel API to Swagger mapping (MRS)

How **optional** properties map to a service's dynamic swagger definition:

```python
ml.service('add-one')
  .version('1.0.1')
  .inputs({ 'x', 'float' }),
  .outputs({ 'answer', 'float' }),
  .description('The Description of the `add-one` service, accepts _markdown_.')
  .deploy()
```

### Produces a service:

- API: `/api/add-one/1.0.1`
- swagger.json: `/api/add-one/1.0.1/swagger.json`

### `name`

`swagger.info.title`

### `version`

`swagger.info.version`  

### `description`

`swagger.info.description`

### `code_fn` or `alias`

The function name or `alias` map to the swagger's `operationId`

### `inputs` and `outputs`

```json
"definitions":{
   "InputParameters":{
       "type":"object",
          "properties":{
              "x":{
                  "type":"float"
               }
            }
   },
   "OutputParameters":{
      "type":"object",
      "properties":{
           "answer":{
              "type":"float"
            }
       }
   }
}
```

## Full swagger.json upon publishing

> Note: things left out for brevity

```json
{
   "swagger":"2.0",
   "info":{
      "description":"The service description in text or `markdown`",
      "title":"add-one",
      "version":"1.0.0"
   },
   "schemes":[
      "http",
      "https"
   ],
   "securityDefinitions":{
      "Bearer":{
         "type":"apiKey",
         "name":"Authorization",
         "in":"header"
      }
   },
   "tags":[
      {
         "name":"add-one"         
      }
   ],
   "consumes":[
      "application/json"
   ],
   "produces":[
      "application/json"
   ],
   "/api/add-one/1.0.0":{
      "post":{
         "tags":[
            "add-one"
         ],
         "description":"Consume the add-one web service.",
         "operationId":"add_one",
         "parameters":[
            {
               "name":"WebServiceParameters",
               "in":"body",
               "required":true,
               "description":"Input parameters to the web service.",
               "schema":{
                  "$ref":"#/definitions/InputParameters"
               }
            }
         ],
         "responses":{
            "200":{
               "description":"OK",
               "schema":{
                  "$ref":"#/definitions/WebServiceResult"
               }
            }
         }
      },
      "definitions":{
         "InputParameters":{
            "type":"object",
            "properties":{
               "x":{
                  "type":"float"
               }
            }
         },
         "OutputParameters":{
            "type":"object",
            "properties":{
               "answer":{
                  "type":"float"
               }
            }
         }
      }
   }
```
