# Part 1: Storage Engines

For revision notes: [Akshat Jain Database Internals Notes](https://github.com/Akshat-Jain/database-internals-notes/tree/main?utm_source=chatgpt.com)

A **storage engine** is the low-level subsystem inside a database that manages how data is physically stored, organized, updated, and retrieved.

It acts as the bridge between:

* the logical database layer (SQL queries, transactions, tables),
* and the actual hardware resources (RAM and disk).

Whenever a query inserts, updates, deletes, or reads data, the storage engine is the component performing the real physical work behind the scenes.

Its responsibilities typically include:

* writing records to disk,
* loading data pages into memory,
* managing indexes,
* organizing file structures,
* handling caching,
* coordinating reads and writes,
* and ensuring durability and recovery.

You can think of the storage engine as the “data organization and persistence layer” of the DBMS.

---

# Storage Engines are Often Pluggable

Some databases allow multiple storage engines to exist within the same DBMS.

This is useful because different workloads require different optimization strategies.

For example, [MySQL Official Site](https://www.mysql.com/?utm_source=chatgpt.com) supports several storage engines, including:

* **InnoDB** → optimized for transactional consistency and crash recovery
* **MyISAM** → simpler engine historically used for read-heavy scenarios
* **RocksDB** → optimized for high write throughput and storage efficiency

In MySQL, the engine can even be selected per table:

```sql id="u2jx9m"
CREATE TABLE users (
    id INT
) ENGINE = InnoDB;
```

This modularity exists because no single storage engine performs best for every access pattern or workload type.

---

# Side Learnings

## PostgreSQL Storage Architecture

[PostgreSQL Official Site](https://www.postgresql.org/?utm_source=chatgpt.com) mainly relies on a single integrated storage engine design instead of exposing multiple pluggable engines like MySQL.

The PostgreSQL community has, however, explored alternative storage implementations such as **zheap**, which aims to improve:

* update efficiency,
* storage utilization,
* and vacuum overhead reduction.

Useful reference:
[Future of Storage in PostgreSQL](https://wiki.postgresql.org/wiki/Future_of_storage?utm_source=chatgpt.com)

The comparison section near the bottom gives a good overview of possible storage engine design directions.

---

# Comparing Databases is Complex

Evaluating databases is difficult because performance depends on many interconnected variables.

Two databases may behave very differently under different workloads even on identical hardware.

Important factors include:

1. Schema structure
2. Concurrent client count
3. Read/write ratio
4. Query complexity
5. Index design
6. Data size
7. Transaction characteristics
8. Hardware and storage devices
9. Available memory
10. Network and replication overhead

Because of this, realistic benchmarking becomes essential.

---

# TPC-C Benchmark

**TPC** stands for **Transaction Processing Performance Council**.

TPC-C is a widely used benchmark designed for measuring **OLTP (Online Transaction Processing)** systems.

Instead of testing only simple reads or writes, it simulates realistic business-style transactional workloads involving:

* concurrent users,
* updates,
* inserts,
* reads,
* and transactional consistency requirements.

Typical scenarios modeled by TPC-C resemble:

* warehouse management,
* payment processing,
* inventory systems,
* and order processing workflows.

The benchmark evaluates how well a database handles:

* throughput,
* latency,
* concurrency,
* transaction processing,
* and scaling behavior under load.

Useful reference:
[Yugabyte TPC-C Benchmark Guide](https://docs.yugabyte.com/preview/benchmark/tpcc-ysql/?utm_source=chatgpt.com)

Benchmarks are not only used for technical comparisons. They are also important in defining:

* infrastructure expectations,
* production capacity,
* and SLA (Service Level Agreement) commitments.

---

# Understanding Storage Engine Trade-Offs

A fundamental principle in database internals is:

> Every storage engine optimizes some operations at the cost of others.

There is no universally optimal design.

Storage engines constantly balance trade-offs involving:

* read speed,
* write speed,
* storage efficiency,
* compression,
* concurrency,
* memory usage,
* recovery complexity,
* and durability guarantees.

---

## Example: Insertion-Order Storage

If records are written in the same order they arrive:

* inserts become cheap,
* sequential writes are efficient,
* and write throughput improves.

However:

* related records may become physically scattered,
* range scans may require more disk seeks,
* and read locality can degrade over time.

This approach is usually favorable for write-heavy systems.

---

## Example: Sorted Storage

If records are maintained in sorted order:

* range queries become faster,
* index traversal improves,
* and nearby records are easier to fetch efficiently.

But maintaining sorted order introduces extra work:

* inserts may trigger page splits,
* data movement becomes necessary,
* and write amplification increases.

This design is often better for read-intensive workloads.

---

Much of database internals is ultimately about engineering efficient compromises between these competing goals.
