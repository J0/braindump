+++
title = "Vitess Deep Dive Session Three"
author = ["Joel Lee"]
tags = ["Vitess"]
date = "2020-12-15"
draft = false
+++

## Cross-Shard Joins 
- How to connect them up using the original query
- Rule which we could apply to most queries: right-most rule. Always use the right most query fragment that an expression addresses and then push that.
- Homework: What can VTGate not handle today?
- If there is work left after stitching together the joins then VTGate won't be able to handle it.VTGate figures out it has to do other work beyond the join. This is the starting point.
- We talk about the performance implementation of doing joins.  

### Recap of operators
- Select from unsharded table: Unsharded route
- Select from  sharded table: Scatter Route
- Select col where id=5 : Unique route -> targets a single sharded
- select col where name=foo: non-unique route

In summary, all routes have one primitive

Simple use case w example

8:43 timestamp

Left hand side route handles table A right hand side route handles table B

Q: Are the left route and right route exectued in parallel?
A: We execute left hand side and we build bind variable a_n and then we loop on the right hand side. We then combine left and right. We work as the relational operator prescribes. 

Q: What kind of metrics should we look for especially for these use cases?
A: Use the total latency of the query.

We can optimize on the number of roundtrips(RTTs)

## Scenario 1: Join on Unique Index 

Timestamp: 16:02

When you receive a query you parse it. Notation is fixed location in oval bottom is from bottom right is where top right is select expression, left hand side is all select expressions.

The query can potentially have to look at all the rows at A so we can provisionally build a route which scans all of A. Because it scans all of A the route opcode is Scatter.

So we start from a FROM clause and attempt to push down as much as we can.

## Scenario 2: Join with no unique VIndex
22:47

- There are scoping rules that we should be careful of when we convert ON clause to a WHERE clause. We talk about it when we come to the symbol table.
- As soon as we push a constraint down to a route we see if that affects the route. 

Q: ORDER BY and Having could affect the route.
A: When we come to the ORDER BY and then we see if it can improve the plan. One limitation of a VIndex


## Symbol table

As described in part one, it ensures that all access is validated. If there's a VIndex reference we recognize that. Where is a.id coming from? the symbol table has to help us answer questions like this. Inside a from clause, no correlations allowed. In case of an on joins only participants of a join are allowed to be referenced. Where clause can only see what is described in the from sequence.

- V3 looks at select clause first and then if it can't find it will search the from clause.
- One symbol table per select
- Symbol table resolves symbol based on state
