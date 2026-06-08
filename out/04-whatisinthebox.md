[← Business Requirement](03-businessrequirement.md) · [Index](00-index.md) · [Next →](04b-firsttry.md)

---

# What's in the Box

**Simple stack. High throughput.**

![Technology stack](../resources/pictures/04-whatisonthebox-technologies.png)

| Layer | Technology |
|---|---|
| Message stream | **Apache Kafka** |
| Storage | **MongoDB** (replica set) |
| Event processing | **MongoDB Change Streams** |
| Application | **Node.js** (KafkaJS) |
| Observability | **Prometheus + Grafana** |
| Orchestration | **Kubernetes** |

No Spark. No Flink. No Hadoop. No dedicated data platform — and that was the point.

---

[← Business Requirement](03-businessrequirement.md) · [Index](00-index.md) · [Next →](04b-firsttry.md)
