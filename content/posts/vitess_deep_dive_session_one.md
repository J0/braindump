+++
title = "Vitess Deep Dive Session One"
author = ["Joel Lee"]
tags = ["Vitess"]
date = "2019-12-22"
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

- Scan is special because it is the only primitve that doesn't take a result as a input. It takes a table name as an input and produces a result. Joins and subqueries cause a total explosion of complexity. Doesn't end with these relational primitives though. We have scalar primitives and Custom Primitives too. 

### Custom Primitive examples {#custom-primitive-examples}

#### Different variants of existing algorithms

- Index Scan 
- Hash Join, BNL Join 
- FileSort 

#### Steps to building the plan
1. Input: SQL 
2. Parser-> AST 
3. PlanBuilder->Relational Primitives 
4. Optimizer->Final Plan with Custom Primitives
 
## Relational Engine also needs a Symbol table {#relational-engine-also-needs-a-symbol-table}

Symbol Engine is a way to validate the correctness of your query. Here are some jobs it performs

- Validate the correctness of your query
- Does it obey scoping rule 

## Track Select {#track-select}

- Used for optimizations. It tells you that this column has an index and what's it's cardinality. It is a critical part of engine design

## What makes joins and subqueries complex? {#what-makes-joins-and-subqueries-complex}

- In the traditional sense, a join is a cross product between two tables. How do you even do a cross-product.
MySQL uses Nested Loop Join. Oracle Uses a Hash Join.
 Joins are bad but subqueries are harder


### We can have a simple subquery or a correlated one. {#we-can-have-a-simple-subquery-or-a-correlated-one-dot}

- We don't care about the queries name. We only care about what we need to do to return the results of a query


### Subqueries have multiple forms {#subqueries-have-multiple-forms}

- They can be used as a simple scalar value, a column tuple, or even a row. It can even be used as rows of columns or a virtual table. Subquery can basically appear anywhere in your current query.
The simple solution is to treat a subquery as a join. Subquery the result is used to apply an additional filter or perform another operation. Subquery is a superset of what a join can do.


## Architecture of Vitess {#architecture-of-vitess}

- Look for diagram
- Big difference is that each of items in Vitess is a full fledged database.
- The vitess version can do a lot more than what SQL Engine version can. 
- Higher round-trip cost to VTGate 
- Sharded data means that not all queries will be efficient 
- Vindex design allows for pluggable indexes which have different types of indexing options.

## Differences in Architecture affect how we create the engine 
- VTGate is kind of a query router but it  is also an engine because it can handle queries of any complexity. Conclusion is that it is both a router and an engine because sometimes it needs to route and sometimes it needs to do relational work
- Generate hybrid opcodes instead of relational opcodes. Generate AST and subsequently SQL from this.

## The Route Primitive {#the-route-primitive}
- Send a query to one database 
- Scatter Route: Sends a query to all shards and combine results

## V3 Strategy {#v3-strategy}
- When we get a query we gfigure out if all the rows we need to satisfy a query is in one datbase. If so, then we can send the construct to that database using the route primitive.
- If not, we split the queries into parts and route them to appropriate parts where each part satisifies the above constraint. VTGate will stitch together any leftover work(stitching together results)

## More on stitching results {#more-on-stitching-results}

- Left table rows in one shard, right table rows in another shard.
- One is a simple join the other is a left join.
- There's only Route and Join but we want support for the other primitives.
- What would it mean if we get support for the other 9 primitives. This is fine for OLTP but if we need OLAP we need to do more work(Complex GroupBy). Under these scenarios, other primitives will be useful.
- There are limits so cross-shard limits
- VTTablet has a limit of 10000 rows  and VTGate has a limit too because things can multiply
You can stream a join


