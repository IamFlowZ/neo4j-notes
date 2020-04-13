# My Notes from Neo4j Graph Academy
[_Neo4j Reference Card_](https://neo4j.com/docs/cypher-refcard/current/)
## Table of Contents
1. [Cypher](#cypher)`
2. [Querying](#querying)
3. [Creation](#creating)

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