# Azure Cosmos DB Graph and SQL API multi-model functionality

In Azure Cosmos DB, any Gremlin API database account is created with an endpoint that allows for document access using the standard SQL API connectivity libraries. 

## Back-end JSON format

Azure Cosmos DB Gremlin API graph collection stores data in the JSON format in the database engine backend. This data can be accessed using the SQL API clients by just using the documents endpoint as illustrated below:

<img src="https://raw.githubusercontent.com/LuisBosquez/azure-cosmos-db-graph-working-guides/master/res/graph-backend-json-1.jpg">

In the default configuration, the vertex and edge objects are stored as JSON documents in the format specified below.

## Vertices
Vertices are stored as JSON documents that use the following format:
``` 
{
  "label": "person",
  "id": "ben",
  "firstName": [
    {
      "_value": "Ben",
      "id": "bd039f6c-2701-4379-81ec-f79c39ffc0a8"
    }
  ],
  "lastName": [
    {
      "_value": "Miller",
      "id": "35ca1484-cacc-4260-be57-61b732dc958d"
    }
  ],
  "_rid": "zGolAOsmbgADAAAAAAAAAA==",
  "_self": "dbs/zGolAA==/colls/zGolAOsmbgA=/docs/zGolAOsmbgADAAAAAAAAAA==/",
  "_etag": "\"3903d7a5-0000-0000-0000-5a69cdde0000\"",
  "_attachments": "attachments/",
  "_ts": 1516883422
}
```

* These fields are the minimum required properties for every Vertex object.
    * `id`: a key-value pair that denotes the unique ID for every Vertex. The value is enforced to be unique - an attempt to insert another document with the same value will result in an error. If this property is not specified, the database engine will automatically assign a GUID.
    * `label`: a key-value pair that is required by the Tinkerpop specification. The best-practice is to use it to represent an entity type in the graph model. When using the Gremlin language, not specifying a label property will make the database engine automatically assign it a value of 'vertex'.
* Gremlin Vertex properties are stored using the following format:
    ```
    "lastName": [
        {
        "_value": "Miller",
        "id": "35ca1484-cacc-4260-be57-61b732dc958d"
        }
    ]
    ```
    * This is known as a *property bag*. It's a JSON array that includes an object with an `id` and a `_value` properties. 
    * This is defined as the default Apache Tinkerpop compliant vertex property. This allows for multiple values to be stored within a single property.
