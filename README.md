# My Notes from Neo4j Graph Academy
[_Neo4j Reference Card_](https://neo4j.com/docs/cypher-refcard/current/)
## Table of Contents
1. [Cypher](#cypher)
2. [Querying](#querying)
3. [Creation](#creating--updating)
4. [Parameters](#parameters)
5. [Analysis](#analysis)
6. [Constraints](#constraints)
7. [Indexes](#indexes)
8. [Importing](#importing)

## Cypher
* Uses ascii art to describe intent to the database
* () = node
* [] = relationship
	* "-" = edge
	* -> or <- = directed relationship
* Labels are denoted by a colon followed by a string (:LABEL) or [:LABEL]
* Properties are surrounded by curly brackets (:LABEL {prop:val})
* Assign variables to nodes before labels "(a:LABEL)" or before patterns "a=()--()"
* (a:Person{name:"John"}) - [:FOLLOWS] -> (b) = a is related to b
* (a:Person) <- [:FOLLOWS] - (b) = b is related to a
* (a:Person) <- [:FOLLOWS] -> (b) = a and b are both related
* Use "MATCH" to search for a pattern
* RETURN to return patterns or properties
* ```MATCH (a:Person{name:"John"}) - [:FOLLOWS] -> (b) RETURN a.name, a.height```

## Querying
* Use "WHERE" to filter results
	* ``` WHERE a.age >= 22 ```
* You can use multiple matches in a single query
	* ``` MATCH (a:Person) - [:KNOWS] - (b:Person {name:"Jack"}), (a) - [:KNOWS] - (c:Person {name:"Cindy"})```
* You can query for varying path lengths with [:LABEL*N]
	* Where "N" is the depth that you would like to traverse
	* You can omit "N" to traverse all relationships of "LABEL"
	* You can also specify a length range with ".." (N..O)
	* ``` [:KNOWS*1..2] ```
		* friend of a friend
	* You can replace "N" with "*" to find the shortest path
* Tons of aggregation functions
	* Don't need to specify grouping key, any non-aggregated keys, becoming grouping keys after all aggregation is completed.
* Use "WITH" to do intermediate filtering
	* ``` WITH a, count(a) as numMutuals ```
	* For example, the initial match pattern may be very general, but you have some range of values in mind. Use this to filter down to those ranges.
	* Variables created in the original match are no longer accessible after the match clause
* Use "DISTINCT" to remove duplicate values in lists
	* Can be paired with "WITH"
	* ``` WITH DISTINCT a ```
* Use "ORDER BY" to sort results
	* ``` ORDER BY a.age DESC ```
* Use "LIMIT" to limit the number of results
	* ``` LIMIT N ```
* "count()" has two forms
	* "count(*)" returns the number of rows a pattern resolves to
	* "count(a)" returns the number of non-null instances of an expression

## Creating & Updating
* Use "CREATE" followed by a pattern to store new information in the database
* If creating patterns where nodes already exist, "MATCH" them first, then fill in the blanks
	* ``` MATCH (a:Person{name:"Tom"}), (b:Person{name:"Ken}) CREATE (a) - [:KNOWS] -> (b) RETURN a, b;```
* You can add additional labels to nodes
	* ``` MATCH (m:Movie) SET m:Action```
* Remove labels with REMOVE
	* ``` REMOVE m:Action ```
* You can use JSON syntax to set properties on a node
	* ```SET x = {prop1: val1, prop2: val2}```
	* ```SET x += {prop3: val3}```
* Set individual properties
	* ```SET x.name="kyle"```
* "MERGE" Should be used carefully
	* Can lead to duplicated data if not used correctly
	* Both modifies and creates data depending on if the query identifies the node.
* Use "ON CREATE " to modify new nodes while merging.
* Use "ON MATCH " to modify already existing nodes.
* When using merge to create relationships, you must first match the nodes to be related, otherwise the nodes are created.

## Parameters
* In production, you should never use hard coded values. You should have the statement hard-coded and pass values in as parameters.
* Use "$" followed by a name to parameterize values
	* ```set a.name = $personName```
* Use ":params" directive in the neo4j GUI to set parameters
	* ``` :params {personName: "Scott"}```
	* View the currently set parameters with ``` :params```
	* remove a value by resubmitting the params object without that key/value
	* Clear the parameters object by passing an empty object ``` :params {} ```

## Analysis
* Two main commands to analyze query execution
	* "EXPLAIN" - estimates the amount of processing that might occur
	* "PROFILE" - provides information about how the engine behaved during the query and executes the query
* Neo4j keeps a cache of common nodes & relationships, the more specific you are with the nodes you're looking for, the more likely that it will hit the cache. Using "PROFILE" to monitor cache vs DB hits can be a easy way to improve query performance.
* If a query is executed and returns a lot of data, there is no way to monitor or kill it while it's returning. This can have significant performance costs.
* If a query is long-running but hasn't returned, you can monitor it with ``` :queries ```
* You can then kill it with ``` CALL dbms.killQuery('query-id')``` or by clicking in the GUI

## Constraints
* Easy way to prevent duplication is through constraints
* Mainly used to guarantee uniqueness
* If any nodes/relationships fail to meet the constraint when the constraint is being created, the constraint will not be created.
* Unique property for a given label
	* ``` CREATE CONSTRAINT on (a:Person) ASSERT a.name is UNIQUE```
* Ensure a property exists
	* ``` CREATE CONSTRAINT on ()-[a:CREATED]-() ASSERT exists(a.date) ```
* Remove constraints with "DROP"
	* ``` DROP CONSTRAINT on ()-[a:CREATED]-() ASSERT exists(a.date) ```
* Use Node Keys to define multiple unique properties for a node label. Internally is used as a composite index for the label
	* ``` CREATE CONSTRAINT on (a:Person) ASSERT (a.name, a.born) IS NODE KEY ```

## Indexes
* Indexes improve read speed at the cost of write-speed and storage size. Indices store redundant data.
* Unlike SQL, no primary keys. Can have multiple required, unique properties 
* Single property indices are used for:
	* Equality checks "="
	* Range comparisons ">, <, <=, >="
	* List membership "IN"
	* String comparisons "STARTS WITH, ENDS WITH, CONTAINS"
	* Existence checks "exists()"
	* Spatial distance searches "distance()"
	* Spatial bounding searches "point()"
* Composite indices are used for: 
	* Equality checks
	* List membership
* For large graphs, it is recommended to create indices AFTER data is loaded.
* Uniqueness constraints == indices, indices != uniqueness constraints
* Use "CREATE INDEX" to create indices for labels
	* ``` CREATE INDEX ON :Movie(released) ```
* Create composite indices by listing properties in the query
	* ``` CREATE INDEX ON :Movie(released, format) ```
	* This creates a second combined index. This and the previous index would both exist.
* List indices with ``` CALL db.indexes() ```
* Delete indices with "DROP INDEX"
	* ``` DROP INDEX ON :Movie(released, format) ```

## Importing
* The most common way to import data into Neo4j is via _.csv_ files.
* Use "LOAD CSV" to pull normalized csv data into Neo4j
	* parses a file in the import directory of the Neo4j installation
	* ``` LOAD CSV WITH HEADERS FROM url-value AS row ```
	* You can specify the csv delimiter with "FIELDTERMINATOR"
		* ``` FIELDTERMINATOR ';' ```
* If you're importing >10k lines you should use "PERIODIC COMMIT" to avoid mishaps or performance hits.