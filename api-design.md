# API Design

## URI

Following REST API conventions are followed for Resource URIs:

* Operations for an entity are available through the Resource URI for a collection `.../api/<version>/entities. Example of Resource URI is .../api/v1/users.`
* Plural of the entity name is used as the collection name. Example collection name for user is `users`.
* Trailing forward slash is not used in the endpoint URI. Example use `.../api/v1/databases instead of .../api/v1/databases/.`
* Resource URI for an instance of an entity is .../api/v1/databases/{id}. UUID is typically used as an identifier instead of Entity name to ensure a stable and unambiguous URI that points to a unique instance of an entity.

## Resource representation

* REST API call returns a response with Resources in JSON Content-Type and Content-Length header that includes length of the response.
* All responses will include the Resource ID field even though the URI entities/{id} already provided by the client in the request to simplify consumption of response at the client.
* Entity names and field names use camelCase per Javascript naming convention.
* All resources include an attribute called href with Resource URI. All relationships of an entity will also include href link to the related resource for easy access.
* Unknown fields sent by the client in API requests are not ignored to ensure the data sent by the client is not dropped at the server without user being aware of it.

## HTTP methods

Following HTTP methods are supported for CRUD operations. HTTP response codes are used per REST API conventions in responses.

### GET

#### Listing Entities

Entities are available as resource collections. Example of a resources collection for Table entity is shown below:

```text
GET /v1/tables

200 OK
{
  “data” : [
    {
      “id”: “123e4567-e89b-42d3-a456-556642440000”
      “name”: “dim_user”,
      “documentation” : “This table has user information...”
    },
    {
      “id”: “4333e4567-e89b-42d3-a456-556642440000”
      “name”: “fact_sales”,
      “documentation” : “This table has sales information...”
    }
    ...
  ]
}
```

In the array of entities returned in the response, the entity may contain only a subset of fields to reduce the payload size. Use fields query parameter with a list of field names to get additional data.

#### Listing entities with only necessary fields

Entities can be listed with fields required by the client using fields query parameter as shown below. Some fields may be included by default whether fields specifies them or not \(example - id and name fields below\):

```text
GET /v1/tables?fields=id,name,columns

200 OK
{
  “data” : [
    {
      “id”: “123e4567-e89b-42d3-a456-556642440000”
      “name”: “dim_user”,
      “columns” : {
         “column1” : {...}
         “column2” : {...}
      }
    },
    {
      “id”: “4333e4567-e89b-42d3-a456-556642440000”
      “name”: “fact_sales”,
      “columns” : {
         “column1” : {...}
         “column2” : {...}
      }
    }
    ...
  ]
}

```

_**TODO: Sort, Pagination**  
**TODO: Relationships returned as EntityReference**_  


#### Getting an entity

```text
GET /v1/tables/123e4567-e89b-42d3-a456-556642440000

200 OK
{
  “id”: “123e4567-e89b-42d3-a456-556642440000”
  “name”: “dim_user”,
  “documentation” : “This table has user information...”
  “columns” : [
    “column1”: {
      ...
    },
    “column2”: {
      ...
    }
    ...
  ]
  ...
}
```

The entity in response may contain only a subset of fields to reduce payload. Use fields query parameter with list of field names to get the required entity fields.

**Getting entities with only necessary fields**

An entity can be read with only required fields as shown below using fields query parameter. Some fields may be included by default whether fields specifies them or not \(example - id and name fields below\):

```text
GET /v1/tables/123e4567-e89b-42d3-a456-556642440000
  ?fields=id,name,documentation

200 OK
{
  “id”: “123e4567-e89b-42d3-a456-556642440000”
  “name”: “dim_user”,
  “documentation” : “This table has user information...”
}
```

**Getting an entity by name**

Using an identifier to identify a resource is a stable and unambiguous way of accessing the resource. Additionally all resources support searching a resource by fully-qualified-name as shown below. These URLs are not stable and may not remain valid if the name of the entity changes.

```text
GET /v1/tables?name=service.user.dim_user

{
  “data” : [
    “id”: “123e4567-e89b-42d3-a456-556642440000”
    “name”: “dim_user”,
    “documentation” : “This table has user information...”
    “columns” : [
      “column1”: {
        ...
      },
      “column2”: {
        ...
      }
      ...
    ]
    …
  ]
}
```

### **POST**

**HTTP POST method is used for creating new entities.**

```text
POST http://localhost:8585/api/v1/users
{
  “name”: “user@domain.com”
}

201 Created
content-length: 151
content-type: application/json
{
  "id": "6feb5287-f3c5-457f-86ae-95bcfb82e867",
  "name": "user@domain.com",
  "href": "http://localhost:8585/api/v1/users/6feb5287-f3c5-457f-86ae-95bcfb82e867"
}
```

* POST request might take a request object different from the entity object returned in the response to keep the APIs simple.
* Required fields in the request object are clearly marked in the JSON schema.
* When an entity is created, 201 Created response is returned along with Entity data as JSON content.

### PUT

PUT request is used for updating an entity or create an entity when it does not exist.

```text
PUT http://localhost:8585/api/v1/users

{
  “name”: “user@domain.com”
}

201 Created
content-length: 151
content-type: application/json
{
  "id": "6feb5287-f3c5-457f-86ae-95bcfb82e867",
  "name": "user@domain.com",
  "href": "http://localhost:8585/api/v1/users/6feb5287-f3c5-457f-86ae-95bcfb82e867"
}
```

* PUT request might take a request object different from the entity object returned in the response to keep the APIs simple.
* Required fields in the request object are clearly marked in the JSON schema.
* When an entity is created, 201 Created response is returned and if the entity already exists, the entity is replaced based on the PUT request and 200 OK response is returned. Both responses include entity data as JSON content.

### PATCH

PATCH request is used for updating an existing entity by sending [JSON patch](http://jsonpatch.com/) document in the request.

```text
PATCH http://localhost:8585/api/v1/users

[
  { "op": "replace", "path": "/displayName", "value": "First Last" },
  { "op": "remove", "path": "/owns/0" }
]


200 OK
{
  "id": "6feb5287-f3c5-457f-86ae-95bcfb82e867",
  "name": "user@domain.com",
  "href": "http://localhost:8585/api/v1/users/6feb5287-f3c5-457f-86ae-95bcfb82e867",
  “displayName” : “First Last”
}
```

* Client first gets Entity using GET request. The fields are then updated with the new values. JSON patch is generated by diffing the original and the updated JSON documents.
* JSON diff is sent using PUT request.
* When diff is successfully applied on the server, 200 OK response is returned along with the updated entity data as content.

### DELETE

DELETE request is used for deleting an existing entity. On successful deletion, the server returns 200 OK response.

```text
DELETE http://localhost:8585/api/v1/users/6feb5287-f3c5-457f-86ae-95bcfb82e867


200 OK
```

### Server Implementation Notes

We use [Dropwizard](https://www.dropwizard.io/en/latest/) Java framework for developing Restful web services. APIs are documented using [Swagger/OpenAPI 3.x](https://swagger.io/specification/). We take schema first approach and define metadata entities and types in [JSON schema](https://json-schema.org/) specification version [Draft-07 to 2019-09](https://json-schema.org/draft/2019-09/release-notes.html). Java code is generated from the JSON schema using [JSON schema 2 pojo](https://www.jsonschema2pojo.org/) tool and Python code is generated using the [Data model code generator](https://github.com/koxudaxi/datamodel-code-generator) tool.

