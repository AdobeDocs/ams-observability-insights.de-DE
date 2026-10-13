---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '491'
ht-degree: 1%
---
## Tutorial 4: Erweiterte Funktionen {#tutorial-4-advanced-functions}

In diesem Tutorial werden die Funktionen und Entitäten behandelt, die für tiefer gehende Untersuchungen sorgen: Variabilität, Wachstum, „Worst Example“-Suchen, Anteil an der Gesamtsumme, Transaktionszusammensetzung und beliebige OpenTelemetry-Metriken.

**In diesem Tutorial:** [9.1 stddev](#91-measure-consistency-with-stddev) ・ [9.2 Derivat](#92-spot-growth-with-derivative) ・ [9.3 Aktueller Rang](#93-find-the-worst-example-with-ranked-latest) ・ [9.4 ProzentOfTotal](#94-find-where-time-goes-with-percentoftotal) ・ [9.5 SpanBreakdown](#95-break-down-transactions-with-spanbreakdown) ・ [9.6 Beliebige Metriken](#96-discover-and-query-any-metric) ・ [9.7 Entität auswählen](#97-choose-the-right-trace-entity) ・ [9.8 Verschachtelte Aggregation](#98-work-around-nested-aggregation)

### Messen der Konsistenz mit `stddev`

Zwei Vorgänge können einen Durchschnitt gemeinsam haben und sich dennoch sehr unterschiedlich verhalten. Die Standardabweichung gibt an, wie stark die Latenz variiert:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name ORDER BY stddevMs DESC LIMIT 20
```

Eine Standardabweichung, die größer ist als der Durchschnitt, verweist auf einen Vorgang mit gelegentlich sehr langsamen Anforderungen. Als Nächstes schauen Sie sich die p99 an.

### Fleckenwachstum mit `derivative`

`derivative(attr)` Gibt die Änderung des Attributs-Durchschnitts zwischen Buckets pro Sekunde zurück. Es ist nützlich, um eine kletternde Anzeige zu erkennen, wie z. B. Heap-Nutzung vor einem Leck:

```sql
SELECT derivative(value) AS heapMBPerSec FROM JvmHeapUsed FACET host TIMESERIES 5 minutes
```

Eine anhaltend positive Linie bedeutet Wachstum. Negative Werte bedeuten, dass die Anzeige fällt (z. B. nach der automatischen Bereinigung). Wie `rate` braucht `derivative` `TIMESERIES` und muss das einzige Aggregat sein.

### Finden Sie das schlechteste Beispiel mit Rang `latest`

Das zweite Argument von `latest` und `earliest` ändert sich in „Zuletzt verwendet“ in „Von diesem Attribut am höchsten“. `latest(traceId, duration)` gibt die Ablaufverfolgungs-ID der langsamsten Anfrage zurück, sodass Sie sie in die Ablaufverfolgungssuche einfügen können:

```sql
SELECT count(*) AS requests, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace, latest(urlPath, duration) AS slowestPath FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 15
```

Verwenden Sie `earliest(traceId, duration)` , um das schnellste Beispiel zum Vergleich abzurufen.

### Finden Sie heraus, wo die Zeit `percentOfTotal`

`percentOfTotal(attr)` zeigt den Anteil jeder Gruppe an der Gesamtzahl an. Angewendet auf `duration` zeigt es an, welche Vorgänge die meiste Serverzeit verbrauchen. Dies ist oft nützlicher als die Frage, welche Vorgänge am langsamsten sind:

```sql
SELECT count(*) AS requests, sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfServerTime FROM Transaction FACET name LIMIT 15
```

`percentOfTotal` können nicht mit `TIMESERIES` kombiniert werden.

### Transaktionen mit `SpanBreakdown` aufschlüsseln

`SpanBreakdown` wird bei der Aufnahme voraggregiert. Es wird aufgezeichnet, welche untergeordneten Vorgänge (`segment`) jede Transaktion (`rootOperation`) Zeit in Minuten pro Minute verbringt. Es antwortet: „Was macht dieser Endpunkt?“ Billig:

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown WHERE rootOperation LIKE '%GET%' FACET rootOperation, segment LIMIT 25
```

Entwicklung eines Segments im Zeitverlauf:

```sql
SELECT average(avgDurationMs) AS avgMs FROM SpanBreakdown FACET segment TIMESERIES AUTO LIMIT 8
```

&#x200B;> [!NOTE]
> `avgDurationMs` wird nicht automatisch mit einer Einheit getaggt. Setzen Sie das Widget **Einheit** auf `ms`.

### Erkennen und Abfragen beliebiger Metriken

SQL hat keine `keyset()` oder `SHOW EVENT TYPES`. Listen Sie stattdessen alle Metriken auf, die Berichte erstellen, zusammen mit der Einheit und dem neuesten Wert:

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricGauge FACET metric, unit LIMIT 200
```

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricSum FACET metric, unit LIMIT 200
```

Fragen Sie dann eine beliebige Metrik nach Namen ab. Beispielsweise wird JVM-Arbeitsspeicher pro Pool verwendet:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool LIMIT 50
```

Um anzuzeigen, **span-Attribute** für jeden Service aufgefüllt werden, zählen Sie nicht leere Werte:

```sql
SELECT count(*) FILTER (WHERE http.route != '') AS withRoute, count(*) FILTER (WHERE db.system != '') AS dbCalls, count(*) FILTER (WHERE peer.service != '') AS withPeer, count(*) FILTER (WHERE error.type != '') AS withErrorType FROM Span FACET service LIMIT 50
```

&#x200B;> [!TIP]
> Wenn eine kuratierte Entität vorhanden ist (z. B. `CpuUtilization`), sollten Sie sie gegenüber `MetricGauge` bevorzugen. Es wendet den richtigen Filter und die richtige Einheitenkonvertierung an und kann rollup-beschleunigt werden.

### Auswählen der richtigen Ablaufverfolgungsentität

| Verwenden Sie | Wenn Sie möchten | Beispiel |
| ------------- | ------------------------------------------------------------------------------------------- | --------------------------------- |
| `Transaction` | KPIs eingehender HTTP-/RPC-Anfragen (Durchsatz, Latenz, Fehler, Apdex). Rollup-beschleunigt. | Goldene Signale |
| `AppRequest` | Dieselben KPIs, **einschließlich Nachrichtenkonsumenten** (Kafka und Ähnliches) | Ereignisgesteuerte Services |
| `Span` | Alles unterhalb der Anfrage: ausgehendes HTTP, DB, Messaging, AEM Sling/JCR/JSP, interne Bereiche | Abhängigkeits- und Code-Pfad-Analyse |

Zählen Sie jede span-Art, um zu sehen, was instrumentiert ist:

```sql
SELECT count(*) AS spans, percentile(duration, 95) AS p95 FROM Span FACET kind
```

Ausgehende Anrufe nach externer Domain:

```sql
SELECT count(*) AS calls, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain LIMIT 20
```

### Umgehen verschachtelter Aggregationen

Mit NRQL können Sie ein Aggregat mit einer Unterabfrage aggregieren, z. B. „der Durchschnitt der maximalen Werte pro Host“. SQL nicht. Im Folgenden finden Sie Alternativen:

| Ziel | SQL-Ansatz |
| --------------------------------------- | ---------------------------------------------------------- |
| Fügen Sie einen Wert pro Gruppe ein und ordnen Sie dann die Gruppen ein | `FACET group` + `ORDER BY alias` + `LIMIT` |
| Ein Verhältnis von zwei Aggregaten | [Formel-Widget](#86-combine-queries-with-formula-widgets) |
| Ein Wert pro Host, dann der „schlechteste Host“ | `FACET host ORDER BY alias DESC LIMIT 1` |
| Ein Durchschnittswert pro Host im Zeitverlauf | `FACET host TIMESERIES AUTO` und lesen Sie die Serie |

Beispiel: der schlechteste Host von p95:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET host ORDER BY p95 DESC LIMIT 1
```
