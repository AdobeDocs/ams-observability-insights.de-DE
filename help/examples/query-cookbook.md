---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '565'
ht-degree: 0%
---
## Cookbook abfragen {#query-cookbook}

Das Cookbook ist eine Bibliothek mit einsatzbereiten Abfragen, gruppiert nach Thema. Jeder Abfrage wird der Diagrammtyp vorangestellt, der am besten zu ihr passt (in **)**. Zeitklauseln werden ausgeschlossen, da die Zeitauswahl das Fenster steuert.

**In diesem Dokument:** [10.1 Datenerkennung](#101-data-discovery) ・ [10.2 Durchsatz](#102-throughput) ・ [10.3 Latenz](#103-latency) ・ [10.4 Fehler](#104-errors) ・ [10.5 APDEX](#105-apdex--user-experience) ・ [10.6 Services und Hosts](#106-services-hosts--instances)10.7 Abhängigkeiten](#107-downstream-dependencies)100 AEM](#108-aem-adobe-experience-manager) ・ [10.9 Infrastruktur](#109-infrastructure) [ ](#1010-jvm--runtime) ・ [10.10.10.10.0 JVM[

> [!TIP]
> Ersetzen Sie Beispielwerte wie `'checkout'` oder `'web-publish-01'` durch echte Werte aus Ihrem Mandanten. Die folgenden Datenerkennungsabfragen führen sie auf.

### Datenerkennung

**Tabelle** - welche Services Traffic melden:

```sql
SELECT count(*) AS requests, uniqueCount(host) AS hosts FROM Transaction FACET service LIMIT 200
```

**Tabelle** - welche Hosts/Ebenen vorhanden sind:

```sql
SELECT count(*) AS requests FROM Transaction FACET hostTierPrefix, host LIMIT 200
```

**Tabelle** - welche Umgebungen und Regionen vorhanden sind:

```sql
SELECT count(*) AS spans FROM Span FACET environment, region LIMIT 50
```

**Tabelle** — jede Metrikmeldung für Messgeräte mit dem neuesten Wert:

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricGauge FACET metric, unit LIMIT 200
```

**Tabelle** - jede Summe/Zähler-Metrik:

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricSum FACET metric, unit LIMIT 200
```

**Tabelle** - Welche span-Attribute werden ausgefüllt (Beispiel für AEM):

```sql
SELECT count(*) FILTER (WHERE sling.script != '') AS slingScripts, count(*) FILTER (WHERE jcr.operation != '') AS jcrOps, count(*) FILTER (WHERE db.system != '') AS dbCalls, count(*) FILTER (WHERE graphql.operation != '') AS graphql FROM Span FACET service LIMIT 50
```

### Durchsatz

**Number** — Anforderungen insgesamt:

```sql
SELECT count(*) AS requests FROM Transaction
```

**LINE** — Durchsatz-Trend:

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

**Area** — Durchsatz nach Service, Top 8:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

**Bar** — Durchsatz nach HTTP-Methode:

```sql
SELECT count(*) AS requests FROM Transaction FACET http.method LIMIT 10
```

**Tabelle** - URL-Pfade mit der größten Auslastung:

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET urlPath ORDER BY requests DESC LIMIT 50
```

**Number** — unterschiedliche Spuren:

```sql
SELECT uniqueCount(traceId) AS traces FROM Span
```

### Latenz

**Zahl** — p95 Reaktionszeit (Einheit automatisch = ms):

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction
```

**LINE** — p50 / p95 / p99 in einem Diagramm:

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

**Line** — p95 pro Service, die 10 am meisten ausgelasteten:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

**Bar** — langsamster Betrieb um p95:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

**Tabelle** — Latenzprofil pro Vorgang:

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99, max(duration) AS maxMs, stddev(duration) AS stddevMs FROM Transaction FACET name ORDER BY p95 DESC LIMIT 50
```

**Bar** — Anteil der insgesamt aufgewendeten Zeit pro Vorgang (sofern das Zeitbudget angegeben wird):

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Transaction FACET name LIMIT 20
```

**Tabelle** - Trace-ID der langsamsten Einzelanforderung pro Vorgang:

```sql
SELECT count(*) AS calls, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTraceId FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 50
```

**Number** — Anteil der Anfragen, die langsamer als 2 s sind:

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowShare FROM Transaction
```

**Tabelle** - Latenz-Buckets über gefilterte Zählungen (ein Histogramm-Ersatz):

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS under100ms, count(*) FILTER (WHERE duration > 100 AND duration <= 500) AS to500ms, count(*) FILTER (WHERE duration > 500 AND duration <= 2000) AS to2s, count(*) FILTER (WHERE duration > 2000) AS over2s FROM Transaction FACET service LIMIT 20
```

### Fehler

**Zahl** — Fehlerrate %:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction
```

**LINE** — Fehlerrate % pro Service:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

**LINE** — Fehlerzählungs-Trend:

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error' TIMESERIES AUTO
```

**Pie** - HTTP-Status-Klassenmix:

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode IS NOT NULL FACET http.statusCode LIMIT 15
```

**Tisch** - 5xx Hotspots:

```sql
SELECT count(*) AS errors5xx FROM Transaction WHERE http.statusCode >= 500 FACET service, urlPath, http.statusCode LIMIT 50
```

**Tabelle** — Fehler nach Ausnahmetyp:

```sql
SELECT count(*) AS errors, uniqueCount(traceId) AS traces, latest(statusMessage) AS lastMessage FROM Span WHERE status = 'Error' FACET service, error.type LIMIT 50
```

**Tabelle** - Fehler im Vergleich zur Gesamtanzahl nebeneinander:

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service ORDER BY errors DESC LIMIT 20
```

**Tabelle** - letzte fehlgeschlagene Anfragen (Ereignisliste):

```sql
SELECT timestamp, service, name, urlPath, http.statusCode, duration, traceId FROM Transaction WHERE status = 'Error' LIMIT 100
```

### Apdex und Anwendererlebnis

**number** — apdex (t = 500 ms):

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

**LINE** — APDEX pro Dienst:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

**Bar** — schlechtester Apdex nach URL-Pfad (strengerer Zielwert von 300 ms):

```sql
SELECT apdex(duration, 300) AS apdex, count(*) AS requests FROM Transaction WHERE urlPath LIKE '/content/%' FACET urlPath ORDER BY apdex ASC LIMIT 20
```

### Services, Hosts und Instanzen

**Tabelle** - Service-Scorecard:

```sql
SELECT count(*) AS requests, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95, apdex(duration, 500) AS apdex, uniqueCount(host) AS hosts FROM Transaction FACET serviceKey ORDER BY requests DESC LIMIT 50
```

**Tabelle** - pro Instanz (AppRequest schließt Verbraucher ein):

```sql
SELECT count(*) AS requests, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95 FROM AppRequest FACET appInstance ORDER BY requests DESC LIMIT 50
```

**LINE** — Durchsatz pro Host:

```sql
SELECT count(*) AS requests FROM Transaction FACET host TIMESERIES AUTO LIMIT 10
```

**Bar** — Lastenausgleichsprüfung (Lastenverteilung pro Host):

```sql
SELECT count(*) AS requests FROM Transaction FACET host LIMIT 30
```

### Downstream-Abhängigkeiten

**Tabelle** — nachgelagerte Services namens:

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Span WHERE kind = 'Client' AND peer.service != '' FACET service, peer.service LIMIT 50
```

**Tabelle** - Externe Domains namens:

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain ORDER BY calls DESC LIMIT 50
```

**Bar** - Datenbanklatenz nach System und Vorgang:

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95 FROM Span WHERE db.system != '' FACET db.system, db.operation LIMIT 30
```

**Table** — Langsame Datenbank-Namespaces:

```sql
SELECT count(*) AS calls, average(duration) AS avgMs, percentile(duration, 99) AS p99 FROM Span WHERE db.system != '' FACET db.system, db.namespace ORDER BY p99 DESC LIMIT 30
```

**LINE** - Messaging-Durchsatz pro Ziel:

```sql
SELECT count(*) AS messages FROM Span WHERE messaging.system != '' FACET messaging.destination TIMESERIES AUTO LIMIT 10
```

**Tabelle** — Produzent vs Verbraucher pro Thema:

```sql
SELECT count(*) FILTER (WHERE kind = 'Producer') AS produced, count(*) FILTER (WHERE kind = 'Consumer') AS consumed, percentile(duration, 95) AS p95 FROM Span WHERE messaging.system != '' FACET messaging.system, messaging.destination LIMIT 30
```

**Tabelle** — RPC-Methoden:

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE rpc.system != '' FACET rpc.system, rpc.method LIMIT 30
```

### AEM (Adobe Experience Manager)

**Area** - Traffic nach AEM-Ebene (Autoren-/Veröffentlichungs-/Dispatcher-Präfix):

```sql
SELECT count(*) AS requests FROM Transaction WHERE hostTierPrefix != '' FACET hostTierPrefix TIMESERIES AUTO LIMIT 10
```

**LINE** — Veröffentlichungsebene p95:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%publish%' FACET host TIMESERIES AUTO LIMIT 10
```

**LINE** — Autorenebene p95:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction WHERE host LIKE '%author%' FACET host TIMESERIES AUTO LIMIT 10
```

**Tabelle** - Seitenanfragen (`/content`) vs. Servlet-Aufrufe (`/bin`):

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/etc.clientlibs/%') AS clientlibs, count(*) FILTER (WHERE urlPath LIKE '/content/dam/%') AS assets FROM Transaction FACET hostTierPrefix LIMIT 20
```

**Tabelle** - langsamste `/bin` Servlets:

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction WHERE urlPath LIKE '/bin/%' FACET urlPath ORDER BY p95 DESC LIMIT 30
```

**Tabelle** - Sling-Anfrage-URIs:

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction WHERE sling.requestUri != '' FACET sling.requestUri ORDER BY requests DESC LIMIT 100
```

**Tabelle** - langsame Sling-Skripte:

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE sling.script != '' FACET sling.script ORDER BY totalMs DESC LIMIT 50
```

**Bar** - Sling-Servlets nach Zeitfreigabe:

```sql
SELECT percentOfTotal(duration) AS timeShare FROM Span WHERE sling.servlet != '' FACET sling.servlet LIMIT 20
```

**Tabelle** - Langsame JSPs:

```sql
SELECT count(*) AS executions, percentile(duration, 95) AS p95 FROM Span WHERE jsp.path != '' FACET jsp.path ORDER BY p95 DESC LIMIT 50
```

**Bar** - JCR-Vorgangs-Mix:

```sql
SELECT count(*) AS operations FROM Span WHERE jcr.operation != '' FACET jcr.operation LIMIT 20
```

**Tabelle** - heißeste JCR-Pfade:

```sql
SELECT count(*) AS operations, percentile(duration, 95) AS p95, sum(duration) AS totalMs FROM Span WHERE jcr.path != '' FACET jcr.operation, jcr.path ORDER BY totalMs DESC LIMIT 50
```

**Tabelle** - Vorgänge in GraphQL (Headless):

```sql
SELECT count(*) AS calls, percentile(duration, 95) AS p95, count(*) FILTER (WHERE status = 'Error') AS errors FROM Span WHERE graphql.operation != '' FACET graphql.operation ORDER BY calls DESC LIMIT 50
```

**Line** — Dispatcher-Anfragerate pro Host:

```sql
SELECT rate(value) AS requestsPerSecond FROM DispatcherRequestRate FACET host TIMESERIES AUTO
```

**Tabelle** — Woraus eine Transaktion besteht (Segmentaufschlüsselung; Mengeneinheit `ms`):

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown WHERE rootOperation LIKE 'GET /content%' FACET rootOperation, segment ORDER BY calls DESC LIMIT 50
```

### Infrastruktur

**LINE** — CPU pro Host:

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

**LINE** — CPU-Benutzerzeit:

```sql
SELECT average(value) AS userPct FROM CpuUserPercent TIMESERIES AUTO
```

> [!NOTE]
> SQL liest eine Entität pro Abfrage, sodass user/system/iowait/steal vier nebeneinander liegende Widgets benötigen (siehe [Blueprint 11.6](#116-infrastructure-capacity-board) W7-W10). Eine Formel würde sie in eine Zahl zusammenfassen, nicht vier Reihen.

**Bar** — CPU p95 nach Host:

```sql
SELECT percentile(value, 95) AS cpuP95 FROM CpuUtilization FACET host LIMIT 30
```

**Tabelle** - Snapshot zur Hostintegrität:

```sql
SELECT average(value) AS avgCpu, max(value) AS peakCpu, latest(value) AS nowCpu FROM CpuUtilization FACET host ORDER BY peakCpu DESC LIMIT 50
```

**LINE** — belegter Speicher %:

```sql
SELECT average(value) AS memory FROM MemoryUtilization FACET host TIMESERIES AUTO
```

**Line** — Speicherwachstumsrate (Lecksuche):

```sql
SELECT derivative(value) AS memoryTrend FROM MemoryUtilization FACET host TIMESERIES AUTO
```

**Tabelle** — Datenträger pro Mount:

```sql
SELECT latest(value) AS usedPct FROM DiskUtilization FACET host, mountpoint ORDER BY usedPct DESC LIMIT 50
```

**Number** — niedrigste freie Root-Festplatte über Hosts hinweg (Wert in Byte; Einheit festlegen `bytes`):

```sql
SELECT min(value) AS freeBytes FROM DiskFreeBytes
```

**LINE** - Lese- und Schreibdurchsatz auf der Festplatte (je einen):

```sql
SELECT rate(value) AS readBytesPerSec FROM DiskReadRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host TIMESERIES AUTO
```

**LINE** — Netzwerk-Empfang/-Übertragung:

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS txBytesPerSec FROM NetworkTransmitRate FACET host TIMESERIES AUTO
```

**LINE** — Netzwerkfehler:

```sql
SELECT rate(value) AS rxErrors FROM NetworkReceiveErrorRate FACET host TIMESERIES AUTO
```

**Linie** — Lastdurchschnitt:

```sql
SELECT average(value) AS load5m FROM LoadAverage5m FACET host TIMESERIES AUTO
```

**Tabelle** - Top-Prozesse nach CPU und Speicher:

```sql
SELECT average(value) AS cpuPct, max(value) AS peakCpuPct FROM ProcessCpuUtilization FACET host, process.name ORDER BY cpuPct DESC LIMIT 30
```

```sql
SELECT average(value) AS memMb FROM ProcessMemoryUsage FACET host, process.name ORDER BY memMb DESC LIMIT 30
```

**Tabelle** — alle Rohmessgeräte nach Namen (keine kuratierte Entität erforderlich):

```sql
SELECT average(value) AS avgValue, max(value) AS maxValue FROM MetricGauge WHERE metric = 'system.cpu.utilization' FACET host, state LIMIT 50
```

### JVM und Runtime

**Line** — Heap pro Host verwendet (MB):

```sql
SELECT average(value) AS heapMb FROM JvmHeapUsed FACET host TIMESERIES AUTO
```

**Tabelle** — Heap verwendet vs. Limit (zwei Widgets oder eine Formel — siehe [Blueprint 11.7](#117-jvm-health-board)):

```sql
SELECT latest(value) AS heapLimitMb FROM JvmHeapLimit FACET host LIMIT 50
```

**LINE** — JVM-Threads:

```sql
SELECT latest(value) AS threads FROM JvmThreadCount FACET host TIMESERIES AUTO
```

**LINE** — JVM-Prozess-CPU:

```sql
SELECT average(value) AS jvmCpu FROM JvmCpuRecentUtilization FACET host TIMESERIES AUTO
```

**Tabelle** - JVM-Speicherpools (allgemeine Summenmetrik):

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool LIMIT 100
```
