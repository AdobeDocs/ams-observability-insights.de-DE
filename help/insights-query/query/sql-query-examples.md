---
title: SQL-Abfragebeispiele
description: Erkunden Sie SQL-Abfragebeispiele für die Observability Insights-Telemetrie.
source-git-commit: 4ad45ae5cbb2a7b348fdad1cd2681a72be12f5eb
workflow-type: tm+mt
source-wordcount: '311'
ht-degree: 10%
---

## SQL-Abfragebeispiele

Hier ein vollständiges Beispiel. Für diese Abfrage:

- Sie wählen `Transaction` als Entität.
- Mit `SELECT` berechnen Sie die Dauer des 95. Perzentils.
- Die Ergebnisse werden nach Dienst in `FACET` gruppiert.
- Verwenden Sie `TIMESERIES AUTO`, um die Daten über ein Intervall mit automatischer Größe darzustellen.

```sql
FROM Transaction SELECT percentile(duration, 95) AS p95 FACET service TIMESERIES AUTO
```

Dadurch wird pro Dienst eine Zeile in einem Liniendiagramm erzeugt, wobei standardmäßig die 10 meistgenutzten Dienste angezeigt werden.

### Grundlegende Abfrage eingehender Anfragen

+++Abfrage anzeigen

Durchschnittliche Reaktionszeit aller eingehenden Anfragen in Millisekunden:

```sql
SELECT average(duration) FROM Transaction
```

+++

### Attributnamen, die Punkte enthalten

+++Abfrage anzeigen

Punktierte Namen müssen nicht in Anführungszeichen gesetzt werden:

```sql
SELECT count(*) FROM Transaction WHERE http.statusCode >= 500 FACET http.route
```

+++

### Abfrage mit mehreren Spalten

+++Abfrage anzeigen

Mehrere Aggregate in einer Abfrage werden zu Spalten in einer Tabelle:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, percentile(duration, 95) AS p95, max(duration) AS maxMs FROM Transaction FACET service
```

+++

### Bedingte Aggregate in einem Durchgang

+++Abfrage anzeigen

Gesamt, Fehler und 5xx nebeneinander:

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, count(*) FILTER (WHERE http.statusCode >= 500) AS http5xx FROM Transaction FACET service
```

+++

### Abfrage einer kuratierten Metrik

+++Abfrage anzeigen

Durchschnittliche CPU pro Host im Zeitverlauf:

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

+++

### Abfrage einer OpenTelemetry-Metrik nach Namen

+++Abfrage anzeigen

Verwenden Sie `MetricGauge`/`MetricSum` für Metriken ohne kuratierte Entität:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool
```

+++

<a id="what-sql-doesnt-do"></a>

## Was SQL nicht tut

SQL ist bewusst klein. Dadurch bleiben Abfragen sicher, schnell und Mandanten-isoliert. Wenn Sie eine dieser Funktionen benötigen, verwenden Sie die Alternative .

| Funktion | Status | Alternative |
| ------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `SELECT *` | Nicht unterstützt | Listen Sie die gewünschten Attribute auf: `SELECT timestamp, name, duration, traceId …`. |
| Mischung von Rohattributen und Aggregaten | Nicht unterstützt | Verwenden Sie `FACET attr` oder `latest(attr)` / `earliest(attr)`. |
| Mathematik in `SELECT` (`a - b`, `count(*) / 60`) | Nicht unterstützt | **Formel**-Widget (Dashboards). |
| Joins und Unterabfragen | Nicht unterstützt | **Formel**-Widget oder zwei Widgets nebeneinander. |
| `HAVING` | Nicht unterstützt | `ORDER BY alias` + `LIMIT`. |
| `COMPARE WITH` | Nicht unterstützt | Ändern Sie die Zeitauswahl oder legen Sie zwei Widgets auf verschiedene Registerkarten. |
| `FACET CASES` | Nicht unterstützt | Eine `count(*) FILTER (WHERE …)` Spalte pro Fall (siehe [Tutorial 3](../../tutorials/tutorial-3.md#group-values-into-cases-with-filter)). |
| `histogram()` | Nicht unterstützt | Spalten mit `FILTER` in Buckets (siehe [Tutorial 3](../../tutorials/tutorial-3.md#build-a-latency-histogram)). |
| `uniques()` / `keyset()` | Nicht unterstützt | `FACET attr LIMIT n` listet verschiedene Werte auf. Discovery-Abfragen listen Metriken und ausgefüllte Attribute auf. |
| `RLIKE` (Regex) | Nicht unterstützt | `LIKE` / `NOT LIKE` mit `%` und `_` Platzhaltern oder `IN (…)`. |
| Kommentare (`--`, `//`, `/* */`) | Nicht unterstützt | Fügen Sie Notizen in den Widget-Titel oder ein **Text**-Widget ein. |
| Absolute Daten in `SINCE` | Nicht unterstützt | Absoluten Bereich der Zeitauswahl verwenden. |
| Typenkonvertierung (`numeric()`, `string()`) | Nicht erforderlich | Attribute werden in den Katalog eingegeben. |
