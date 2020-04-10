# My Notes from Neo4j Graph Academy
[_Neo4j Reference Card_](https://neo4j.com/docs/cypher-refcard/current/)
## Table of Contents
1. [Cypher](#cypher)
2. [Querying](#querying)

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



