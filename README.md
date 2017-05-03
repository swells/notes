# Python Client Library Package `mldeploy`

## Introduction

July 2017, we will be working to introduce a Python Client Library `mldeploy`
that is comparable in functionality with the existing `mrsdeploy` R package for 
model operationalize. Focus will be given to produce a light-weight progressive 
set of APIs that are crafted for flexibility, readability, and a low learning 
curve.

## Out of Scope

- Remote execution functionality
- Realtime Scoring functionality (we will design for it)
- `mldeploy` Python package install and distribution mechanism (No R Client equivalent) 

## Assumptions

- Knowledge of the R package `mrsdeploy` 
- General intent is functional symmetry with the R package `mrsdeploy` 
- `AML VNext` implies Azure Machine Learning next latest stable version

## Requirments

### Environments and Ecosystem

The `mldeploy` library will integrate and run successfully in both python 
Applications as scripts, interactive REPL, in addition to IPython/Jupyter data science notebooks.
A productive workflow must be maintained within these ecosystems such that a user can:

- Rapidly and iteratively operationalize models
- Easy to consume and test deployed models 
- Remove the learning curve of swagger from the data scientist

### Authentication

The `mldeploy` library will make it easy for python applications and
IPython/Jupyter data science notebook users to authenticate against 
Active Directory and Azure Active Directory (AAD) in order to access 
protected web resources.

To use the functions in the mldeploy library, you must log into the ML Server as an 
authenticated user. Users have the following authentication options:

- Active Directory
- Azure Active Directory (AAD)

To fully support AML VNext and ML Server in an uniform manner, expanding and normalizing the AAD 
context to acquire authentication tokens will be necessary:

- Acquire Token with _Client Credentials_
- Acquire Token with _Username and Password_
- Acquire Token with _Client Certificate_
- Acquire Token with _Refresh Token_
- Acquire Token with _Device Code_
- Acquire Token with _Authorization Code_

