[← What's in the Box](04-whatisinthebox.md) · [Index](00-index.md) · [Next →](05-solution.md)

---

# First Tries

**Two dead ends — both caused by the same root problem.**

![First tries](../resources/pictures/04-firsttry-data-lake-002.png)

### Attempt 1 · Apache Spark

The technically correct choice. Kafka → Spark → MongoDB.

**What went wrong:**
- Operational overhead: constant tuning of memory, executors, shuffle partitions
- Resource contention: running Spark on the same infra as the application caused unpredictable failures
- **No metrics. No dashboards.** Every incident was a 2-hour investigation in the dark

### Attempt 2 · Scheduled Aggregation Jobs

Simpler: Kafka → MongoDB raw collection → batch job every 60 seconds.

**What went wrong:**
- Aggregation throughput < ingestion throughput — the queue never caught up
- A 35-second job left 35 seconds of new data already waiting for the next run
- Lag grew from minutes to hours
- **Again: no monitoring.** We discovered the problem when users complained about stale data

---

> The failures weren't Spark or batch jobs. The failure was flying blind.

---

[← What's in the Box](04-whatisinthebox.md) · [Index](00-index.md) · [Next →](05-solution.md)
