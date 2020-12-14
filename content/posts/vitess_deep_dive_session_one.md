+++
title = "Vitess Deep Dive Session One"
author = ["Joel Lee"]
tags = ["Vitess"]
date = "2020-12-14"
draft = false
+++


## Relational Primitives {#relational-primitives}


-   Scan
-   Join
-   LeftJoin
-   Filter
-   Select
-   Aggregate
-   Sort
-   Limit
-   Merge

Of all the primitives, scan is special because it is the only primitve that doesn't take a result as a input. It takes a table name as an input and produces a result. Joins and subqueries cause a total explosion of complexity. Doesn't end with these relational primitives though. We have scalar primitives and Custom Primitives too. 





#### Different variants of existing algorithms

- Index Scan 
- Hash Join, BNL Join 
- FileSort 

#### Steps to building the plan
1. Input: SQL 
2. Parser-> AST 
3. PlanBuilder->Relational Primitives 
4. Optimizer->Final Plan with Custom Primitives

In addition, we also have a symbol table which is used to validate the correctness of the query. For instance, it checks if a table exists and if scoping rules are obeyed 

Here's an overview of the parsing process:

{{< figure src="/ox-hugo/parsing_process.png" caption="Overview of the parsing process" >}}

## Track Select {#track-select}

- Used for optimizations. It tells you that this column has an index and what's it's cardinality. It is a critical part of engine design

## What makes joins and subqueries complex? {#what-makes-joins-and-subqueries-complex}

- In the traditional sense, a join is a cross product between two tables. How do you even do a cross-product.
MySQL uses Nested Loop Join. Oracle Uses a Hash Join. Joins are bad but subqueries are harder. We can have a simple subquery or a correlated one. As a recap, here's what a nested loop join looks like:

```
for each row in t1 matching range {
  for each row in t2 matching reference key {
    for each row in t3 {
      if row satisfies join conditions, send to client
    }
  }
}
```


Here's an example of a correlated query. There are different ways the query could be executed but it doesn't matter at this point.

{{< figure src="/ox-hugo/correl_query.png" caption="Example of correlated subquery" >}}


### Subqueries have multiple forms {#subqueries-have-multiple-forms}

- They can be used as a simple scalar value, a column tuple, or even a row. It can even be used as rows of columns or a virtual table. Subquery can basically appear anywhere in your current query.
The simple solution is to treat a subquery as a join. Subquery the result is used to apply an additional filter or perform another operation. Subquery is a superset of what a join can do.


## Architecture of Vitess As Compared To MySQL {#architecture-of-vitess}

{{< figure src="/ox-hugo/vitess_arch.png" caption="Vitess Architecture vs MySQL Architecture">}}

- Big difference is that each of items in Vitess is a full fledged database.
- The vitess version can do a lot more than what SQL Engine version can. 
- Higher round-trip cost to VTGate 
- Sharded data means that not all queries will be efficient 
- Vindex design allows for pluggable indexes which have different types of indexing options.

## Differences in Architecture affect how we create the engine 
- VTGate is kind of a query router but it  is also an engine because it can handle queries of any complexity. Conclusion is that it is both a router and an engine because sometimes it needs to route and sometimes it needs to do relational work
- Generate hybrid opcodes instead of relational opcodes. Generate AST and subsequently SQL from this.

### The Route Primitive {#the-route-primitive}
- Send a query to one database 
- Scatter Route: Sends a query to all databases. If no routing key then the data can befound on any shard.

### V3 Strategy {#v3-strategy}
- When we get a query we figure out if all the rows we need to satisfy a query is in one database. If so, then we can send the construct to that database using the route primitive.
- If not, we split the queries into parts and route them to appropriate parts where each part satisifies the above constraint. VTGate will stitch together any leftover work(stitching together results)

### More on stitching results {#more-on-stitching-results}

- Left table rows in one shard, right table rows in another shard.
- One is a simple join the other is a left join.
- There's only Route and Join but we want support for the other primitives.
- What would it mean if we get support for the other 9 primitives. This is fine for OLTP but if we need OLAP we need to do more work(Complex GroupBy). Under these scenarios, other primitives will be useful.
- There are limits so cross-shard limits
- VTTablet has a limit of 10000 rows and VTGate has a limit too because things can multiply

### Cross Shard Join
- It was mentioned that in a Join + cross shard join we could loop over rows in a row set and pull from all shards to check for row matching the query. It was suggested that this could be a non starter because of the network round trip time involved.
- The counterpoint was that the user will be warned of this and that there are only a handful of rows typically involved when building such a query in Vitess(It is a last resort solution for Vitess and if done the workload is low).
- It was also mentioned that there is an easy optimization in which there is a unique VIndex and we can fetch blocks in batches(10  queries from origin). In some ways this is like Block Nested Loop Join in MySQL.

### Misc

- You can stream a join
- At Youtube, they had functionality which could take a Complex Query and turn it into a MapReduce Job via Hive.
- One advantage that Vitess has over NoSQL is that Vitess can push down complex work to MySQL.
- Perhaps if relational databases could have relaxed ACID properties earlier then more users would have stuck to relational solutions instead of defecting to NoSQL.


