+++
title = "CMU Advanced Database Systems Lect03 Notes"
author = ["Joel Lee"]
tags = ["Database Systems", "CMU Advanced Database Systems", "MVCC" ]
date = "2020-01-04"
draft = false
+++

## In-Memory Multiversion Concurrency Control(MVCC) {#in-memory-multiversion-concurrency-control--mvcc}

-   DBMS maintains multiple physical versions of a single logical object that's modified.
-   Database needs to figure out which is the correct version when a query is launched
-   Proposed in 1978, implemented in InterBase. This was sold to Borland and it was open sourced as Firebird. Firefox is called firefox because Netscape realized the name Firebird was taken


## Benefits of MVCC {#benefits-of-mvcc}

-   Writers don't block readers and in many cases readers don't block writers.
-   MVCC supports time travel queries. You can say: Go get me this tuple as it existed five years ago. This can be useful in finance where you need data for regulations. Postgres had this in the 1980s before it was used outside of Academia.
-   MVCC is more than a concurrency control protocol.


### Snapshot Isolation {#snapshot-isolation}

-   When a txn starts it sees a **Consistent** snapshot of the database that existed at the moment when the txn started.
-   If two txns updates the same object the first writer wins.
-   We get snapshot isolation for free with MVCC. Need to do extra stuff to get Serializable isolation.(refer to paper)


### MVCC Design decisions {#mvcc-design-decisions}

-   Concurrency Control Protocol, Version Storage, Garbage Collection(GC), Index management


#### Approaches to MVC {#approaches-to-mvc}

1.  Timestamp Ordering
    -
2.  Optimistic Concurrency Control
3.  Two-Phase Locking


#### What goes into a tuple? {#what-goes-into-a-tuple}

-   Transaction ID
-   Begin and end timestamp -> Allows us to figure out version timestamp.
-   Pointer pointing to next/previous version(Singly linked list)
-   Additional metadata

This doesn't seem like it's a lot but it depends on what the size of your data is. Overhead could be non-trivial if data is small


#### Multi-Version Timestamp ordering {#multi-version-timestamp-ordering}

-   Read timestamp to keep track of last timestamp which read the tuple.
-   View the example at 23:14
-   How do we know that the transaction committed? We need a global structure to tell us that we're reading from a txn that's committed. Sometimes there's an extra bit used.


#### Multi-version Two-Phase Locking {#multi-version-two-phase-locking}

-   Transactions use tuple's read count field as a shared lock. Txn-id and read-cnt together work as an exclusive lock. If txn-id is zero then the txn acquires the shared lock by incrementing the read-cnt field.
-   When we're done we flip everything back to zero.
-   Most systems do deadlock detection not deadlock prevention
-   If you reach the max value for your timestamp then you get weird behaviour
-   When we wraparound rows that should be deleted now become observable
-   The Postgres solution: Additional flag that denotes a version is frozen. Flip the bit when we're close to the limit and we turn on the vacuum.
-   When we're running the vacuum do we need to ensure that no one is usingt the version that we're using? If we have open transactions when we wrap around everything has to stop.
-   What if we have Multiple wraparounds? We won't compare between multiple wraparounds. Though the version chain will give you info.
-   We use the pointer field to create a latch free version chain per logical tuple. Allows us to find version that is visible. We scan until we find the version we want. Head can be either oldest or newest version.
-   All versions will be stored in local memory regions to avoid contention


### Version storage {#version-storage}

1.  Append Only Storage
2.  Time Travel Storage
3.  Delta Storage


#### Append-only storage {#append-only-storage}

-   All physical versions of tuple are stored in the same table space. Versions are mixed together
-   On every update, append a new version of tuple into an empty space
-   Can go oldest to newst or newest to oldest.
-   In O2N(Old to new) you just append to end of chain but you have ot traverse chain on lookups
-   in N2O(New to old) Have to update index pointers for every version
-   Don't have to traverse chain on lookups


#### Time Travel storage {#time-travel-storage}

-   Main table and a time travel table
-   On every update copy the current version to the time travel table and then update the table.
-   Since main table always has the latest version we can just drop the time travel table when we're doing garbage collection.


#### Delta Storage {#delta-storage}

-   On update, copy only the values that were modified to the delta storage and then overwrite the master version
-   Only copy things that are modified but you have to scan back further and further to find what you want.
-   Always done with newest to oldest. This is done by oracle and MySQL.


### How to handle non-inline attributes {#how-to-handle-non-inline-attributes}

-   Use a reference counter to keep track of how many times we are pointing to an object.
-   Dictionary compression does the exact same thing for you for free.
-   No active transaction can "see"/access the old version.


#### Tuple level {#tuple-level}

-   Look at data in table themselves and decide if we can collection them. Either through background vacuuming or Cooperative Cleaning
-   Look at 55:16. Background vacuuming scans the table and looks for reclaimable versions. It works with any storage. It can get expensive. We keep a bitmap to see what has been modified
-   Cooperative cleaning: txn will know what's active and then you can do lookup through index and just delete accordingly. Update index to point to new head of version chain. One problem that could arise is that but if no one reads the lookup then the tuple will never be cleaned


#### Transaction level GC {#transaction-level-gc}

-   Each txn keeps track of it's read/write set. DBMS determines when all versions created by a txn are no-longer visible. It's too slow but it requires multiple threads to reclaim memory fast enough.


### Index-Management {#index-management}

-   PKey indexes always points to the version chain head. DBMS has to updathe pkey index. If a txn update a tuple's pkey attribute this is a delete then an insert
-   This gets more complicated for secondary indexes. Refer to Uber's article about why they switched from Postgres to MySQL


#### Logical Pointers {#logical-pointers}

-   Unqiue value for tuple that does not change
-   Map logical ID to actual physical location. Primary key vs tuple id


#### Physical Pointer {#physical-pointer}

-   Use the physical address of the head


### Questions? {#questions}

-   Do I malloc ahead of time? We Malloc a big chunk and then slice them up on a per chunk basis.
-   We can't modify two memory locations without acquiring a latch
