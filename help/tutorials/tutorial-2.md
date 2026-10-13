---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '344'
ht-degree: 0%
---
## Tutorial 2: Steuern Ihrer Daten {#tutorial-2-control-your-data}

In diesem Tutorial lernen Sie die Funktionen und Klauseln kennen, die aus rohen Zahlen Antworten machen: eindeutige Zahlen, aktuelle Werte, Prozentsätze, Perzentile, Platzhalter, Sortierung und mehrdimensionale Gruppierung.

**In diesem Tutorial:** [7.1 uniqueCount](#71-count-distinct-values-with-uniquecount) ・ [7.2 neueste/früheste](#72-find-the-latest-and-earliest-values) ・ [7,3 Prozentsatz](#73-calculate-percentages) ・ [7,4 Perzentil](#74-understand-latency-with-percentile) ・ [7,5 Beschriftungen](#75-make-results-readable-with-labels) ・ [7,6 Platzhalter und Listen](#76-match-patterns-and-lists) ・ [7,7 Sortierung](#77-sort-and-rank-with-order-by) ・ [7,8 Mehrere Facetten](#78-group-by-several-attributes) ・ [7,9 Frühere Fenster](#79-look-at-a-past-window)

### Unterschiedliche Werte mit `uniqueCount` zählen

Wie viele Hosts und eindeutige Spuren hat jeder Service?

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

`uniqueCount` ist exakt, keine Schätzung.

Um **Auflisten** der einzelnen Werte selbst (was NRQL `uniques()` aufruft), facettieren Sie das -Attribut:

```sql
SELECT count(*) AS requests FROM Transaction FACET host LIMIT 100
```

### Finden Sie die neuesten und frühesten Werte

`latest(attr)` gibt den neuesten Wert und `earliest(attr)` den ältesten zurück. Sie sind nützlich für „aktuelle Status“-Tabellen:

```sql
SELECT latest(value) AS currentCpu FROM CpuUtilization FACET host
```

```sql
SELECT earliest(timestamp) AS firstSeen, latest(timestamp) AS lastSeen FROM Transaction FACET service
```

### Prozentsätze berechnen

`percentage(*, WHERE condition)` gibt den Anteil der Zeilen zurück, die einer Bedingung entsprechen. Dies ist die Fehlerrate:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction
```

Trend pro Service:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

Verfolgen Sie den Anteil langsamer Anfragen:

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction FACET service
```

### Verstehen der Latenz mit `percentile`

Durchschnittswerte blenden Ausreißer aus. Die Perzentile geben an, was die meisten Benutzer erleben (p50) und was die Unglücklichen erleben (p95, p99):

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

&#x200B;> [!TIP]
> P50, P95 und P99 von `Transaction.duration` werden aus Datenaggregationen über lange Fenster bereitgestellt, sodass sie für wöchentliche Ansichten schnell bleiben.

### Ergebnisse mit Kennzeichnungen lesbar machen

Verwenden Sie `AS` mit einfachen Anführungszeichen für Beschriftungen, die Leerzeichen enthalten:

```sql
SELECT count(*) AS 'Requests', percentile(duration, 95) AS 'p95 (ms)' FROM Transaction FACET service
```

&#x200B;> [!NOTE]
> Aliase gelten nur für `SELECT`. Facettenspalten sind immer mit dem Attributnamen gekennzeichnet.

### Muster und Listen abgleichen

`LIKE` verwendet `%` für eine beliebige Anzahl von Zeichen und `_` für genau ein Zeichen:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath LIKE '/content/%.html' FACET urlPath LIMIT 20
```

Ausschließen von statischen Assets mit `NOT LIKE`:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath NOT LIKE '%.js' AND urlPath NOT LIKE '%.css' AND urlPath NOT LIKE '%.png' FACET urlPath LIMIT 20
```

Abgleich mit einer Liste mit `IN` und `NOT IN` (bis zu 100 Werte):

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.method IN ('POST', 'PUT', 'DELETE') FACET service
```

&#x200B;> [!NOTE]
> SQL hat keine regulären Ausdrücke (`RLIKE`). Kombinieren Sie `LIKE` mit `OR` oder verwenden Sie `IN`.

### Mit `ORDER BY` sortieren und ordnen

Standardmäßig werden Facettenergebnisse nach dem ersten Aggregat sortiert. Verwenden Sie `ORDER BY`, um nach etwas anderem zu sortieren, z. B. nach den langsamsten Vorgängen auf p95:

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

Oder zuerst die am wenigsten genutzten Dienste:

```sql
SELECT count(*) AS requests FROM Transaction FACET service ORDER BY requests ASC LIMIT 10
```

&#x200B;> [!TIP]
> Es gibt keine `HAVING`. Um nur „Besetzt“-Vorgänge zu finden, reihen Sie `requests` nach und behalten Sie die obersten N bei, oder fügen Sie eine `WHERE` hinzu, die Geräusche beseitigt (z. B. `WHERE http.method != 'OPTIONS'`).

### Nach mehreren Attributen gruppieren

`FACET` akzeptiert bis zu fünf Attribute. Jede Kombination wird zu einer Zeile oder Serie:

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode >= 400 FACET service, http.method, http.statusCode LIMIT 30
```

### Blick in ein vergangenes Fenster

Es gibt keine `COMPARE WITH`. Ändern Sie in den Dashboards und im Explorer die **Zeitauswahl**. Wenn Sie die -API ohne expliziten Bereich aufrufen, erhalten Sie `SINCE … UNTIL …` alle vorherigen Fenster:

```sql
SELECT count(*) AS lastWeekSameHour FROM Transaction SINCE 169 hours ago UNTIL 168 hours ago
```
