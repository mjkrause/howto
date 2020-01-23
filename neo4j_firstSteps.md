(v. 2019-12-20)

# `neo4j` Graph Database

Graph databases are highly performant and scalable. The logical model is basically the physical model, and chances in the model can be implemented relatively easy. Queries looking for the same records and information are much shorter for graph databases compared to reltionals databases. In practice Neo4j is used in fraud detection, real-time recommendations, social networks, and data center management. 

Data organization and represenation in graphs (nodes and relationships) is more similar to human's intuition compare to organization as tables and keys.

# Installation 

Pull down a neo4j docker container and run it. Import some data into it. Data can be in a text file where each line is a Cypher command. Once it is running there are two ways to interact with a database: either using the browser by pointing it to `localhost:7474/browser` (HTTP and local only, not secure) and follow the instructions, or by using a shell named `Cypher` (but the client needs to be installed separately). Cypher is a query language, which is similar to SQL clauses. 

To just get a feel for a Neo4j DB you can also point your browser to `http://console.neo4j.org/`, which permits input of queries to an existing database evolving around the movie The Matrix, and shows query results as a relationship graph.


## Accessing it through the browser
I first used the browser and connected to it as username `neo4j` and password `admin`. When I typed in that password it redirected to another page in which I could choose a password. From here the Neo4j's REST API can be explored. 

## Accessing it through `cypher-shell`

If the app is running in a container you need to access the database by entering the container:
```bash
docker exec -it name_of_container bash
```
Databases are in `data/databases`. Executing `cypher-shell` launches the client. It will ask for a username and password. Create a new credentials or in case you have created one through the browser already use those credentials. The successful log-in should get into the `neo4j` prompt.

## Core concepts

* Neo4j is a graph database, basically it can be a directed, acyclic graph (DAG). (are all Neo4j DBs DAGs?)

* The DB consists of nodes and relationships.

* A node in a graph represents a thing, and it can have a relationship with one or more nodes, including itself. 

* A node is has a label that starts with a colon, like so: `:someNode`

* Similarly, a relationship has a label that starts with colon

* Relationships can have names and properties. Properties are simple JS objects (JSON), but they can't be nested. JSON objects returned by search queries, however, can be nested.

* Relationships can only exist if there are nodes connecting them.

* Relationships are always directional, sometimes this is implicit.

* Value types are
    * boolean
	* text (string)
	* numbers (int and floats)
	* lists (all elements must be of same type)
	
* Deleting properties just means that you delete a key-value pair from a relationship property. 

## Drivers and connections

A client interfaces with Neo4j by using either the Bolt protocol or its HTTP API. For HTTP use port `7474`, for HTTPS use port `7473`, and for Bolt use port `7687`.

A driver for Node.js exists to connect to a Neo4j DB. You would use the _Neo4jJavaScript Driver_, wrap it in a function, and query the DB by using the endpoint directly, e.g., using the `request` Node module.

In our case we use LoopBack, and connect to the database by defining it as one of the datasources, and use LoopBack's Neo4j connector to connect to it.
