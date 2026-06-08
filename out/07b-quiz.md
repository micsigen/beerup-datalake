[← Monitoring](06-monitoring.md) · [Index](00-index.md) · [Next →](07-qa.md)

---

# Kvíz

**Miért vallott kudarcot az ütemezett aggregációs job megközelítés 1 500 msg/mp-nél?**

---

- **A)** Az aggregáció átviteli sebessége alacsonyabb volt a betöltési sebességnél — a lemaradás folyamatosan nőtt ✅
- **B)** A MongoDB replica set nem bírta a párhuzamos írási terhelést
- **C)** A Kafka consumer nem tudta elég gyorsan commitolni az offseteket
- **D)** A Node.js egyszálú, és minden batch-nél blokkolt

---

> *A:* Ha egy job 35 másodpercig fut, a következő már 35 másodpercnyi új adattal indul. A sor sosem fogy el — csak növekszik.

---

[← Monitoring](06-monitoring.md) · [Index](00-index.md) · [Next →](07-qa.md)
