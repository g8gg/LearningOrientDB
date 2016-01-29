# OrientDB Primer

[toc]


# Overview 

## Multi-Model Database

> The OrientDB engine supports Graph, Document, Key/Value, and Object models, so you can use OrientDB as a replacement for a product in any of these categories.



### Document Model

> The data in this model is stored inside documents. A document is a set of key/value pairs (also referred to as fields or properties) where a key allows access to its value. Values can hold primitive data types, embedded documents, or arrays of other values.

![Alt text](./1441167136838.png)


### Graph Model

> A graph represents a network-like structure consisting of Vertices (also known as Nodes) interconnected by Edges (also known as Arcs). OrientDB's graph model is represented by the concept of a property graph, which defines the following:
* Vertex - an entity that can be linked with other Vertices and has the following mandatory properties:
  * unique identifier
  * set of incoming Edges
  *  set of outgoing Edges
* Edge - an entity that links two Vertices and has the following mandatory properties: 
  * unique identifier
  * link to incoming Vertex (also known as head)
  * link to outgoing Vertex (also known as tail)
  * label that defines the type of connection/relationship between head and tail vertex
  
![Alt text](./1441167250878.png)

### Object Model

> This model has been inherited by Object Oriented programming and supports Inheritance between types (sub-types extends the super-types), Polymorphism when you refer to a base class, and Direct binding from/to Objects used in programming languages.

![Alt text](./1441167320796.png)




## Basic Concept

