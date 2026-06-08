[← First Tries](04b-firsttry.md) · [Index](00-index.md) · [Next →](06-monitoring.md)

---

# Solution

**From pull to push: event-driven architecture.**

After two failed approaches (Spark, scheduled batch jobs), the breakthrough was switching from *polling* to *streaming*.

```mermaid
graph TD
    A["📬 Kafka<br/>~1500-1700 msg/sec"] -->|Batch inserts<br/>100-500 docs| B["🗄️ MongoDB<br/>trm-satelite<br/>Raw Data · 10h TTL"]
    B -->|Change Stream<br/>INSERT events| C["⚡ Change Stream Processor<br/>Node.js"]
    C -->|Micro-batch buffer<br/>200-500ms window| D["🔄 Aggregation Logic<br/>businessKey + timeSlot"]
    D -->|Bulk write · $inc + upsert| E["🗄️ MongoDB<br/>Aggregated Data"]
    C -->|Save position| F["💾 Resume Token"]
    F -->|Load on restart| C
    E -->|Read| G["🎨 UI / Dashboard"]
    E --> H["⏰ Scheduled Jobs<br/>Recalc · Backfill · Cleanup"]
    B --> H
    C -->|Metrics| I["📊 Prometheus + Grafana"]
```

**Key decisions:**

| Decision | Why it matters |
|---|---|
| **Change Streams** | Events flow in real-time, no re-scanning |
| **Micro-batch (200–500ms)** | Reduces MongoDB round-trips 200–500× |
| **Resume token** | Crash-safe, no data loss on restart |
| **Single processor instance** | No distributed consensus needed |
| **Scheduled jobs** | Backfill, cleanup, and complex metric recalculation |

**Result:** 6,000+ msg/sec capacity · 1–2 sec end-to-end latency · 3–4× headroom

![Architecture concept](../resources/pictures/hypercube.png)

---

[← First Tries](04b-firsttry.md) · [Index](00-index.md) · [Next →](06-monitoring.md)
