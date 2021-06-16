# Schema Modeling

Open Metadata takes a schema first approach modeling Entities and Types in the system to establish a consistent vocabulary for metadata. We use [JSON schema](https://json-schema.org/) for modeling the metadata as it offers several advantages:

* Makes it easy to describe the structure of data with readable documentation that is both human and machine consumable.
* Models can include constraints, such as required/optional fields, default values, allowed values that not only serve automated validation but also as documentation.
* Common types are developed once and can be reused as building blocks in many different schemas and become the basis of vocabulary development.
* Supports a rich set of tools for generating code and validation in various languages, reducing the manual boilerplate coding.
* Support for rich formats help to convert schema types into native standard types during code generation, such as URI, date and time. 

## References

1. [JSON schema](https://json-schema.org/) specification version [Draft-07 to 2019-09](https://json-schema.org/draft/2019-09/release-notes.html)
2. [JSON schema 2 pojo](https://www.jsonschema2pojo.org/) tool used for Java code generation from JSON schema
3. [Data model code generator](https://github.com/koxudaxi/datamodel-code-generator) for generating python code from JSON schema

## **Types**

JSON schema supports many native types - null, boolean, object, array, number, and string. In addition, to develop clear and consistent vocabulary, domain specific reusable types are defined ranging from simple types, such as UUID, timestamp, and email to more complex object types, such as Tags, Ownership, and Usage.

## **Entity**

An Entity is a special Type that represents an object that is either real or conceptual with an identity. An entity can be related to another entity through relationships. An Entity has two types of Fields - Attributes and Relationships:

Attributes represent an Entity’s data. Entities MUST include an attribute called ID that uniquely identifies an instance of an entity. An entity might optionally include a human-readable fullyQualitifedName attribute that uniquely identifies the entity. An attribute of an entity MUST not be another Entity and should be captured through relationship. Entities typically SHOULD have the following common attributes:

| **id** | name | fullyQualifiedName | displayName | description | owner | href |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Mandatory attribute of type UUID that identifies the entity instance | Name of the entity \(example database name\). For some entities, the name may uniquely identify an entity. | Unique human readable name that identifies an entity \(and attributes of an entity\) that is formed using all the names in the hierarchy above the given entity. Example - databaseService.database.table. | Optional name used for display purposes. For example name could be “[john.smith@domain.com](mailto:john.smith@domain.com)” and displayName could be “John Smith” | Description of the entity instance. Not all entities need description. For example, User entity might not need a description and just the name of the user might suffice. A Database entity needs description to provide details of what is stored in the database, when to use it, and other information on how to use it. | Optional attribute used to capture the ownership information. Not all entities have ownership information \(for example User, Team and Organization\). | An attribute generated on the fly as part of API response to provide URL link to the entity returned. |

  
**Relationship** captures information about association of an Entity with another Entity. Relationships can have cardinality - One-to-one, One-to-many, Many-to-one, and Many-to-many. Example of relationships:

* One-to-one: A Table is owned by a User.
* One to Many: a Database contains multiple Tables.
* Many-to-Many: A User belongs to multiple Teams. A Team has multiple Users.

All relationships are captured using the EntityReference type.  


Following is an example of a JSON schema of the User entity with attributes id, displayName, and email. User entity has one-to-many relationships to another entity TEam \(user is member of multiple teams\).

```text
{
  "title": "User entity",
  "type": "object",

  "properties" : {
    "id": {
      "description": "Unique identifier for instance of a User",
      "$ref": "#/definitions/uuid"
    },
    "displayName": {
      "description": "Name used for display purposes. Example 'John Smith'",
      "type" : "string"
    },
    "email": {
      "description": "User's Email",
      "type": "string"
    },
   "teams" : {
      "description": "Teams that this user belongs to",
      "type": "array",
      "items" :{
        "$ref": "#/definitions/entityReference"
      }
   }
  }
}
```

## Metadata system entities

Metadata system has the following core entities:

1. **Data Entities** - These entities represent data, such as databases, tables, and topics, and assets created using data, such as Dashboards, Reports, Metrics, and ML Features. It also includes entities such as Pipelines that are used for creating data assets.
2. **Services** - Services represent platforms and services used for storing and processing data. It includes Online Data Stores, Data Warehouses, ETL tools, Dashboard services etc.
3. **Users & Teams** - These entities represent users within an organization and teams that they are organized under.
4. **Activities** - These entities are related to feeds, posts, and notifications for collaboration between users.
5. **Glossary and Tags** - Entities for defining business glossary that includes hierarchical tags.

