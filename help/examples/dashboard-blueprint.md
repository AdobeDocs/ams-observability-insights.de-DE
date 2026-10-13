---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '1311'
ht-degree: 1%
---
## Dashboard-Blueprints {#dashboard-blueprints}

Jeder Blueprint ist ein vollständiges, komplexes Dashboard. Es listet das Ziel, das Layout der Registerkarten und Abschnitte sowie jedes Widget mit seinem Diagrammtyp, seiner Einheit, den Schwellenwerten, seiner Größe und seiner Abfrage auf. Die Größen werden im 12-spaltigen Raster `width×height`. Jeden von oben nach unten erstellen und dann ziehen, um ihn anzupassen.

| Blueprint | Zielgruppe | Lichter |
| ------------------------------------------------------------------------------------ | --------------------- | ---------------------------------------------------------------------------------- |
| [11.1 Goldene Signale / SLO](#111-golden-signals--slo-board) | SRE, auf Abruf | KPI-Kacheln mit Schwellenwerten, Apdex, Fehler %, Sättigung |
| [11.2 Fehlerklassifizierung](#112-error-triage-board) | auf Abruf | Fehlerklassen, 5xx Hotspots, fehlgeschlagene Anfragen mit Trace-IDs |
| [11.3 Latenz - Vertiefung](#113-latency-deep-dive-board) | Performance-Ingenieure | Perzentile, Latenz-Buckets, Variabilität, Transaktionszusammensetzung, langsamste Spuren |
| [11.4 Downstream-Abhängigkeiten](#114-downstream-dependencies-board) | Dienstbesitzer | Ausgehendes HTTP, DB, Messaging, RPC |
| [11.5 AEM Author / Publish / Dispatcher](#115-aem-author--publish--dispatcher-board) | AEM-Teams | Abschnitte pro Ebene, Sling/JSP/JCR, Dispatcher-Last |
| [11.6 Infrastrukturkapazität](#116-infrastructure-capacity-board) | Plattform/Kapazität | CPU-Modi, Arbeitsspeicher, Festplatte, Netzwerkraten, Formeln |
| [11.7 JVM-Status](#117-jvm-health-board) | Java-Teams | Heap %-Formeln, Heap pro Host und Pool, Threads, JVM CPU |
| [11.8 Executive / NOC Wand](#118-executive--noc-wall-board) | NOC, Führung | KPI-Kacheln, Verfügbarkeitsformel, Traffic nach Ebene, SLO-Hinweis |
| [11,9-Verhältnisse und Reserven](#119-ratios--headroom-board-formula-showcase) | Alle | Formel-Widget-Showcase |
| [11.10 Service-Explorer](#1110-service-explorer-variable-driven) | Jedes Team | Variablengesteuert, wiederverwendbar |

**Diagrammlegende:** **N** = Zahl, **L** = Linie, **A** = Fläche, **B** = Balken, **P** = Kreis, **T** = Tabelle.

&#x200B;---

### Goldene Signale / SLO-Board

**Ziel:** ein Bildschirm, der antwortet: „Ist die Plattform gerade gesund?“ — Traffic, Fehler, Latenz, Sättigung.
**Zeitauswahl:** Letzte 1 Stunde (Vorfall) oder 24 Stunden (tägliche Überprüfung).

```text
┌──────────── Section: SLO KPIs ────────────────────────────────┐
│ N Requests │ N Error % │ N p95 │ N Apdex │ N Slow % │ N Hosts  │  (6 × 2w×2h)
├──────────── Section: Traffic & Errors ────────────────────────┤
│ A Throughput by service (6w) │ L Error % by service (6w)       │
├──────────── Section: Latency ─────────────────────────────────┤
│ L p50/p95/p99 (6w)           │ L Apdex by service (6w)         │
├──────────── Section: Saturation ──────────────────────────────┤
│ L CPU by host (4w) │ L Memory by host (4w) │ L Load 5m (4w)    │
└───────────────────────────────────────────────────────────────┘
```

**W1 ・ Anfragen** — N, 2×2

```sql
SELECT count(*) AS requests FROM Transaction
```

**W2 ・ Fehler %** — N, Einheit `%`, 2×2. Schwellenwerte: `0 → Green`, `1 → Amber`, `5 → Red`.

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction
```

**W3 ・ p95 Latenz** — N, Einheit `ms`, 2×2. Schwellenwerte: `0 → Green`, `800 → Amber`, `2000 → Red`.

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction
```

**W4 ・ APDEX** — N, 2×2. Schwellenwerte: `0 → Red`, `0.7 → Amber`, `0.85 → Green` (die höchste passende Band gewinnt, sodass niedrige Werte rot bleiben).

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

**W5 ・ Anfragen > 2 s** — N, Einheit `%`, 2×2. Schwellenwerte: `0 → Green`, `2 → Amber`, `10 → Red`.

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction
```

**W6 ・ Aktive Hosts** — N, 2×2

```sql
SELECT uniqueCount(host) AS hosts FROM Transaction
```

**W7 ・ Durchsatz nach Service** — A, 6×4

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**W8 ・ Fehler % nach Dienst** — L, Einheit `%`, 6×4

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**W9 ・ Latenzperzentile** — L, Einheit `ms`, 6×4

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

**W10 ・ Apdex nach Service** — L, 6×4

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**W11 ・ CPU nach Host** — L, 4×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO LIMIT 10
```

**W12 ・ Speicher nach Host** - L, 4×4

```sql
SELECT average(value) AS memory FROM MemoryUtilization FACET host TIMESERIES AUTO LIMIT 10
```

**W13 ・ Last 5m** — L, 4×4

```sql
SELECT average(value) AS load5m FROM LoadAverage5m FACET host TIMESERIES AUTO LIMIT 10
```

&#x200B;---

### Pinnwand für Fehlersuche

**Ziel:** von „Fehlerrate ist hoch“ zu „Dieser Endpunkt, diese Ausnahme, diese Verfolgung“ in drei Klicks.

```text
Tab "Overview"
│ N Errors │ N Error % │ N Affected traces │ N 5xx count │                 (4 × 3w×2h)
│ L Errors by service (8w) │ P Status code mix (4w)                       │
│ T Errors vs total by service (12w)                                       │
Tab "Drill-down"
│ T 5xx hot spots: service × path × code (12w)                             │
│ T Exception types (6w) │ T Downstream failures (6w)                      │
│ T Latest failing requests – event list (12w×6h)                          │
```

**Registerkarte „Übersicht“**

**W1 ・ Fehler** — N, Schwellenwerte `0 → Green`, `1 → Amber`, `100 → Red`

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error'
```

**W2 ・ Fehler %** — N, Einheit `%`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction
```

**W3 ・ Betroffene Spuren** — N

```sql
SELECT uniqueCount(traceId) AS traces FROM Span WHERE status = 'Error'
```

**W4 ・ HTTP 5xx** — N, Schwellenwerte `0 → Green`, `1 → Red`

```sql
SELECT count(*) AS http5xx FROM Transaction WHERE http.statusCode >= 500
```

**W5 ・ Fehler nach Service** — L, 8×4

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error' FACET service TIMESERIES AUTO LIMIT 10
```

**W6 ・ Status-Code-Mischung** — P, 4×4

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode >= 400 FACET http.statusCode LIMIT 10
```

**W7 ・ Fehler vs. insgesamt nach Service** — T, 12×4

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentage(*, WHERE status = 'Error') AS errorPct, count(*) FILTER (WHERE http.statusCode >= 500) AS http5xx, count(*) FILTER (WHERE http.statusCode >= 400 AND http.statusCode < 500) AS http4xx FROM Transaction FACET service ORDER BY errors DESC LIMIT 30
```

**Registerkarte „Aufschlüsselung“**

**W8 ・ 5xx Hotspots** — T, 12×5

```sql
SELECT count(*) AS errors, percentile(duration, 95) AS p95, latest(traceId) AS lastTraceId FROM Transaction WHERE http.statusCode >= 500 FACET service, urlPath, http.statusCode ORDER BY errors DESC LIMIT 100
```

**W9 ・ Ausnahmetypen** — T, 6×5

```sql
SELECT count(*) AS errors, latest(statusMessage) AS lastMessage FROM Span WHERE status = 'Error' AND error.type != '' FACET service, error.type ORDER BY errors DESC LIMIT 50
```

**W10 ・ Downstream-Ausfälle** — T, 6×5

```sql
SELECT count(*) AS failures, percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND status = 'Error' FACET service, peer.service, externalDomain ORDER BY failures DESC LIMIT 50
```

**W11 ・ Neueste fehlgeschlagene Anfragen** — T, 12×6

```sql
SELECT timestamp, service, host, name, urlPath, http.statusCode, duration, statusMessage, traceId FROM Transaction WHERE status = 'Error' LIMIT 200
```

&#x200B;---

### Deep-Dive-Board mit Latenz

**Ziel:** finden _wo_ wie viel Zeit vergeht - welche Vorgänge, welche Segmente, wie konsistent.

```text
│ L p50/p95/p99 (8w)               │ N p99 (4w)                │
│ B Time share by operation (6w)   │ B Slowest ops by p95 (6w) │
│ T Latency profile per operation (12w)                         │
│ T Latency buckets per service (6w) │ B Variability (6w)       │
│ T Transaction composition – SpanBreakdown (12w)               │
│ T Slowest trace per operation (12w)                           │
```

**W1 ・ Perzentile** — L, `ms`, 8×4

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

**W2 ・ p99** — N, `ms`, Schwellenwerte `0 → Green`, `1500 → Amber`, `4000 → Red`, 4×4

```sql
SELECT percentile(duration, 99) AS p99 FROM Transaction
```

**W3 ・ Zeitanteil nach Betrieb** — B, `%`, 6×4

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Transaction FACET name LIMIT 15
```

**W4 ・ Langsamster Betrieb (p95)** — B, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 15
```

**W5 ・ Latenzprofil** — T, 12×5

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99, max(duration) AS maxMs FROM Transaction FACET service, name ORDER BY p95 DESC LIMIT 100
```

**W6 ・ Latenz-Buckets** — T, 6×4

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS fast, count(*) FILTER (WHERE duration > 100 AND duration <= 500) AS ok, count(*) FILTER (WHERE duration > 500 AND duration <= 2000) AS slow, count(*) FILTER (WHERE duration > 2000) AS verySlow FROM Transaction FACET service LIMIT 20
```

**W7 ・ Latenzvariabilität** - B, `ms`, 6×4

```sql
SELECT stddev(duration) AS stddevMs FROM Transaction FACET service ORDER BY stddevMs DESC LIMIT 15
```

**W8 ・ Transaktionszusammensetzung** — T, Einheit `ms`, 12×5

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown FACET service, rootOperation, segment ORDER BY calls DESC LIMIT 100
```

**W9 ・ Langsamste Spur pro Betrieb** — T, 12×5

```sql
SELECT count(*) AS calls, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTraceId, latest(host, duration) AS slowestHost FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 50
```

&#x200B;---

### Pinnwand für nachgelagerte Abhängigkeiten

**Ziel:** sind wir oder sie es? Zustand aller Datenbanken, API, Warteschlangen und RPC, die wir aufrufen.

```text
│ N Outbound calls │ N Outbound error % │ N Outbound p95 │            (3 × 4w×2h)
│ T Downstream services (12w)                                       │
│ L External domain p95 (6w) │ T External domains (6w)              │
│ B DB p95 by system/op (6w) │ T Slow DB namespaces (6w)            │
│ L Messages per destination (6w) │ T Producer vs consumer (6w)     │
│ T RPC methods (12w)                                               │
```

**W1 ・ Ausgehende Anrufe** — N

```sql
SELECT count(*) AS calls FROM Span WHERE kind = 'Client'
```

**W2 ・ Ausgehender Fehler %** — N, `%`, Schwellenwerte `0 → Green`, `2 → Amber`, `10 → Red`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE kind = 'Client'
```

**W3 ・ Ausgehender p95**-n, `ms`

```sql
SELECT percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client'
```

**W4 ・ Downstream Services** — T, 12×5

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE kind = 'Client' FACET service, peer.service ORDER BY calls DESC LIMIT 100
```

**W5 ・ Externe Domäne p95** — L, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain TIMESERIES AUTO LIMIT 8
```

**W6 ・ Externe Domains** — T, 6×4

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain ORDER BY calls DESC LIMIT 50
```

**W7 ・ DB P95** — B, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Span WHERE db.system != '' FACET db.system, db.operation ORDER BY p95 DESC LIMIT 20
```

**W8 ・ Langsame DB-Namespaces** - T, 6×4

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 99) AS p99, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE db.system != '' FACET service, db.system, db.namespace ORDER BY p99 DESC LIMIT 50
```

**W9 ・ Nachrichten pro Ziel** — L, 6×4

```sql
SELECT count(*) AS messages FROM Span WHERE messaging.system != '' FACET messaging.destination TIMESERIES AUTO LIMIT 10
```

**W10 ・ Hersteller vs Verbraucher** — T, 6×4

```sql
SELECT count(*) FILTER (WHERE kind = 'Producer') AS produced, count(*) FILTER (WHERE kind = 'Consumer') AS consumed, percentile(duration, 95) AS p95 FROM Span WHERE messaging.system != '' FACET messaging.system, messaging.destination LIMIT 50
```

**W11 ・ RPC-** — T, 12×4

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE rpc.system != '' FACET service, rpc.system, rpc.method ORDER BY calls DESC LIMIT 50
```

&#x200B;---

### AEM-Autoren-/Veröffentlichungs-/Dispatcher-Board

**Ziel** Ein AEM Operations Board, gruppiert nach Ebene, das das klassische New Relic AEM-Layout widerspiegelt.
Verwenden Sie **Abschnittsheader** namens `PUBLISH`, `AUTHOR`, `DISPATCHER` und End-Widget-Titel mit `- PUBLISH`/`- AUTHOR`, um den Farb-Cue zu erhalten.

```text
Tab "Tiers"
├─ Section: OVERVIEW
│ A Traffic by tier (8w) │ T Content mix by tier (4w)
├─ Section: PUBLISH
│ N Requests - PUBLISH │ N Error % - PUBLISH │ N p95 - PUBLISH │ N Apdex - PUBLISH
│ L p95 by host - PUBLISH (6w) │ L CPU - PUBLISH (6w)
│ L Heap - PUBLISH (6w) │ T Slowest /bin servlets - PUBLISH (6w)
├─ Section: AUTHOR
│ N Requests - AUTHOR │ N Error % - AUTHOR │ N p95 - AUTHOR │ N Active authors' hosts - AUTHOR
│ L p95 by host - AUTHOR (6w) │ L Heap - AUTHOR (6w)
├─ Section: DISPATCHER
│ L Dispatcher req/s (8w) │ L CPU - DISPATCHER (4w)
Tab "Code hot spots"
│ T Slow Sling scripts (6w) │ T Slow JSPs (6w)
│ T Hottest JCR paths (6w)  │ B JCR operation mix (6w)
│ T Sling request URIs (6w) │ T GraphQL operations (6w)
│ B Sling servlet time share (12w)
```

**Übersicht**

**W1 ・ Traffic nach Stufe** — A, 8×4

```sql
SELECT count(*) AS requests FROM Transaction WHERE hostTierPrefix != '' FACET hostTierPrefix TIMESERIES AUTO LIMIT 10
```

**W2 ・ Content-Mix nach Stufe** - T, 4×4

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/content/dam/%') AS assets FROM Transaction FACET hostTierPrefix LIMIT 20
```

**Abschnitt Veröffentlichen**

**W3 ・ Anfragen - VERÖFFENTLICHEN** — N

```sql
SELECT count(*) AS requests FROM Transaction WHERE host LIKE '%publish%'
```

**W4 ・ Fehler % -**: N, `%`, `0 → Green`, `1 → Amber`, `5 → Red`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction WHERE host LIKE '%publish%'
```

**W5 ・ p95 - VERÖFFENTLICHEN** — N, `ms`, `0 → Green`, `1000 → Amber`, `3000 → Red`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%publish%'
```

**W6 ・ APDEX - VERÖFFENTLICHEN** — N, Schwellenwerte `0 → Red`, `0.7 → Amber`, `0.85 → Green`

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction WHERE host LIKE '%publish%'
```

**W7 ・ p95 nach Host - PUBLISH** — L, `ms`, 6×4

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO LIMIT 10
```

**W8 ・ CPU - PUBLISH** — L, `%`, 6×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO
```

**W9 ・ Heap - VERÖFFENTLICHEN** — L, `MB`, 6×4

```sql
SELECT average(value) AS heapMb FROM JvmHeapUsed WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO
```

**W10 ・ Langsamste /bin-Servlets - VERÖFFENTLICHEN** - T, 6×4

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction WHERE host LIKE '%publish%' AND urlPath LIKE '/bin/%' FACET urlPath ORDER BY p95 DESC LIMIT 30
```

**Autorenabschnitt** - Duplizieren Sie W3-W9 und ändern Sie `'%publish%'` in `'%author%'` und das Titelsuffix in `- AUTHOR`. Zusätzliches Widget:

**W11 ・ Autoren-Hosts aktiv - AUTOR** — N

```sql
SELECT uniqueCount(host) AS hosts FROM Transaction WHERE host LIKE '%author%'
```

**Dispatcher-Abschnitt**

**W12 ・ Dispatcher req/s** — L, `req/s`, 8×4

```sql
SELECT rate(value) AS requestsPerSecond FROM DispatcherRequestRate FACET host TIMESERIES AUTO
```

**W13 ・ CPU - DISPATCHER** — L, `%`, 4×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization WHERE host LIKE '%dispatcher%' FACET host TIMESERIES AUTO
```

**Registerkarte „Code-Hotspots“**

**W14 ・ Langsame Sling-Skripte** — T

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE sling.script != '' FACET sling.script ORDER BY totalMs DESC LIMIT 50
```

**W15 ・ Langsame JSPs** - T

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95 FROM Span WHERE jsp.path != '' FACET jsp.path ORDER BY p95 DESC LIMIT 50
```

**W16 ・ Heißeste JCR-Pfade** — T

```sql
SELECT count(*) AS operations, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE jcr.path != '' FACET jcr.operation, jcr.path ORDER BY totalMs DESC LIMIT 50
```

**W17 ・ JCR-Betriebsmix** — B

```sql
SELECT count(*) AS operations FROM Span WHERE jcr.operation != '' FACET jcr.operation LIMIT 20
```

**W18 ・ Sling-Anfrage-URIs** — T

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction WHERE sling.requestUri != '' FACET sling.requestUri ORDER BY requests DESC LIMIT 100
```

**W19 ・ GraphQL-Betrieb** — T

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE graphql.operation != '' FACET graphql.operation ORDER BY calls DESC LIMIT 50
```

**W20 ・ Sling Servlet-Zeitfreigabe** - B, `%`, 12×4

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Span WHERE sling.servlet != '' FACET sling.servlet LIMIT 20
```

&#x200B;---

### Infrastrukturkapazitäts-Board

**Ziel:** Kapazitätsplanung und Sättigungssuche über Hosts hinweg.
**Zeitauswahl:** 7 T für Trends (Datenaggregationen machen das schnell), 1 H für Vorfälle.

```text
Tab "Compute"
│ N Hosts reporting │ N Peak CPU % │ N Peak memory % │ N Max load 5m │
│ L CPU by host (6w) │ L Memory by host (6w)                         │
│ L CPU user / system / iowait / steal (4 × 3w)                     │
│ T Host snapshot (12w)                                              │
│ T Top processes by CPU (6w) │ T Top processes by memory (6w)       │
Tab "Storage & Network"
│ T Disk used per mount (6w) │ N Lowest free root disk (6w)          │
│ L Disk read (6w) │ L Disk write (6w)                               │
│ L Network rx (6w) │ L Network tx (6w)                              │
│ L Total network I/O for one host – formula (12w)                   │
```

**W1 ・ Hosts-Reporting** — N

```sql
SELECT uniqueCount(host) AS hosts FROM CpuUtilization
```

**W2 ・ Spitzenwert CPU** — N, `%`, Schwellenwerte `0 → Green`, `75 → Amber`, `90 → Red`

```sql
SELECT max(value) AS peakCpu FROM CpuUtilization
```

**W3 ・ Spitzenspeicher** - N, `%`, `0 → Green`, `80 → Amber`, `92 → Red`

```sql
SELECT max(value) AS peakMemory FROM MemoryUtilization
```

**W4 ・ Max. Belastung 5m** — N

```sql
SELECT max(value) AS maxLoad FROM LoadAverage5m
```

**W5 ・ CPU nach Host** - L, `%`, 6×4

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

**W6 ・ Speicher nach Host** - L, `%`, 6×4

```sql
SELECT average(value) AS memory FROM MemoryUtilization FACET host TIMESERIES AUTO
```

**W7-W10 ・ CPU nach Modus** — L, `%`, jeweils ×:4

```sql
SELECT average(value) AS userPct FROM CpuUserPercent FACET host TIMESERIES AUTO LIMIT 10
```

```sql
SELECT average(value) AS systemPct FROM CpuSystemPercent FACET host TIMESERIES AUTO LIMIT 10
```

```sql
SELECT average(value) AS iowaitPct FROM CpuIowaitPercent FACET host TIMESERIES AUTO LIMIT 10
```

```sql
SELECT average(value) AS stealPct FROM CpuStealPercent FACET host TIMESERIES AUTO LIMIT 10
```

**W11 ・ Host-Snapshot** - T, 12×5

```sql
SELECT average(value) AS avgCpu, percentile(value, 95) AS p95Cpu, max(value) AS peakCpu, latest(value) AS nowCpu FROM CpuUtilization FACET host ORDER BY p95Cpu DESC LIMIT 100
```

**W12 ・ Top-Prozesse von CPU** - T, 6×5

```sql
SELECT average(value) AS cpuPct, max(value) AS peakPct FROM ProcessCpuUtilization FACET host, process.name ORDER BY cpuPct DESC LIMIT 30
```

**W13 ・ Top-Prozesse nach Speicher** - T, `MB`, 6×5

```sql
SELECT average(value) AS memMb, max(value) AS peakMb FROM ProcessMemoryUsage FACET host, process.name ORDER BY memMb DESC LIMIT 30
```

**Registerkarte „Speicher und Netzwerk“**

**W14 ・ Verwendete Festplatte pro Mount** — T, `%`

```sql
SELECT latest(value) AS usedPct, max(value) AS peakPct FROM DiskUtilization FACET host, mountpoint ORDER BY usedPct DESC LIMIT 50
```

**W15 ・ Niedrigste freie Wurzelscheibe** — N, `bytes`, `0 → Red`, `5000000000 → Amber`, `20000000000 → Green`

```sql
SELECT min(value) AS freeBytes FROM DiskFreeBytes
```

**W16 / W17 ・ Festplattendurchsatz** — L, `bytes/s`

```sql
SELECT rate(value) AS readBytesPerSec FROM DiskReadRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host TIMESERIES AUTO
```

**W18 / W19 ・ Netzwerkdurchsatz** — L, `bytes/s`

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS txBytesPerSec FROM NetworkTransmitRate FACET host TIMESERIES AUTO
```

**W20 ・ Netzwerk-E/A insgesamt für einen Host (Formel)** - Formula-Widget, L, `bytes/s`, 12×4. Formeln können nicht `FACET` werden und `rate()` müssen auf einen Host angewendet werden (siehe [Kuratierte Metrikentitäten](#53-curated-metric-entities)), also heften Sie den Host an - ersetzen Sie `web-publish-01` durch einen echten Host-Namen aus W5.

`netRx`:

```sql
SELECT rate(value) AS v FROM NetworkReceiveRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

`netTx`:

```sql
SELECT rate(value) AS v FROM NetworkTransmitRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

Formel:

```text
netRx + netTx
```

&#x200B;---

### JVM-Health Board

**Ziel:** Heap-Druck, Thread-Lecks und JVM-CPU-Spitzen zu erkennen, bevor sie zu Ausfällen führen.

```text
│ N Heap used % (formula) │ N Peak threads │ N JVM CPU % │ N Max heap MB │
│ L Heap used % over time – formula (12w)                                │
│ L Heap used by host (6w) │ T Heap limit by host (6w)                    │
│ L Threads by host (6w)   │ L JVM CPU by host (6w)                       │
│ T Memory pools (12w)                                                   │
```

**W1 ・ Verwendeter Heap % (Formel)** — Formel, N, `%`, `0 → Green`, `75 → Amber`, `90 → Red`

`used`:

```sql
SELECT average(value) AS v FROM JvmHeapUsed
```

`limit`:

```sql
SELECT average(value) AS v FROM JvmHeapLimit
```

Formel:

```text
(used / limit) * 100
```

**W2 ・ Spitzengewinde** — N

```sql
SELECT max(value) AS peakThreads FROM JvmThreadCount
```

**W3 ・ JVM CPU** — N, `%`

```sql
SELECT average(value) AS jvmCpu FROM JvmCpuRecentUtilization
```

**W4 ・ Maximal verwendeter Heap** — N, `MB`

```sql
SELECT max(value) AS maxHeapMb FROM JvmHeapUsed
```

**W5 ・ Verwendeter Heap % im Zeitverlauf (Formel)** — Formel, L, `%`, 12×4

`used`:

```sql
SELECT average(value) AS v FROM JvmHeapUsed TIMESERIES AUTO
```

`limit`:

```sql
SELECT average(value) AS v FROM JvmHeapLimit TIMESERIES AUTO
```

Formel:

```text
(used / limit) * 100
```

&#x200B;> [!NOTE]
> Um eine Formel auf einen Host zu beschränken, fügen Sie `WHERE host = 'my-publish-01'` zu (**)** Abfragen hinzu.

**W6 ・ Vom Host verwendeter Heap** — L, `MB`, 6×4

```sql
SELECT average(value) AS heapMb FROM JvmHeapUsed FACET host TIMESERIES AUTO
```

**W7 ・ Heap-Limit nach Host** — T, `MB`, 6×4

```sql
SELECT latest(value) AS heapLimitMb FROM JvmHeapLimit FACET host LIMIT 50
```

**W8 ・ Threads nach Host** — L, 6×4

```sql
SELECT latest(value) AS threads FROM JvmThreadCount FACET host TIMESERIES AUTO
```

**W9 ・ JVM CPU nach Host** — L, `%`, 6×4

```sql
SELECT average(value) AS jvmCpu FROM JvmCpuRecentUtilization FACET host TIMESERIES AUTO
```

**W10 ・ Speicherpools** - T, 12×5

```sql
SELECT latest(value) AS bytesUsed, max(value) AS peakBytes FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool ORDER BY peakBytes DESC LIMIT 100
```

&#x200B;---

### 11.8 Executive / NOC Wandplatte

**Ziel** Ein brillanter, farbcodierter Wandbildschirm. Hauptsächlich Anzahl Widgets mit Schwellenwerten, ein Trend pro Zeile.

```text
│ N Requests │ N Availability % (formula) │ N Apdex │ N p95 │ N 5xx │ N Hosts │
│ L Throughput (12w)                                                          │
│ B Requests by service (6w) │ P Traffic by tier (6w)                         │
│ Text: "SLO: 99.9 % availability, p95 < 800 ms. Owner: platform-sre" (12w×1h) │
```

**W1 ・ Anfragen** — N

```sql
SELECT count(*) AS requests FROM Transaction
```

**W2 ・ Verfügbarkeit % (Formel)** — Formel, N, `%`, `0 → Red`, `99 → Amber`, `99.9 → Green`

`ok`:

```sql
SELECT count(*) AS v FROM Transaction WHERE status != 'Error'
```

`total`:

```sql
SELECT count(*) AS v FROM Transaction
```

Formel:

```text
(ok / total) * 100
```

**W3 ・ APDEX** — N, Schwellenwerte `0 → Red`, `0.7 → Amber`, `0.85 → Green`

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

**W4 ・ p95** — N, `ms`, Schwellenwerte `0 → Green`, `800 → Amber`, `2000 → Red`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction
```

**W5 ・ 5xx** — N, Schwellenwerte `0 → Green`, `1 → Amber`, `50 → Red`

```sql
SELECT count(*) AS http5xx FROM Transaction WHERE http.statusCode >= 500
```

**W6 ・ Hosts** — N

```sql
SELECT uniqueCount(host) AS hosts FROM Transaction
```

**W7 ・ Durchsatz** - L, 12×4

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

**W8 ・ Anfragen nach Service** — B, 6×4

```sql
SELECT count(*) AS requests FROM Transaction FACET service LIMIT 10
```

**W9 ・ Traffic nach Stufe** — P, 6×4

```sql
SELECT count(*) AS requests FROM Transaction WHERE hostTierPrefix != '' FACET hostTierPrefix LIMIT 10
```

**W10 ・ SLO Hinweis** — Text-Widget, 12×1.

&#x200B;---

### Verhältnisse &amp; Headroom Board (Formula Showcase)

**Ziel:** abgeleitete KPIs, für die mehr als eine Abfrage erforderlich ist.

**W1 ・ Anforderungen pro Host** — Formel, L, `req/host`

`requests`:

```sql
SELECT count(*) AS v FROM Transaction TIMESERIES AUTO
```

`hosts`:

```sql
SELECT uniqueCount(host) AS v FROM Transaction TIMESERIES AUTO
```

Formel:

```text
requests / hosts
```

**W2 ・ 5xx Anteil aller Fehler** — Formel, N, `%`

`e5xx`:

```sql
SELECT count(*) AS v FROM Transaction WHERE http.statusCode >= 500
```

`errors`:

```sql
SELECT count(*) AS v FROM Transaction WHERE http.statusCode >= 400
```

Formel:

```text
(e5xx / errors) * 100
```

**W3 ・ Zeit in der Datenbank im Vergleich zur gesamten Serverzeit** - Formel, L, `%`

`db`:

```sql
SELECT sum(duration) AS v FROM Span WHERE db.system != '' TIMESERIES AUTO
```

`server`:

```sql
SELECT sum(duration) AS v FROM Transaction TIMESERIES AUTO
```

Formel:

```text
(db / server) * 100
```

**W4 ・ Heap-Reserven (MB)** - Formel, L, `MB`

`limit`:

```sql
SELECT average(value) AS v FROM JvmHeapLimit TIMESERIES AUTO
```

`used`:

```sql
SELECT average(value) AS v FROM JvmHeapUsed TIMESERIES AUTO
```

Formel:

```text
limit - used
```

**W5 ・ Netzwerk-Out/In-Verhältnis für einen Host** — Formel, L (Host anheften; siehe [Kuratierte Metrikentitäten](#53-curated-metric-entities))

`rx`:

```sql
SELECT rate(value) AS v FROM NetworkReceiveRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

`tx`:

```sql
SELECT rate(value) AS v FROM NetworkTransmitRate WHERE host = 'web-publish-01' TIMESERIES AUTO
```

Formel:

```text
tx / rx
```

**W6 ・ Traffic-Verhältnis Veröffentlichung vs. Autor** — Formel, N

`publish`:

```sql
SELECT count(*) AS v FROM Transaction WHERE host LIKE '%publish%'
```

`author`:

```sql
SELECT count(*) AS v FROM Transaction WHERE host LIKE '%author%'
```

Formel:

```text
publish / author
```

**W7 ・ Gewichtete Fehler-Budget-Verbrennung (SLO 99,9 %)** — Formel, L

`errors`:

```sql
SELECT count(*) AS v FROM Transaction WHERE status = 'Error' TIMESERIES AUTO
```

`total`:

```sql
SELECT count(*) AS v FROM Transaction TIMESERIES AUTO
```

Formel (Burn-Rate: 1.0 = Budget genau planmäßig verbrauchen):

```text
(errors / total) / 0.001
```

&#x200B;---

### Service-Explorer (variablengesteuert)

**Ziel:** eine Pinnwand, auf die jedes Team auf _seinen Service_ Umgebung verweisen kann.

&#x200B;> [!WARNING]
> Siehe die **aktuelle Build-Beschränkung** in [Tutorial 3, Schritt 8.7](#87-make-dashboards-reusable-with-variables): `{{variables}}` in Widget SQL kann noch nicht in der Vorschau angezeigt oder gespeichert werden. Bis dies behoben ist, ersetzen Sie jedes `{{…}}` durch ein Literal (z. B. `'checkout'`) und verwenden Sie _Duplizieren_ um die Pinnwand pro Dienst zu klonen. Das folgende Design ist bereit für die Ersetzung, wenn End-to-End unterstützt wird.

**Variablen**

| Name | Label | Typ | Multi | Nummerisch | Definition |
| -------- | ------------------- | ------------ | ----- | ------- | ---------------------------------------------------------- |
| `svc` | Dienst | Aus einer Abfrage | ✔ |         | `SELECT count(*) FROM Transaction FACET service LIMIT 200` |
| `env` | Umgebung | Aus einer Abfrage | ✔ |         | `SELECT count(*) FROM Span FACET environment LIMIT 50` |
| `slowMs` | Langsamer Schwellenwert (ms) | Textfeld |       | ✔ | `2000` |

```text
Section: "{{svc}} in {{env}}"
│ N Requests │ N Error % │ N p95 │ N Slow requests │
│ L Throughput by service (6w) │ L p95 by service (6w)   │
│ T Operations (12w)                                     │
│ T Slow requests over {{slowMs}} ms – events (12w)       │
│ T Downstream calls (12w)                               │
```

**W1 ・ Anfragen —{{svc}}** — N

```sql
SELECT count(*) AS requests FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}})
```

**W2 ・ Fehler %** — N, `%`

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}})
```

**W3 ・ p95** — N, `ms`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}})
```

**W4 ・ Langsame Anfragen** — N

```sql
SELECT count(*) AS slow FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) AND duration > {{slowMs}}
```

**W5 ・ Durchsatz nach Service** — L

```sql
SELECT count(*) AS requests FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) FACET service TIMESERIES AUTO
```

**W6 ・ p95 durch Service** — L, `ms`

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) FACET service TIMESERIES AUTO
```

**W7 ・ Betrieb** — T

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction WHERE service IN ({{svc}}) AND environment IN ({{env}}) FACET service, name ORDER BY calls DESC LIMIT 100
```

**W8 ・ Langsame Anfragen (Ereignisse)** — T

```sql
SELECT timestamp, service, host, name, urlPath, duration, traceId FROM Transaction WHERE service IN ({{svc}}) AND duration > {{slowMs}} ORDER BY duration DESC LIMIT 200
```

**W9 ・ Downstream-Anrufe** — T

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE kind = 'Client' AND service IN ({{svc}}) FACET peer.service, externalDomain, db.system ORDER BY calls DESC LIMIT 50
```