* The following are system properties – automatically added by Cosmos DB. They shouldn’t be modified manually: `_rid`, `_self`, `_etag`, `_attachments` and `_ts`. 
* For partitioned collections, the partitioning key will be stored as a flat property:
    ```
    "partitionKey": "value"
    ```
    For more information, please visit the [Graph Partitioning topic](https://docs.microsoft.com/en-us/azure/cosmos-db/graph-partitioning).

## Edges
Edges are stored as JSON documents that use the following format:
``` 
{
  "label": "knows",
  "id": "3f7c5e73-cb44-42f2-962e-30121583c179",
  "relationship": "friends",
  "_sink": "target_id",
  "_sinkLabel": "target_label",
  "_vertexId": "source_id", 
  "_vertexLabel": "source_label",
  "partitionKey": "value",
  "_isEdge": true, 
  "_rid": "zGolAOsmbgAHAAAAAAAAAA==",
  "_self": "dbs/zGolAA==/colls/zGolAOsmbgA=/docs/zGolAOsmbgAHAAAAAAAAAA==/",
  "_etag": "\"3903eba5-0000-0000-0000-5a69ce230000\"",
  "_attachments": "attachments/",
  "_ts": 1516883491
}
```
* These fields are the minimum required properties for every Vertex object.
    * `id`: a key-value pair that denotes the unique ID for every Edge. Similarly to the vertex Id, this value is enforced to be unique - an attempt to insert another document with the same value will result in an error. If this property is not specified, the database engine will automatically assign a GUID.
    * `label`: a key-value pair that is required by the Tinkerpop specification. The best-practice is to use it to represent a type of relationship in the graph model. When using the Gremlin language, not specifying a label property will result in an error.
* Edge properties are just stored as a set of flat key-value pairs. Multi-valued properties are not supported for edges.
* Edge relationship metadata is stored in the following way:
    * `_sink`: key-value pair that stores the ID of the **target** Vertex.
    * `_sinkLabel`: key-value pair that stores the label of the **target** Vertex.
    * `_vertexId`: key-value pair that stores the ID of the **source** Vertex.
    * `_vertexLabel`: key-value pair that stores the label of the **source** Vertex.
* For partitioned collections, the Edge object will store the following partition information:
    * `_sinkPartition`: key-value pair that stores the partitioning key value of the **target** Vertex. This will be used by the database engine to locate the exact partition of the target Vertex.
    * `partitionKey`: key-value pair that stores the partitioning key of the **Edge itself**. This property name and value will be inherited from the **source Vertex** object. 
    * For more information, please visit the [Graph Partitioning topic](https://docs.microsoft.com/en-us/azure/cosmos-db/graph-partitioning).
* The following are system properties – automatically added by Cosmos DB. They shouldn’t be modified manually: `_rid`, `_self`, `_etag`, `_attachments` and `_ts`. 

## Caveats
A JSON document that is structured in a way that isn't consistent with the format specified above will result in properties not being recognized by the Gremlin engine. 


# Using Multi-model

After you have retrieved the document endpoint of your graph database (with the format of `****.documents.azure.com`, then you can proceed to issue queries using the [SQL API of Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-query-getting-started).

You can use any of the SDKs of Cosmos DB SQL API:
- [.NET](https://docs.microsoft.com/en-us/azure/cosmos-db/create-sql-api-dotnet#code-examples)
- [Java](https://docs.microsoft.com/en-us/azure/cosmos-db/create-sql-api-java)
- [Node.js](https://docs.microsoft.com/en-us/azure/cosmos-db/create-sql-api-nodejs)
- [Python](https://docs.microsoft.com/en-us/azure/cosmos-db/create-sql-api-python)

Some advantages of using multi-model access are the following:
- Paginated queries: You can take advantage of the pagination feature in Cosmos DB to consume large amounts of data that result from a `SELECT` query in a controlled way. Learn more about [continuation tokens](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.documents.client.feedoptions.requestcontinuation?view=azure-dotnet).
- Change Feed support: You can use [Change Feed](https://docs.microsoft.com/en-us/azure/cosmos-db/change-feed) to execute operations on created and updated documents across all partitions.
- Polyglot database operations: This allows for document operations and graph operations on the same data overall.

## Document queries on graph objects

As noticed above, vertices and edges are all represented as JSON documents. The main distinction is that they are stored in a format that is compliant with the GraphSON format. However, JSON documents for Vertices and Edges are distinct in the following ways:
- Edges have a special system property called `_isEdge`.
- Edges have all flat properties: Only key-value pairs are used and not property bags.
- Vertices make use of property bags (unless the FlatSchema property has been enabled on your account).

After you use the document endpoint and connect with the SQL API, you can issue document queries like the following.

### Example queries

Obtain all **Vertices** of a graph database:

```sql
SELECT * FROM c WHERE NOT is_defined(c._isEdge)
```

Obtain all **Edges** of a graph database:

```sql
SELECT * FROM c WHERE c._isEdge = true
```
Obtain all **Vertices** of a graph database with a given **property**:

```sql
// Note that properties from vertices need to be queried in the following pattern c.<name of property>._value = <value of property>
// This is because of the property bag format that vertices require.
SELECT * FROM c WHERE NOT is_defined(c._isEdge) AND c.name[0]._value = "Luis"

// If the account has a FlatSchema setting enabled, then the following query should be used.
SELECT * FROM c WHERE NOT is_defined(c._isEdge) AND c.name = "Luis"
```

Obtain all **Edges** of a graph database with a given **property**:

```sql
// Edges, on the other side, have all flat properties. This is regardless of the FlatSchema setting.
SELECT * FROM c WHERE c._isEdge = true AND c.weight = 1
```

Obtain all **Vertices** of a graph database with a given **partition key**:

```sql
// Partition key properties are always flat (key-value). This is regardless of the FlatSchema setting.
SELECT * FROM c WHERE NOT is_defined(c._isEdge) AND c.pk  = "partition key value"
```
