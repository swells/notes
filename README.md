# Python Client Library Package `mldeploy`


```python
# -*- coding: utf-8; tab-width: 4; indent-tabs-mode: t; python-indent: 4 -*-

"""
Role
====

`Operationalization` is designed to be a low-level abstract foundation class 
from which other service operationalization attribute classes in the `mldeploy` 
package can be derived. It provides a standard template for creating
attribute-based operationalization lifecycle phases providing a consistent 
`__init()__`, `__del()__` sequence that chains initialization (initializer),
authentication (authentication), and destruction (destructor) methods for the 
class hierarchy.
"""

import six

# python 2 and python 3 compatibility library
from six import add_metaclass
from abc import ABCMeta, abstractmethod


@add_metaclass(ABCMeta)
class Operationalization(object):

    def initializer(self, api_client, config):
        """
        Init lifecycle method, invoked during construction. Sets up attributes 
        and invokes initializers for the class hierarchy.

        An optional _noonp_ method where subclass implementers MAY provide this 
        method definition by overriding.

        Object with configuration property name/value pairs
		"""        
        pass
        
    def destructor(self):
        """
        Destroy lifecycle method. Invokes destructors for the class hierarchy.

        An optional _noonp_ method where subclass implementers MAY provide this 
        method definition by overriding.
        """        
        pass
        
    def authentication(self, context):
        """
        Authentication lifecycle method. Invokes the authentication enterypoint
        for the class hierarchy.        

        An optional _noonp_ method where subclass implementers MAY provide this 
        method definition by overriding.
        """         
        pass
        
    @abstractmethod
    def get_service(self, name):
        """Retrieve service meta-data from the name source and return an new 
        service instance.
        """

    @abstractmethod
    def list_services(self, name=None, version=None):
        """
        return a list of service meta-data
        """

    @abstractmethod
    def delete_service(self, name):
        """
        return True|False
        """     

    @abstractmethod
    def deploy_service(self, name, **kwargs):
        """
        return a new service instance.        
        """

    @abstractmethod
    def redeploy_service(self, name, **kwargs):
        """
        return a new service instance.        
        """
        
    def service(self, name):
        """
        return a `OperationalizationDefinition` for fluent API.
        """        
        return OperationalizationDefinition(name, self)   


```








  * [Introduction](#introduction)
  * [Out of Scope](#out-of-scope)
  * [Assumptions](#assumptions)
  * [Requirments (P1)](#requirments-p1)
    * [Environments and Ecosystem](#environments-and-ecosystem)
    * [Authentication](#authentication)
    * [AML VNext flexibility](#aml-vnext-flexibility)
    * [Service API Parity with `mrsdeploy`](#service-api-parity-with-mrsdeploy)
    * [Specifying packages](#specifying-packages)
  * [Encoding Types](#encoding-types)
  * [Package Structure](#package-structure)
  * [Security](#security)
  * [Testing](#testing)
  * [Documentation](#documentation)
  * [Open Source Dependencies](#open-source-dependencies)
    * [Runtime](#runtime)
    * [Development](#development)
  * [ShipR Integration](#shipr-integration)
  * [Full API Overview](#full-api-overview)
  * [Import](#import)
  * [Supported Functions and Objects](#supported-functions-and-objects)
    * [`MLDeploy`](#mldeploy)
    * [`ServiceDefinition`](#servicedefinition)
    * [Discover/Get a Service](#discoverget-a-service)
    * [Delete a Service](#delete-a-service)
    * [List Services](#list-services)
    * [Publish Service](#publish-service)
    * [Update Service](#update-service)
  * [Errors](#errors)
    * [Validation Errors](#validation-errors)
    * [Runtime Errors](#runtime-errors)
  * [Package API](#package-api)
  * [Highlevel API to Swagger mapping (MLS)](#highlevel-api-to-swagger-mapping-mls)
  * [Public API and Object Matrix](#public-api-and-object-matrix)

## Introduction

July 2017, we will be working to introduce a Python Client Library `mldeploy`
that is comparable in functionality with the existing `mrsdeploy` R package for 
model operationalization. Focus will be given to produce a light-weight progressive 
set of APIs that are crafted for flexibility, readability, and a low learning 
curve.

## Out of Scope

- Python Remote execution functionality
- Python Realtime Scoring functionality (we will design for it)
- The July 2017 depreciation of the `mrsdeploy` R package in place of the `mldeploy` R equivalent
- `mldeploy` Python package install and distribution mechanism (No R Client equivalent) 

## Assumptions

- Knowledge of the R package `mrsdeploy` 
- General intent is functional symmetry with the R package `mrsdeploy` 
- `AML VNext` implies Azure Machine Learning next latest stable version

## Requirments (P1)

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

To use the functions in the `mldeploy` library, you must log into the ML Server as an 
authenticated user. Users have the following authentication options:

- Active Directory
- Azure Active Directory (AAD)

There will be no authentication User Interface (UI) prompts as in `mrsdeploy`under `remoteLogin()`.
The authentication UI prompts are intrinsically a _remote-execution_ characteristic.
The motivation for the divergence is to further articulate that _remote-execution_ and _services_ 
are truly separate pieces of functionality that work independently of one another as well as together.

### AML VNext flexibility

All public APIS should apply to both ML Server and AML VNext. This will require abstractions
around: (Authentication, Service Management, and Service Consumption)
to allow for the correct implementation details to be provided while keeping a
common interfaces. In addition, this package should be constructed in a conventional 
fashion that makes it easy to be included as a dependency inside other packages (important for future CLI functionality).

### Service API Parity with `mrsdeploy` 

Full symmetry for service management and consumption operations will 
be implemented in python, this to include: 

> `publishService()`

#### Arguments

- `name` string 
- `code` string/ R file/ R function 
- `model` R object/ R file (optional)
- `snapshot` string (optional)
- `inputs` list of param name and type (optional)
- `outputs` list of param name and type (optional)
- `artifacts` string array (optional)
- `v` string (optional)
- `alias` string (optional)

> `updateService()`

**Arguments**

- `name` string 
- `code` string/ R file/ R function 
- `model` R object/ R file (optional)
- `snapshot` string (optional)
- `inputs` list of param name and type (optional)
- `outputs` list of param name and type (optional)
- `artifacts` string array (optional)
- `v` string (optional)
- `alias` string (optional)

> `getService()`

**Arguments**

- `name` string
- `v` string (optional)

> `deleteService()`

**Arguments**

- `name` string
- `v` string (optional)

> `listServices()` 

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

> **Note** 1. Are we using both pip and conda. Should there be some flexibility with `$pip freeze`?

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

> **Note** Do we need something like `tox` given what ShipR supports?

## Documentation

> **Pending** based on internal convention agreement since Python docstrings can be written following several different formats.

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
- [Mock](https://pypi.python.org/pypi/mock) and [pytest-mock](https://pypi.python.org/pypi/pytest-mock) 
- [tox](https://pypi.python.org/pypi/tox)

## ShipR Integration

All development and runtime dependencies will be stored in a similar fashion
as R dependencies within their respective Python `Packages-OSS` and `Packages-NonOSS`
repository locations. All versions will be shrink-wrapped and frozen.

## Full API Overview

```python
from mldeploy import MLDeploy
from sklearn import datasets
import numpy as np

# --- Create authenticated session via LDAP/AD ---
ml = MLDeploy('http://localhost:12800', auth=('username', 'password'))

## --- Prepare for publishing ---
local_obj = 'Local Object'
model = 'Model'

def init:
   # Any objects defined here are evaluated first and are accessible in `code`
   init_obj = 'Initialized Object'

def add_one(x):
   print init_obj
   print local_obj
   print model
   return x + 1

# --- Publish service via fluent API ---
ml.service('add-one')
   .version('1.0.1')
   .code_fn(add_one, init)
   .inputs({ 'x': 'float' }),
   .outputs({ 'answer': 'float' }),
   .models([ model ]),
   .objects([ local_obj ])
   .packages([ 'pandas==0.18.0', 'sklearn', np ])
   .artifacts([ 'histogram.png' ])
   .description('The Description of the `add-one` service, accepts _markdown_.')
   .deploy() 

# --- Or Publish service via `kwargs` dictionary (alternative equivalent) ---
kwargs = { 
    'version': '1.0.1', 
    'code_fn': [ add_one, init ]    
    'objects': [ local_obj ],
    'models': [ model ],
    'inputs': { 'x': 'float' },
    'outputs': { 'answer': 'float' },
    'packages': [ 'pandas==0.18.0', 'sklearn', np ],
    'artifacts' : [ 'histogram.png' ],
    'description': 'The Description of the `add-one` service, accepts _markdown_.' 
}

ml.deploy_service('add-one', **kwargs)

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
ml.redeploy_service('add-one', **kwargs)
```

## Import

Before beginning building and managing a service, be sure to import the `mldeploy` package:

```python
from mldeploy import MlDeploy, AuthenticationContext
```
## Authentication

To use the functions in the `mldeploy` library, you must log into the ML Server as an 
authenticated user. Users have the following authentication options that are represented 
as a tuple of immutable strings:

- Active Directory:  `('username', 'pasword')`
- Azure Active Directory (AAD): `('username', 'password', 'authuri', 'tenant', 'resource', 'clientid')`

Example:

```python
import mldeploy
from mldeploy import MLDeploy

# --- Active Directory ---
auth = ('username', 'password')
ml = MLDeploy('env-url', auth=auth) 

# --- Azure Active Directory ---
auth = ('username', 'password', 'authuri', 'tenant', 'resource', 'clientid')
ml = MLDeploy('env-url', auth=auth) 
```

## Supported Functions and Objects

### `MLDeploy`

Used for authentication and global configurations such as logging and environment setting. The `MlDeploy` is a factory for an authenticated session returning an `ml` object to be used for service management and consumption.

> `MLDeploy` extends `BaseOperationalization`

```
BaseOperationalization
 + deploy_service(name: str, **kwargs) -> Service
 + redeploy_service(name: str, **kwargs) -> Service
 + get_service(name: str, version=None) -> Service
 + list_services(name:None, version=None) -> list
 + delete_service(name: str, version=None) -> bool
 + service(name: str) -> ServiceDefinition
 + logging(on: bool) -> void
 + environment(url: str, auth: tuple) -> void
```

#### Arguments

- url:str
- auth:tuple
- logging=None (bool) Default is `False`

Example:

```python
import mldeploy
from mldeploy import MLDeploy

# --- Active Directory ---
auth = ('username', 'password')
ml = MLDeploy('url', auth=auth)

# --- Azure Active Directory ---
auth = ('username', 'password', 'authuri', 'tenant', 'resource', 'clientid')
ml = MLDeploy('url', auth=auth, logging=True)
```

### `ServiceDefinition`

```
+ __init__(self: ServiceDefinition, name: str) -> this
+ version(version: string) -> this
+ code_fn(add_one, init=None) ->  this
+ code_str(code: str, init=None) -> this
+ inputs(name_type: dict) -> this
+ outputs(name_type: dict) -> this
+ input(name: str, type: str) -> this
+ output(name: str, type: str) -> this
+ objects(objects: tuple) -> this
+ object(object: Object) -> this
+ model(model: object) -> this
+ models(models: tuple) -> this
+ packages(packages: tuple) -> this
+ package(package: str) -> this
+ artifacts(filenames: list) -> this
+ alias(operation: str) -> this
+ description(description: str) -> this
+ deploy() -> Service
+ redeploy() -> Service
```

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
+ code_fn(add_one, init=None) ->  this
+ code_str(code: str, init=None) -> this
+ inputs(name_type: dict) -> this
+ outputs(name_type: dict) -> this
+ input(name: str, type: str) -> this
+ output(name: str, type: str) -> this
+ objects(objects) -> this
+ object(object) -> this
+ model(model: object) -> this
+ models(models: list) -> this
+ packages(packages: list) -> this
+ package(package: str) -> this
+ artifacts(filenames: list) -> this
+ alias(operation: str) -> this
+ description(description: str) -> this
+ deploy() -> Service
+ redeploy() -> Service
```

##### Non-Fluent equivalent

`ml.deploy_service(name: str, **kwargs)`

#### Publish basics

The fluent APIS designed for optional configurations where the readability of the invocation is close to that of the ordinary written prose (grammatical structure).

A `publish` can be initiated by invoking the `ml.service(name)` object then calling `.deploy()` to send the request.

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
+ code_fn(add_one, init=None) ->  this
+ code_str(code: str, init=None) -> this
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
+ description(description: str) -> this
+ deploy() -> Service
+ redeploy() -> Service
```

##### Non-Fluent equivalent

`ml.redeploy_service(name: str, **kwargs)`

#### Update basics

The fluent APIS designed for optional configurations where the readability of the invocation is close to that of the ordinary written prose (grammatical structure).

An `update` can be initiated by invoking the `ml.service(name)` object then calling `.redeploy()` to send the request.

For example, we are `updating` an existing `service` named `add-one/v1` with a new `description` and `deploying` it again.

```py

ml.service('add-one')
  .version('v1')
  .description('The updated description for the `add-one/v1` service')
  .redeploy()
```

## Errors

There are two classifications of error scenarios:

1. [Validation Errors](#validation-errors)
2. [Request Errors](#request-errors)

### Validation errors 

Validation errors are used to ensure that only valid data is sent on a request. These errors will be raised when user provided values are incorrect and will always display within the context of the action.

### Request errors

All errors will be managed and realized as simple HTTP errors. All HTTP `4xx` and `5xx` responses will be considered an error by default. 
Errors raised on the request will be internally caught, normalized, formatted, and returned.

Example:
```python
 # --- Internal error handling ---
 try:
    response = send('url', 'method', 'data')

    # Consider any status other than 2xx an error
    if not response.status_code // 100 == 2:
       return "Error: Unexpected response {}".format(response)

except requests.exceptions.RequestException as e:
        return "Error: {}".format(e)
```

Request errors will breakdown into two categories:

1. [Service management errors](#service-management-errors)
2. [Service consumption errors](#service-consumption-errors)

#### Service management errors

Service management errors will be raised for any of the typical HTTP `4xx` and `5xx` error responses during:

- `get_service(name:str, version=None)`
- `delete_service(name:str, version=None)`
- `list_services(name=None, version=None)`
- `service_deploy(name:str, **kwargs)`
- `service_redeploy(name:str, **kwargs)`
- `service(name:str).deploy()`
- `service(name:str).redeploy()`

#### Service consumption errors

Consumption errors are raised during api service invocations and are realized via the `ApiException` class. The intention will be to extend the HTTP `4xx` and `5xx` errors and provide additional information about the request such as: I/O, headers, request, and response payloads.

```python

class ApiException(Exception):

    def __init__(self, status=None, reason=None, http_resp=None):
        if http_resp:
            self.status = http_resp.status
            self.reason = http_resp.reason
            self.body = http_resp.data
            self.headers = http_resp.getheaders()
        else:
            self.status = status
            self.reason = reason
            self.body = None
            self.headers = None

    def __str__(self):
        """
        Custom error messages for exception
        """
        error_message = "({0})\n"\
                        "Reason: {1}\n".format(self.status, self.reason)
        if self.headers:
            error_message += "HTTP response headers: {0}\n".format(self.headers)

        if self.body:
            error_message += "HTTP response body: {0}\n".format(self.body)


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


## Highlevel API to Swagger mapping (MLS)

How **optional** properties map to a service's dynamic swagger definition:

```python
ml.service('add-one')
  .version('1.0.1')
  .inputs({ 'x': 'float' }),
  .outputs({ 'answer': 'float' }),
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

## Public API and Object Matrix

| Public Function                                   | Returns           |
| ------------------------------------------------- |:-----------------:|
| `ml = MLDeploy(url:str, auth:tuple, logging=None)`| MLDeploy          | 
| `ml.deploy_service(name:str, **kwargs)`           | Service           |
| `ml.redeploy_service(name:str, **kwargs)`         | Service           |
| `ml.list_services(name=None, version=None)`       | list              |
| `ml.get_service(name:str, version=None)`          | Service           |
| `ml.delete_service(name:str, version=None)`       | bool              |
| `ml.service(name:str)`       _Fluent API*_        | ServiceDefinition |

***Fluent API:**

``` 
ServiceDefinition
 + version(version: str) -> this
 + code_fn(add_one, init=None) ->  this
 + code_str(code: str, init=None) -> this
 + inputs(name_type: dict) -> this
 + outputs(name_type: dict) -> this
 + input(name: str, type: str) -> this
 + output(name: str, type: str) -> this
 + objects(objects: tuple|list) -> this
 + object(object: Object) -> this
 + model(model: object) -> this
 + models(models: tuple|list) -> this
 + packages(packages: tuple|list) -> this
 + package(package: str) -> this
 + artifacts(filenames: tuple|list) -> this
 + artifact(filenames: str) -> this
 + alias(operation: str) -> this
 + description(description: str) -> this
 + deploy() -> Service
 + redeploy() -> Service
```

****kwargs (equivalent Dictionary):**

Same names:

``` 
kwargs = { 
    'version': '1.0.1', 
    'code_fn': [ add_one, init ],
    'code_str:' ['add-one', 'init'],
    'objects': [ local_obj ],
    'models': [ model ],
    'inputs': { 'x': 'float' },
    'outputs': { 'answer': 'float' },
    'packages': [ 'pandas==0.18.0', 'sklearn', np ],
    'artifacts' : [ 'histogram.png' ],
    'description': 'The Description of the `add-one` service, accepts _markdown_.' 
}
```