### Schema
参见[Schema](http://orientdb.com/docs/2.1/Schema.html)
 Although OrientDB can work in schema-less mode, sometimes you need to enforce your data model using a schema. OrientDB supports schema-full or schema-hybrid solutions where the second one means to set such constraints only for certain fields and leave the user to add custom fields to the records. This mode is at class level, so you can have the Employee class as schema-full and EmployeeInformation class as schema-less.

* Schema-Full: enable the strict-mode at class level and set all the fields as mandatory
* Schema-Less: create classes with no properties. Default mode is non strict-mode so records can have arbitrary fields
* Schema-Hybrid, called also Schema-Mixed is the most used: create classes and define some fields but leave the record to define own custom fields


> Changes to the schema are not transactional, so execute them outside a transaction.

To access to the schema, you can use SQL or API. Will follow examples using Java API.

To gain access to the schema APIs you need in the Schema instance of the database you're using.
```java
OSchema schema = database.getMetadata().getSchema();
```

#### Working with Fields
参见：[Working with Fields](http://orientdb.com/docs/2.0/orientdb.wiki/Document-Field-Part.html)

> OrientDB has a powerful way to extract parts of a Document field. This applies to the Java API, SQL Where conditions, and SQL projections.
To extract parts you have to use the square brackets.



##### Extract punctual items
---------
###### Single item

> Example: tags is an EMBEDDEDSET of Strings containing the values ['Smart', 'Geek', 'Cool'].
The expression tags[0] will return 'Smart'.

###### Single items

> Inside square brackets put the items separated by comma ",".
Following the tags example above, the expression tags[0,2] will return a list with [Smart, 'Cool'].
###### Range items

> Inside square brackets put the lower and upper bounds of an item, separated by "-".
> Following the tags example above, the expression tags[1-2] returns ['Geek', 'Cool'].
Usage in SQL query

Example:
```sql
SELECT * FROM profile WHERE phones['home'] like '+39%'
```
Works the same with double quotes.

You can go in a chain (contacts is a map of map):

```sql
SELECT * FROM profile WHERE contacts[phones][home] like '+39%'
```
With lists and arrays you can pick an item element from a range:
```sql
SELECT * FROM profile WHERE tags[0] = 'smart'
```
and single items:
```sql
SELECT * FROM profile WHERE tags[0,3,5] CONTAINSALL ['smart', 'new', 'crazy']
```
and a range of items:
```sql
SELECT * FROM profile WHERE tags[0-5] CONTAINSALL ['smart', 'new', 'crazy']
```

##### Condition

> Inside the square brackets you can specify a condition. Today only the equals condition is supported.

Example:
```java
employees[label = 'Ferrari']
```
##### Use in graphs

You can cross a graph using a projection. This an example of traversing all the retrieved nodes with name "Tom". "out" is outEdges and it's a collection. Previously, a collection couldn't be traversed with the . notation. Example:
```sql
SELECT out.in FROM v WHERE name = 'Tom'
```
This retrieves all the vertices connected to the outgoing edges from the Vertex with name = 'Tom'.

A collection can be filtered with the equals operator. This an example of traversing all the retrieved nodes with name "Tom". The traversal crosses the out edges but only where the linked (in) Vertex has the label "Ferrari" and then forward to the:
```sql
SELECT out[in.label = 'Ferrari'] FROM v WHERE name = 'Tom'
```
Or selecting vertex nodes based on class:
```sql
SELECT out[in.@class = 'Car'] FROM v WHERE name = 'Tom'
```
Or both:
```sql
SELECT out[label='drives'][in.@class = 'Car'] FROM v WHERE name = 'Tom'
```
As you can see where brackets ([]) follow brackets, the result set is filtered in each step like a Pipeline.

> NOTE: This doesn't replace the support of GREMLIN. GREMLIN is much more powerful because it does thousands of things more, but it's a simple and, at the same time, powerful tool to traverse relationships.


### Class

A Class is a concept taken from the Object Oriented paradigm. In OrientDB defines a type of record. It's the closest concept to a Relational DBMS Table. Class can be schema-less, schema-full or mixed.

A class can inherit from another, shaping a tree of classes. This means that the sub-class extends the parent one inheriting all the attributes.

Each class has its clusters that can be logical (by default) or physical. A class must have at least one cluster defined (as its default cluster), but can support multiple ones. In this case by default OrientDB will write new records in the default cluster, but reads will always involve all the defined clusters.

When you create a new class by default a new physical cluster is created with the same name of the class in lower-case.


#### Create a persistent class
Each class contains one or more properties. This mode is similar to the classic Relational DBMS approach where you define tables before storing records.

Example of creation of Account class. By default a new [Cluster](Concepts#cluster] will be created to keep the class instances:

```java
OClass account = database.getMetadata().getSchema().createClass("Account");
```

#### Get a persistent class
To retrieve a persistent class use the getClass(String) method. If the class not exists NULL is returned.

```java
OClass account = database.getMetadata().getSchema().getClass("Account");
```

#### Drop a persistent class
To drop a persistent class use the OSchema.dropClass(String) method.

```java
database.getMetadata().getSchema().dropClass("Account");
```
The records of the removed class will be not deleted unless you explicitly delete them before to drop the class. Example:
```java
database.command( new OCommandSQL("DELETE FROM Account") ).execute();
database.getMetadata().getSchema().dropClass("Account");
```

### Clusters

### Record ID


##  Key Concept

### SQL

### Relationships

> “The most important feature of a graph database is the management of relationships. Many users come to OrientDB from MongoDB or other document databases because they lack efficient support of relationships.”

#### Relational Model

> “The relational model (and RDBMS - relational database management systems) has long been thought to be the best way to handle relationships. Graph databases suggest a more modern approach to this topic.”

##### 1-to-1 relationship

##### 1-to-Many relationship

##### Many-to-Many relationship

#### Relations in OrientDB

> OrientDB doesn't use JOINs. Instead it uses LINKs. A LINK is a relationship managed by storing the target RID in the source record. It's much like storing a pointer between 2 objects in memory. When you have Invoice -> Customer, then you have a pointer to Customer inside Invoice as an attribute. It's exactly the same. In this way it's like your database was in memory, a memory of several exabytes.
> What about 1-to-N relationships? These relationships are handled as a collection of RIDs, like you would manage objects in memory. OrientDB supports different kinds of relationships:
> * **LINK**, to point to one record only
> * **LINKSET**, to point to several records. Like Java Sets, the same RID can only be included once. The pointers also have no order
> * **LINKLIST**, to point to several records. Like Java Lists, they are ordered and can contain duplicates
> * **LINKMAP**, to point to several records with a key stored in the source record. The Map values are the RIDs. Works like the Java Map<?,Record>.


## Hardcore hacking

### Lightweight Edges
> OrientDB supports Lightweight Edges from v1.4. Lightweight Edges are like regular edges, but they have no identity on database. Lightweight edges can be used only when:
> * no properties are defined on edge
> * two vertices are connected by maximum 1 edge, so if you already have one edge between two vertices and you're creating a new edge between the same vertices, the second edge will be regular

> By avoiding the creation of the underlying Document, Lightweight Edges have the same impact on speed and space as with Document LINKs, but with the additional bonus to have bidirectional connections. This means you can use the MOVE VERTEX command to refactor your graph with no broken LINKs.

Starting from OrientDB v2.0, Lightweight Edges are disabled by default with new databases. This is because having regular edges makes easier to act on edges from SQL. Many issues from beginner users were on Lightweight Edges.

If you want to use Lightweight Edges, enable it via API:
```java
OrientGraph g = new OrientGraph("mygraph");
g.setUseLightweightEdges(true);
```
or via SQL
```sql
ALTER DATABASE custom useLightweightEdges=true
```

> **Changing useLightweightEdges setting to true, will not transform previous edges, but all new edges could be Lightweight Edges if they meet the requirements.**

### When use Lightweight Edges?
> These are the PROS and CONS of Lightweight Edges vs Regular Edges:
***PROS:***
>* faster in creation and traversing, because don't need an additional document to keep the relationships between 2 vertices
CONS:
>*  cannot store properties
    harder working with Lightweight edges from SQL, because there is no a regular document under the edge


参见：[Lightweight Edges](http://orientdb.com/docs/2.1/Lightweight-Edges.html)


## Let's work with Graphs
>  We already met the Graph Model a few pages ago. Now we have all the basic knowledge needed to work with OrientDB as a GraphDB! This requires the graph edition of OrientDB. Connect to the GratefulDeadConcerts database for experimentation. It contains the concerts performed by the "滴滴找布" [^fn0] product.



### Create Vertexes and Edges
> OrientDB comes with a generic Vertex persistent class called "V" (OGraphVertex in previous releases) and "E" (OGraphEdge in the past) for Edge. 

#### Graph Commands
> In effect, the GraphDB model works on top of the underlying Document model, so all the stuff you have learned until now (Records, Relationships, etc.) remains valid.

> By using graph commands, OrientDB takes care of ensuring that the graph remains always consistent. All the Graph commands are:
> * CREATE VERTEX
> * DELETE VERTEX
> * CREATE EDGE
> * DELETE EDGE

#### Create custom Vertices and Edges classes
> Even though you can work with Vertices and Edges, OrientDB provides the possibility to extend the V and E classes. The pros of this approach are:
> * better understanding about meaning of entities
> * optional constraints at class level
> * performance: better partitioning of entities
> * object-oriented inheritance among graph elements
> 
> So from now on, we will avoid using plain V and E and will always create custom classes. 

Let's develop an example graph to model a social network based on "滴滴找布" data model [^fn1]:

#### Tips
##### Drop/Delete/Remove Field
```sql
update <class> remove field
```


### Server Configuration
#### Change the Server's database directory
> By default OrientDB server manages the database under the directory "\$ORIENTDB_HOME/databases" where \$ORIENTDB_HOME is the OrientDB installation directory. By setting the configuration parameter "server.database.path" in server orientdb-server-config.xml you can specify a custom path. 

```xml
<properties>
<entry value="1" name="db.pool.min"/>
<entry value="50" name="db.pool.max"/>
<entry value="true" name="profiler.enabled"/>
<entry value="info" name="log.console.level"/>
<entry value="fine" name="log.file.level"/>
<entry value="/Users/gosber/odb" name="server.database.path" />
</properties>
```



### ETL

#### ETL Transformers
参见 [ETL Transformers](http://orientdb.com/docs/last/Transformer.html)

>Transformer components are executed in pipeline. They work against the received input returning an output.
Before the execution, the $input variable is always assigned, so you can get at run-time and use if needed.

|Available Transformers||||
|--------|--------|--------|--------|
|CSV 	|FIELD 	|MERGE 	|VERTEX
|CODE 	|LINK 	|EDGE 	|FLOW
|LOG 	|BLOCK 	|COMMAND


#### ETL - Import from RDBMS
参见： [Import from RDBMS](http://orientdb.com/docs/2.0/orientdb-etl.wiki/Import-from-DBMS.html)



#### Import from a huge csv
参见：[ how to import a huge csv and generate different vertexes/edges based on the csv column values](https://groups.google.com/d/msg/orient-database/8LkBtFCzb6w/TL4OSrLQzmoJ)



#### Import from MySQL/CSV

##### Export raw data at first
```sql
-- 导出用户数据到ddzbUsers911.csv
SELECT a.userid, b.`name`, a.userAccount, a.createDT, a.lastLoginDT, b.latitude, b.longitude FROM `user` a LEFT JOIN `userprofile` b ON a.userID = b.userID WHERE a.userid > 100000
```

```sql
-- 导出用户的找布需求(审核通过)到ddzbUserPurchases.csv
select userPurchaseId,userId,purchaseDesc,createDT,updateDT from userpurchase where userId>100000 and reviewStatus=1;
```

```sql
-- 导出报价数据到ddzbUserBids.csv
select userBidId,userPurchaseId,userId,createDT,updateDT from userbid where userId>100000 and flag <> -1;
```


##### Tips
> Format the datetime column accroding by OrientDB server configuration to avoiding NULL.

>orientdb {db=ddzb}> list properties
>|DATABASE | PROPERTIES|
|-----|-----|
| NAME                           | VALUE                                              |-----|-----|
| Name                           | null                                               |
| Version                        | 14                                                 |
| Conflict Strategy              | version                                            |
 |Date format                    | yyyy-MM-dd                                         |
 |Datetime format                | yyyy-MM-dd HH:mm:ss                                |
 |Timezone                       | Asia/Shanghai                                      |
 |Locale Country                 | CN                                                 |
 |Locale Language                | zh                                                 |
 |Charset                        | UTF-8                                              |
 |Schema RID                     | #0:1                                               |
 |Index Manager RID              | #0:2                                               |
 |Dictionary RID                 | null                                               |


* **导出时指定Datetime格式**
> ![Alt text](./1442049389102.png)

* **直接用SQL指定格式，如MySQL：**
> ```sql 
> SELECT DATE_FORMAT(NOW(),'%Y-%m-%d %H:%i:%s')
> ```

* **Care about reserved field like "id" in Blueprints standard**
> You can rename it (with "field" transformers) or set this in Orient Loader:
> ```json
> "loader" : {
 >   "orientdb": {
>		...
>		"standardElementConstraints": false,
>		...
>		}
>	}
> ```



##### 使用OETL脚本进行数据导入工作...

>  * 以ddzb为例，先导入用户数据，再导入采购，并使得V(User)→E(Want)→V(Purchase)
>  * 导入抢单(报价)数据，并使得V(Bid)→E(Offer)→V(Purchase) 

Step 1.

```bash
-- remote mode ddzbUser.json
#bin ./oetl.sh ~/source/RddzbUsers.json
```

```json
-- ddzbUsers.json
{
  "config": {
    "log": "debug"
  },
  "source" : {
    "file": { "path": "/Users/gosber/source/ddzbUsers911.csv" }
  },
  "extractor": { "row": {} },
  "transformers": [
    { "csv": { "separator": ",",
               "columnsOnFirstLine": false,
               "columns":["userID","userName","userAccount",
                 "createDT:datetime","lastLoginDT:datetime",
                 "latitude","longitude"] } },
    { "vertex": { "class": "User" } },
  ],
  "loader": {
    "orientdb": {
       "dbURL": "remote:/Users/gosber/odb/ddzb",
       "dbType": "graph",
       "wal": false,
       "batchCommit": 1000,
       "tx": true,
       "txUseLog": false,
       "useLightweightEdges" : false,
       "classes": [
         {"name": "User", "extends": "V"}
       ]
    }
  }
}
```
------------
Step 2.

```bash
-- remode mode ddzbUserPurchase.json
#bin ./oetl.sh ~/source/ddzbUserPurchase.json
```

```json
--ddzbUserPurchase.json
{
  "config": {
    "log": "debug"
  },
  "source" : {
    "file": { "path": "/Users/gosber/source/ddzbUserPurchases911.csv" }
  },
  "extractor": { "row": {} },
  "transformers": [
    { "csv": { "separator": ",",
               "nullValue": "",
               "columnsOnFirstLine": false,
               "columns":["userPurchaseId","userID","purchaseDesc",
                 "createDT:datetime","updateDT:datetime"
                 ] } },
    { "vertex": {"class": "Purchase"}},
    { "edge":
      {"class":"Want",
        "direction":"in",
        "joinFieldName":"userID",
        "lookup":"User.userID"
        }
    }
  ],
  "loader": {
    "orientdb": {
       "dbURL": "remote:/Users/gosber/odb/ddzb1",
       "dbType": "graph",
       "wal": false,
       "batchCommit": 1000,
       "tx": true,
       "txUseLog": false,
       "useLightweightEdges" : false,
       "dbAutoCreate": true,
       "classes": [
         {"name": "Purchase", "extends": "V"},
         {"name": "User", "extends": "V"},
         {"name": "Want", "extends": "E"}
       ]
    }
  }
}
```
------------
Step 3.

```bash
-- remode mode ddzbUserPurchase.json
#bin ./oetl.sh ~/source/ddzbUserBids.json
```

```json
--ddzbUserBids.json
{
  "config": {
    "log": "info"
  },
  "source" : {
    "file": { "path": "/Users/gosber/source/ddzbUserBids911.csv" }
  },
  "extractor": { "row": {} },
  "transformers": [
    { "csv": { "separator": ",",
               "columnsOnFirstLine": false,
               "columns":["userBidId","userPurchaseId","userId",
                 "createDT:datetime","updateDT:datetime"
                 ] } },
    { "vertex": { "class": "Bid" } },
    { "edge":
      {"class":"Offer",
        "direction":"out",
        "joinFieldName":"userPurchaseId",
        "lookup":"Purchase.userPurchaseId"
      }
    }
  ],
  "loader": {
    "orientdb": {
       "dbURL": "remote:/Users/gosber/odb/ddzb1",
       "dbType": "graph",
       "wal": false,
       "batchCommit": 1000,
       "tx": true,
       "txUseLog": false,
       "useLightweightEdges" : false,
       "classes": [
         {"name": "Bid", "extends": "V"},
         {"name": "Purchase", "extends": "V"},
         {"name": "Offer", "extends": "E"}
       ]
    }
  }
}

```

##### 验证读取数据
* 从Purchase中找到所有/指定的in的Vertex的信息
```sql
select expand(in() from #12:0 (@rid of Purchase)
select expand(in('Want')) from #12:0 (@rid of Purchase)
```

* 从Purchase中找到所有/指定的in的Edge的信息
```sql
select expand(inE() from #12:0 (@rid of Purchase)
select expand(inE('Want')) from #12:0 (@rid of Purchase)
```

```
SELECT traversedElement(-1) FROM ( TRAVERSE in() from #12:0 WHILE $depth <= 1 )
```

### FAQ / Tips
#### Indexes

##### Get all the configured indexes:
```sql
select expand(indexes) from metadata:indexmanager
```

>| \#   |@CLASS|mapRid|clusters|algorithm|type     |name     |indexVers|valueContain|metadata          |indexDefinitionClass        |indexDefinition
>  | ------ | --------: | :--------| :------: | --------: | --------:| :------: | :-------- | --------:| :------: | :-------- |
| 0   |null  |null  |[0]     |SBTREE   |DICTIO...|dictio...|1        |NONE        |{durableInNonTx...|com.orientechnologies.ori...|{keyTypes:[1],collate:defau...
|1   |null  |null  |[3]     |LUCENE   |SPATIAL  |User.P...|1        |NONE        |null              |com.orientechnologies.ori...|{className:User,indexDefini...
|2   |null  |null  |[1]     |SBTREE   |UNIQUE   |OUser....|1        |NONE        |null              |com.orientechnologies.ori...|{className:OUser,field:name...
|3   |null  |null  |[1]     |SBTREE   |UNIQUE   |ORole....|1        |NONE        |null              |com.orientechnologies.ori...|{className:ORole,field:name...


#### Properties

##### Get all properties for special @Class
```sql
select expand(properties) from (select expand(classes) from metadata:schema) where name = 'User'
```

>  | \# |@CLASS|name       |type|globalId|mandatory|readonly|notNull|defaultValue|min |max |regexp|customFields|collate
>  | ------ | --------: | :--------| :------: | --------: | --------:| :------: | :-------- | --------:| :------: | :-------- | --------:| :------: |------|
|0   |null  |longtitude |5   |27      |false    |false   |false  |null        |null|null|null  |null        |default
|1   |null  |longitude  |5   |26      |false    |false   |false  |null        |null|null|null  |null        |default
|2   |null  |createDT   |6   |23      |false    |false   |false  |null        |null|null|null  |null        |default
|3   |null  |userAccount|7   |21      |false    |false   |false  |null        |null|null|null  |null        |default
|4   |null  |userName   |7   |22      |false    |false   |false  |null        |null|null|null  |null        |default
|5   |null  |userID     |1   |20      |false    |false   |false  |null        |null|null|null  |null        |default
|6   |null  |latitude   |5   |25      |false    |false   |false  |null        |null|null|null  |null        |default
|7   |null  |lastLoginDT|6   |24      |false    |false   |false  |null        |null|null|null  |null        |default


### Troubleshooting
#### ETL
##### Import
###### issue 1
* ETL import csv with date time field, but only date converted, time becomes 00:00:00 
> [ETL force format datetime field](https://www.mail-archive.com/orient-database@googlegroups.com/msg12016.html)
> Try forcing datetime. Look at:
http://orientdb.com/docs/last/orientdb-etl.wiki/Transformer.html#csv and
set the "columns" field in CSV transformer, like:
{ "csv": { "separator": ",", "columns":["date:datetime", "type:string",
"phone:string", "old_token:string", "new_token:string"] } }

###### issue 2
* Convenience command for renaming a field through SQL API
参见：[讨论](https://github.com/orientechnologies/orientdb/issues/3350)
```sql
UPDATE <class> RENAME <field> <newFieldName>
```
> Yeah, I understood that. However, I am talking about when in schemaless mode and not using a property. Attempting to use ALTER PROPERTY before creating the property failed for me (or is that a bug?).
Chatting to Luca over on the forum, I see the solution currently is:


```sql
UPDATE <class> SET <newField> = <oldField>
UPDATE <class> REMOVE <oldField>
```
> I still think there is a case for a more obvious RENAME command though. I searched for quite a bit on the forum and docs on how to rename a field and came up blank when ALTER PROPERTY didn't work. Perhaps just an addition to the documentation would suffice though, although perhaps a RENAME would help for the case you mention of handling custom indexes.


##### Export

#### SQL
#### Admin
#### Backup


 
[^fn0]:**滴滴找布**是人脉通的一款垂直B2B撮合交易产品，[官网](http://www.ddzhaobu.com)

[^fn1]:如图：