[See authentication API details](#authentication-api)

### AML VNext flexibility

All public APIS should apply to both ML Server and AML VNext. This will require abstractions
around: (Authentication, Service Management, and Service Consumption)
to allow for the correct implementation details to be provided while keeping a
common interfaces. In addition, this package should be constructed in a conventional 
fashion that makes it easy to be included as a dependency inside other packages (important for future CLI functionality).

### Service API Parity with `mrsdeploy` 

Full symmetry for service management and consumption operations will 
be implemented in python, this to include: 

`publishService()`

**Arguments**

- `name` string 
- `code` string/ R file/ R function 
- `model` R object/ R file
- `snapshot` string 
- `inputs` list of param name and type 
- `outputs` list of param name and type
- `artifacts` string array 
- `v` string 
- `alias` string NULL 

`updateService()`

**Arguments**

- `name` string 
- `v` string
- `code` string/ R file/ R function 
- `model` R object/ R file
- `snapshot` string 
- `inputs` list of param name and type 
- `outputs` list of param name and type
- `artifacts` string array 
- `v` string 
- `alias` string NULL 

`getService()`

**Arguments**

- `name` string
- `v` string (optional)

`deleteService()`

**Arguments**

- `name` string
- `v` string (optional)

`listServices()` 

**Arguments**

- `name` string (optional)
- `v` string (optional)

See [functions](#service-consumption-api) for more details.

### Specifying packages

> API design subject to change pending the larger package design review.

Service creators will be able to define python package dependencies during
service deployment and redeployment operations. Support on the API will allow 
for the following:

- Specifying a _package repository source destination
- Specifying the option to install the latest version of a package (default)
- Specifying the option to install a specific version of a package
- Specifying the option to install from a `pip-freeze` txt file (AML Vnext flexibility)
- Flexibility and convenience to define dependencies via package module references

> **note** 1. Can/Should we be using both pip and conda? 2. Should there be some flexibility with $pip freeze?

[See packages API details](#packages-api)

## Encoding Types

Supported service I/O encoding will have comparable type support as in R.

## Package Structure

The `mldeploy` library will be laid out as a Python package in a conventional 
fashion in order to foster ease of use:

- Easy to install with `pip` or `easy_install`? or from `git`
- Easy to specify as a dependency for another package
- Easy for other users to download and run tests
- Easy for other users to work on and have immediate familiar with the basic directory structure
- Easy to add and distribute documentation

## Security

A review will be scheduled with the security team upon sign off.

## Testing

Testing will follow our existing conventions and restrictions to help drive 
quality. This will include unit-testing, integration-testing, and strong code 
coverage with gated checkins.

- [pytest](https://docs.pytest.org/en/latest/) to run our tests and for expectations
- [pytest-cov](https://pypi.python.org/pypi/pytest-cov) for test coverage
- [Mock](https://pypi.python.org/pypi/mock) and [pytest-mock](https://pypi.python.org/pypi/pytest-mock) for mocking
- [tox](https://pypi.python.org/pypi/tox) to run tests against all the supported python versions

> **note** Do we need something like `tox` given what ShipR supports?

## Documentation

> **Pending** based on internal convention accept since Python docstrings can be written following several different formats.

Documentation will be supplied in both the package as well as more verbose
overview and examples on MSDN and github. All public API functions must be fully documented, including a README, and support for ` help(package)` during an import.

## Open Source Dependencies 

The latest stable version dependencies will be used unless.

### Runtime

- [requests](http://docs.python-requests.org/en/master/)
- [adal](https://github.com/AzureAD/azure-activedirectory-library-for-python)

### Development

- [pytest](https://docs.pytest.org/en/latest/)
- [pytest-cov](https://pypi.python.org/pypi/pytest-cov)
- [Mock](https://pypi.python.org/pypi/mock and [pytest-mock](https://pypi.python.org/pypi/pytest-mock) 
- [tox](https://pypi.python.org/pypi/tox)

## ShipR Integration

All development and runtime dependencies will be stored in a similar fashion
as R dependencies within their respective Python `Packages-OSS` and `Packages-NonOSS`
repository locations. All versions will be shrink-wrapped and frozen.

## Full API Overview

```python
from mldeploy import MlDeploy, AuthenticationContext
from sklearn import datasets
import numpy as np

# --- Create authenticated session via LDAP/AD ---
context = AuthenticationContext()
ml = MlDeploy('url', auth=context.active_directory('username', 'password'))

## --- Prepare for publishing ---
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

# --- Or Publish service via `kwargs` dictionary (alternative equivalent) ---
kwargs = { 
    'version': '1.0.1', 
    'code_fn': [ add_one, init ]    
    'objects': [ local_obj ], 
    'inputs': { 'x', 'float' },
    'outputs': { 'answer', 'float' },
    'packages': [ 'pandas==0.18.0', 'sklearn', np ],
    'description': 'The Description of the `add-one` service, accepts _markdown_.' 
}

ml.deploy_service('add-one', **kargs)

# --- Discover service by `name` or `name` and optional `version` ---
service = ml.get_service('add-one')

# consume/test the service (raw response)
res = service.add_one(5)

print res.output('answer')

# --- service management ---
services = ml.list_services('add-one') # all versions of service `add-one`
service = ml.get_service('add-one', version = '1.0.1') # get `v1.0.1` otherwise latest
ml.delete_service('add-one')

# --- service update and re-deploy ---
ml.service('add-one')
   .version('1.0.1')
   .description('Update the description field.')
   .redeploy()

# --- Or service update via `kwargs` dictionary (alternative equivalent) ---
kwargs = { 
    'version': '1.0.1', 
    'description': 'Update the description field.' 
}
ml.redeploy_service('add-one', **kargs)
```

## Import:

Before beginning building and managing a service, be sure to import the `mldeploy` package:

```python
from mldeploy import MlDeploy, AuthenticationContext
```

## Supported Functions and Objects

### `AuthenticationContext`

```python
import mldeploy
from mldeploy import MLDeploy, AuthenticationContext

context = AuthenticationContext()
auth = context.active_directory('username', 'password')
auth = context.azure_active_directory_client_credentials('authuri', 'tenant', 'resource', 'clientid')
auth = context.azure_active_directory_username_password('authuri', 'tenant', 'resource', 'clientid', 'username', 'password')
auth = context.azure_active_directory_refresh_token('authuri', 'tenant', 'resource',  'clientid', 'refresh_token')
auth = context.azure_active_directory_client_certificate('authuri', 'tenant', 'resource', 'clientid', 'privateKeyFile', 'privateKeyFile')
auth = context.azure_active_directory_device_code('authuri', 'tenant', 'resource', 'clientid')
auth = context.azure_active_directory_authorization_code()

# -- Create authenticated service session --
ml = MLDeploy('staging-env-url', auth=auth)
```

### `MLDeploy`

Used for authentication and global configurations such as logging and environment setting. The `MlDeploy` is a factory for an authenticated session returning an `ml` object to be used for service management and consumption.

`MLDeploy` extends `BaseOperationalization`:

```
BaseOperationalization
 + deploy_service(name: str, **kargs): Service
 + redeploy_service(name: str, **kargs): Service
 + get_service(name: str, version:None): Service
 + list_services(name:None, version:None): list
 + delete_service(name: str, version:None): bool
 + service(name: str): ServiceDefinition
 + logging(on: bool): void
 + environment(url: str, auth: Authentication): void

```

Example:

```python
import mldeploy
from mldeploy import MLDeploy, AzureML, AuthenticationContext

ml = MLDeploy('production-env-url', auth=auth) 
ml = AzureML('production-env-url', auth=auth) 
```
**note** Does Azure ML have the notion of a service `version`? 


### Discover/Get a Service

`ml.get_service(name: str, version=None)`

### Delete a Service

`ml.delete_service(name: str, version:None)`

### List Services

`ml.list(name=None, version=None)`

List the different published web services. The service name and service version are optional. This call allows you to retrieve service information regarding:

1. All services published
2. All versioned services for a specific named service
3. A specific version for a named service

### Publish Service

##### Fluent API

`ml.service(name: str).deploy())`

```
ServiceDefinition
+ __init__(self: ServiceDefinition, name: str) -> this
+ version(version: string) -> this
+ code_fn(add_one, init) ->  this
+ code_str(code: str, init: str) -> this
+ inputs(name_type: dict) -> this
+ outputs(name_type: dict) -> this
+ input(name: str, type: str) -> this
+ output(name: str, type: str) -> this
+ objects(objects) -> this
+ object(object) -> this
+ packages(packages: list) -> this
+ package(package: str) -> this
+ artifacts(filenames: list) -> this
+ alias(operation: str) -> this
+ description(description: script) -> this
+ deploy() -> Service
+ redeploy() -> Service
```

##### Non-Fluent equivalent

`ml.deploy_service(name: str, **kargs)`

#### Publish basics

The fluent APIS designed for optional configurations where the readability of the invocation is close to that of the ordinary written prose (grammatical structure).

A `publish` or `update` can be initiated by invoking the appropriate method on the `ml.service(name)` or `ml.service(name, version)` object, then calling `.deploy()` to send the request.

For example, a simple `publish` request:

Here we are publishing a service named `add-one` that is comprised of a block of code `answer = x + 1`. It accepts `inputs` named x of type float and returns `outputs` of type float. Finishing the configuration, we call `deploy` to indicate we are done and to deploy the service.

```py
ml.service('add-one')
   .code_str('answer = x + 1')
   .inputs({ 'x': 'float' })
   .outputs({ 'answer': 'float' })
   .deploy()
```

### Update Service

##### Fluent API

```python
ml.service(name: str).redeploy()
```

```
ServiceDefinition
+ __init__(self: ServiceDefinition, name: str) -> this
+ version(version: string) -> this
+ code_fn(add_one, init) ->  this
+ code_str(code: str, init: str) -> this
+ inputs(name_type: dict) -> this
+ outputs(name_type: dict) -> this
+ input(name: str, type: str) -> this
+ output(name: str, type: str) -> this
+ objects(objects) -> this
+ object(object) -> this
+ packages(packages: list) -> this
+ package(package: str) -> this
+ artifacts(filenames: list) -> this
+ alias(operation: str) -> this
+ description(description: script) -> this
+ deploy() -> Service
+ redeploy() -> Service
```

##### Non-Fluent equivalent

`ml.redeploy_service(name: str, **kargs)`

#### Update basics

The fluent APIS designed for optional configurations where the readability of the invocation is close to that of the ordinary written prose (grammatical structure).

For example, we are `updating` an existing `service` named `add-one/v1` with a new `description` and `deploying` it again.

```py

ml.service('add-one')
  .version('v1')
  .description('The updated description for the `add-one/v1` service')
  .redeploy()
```

## Package API

Packages can be installed during service publishing in one of two ways:

1. Collectively using the `.packages(['pkg, 'pkg'])` function
2. Individually using the `.package('pkg')` function

```python
# 1. --- Add packages dependencies collectively --

ml.service('add-one')
   .code_fn(add_one)
   .packages(['pandas==0.15.2', 'numpy'])
   .deploy()

# 2. --- Add packages dependencies individually ---

ml.service('add-one')
   .code_fn(add_one)
   .package('pandas==0.15.2')
   .package('numpy')
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
