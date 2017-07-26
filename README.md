# Neo4J 101 Workshop

Slides can be found [here](http://slides.com/carlos-hernandez/graph-databases)

## What is Neo4j?
Neo4j is an open-source NoSQL graph database implemented in Java and Scala. It implements the Property Graph Model efficiently down to the storage level, and provides full database characteristics including ACID transaction compliance, cluster support and runtime failover.

## Requirements
* Java 8
* Neo4J 3.2+

## Installation
By default Neo4j requires authentication and requires you to login with neo4j/neo4j at the first connection and set a new password.

### Brew
1. Run `brew install neo4j`. This will update `brew` and install Neo4J (version 3.2 or higher).
1. Run `neo4j version` to verify the version installed.
1. Set the password before starting up the database for the first time:
    * `neo4j-admin set-initial-password <password>`

### Docker
1. Change the local repo path and password in the following command and run it:
    ```Shell
    docker run \
        --publish=7474:7474 --publish=7687:7687 \
        --volume=$HOME/neo4j/data:/data \
        --volume=$HOME/neo4j/logs:/logs \        
        --volume=/<path_to__ws_repo>:/var/lib/neo4j/import \        
        --env NEO4J_AUTH=neo4j/<password> \
        --detach --name neo4j-101-ws \
        neo4j:latest
    ```

## Cypher

### What is Cypher?
Cypher is a declarative graph query language that allows for expressive and efficient querying and updating of the graph store.

### Connecting to the Cypher Shell
* Brew:
    * `neo4j start`
    * `cypher-shell --username neo4j`
* Docker:
    * `docker exec --interactive --tty neo4j-101-ws bin/cypher-shell --username neo4j`

### Cypher Shell helpful commands:
* `:exit`
* `:help`
* `:history`

## Cypher: Basic syntax

### Node syntax
```Cypher
()
(movie)
(:Movie)
(movie:Movie)
(movie:Movie { title: "The Lord of the Rings: The Fellowship of the Ring" })
(movie:Movie { title: "The Lord of the Rings: The Fellowship of the Ring", released: 2001 })
```

### Relationship syntax
```Cypher
-->
-[role]->
-[:ACTED_IN]->
-[role:ACTED_IN]->
-[role:ACTED_IN { roles: ["Aragorn"] }]->
```

### Pattern syntax
```Cypher
(actor:Actor { name: "Viggo Mortensen" })
-[role:ACTED_IN { roles: ["Aragorn"] }]->
(movie:Movie { title: "The Lord of the Rings: The Fellowship of the Ring" })

acted_in = (:Person)-[:ACTED_IN]->(:Movie)
```

## Cypher: Data manipulation

### Creating nodes 
```Cypher
CREATE (actor:Actor { name:"Viggo Mortensen", born:1958 });
```

### Creating relationships
```Cypher
MATCH (actor:Actor { name:"Viggo Mortensen"})
CREATE (movie:Movie { title: "The Lord of the Rings: The Fellowship of the Ring", released: 2001 })
CREATE (actor)-[role:ACTED_IN { roles: ["Aragorn"] }]->(movie);
```

### Update a node and create it if it doesn't exist
```Cypher
MERGE (movie:Movie { title:"Alatriste" })
ON CREATE SET movie.released = 2006;
```

### Update a relationship and create it if it doesn't exist
```Cypher
MATCH (movie:Movie { title:"Alatriste" })
MATCH (actor:Actor { name:"Viggo Mortensen" })
MERGE (actor)-[role:ACTED_IN]->(movie)
ON CREATE SET role.roles = ["Captain Diego Alatriste"];
```

### Set a property
```Cypher
MATCH (movie { title: "Alatriste" })
SET movie.country = "Spain";
```

### Remove a property
```Cypher
MATCH (movie { title: "Alatriste" })
REMOVE movie.country;
```

### Deleting nodes
```Cypher
MATCH (n)
DELETE n;
```

### Deleting relationships
```Cypher
MATCH ()-[r]-()
DELETE (r);
```

## Exercise 1: Data manipulation
1. Create a single node with no labels and no properties
1. Create multiple nodes in a single `CREATE` statement
1. Create a node with multiple labels
1. Remove a label from a node
1. Create a node and return it in the same statement
1. Create a full path in a single `CREATE` statement
1. Update an existing node and set a property if a match is found
1. Copy properties between two nodes
1. Delete a node with existing relationships
1. Delete an specific type of relationship
1. Delete a node with a specific label

## Cypher: Retrieving data

To test the following queries you can execute the following:
```Cypher
MATCH (n) DETACH DELETE n;

CREATE (viggo:Actor { name:"Viggo Mortensen", born:1958 }),
    (lotr:Movie { title: "The Lord of the Rings: The Fellowship of the Ring", released: 2001 }),
    (alatriste:Movie { title:"Alatriste", country: "Spain", released: 2006 }),
    (viggo)-[:ACTED_IN { roles: ["Aragorn"] }]->(lotr),
    (viggo)-[:ACTED_IN { roles: ["Captain Diego Alatriste"] }]->(alatriste);
```

### Get all nodes
```Cypher
MATCH (n)
RETURN n;
```

### Match by labels and properties
```Cypher
MATCH (:Actor { name: 'Viggo Mortensen' })--(movie:Movie)
RETURN movie.title AS title;
```

### Get relationship type
```Cypher
MATCH (:Actor { name: 'Viggo Mortensen' })-[r]->(movie)
RETURN DISTINCT type(r);
```

### Match on relationship type
```Cypher
MATCH (alatriste:Movie { title: 'Alatriste' })<-[:ACTED_IN]-(actor)
RETURN actor.name;
```

### Named paths
```Cypher
MATCH p =(viggo { name: 'Viggo Mortensen' })-->()
RETURN p;
```

### Return all elements
```Cypher
MATCH p =(viggo { name: 'Viggo Mortensen' })-->()
RETURN *;
```

### Returning different kind of elements
```Cypher
MATCH (actor:Actor)
WHERE actor.name CONTAINS 'Viggo'
RETURN actor.born > 1970, "I'm a literal", (actor)-->();
```

### Property existence checking
```Cypher
MATCH (n)
WHERE exists(n.title)
RETURN n;
```

## Filter on aggregate function results
```Cypher
MATCH (actor:Actor)-->(movie)
WITH actor, count(*) AS appearances
WHERE appearances > 1
RETURN actor.name;
```

### Sort results before using collect on them
```Cypher
MATCH (n)
WITH n
ORDER BY n.title DESC LIMIT 3
WHERE exists(n.title)
RETURN collect(n.title);
```

## Loading data from CSV files

Given the following CSV files:

**persons.csv**
```CSV
id,name
1,Charlie Sheen
2,Oliver Stone
3,Michael Douglas
4,Martin Sheen
5,Morgan Freeman
```

**movies.csv**
```CSV
id,title,country,year
1,Wall Street,USA,1987
2,The American President,USA,1995
3,The Shawshank Redemption,USA,1994
```

**roles.csv**
```CSV
personId,movieId,role
1,1,Bud Fox
4,1,Carl Fox
3,1,Gordon Gekko
4,2,A.J. MacInerney
3,2,President Andrew Shepherd
5,3,Ellis Boyd 'Red' Redding
```

we can import its data to Neo4J as follows:
```Cypher
CREATE CONSTRAINT ON (person:Person) ASSERT person.id IS UNIQUE;
LOAD CSV WITH HEADERS FROM "file:///persons.csv" AS csvLine
CREATE (p:Person { id: toInteger(csvLine.id), name: csvLine.name });

CREATE CONSTRAINT ON (movie:Movie) ASSERT movie.id IS UNIQUE;
CREATE INDEX ON :Country(name);
LOAD CSV WITH HEADERS FROM "file:///movies.csv" AS csvLine
MERGE (country:Country { name: csvLine.country })
CREATE (movie:Movie { id: toInteger(csvLine.id), title: csvLine.title, year:toInteger(csvLine.year)})
CREATE (movie)-[:MADE_IN]->(country);

USING PERIODIC COMMIT 500
LOAD CSV WITH HEADERS FROM "file:///roles.csv" AS csvLine
MATCH (person:Person { id: toInteger(csvLine.personId)}),(movie:Movie { id: toInteger(csvLine.movieId)})
CREATE (person)-[:PLAYED { role: csvLine.role }]->(movie);

DROP CONSTRAINT ON (person:Person) ASSERT person.id IS UNIQUE;
DROP CONSTRAINT ON (movie:Movie) ASSERT movie.id IS UNIQUE;

MATCH (n)
WHERE n:Person OR n:Movie
REMOVE n.id;
```

For security reasons, all `file://` URLs are always relative to the standard import directory (`$NEO4J_HOME/import`).

## Exercise 2
Given the following ASCII art:
```
(:NationalTeam {name})<-[:REPRESENTS]-(:Player {name})-[:PLAYS_FOR]->(:Club {name})
```

1. Load the CSV files found in the `exercise02` folder to create this graph.
  
    `$NEO4J_HOME` is `/usr/local/Cellar/neo4j/<version>/libexec` if you installed Neo4J using Brew. Copy the files into the import directory:
    ```Shell
    cp -R exercise02 /usr/local/Cellar/neo4j/<version>/libexec/import
    ```
    
    For Docker, this data is already available in the right folder if you followed the instructions.
1. Once the data is loaded, answer the following questions:
    * Which are the top 10 clubs with the most players being called by their national teams? Display the club name and the number of international players.
    * Using the top 1 club from the previous question, who are the players and where do they come from? Display the country name, the number of players and a list with its names.
    * In which clubs are playing the members of a given national team? Display the club name, the number of players and a list with its names.
    * What country needs to pick from less clubs to build their national team? Display the country name and the number of distinct clubs.

**NOTE:** You can connect to http://localhost:7474 to use the Neo4J Browser to resolve this exercise.

Also, you can use the [Neo4J Cypher Refcard](https://neo4j.com/docs/cypher-refcard/current/) to find more information about Cypher.
