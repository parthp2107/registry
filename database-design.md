# Database Design

One of the important goals of OpenMetadata is to ensure it is able to handle the varied set of requirements that diverse types of organizations from different industries bring. Further each organization may also need additional customization in storing org specific metadata. Following types of extensibility is key to address these needs:

1. Ability to add new entities - it must be easy to add new entities to the metadata system.
2. Ability to extend existing entities - it must be easy to add new attributes and relationships to an existing entity.
3. Extensibility must not require complicated database upgrades.
4. Extensibility must not break backward compatibility for existing users.

We use MySQL for storing metadata. In order to ensure extensibility of an entity, the entity data is stored as a JSON document in MySQL JSON column type as shown in the example below. Adding new attributes to an entity does not require altering the schema of the entity table

```text
CREATE TABLE IF NOT EXISTS user_entity (
   id VARCHAR(36) GENERATED ALWAYS AS (json ->> '$.id') STORED NOT NULL,
   name VARCHAR(256) GENERATED ALWAYS AS (json ->> '$.name') NOT NULL,
   json JSON NOT NULL,
   timestamp BIGINT,
   PRIMARY KEY (id),
   UNIQUE KEY unique_name(name)
);
```

MySQL generated columns externalize some entity fields \(see user.id and user.name from the above example\) from the JSON document to easily add table constraints and to support efficient search/filtering. Each entity gets a separate table where it is stored, such as `database_entity, table_entity, dashboard_entity` etc. For more details on the MySQL database schema, see TODO.

In the JSON column, the document stored is based on the JSON schema of an entity. Example of JSON schema for User entity can be found here TODO. All the JSON types, entities, and API request schemas are available here TODO. In the JSON column, only the attribute fields of an entity are stored and relationship fields are constructed on-the-fly based on relationships between entities stored in the relationship table as show below:

```text
CREATE TABLE IF NOT EXISTS relationship (
   fromId VARCHAR(36) NOT NULL,
   toId VARCHAR(36) NOT NULL,
   fromEntity VARCHAR(256) NOT NULL,
   toEntity VARCHAR(256) NOT NULL,
   relation TINYINT NOT NULL,
   timestamp BIGINT,
   INDEX edgeIdx (fromId, toId, relation),
   INDEX fromIdx (fromId, relation),
   INDEX toIdx (toId, relation),
   PRIMARY KEY (fromId, toId, relation)
);
```

Each row in this table captures a relationship from one entity to another entity. Example - fromEntity “team” with fromId “xxx” has relation “contains” toEntity “user” with toId “yyy”.

