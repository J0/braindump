+++
title = "Vitess Deep Dive Session Two"
author = ["Joel Lee"]
tags = ["Vitess"]
date = "2020-01-02"
draft = false
+++

## Filter handles both where and having {#filter-handles-both-where-and-having}

Eight out of nine Opcodes are needed. VTGate has to become a full database engine


## If you knew at the beginning, would you have used MySQL at the VTGate layer instead. {#if-you-knew-at-the-beginning-would-you-have-used-mysql-at-the-vtgate-layer-instead-dot}

This will work if a single VTGate node can handle your result. Otherwise, it won't work. The very fact that you've sharded your database means that you probably don't want all your data to go through one node. This probably won't have scelled though so it's fine that we are where we are today.


## Failovers we need to maintain failovers for mysql scenarios. {#failovers-we-need-to-maintain-failovers-for-mysql-scenarios-dot}

Fact that we're using a storage engine. Route was about to do all 9 operations.


## When we have a transaction at VTGate level does VTGate need to have special knowledge abut binlog {#when-we-have-a-transaction-at-vtgate-level-does-vtgate-need-to-have-special-knowledge-abut-binlog}

VTGate is  stateless. Session information is stored on client side and it is exchanged like a cookie. Start a transaction with one VTGate and continue it using another one. If there are lookup Vindexes and it has to do a delete. VTGate is also a 2PC coordinator then it does perform a 2PC coordination. Most DML's are sent to VTTablet.


## Differences {#differences}

-   We try to preserve the original query  so vtgate can act like a router and not an engine


## Use Case Scenarios of VTGate as a router {#use-case-scenarios-of-vtgate-as-a-router}

-   select a.col from a where a.id=:id. In this scenario, id is the sharding key. Look up which shard the key is in and send the query to that shard. Index tells you given a where clause you don't need to scan all rows. Index restricts number of rows you need to scan. VIndex tells you that you only need to scan this one shard. Intuitive case it's easy to say this is the shard

-select a.col from a where a.id in ::list ask vindex for mapping of indexes to shards. Vindex will build seperate query for each shard.  This is a smart route.

-   Select a.col from a where a.col = :col . The column does not have a sharding key.. Rows can be found in any shard. The query is scatterable if you ran a query and the query does not have external dependencies
-   select a.col, b.col from a join b on b.id=a.id where a.id=:id . For this query. Let's look at something more graspable by the brain item table. Fetch all the items of order X. This is a single id even though it's join because all. This query should be sent to the shard that contains the order id
-   select a.col, b.col from a join b on b.id=a.id there is no where clause so we can't route. The question is whether we can scatter. If we subset the query to a smaller set of roles we would still get the result that we would otherwise have gotten


## What does it mean to scatter a query? {#what-does-it-mean-to-scatter-a-query}

Send your query to all shards and return the results as they come. If there is cross shard dependency then VTGate has to do it. If there is a subquery that looks outside.


## Cross Shard joins {#cross-shard-joins}

-   We are doing a join but join is on two different sharding keys

VTGate has to break up the query into two parts and then do the join

-   Fetch a.id and a.col  we need a.col because it's used in the join. Can be in same keyspace but different shard.
-   Break up query into different routes. No restriction what type of routes each fragment of a query. If there is a scatter that joins with another scatter is going to be way more expensive. Depends on what the routes do
-   A and b on same shard C on different sharding key. Send a and b to one shard and use C to derive  the key.
-   How does vitess break the query into routes?


## The right most route rule {#the-right-most-route-rule}

-   Select a.col, b.col from a join b on sin(b.id) = cos(a.id)
-   this becomes select a.col, a.id from a (Scatter)
-   select b.col from b where sin(b.id) = cos(:\_a\_col)
-   if vitess is going to break a query any expression will become to the right most table. Wherever possible we push down work into mysql itself.

DOn't write a query like this

-   select a.col, b.col from a join b on b.id2=a.id where a.id=:id and b.col > a.col (This is an inequality in the where clause)
-   Cascading joins
-   How do we handle subqueries?
-   select id, count(\*) from a group by id

Number of itme count for each order. The query can be scattered because there is not cross shard dependency

-   select a.col, b.col from a join b where b.id=a.id order by b.id

As long as order by order matches the join order it's fine.

See [Vitess Deep Dive Session One]({{<relref "vitess_deep_dive_session_one.md" >}})
