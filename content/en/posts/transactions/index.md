---
title: "Transactions"
date: 2025-04-06
description: "Summary of ACID guarantees and isolation levels in database systems."
toc: true
---

This chapter explores how **database systems** manage **concurrency** and **failures** to provide guarantees of **atomicity**, **isolation**, **durability**, and ultimately **serializability**, at the cost of various **performance** and **availability** trade‑offs.

---

### The Slippery Concept of a Transaction

A **transaction** groups several reads and writes into an atomic unit: either all succeed (_commit_) or none (_abort_) :contentReference[oaicite:0]{index=0}.

- Simplifies error handling: just retry on abort.
- Not every application needs them; sometimes guarantees are traded for **performance** or **availability**.

---

### The Meaning of ACID

ACID defines the safety guarantees of transactions:

| Letter | Meaning                                                                                                                           | Responsible               |
| ------ | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| **A**  | **Atomicity**: all‑or‑nothing; partial changes are rolled back on abort. :contentReference[oaicite:1]{index=1}                    | Database engine           |
| **C**  | **Consistency**: transactions preserve application‑defined invariants. :contentReference[oaicite:2]{index=2}                      | Application (constraints) |
| **I**  | **Isolation**: prevents interference between concurrent transactions; ideally serializable. :contentReference[oaicite:3]{index=3} | Database engine           |
| **D**  | **Durability**: committed data survives failures (fsync, WAL, replication). :contentReference[oaicite:4]{index=4}                 | Database engine           |

---

### Single‑Object vs Multi‑Object Operations

- **Single‑object**: atomicity and isolation at the level of one record (e.g., writing a 20 KB JSON document). :contentReference[oaicite:5]{index=5}
- **Multi‑object**: coordinating changes across multiple records (denormalization, secondary indexes) requires classic transactions to avoid inconsistencies. :contentReference[oaicite:6]{index=6}

---

## Weak Isolation Levels

Weaker than **serializability**, offering better performance but exposing various **anomalies**:

| Anomaly      | Read Committed | Snapshot Isolation | Serializable |
| ------------ | -------------- | ------------------ | ------------ |
| Dirty Read   | ❌             | ✅                 | ✅           |
| Dirty Write  | ❌             | ✅                 | ✅           |
| Lost Update  | ❌             | Partial            | ✅           |
| Write Skew   | ❌             | ❌                 | ✅           |
| Phantom Read | ❌             | ✅ (reads only)    | ✅           |

### Read Committed

- Prevents **dirty reads** and **dirty writes**: you never see uncommitted data, nor overwrite it.
- Typical implementation: row‑level locks plus MVCC for lock‑free reads.
<Figure 7-4> :contentReference[oaicite:7]{index=7}

### Snapshot Isolation & Repeatable Read

- Each transaction reads from a **consistent snapshot** taken at its start.
- MVCC retains multiple versions; readers never block writers and vice versa.
- Known as _Repeatable Read_ in PostgreSQL/MySQL and _Serializable_ in Oracle.
<Figure 7-7> :contentReference[oaicite:8]{index=8}

---

## Preventing Lost Updates

When two transactions perform a **read‑modify‑write**, one update can overwrite the other :contentReference[oaicite:9]{index=9}.  
**Solutions**:

- **Atomic update operations** (e.g., `UPDATE counters SET value = value + 1`).
- **SELECT … FOR UPDATE** to explicitly lock rows.
- Automatic detection in engines with SSI. :contentReference[oaicite:10]{index=10}

---

## Write Skew & Phantoms

- **Write Skew**: two transactions read the same data, make decisions, and write to different records, violating business rules (e.g., “at least one doctor must remain on call”). :contentReference[oaicite:11]{index=11}
  <Figure 7-8>
- **Phantoms**: inserting rows that change the result of a prior `SELECT`. Requires **predicate locks** or **index‑range locks**.

---

## Serializability

Ensures that the concurrent execution of transactions is equivalent to some **serial** order.

### 1. Actual Serial Execution

- A single thread processes transactions in memory (VoltDB, Redis, Datomic).
- Transactions must be very short and the working set must fit in RAM. :contentReference[oaicite:12]{index=12} :contentReference[oaicite:13]{index=13}

### 2. Two‑Phase Locking (2PL)

- **Pessimistic**: shared/exclusive locks; readers block writers and vice versa.
- Prevents all anomalies but reduces concurrency and increases latency.
- **Index‑range locks** approximate predicate locks with lower overhead. :contentReference[oaicite:14]{index=14} :contentReference[oaicite:15]{index=15}

### 3. Serializable Snapshot Isolation (SSI)

- **Optimistic** MVCC: transactions proceed without blocking; at _commit_ the system detects illegal dependencies.
- Provides serializability with predictable latency and partition scalability.
<Figure 7-11> :contentReference[oaicite:16]{index=16}

---

## Summary

Transactions simplify error handling and concurrency control, but there is a **trade‑off** between **guarantees** and **performance**.

- **Weak isolation**: high performance, vulnerable to anomalies.
- **Serializability**: safe, but at the cost of latency or scalability.

Choosing the right isolation level—or combining techniques (locks, MVCC, SSI)—is key to maintaining **consistency** without sacrificing **performance**.
