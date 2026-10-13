---
title: SQL-Referenz
description: Verweisen Sie auf die SQL-Syntax, -Klauseln und -Funktionen, die von Observability Insights unterstützt werden.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '1492'
ht-degree: 9%
---

## SQL-Referenz {#sql-reference}

Diese Referenz behandelt SQL-Syntax, -Klauseln, -Funktionen und -Operatoren. Erweitern Sie jeden Eintrag, um Details und Beispiele anzuzeigen.

**In diesem Dokument:** [Abfragekomponenten](#41-query-components) ・ [Erforderliche Klauseln](#42-required-clauses) ・ [Optionale Klauseln](#43-optional-clauses) ・ [Aggregatorfunktionen](#44-aggregator-functions) ・ [Operatoren und Bedingungen](#45-operators-and-conditions) ・ [Typbehandlung](#46-type-handling-blank-vs-null) ・ [SQL zu SQL](#47-translate-sql-to-sql) ・ [Importieren vorhandener Dashboards](#48-importing-existing-dashboards-to-insights)

### Abfragekomponenten

```text
SELECT <select-item> [, <select-item> ...]
FROM   <Entity>
[WHERE      <condition>]
[FACET      <attribute> [, <attribute> ...]]          -- up to 5
[SINCE      <n> <unit> [AGO]]
[UNTIL      NOW | <n> <unit> [AGO]]
[TIMESERIES AUTO | MAX | <n> <unit>]
[ORDER BY   <alias-or-selected-name> [ASC | DESC]]
[LIMIT      <1..5000>]

<select-item> := <function>(<args>) [FILTER (WHERE <condition>)] [AS <alias>]
               | <attribute> [AS <alias>]
<unit>        := second | seconds | minute | minutes | hour | hours | day | days | week | weeks
```

>[!NOTE]
>
> Die oben genannten `--` Anmerkungen dienen nur der Dokumentation. SQL selbst akzeptiert keine Kommentare.

### Erforderliche Klauseln

<details>
<summary><code>SELECT</code> — Was zurückgegeben werden soll</summary>

`SELECT` listet entweder **Raw-Attribute** (eine Ereignisabfrage) oder **Aggregatorfunktionen** (eine Aggregatabfrage) auf. Man kann die beiden nicht vermischen.

| Formular | Beispiel | Ergebnis |
| ------------ | -------------------------------------------------- | ------------------------------------------------- |
| Attribute | `SELECT timestamp, name, duration` | Zeilen mit einzelnen Ereignissen (Standard: 100) |
| Aggregate | `SELECT count(*), average(duration)` | Ein Wert pro Aggregat (pro Facetten-/Zeitbereich) |
| Mit Aliassen | `SELECT count(*) AS requests` | `requests` der Spaltentitel |
| Zitatalias | `SELECT percentile(duration, 95) AS 'p95 latency'` | Titel mit Leerzeichen |

Ereignisliste:

```sql
SELECT timestamp, service, name, duration, status, traceId FROM Transaction WHERE status = 'Error' LIMIT 50
```

Mehrere Aggregate:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

Regeln:

- `SELECT *` wird nicht unterstützt. Listen Sie die gewünschten Attribute auf.
- Die Mathematik zwischen Attributen oder Aggregaten (`sum(a) / count(*)`) wird nicht unterstützt. Verwenden Sie ein [Formel-Widget](#86-combine-queries-with-formula-widgets).
- Das Mischen von Attributen und Aggregaten schlägt mit fehl _Rohattribute können nicht mit Aggregaten gemischt werden; verwenden Sie FACET, um Ergebnisse zu gruppieren_.

</details>

<details>
<summary><code>FROM</code> - Welche Entität abgefragt werden soll</summary>

`FROM` benennt genau **eine** Entität aus dem [Datenwörterbuch](#data-dictionary). Beim Entitätsnamen wird nicht zwischen Groß- und Kleinschreibung unterschieden.

```sql
SELECT count(*) FROM Transaction
```

```sql
FROM Span SELECT count(*) FACET kind
```

Jede Entität hat einen festen Umfang. Zum Beispiel ist `Transaction` immer Serverspannen und `CpuUtilization` ist immer die Metrik &quot;CPU-beschäftigt“, wobei `value` in Prozent konvertiert wird. Sie können nicht mehr als eine Entität in einer Abfrage abfragen. Um Entitäten zu kombinieren, verwenden Sie ein [Formel-Widget](#86-combine-queries-with-formula-widgets).

</details>

### Optionale Klauseln

<details>
<summary><code>AS</code> - eine Spalte kennzeichnen</summary>

Verwenden Sie `AS`, um einem Aggregat oder Attribut einen lesbaren Namen zu geben. Der Alias wird zur Spaltenüberschrift, Legendenbezeichnung und zum Namen, auf den Sie in `ORDER BY` verweisen.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET service ORDER BY p95 DESC LIMIT 10
```

Verwenden Sie für Beschriftungen mit Leerzeichen eine Zeichenfolge in Anführungszeichen:

```sql
SELECT percentile(duration, 95) AS 'p95 latency (ms)' FROM Transaction TIMESERIES AUTO
```

</details>

<details>
<summary><code>FACET</code> — Gruppenergebnisse</summary>

`FACET` gruppiert Ergebnisse nach bis zu **5** Attributen, z. B. SQL-`GROUP BY`.

```sql
SELECT count(*) FROM Transaction FACET service, http.method, http.statusCode LIMIT 30
```

| Abfrage | `LIMIT n` bedeutet | Standard |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `FACET` ohne `TIMESERIES` | Anzahl Zeilen | 10 |
| `FACET` mit `TIMESERIES` | **Top n Reihe**, sortiert nach der Summe des ersten Aggregats über das gesamte Fenster. Jedes Mal, wenn Eimer für diese Serie aufbewahrt wird. | Alle Reihen (begrenzt auf 5.000 Zeilen) |

Facettierte Zeitreihen, Top 5:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Facettenwerte können Zeichenfolgen oder Zahlen sein. Leere Zeichenfolgen bilden eine eigene Gruppe. Mit `WHERE attr != ''` ausschließen.

</details>

<details>
<summary><code>FILTER</code> — Bedingte Aggregate</summary>

Hängen Sie `FILTER (WHERE condition)` an ein Aggregat an, um es über eine Teilmenge von Zeilen zu berechnen. Auf diese Weise können Sie mehrere Bedingungen in einem Durchgang vergleichen.

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentile(duration, 95) FILTER (WHERE http.method = 'GET') AS getP95 FROM Transaction FACET service LIMIT 20
```

`FILTER` ist mit `apdex`, `rate` oder `derivative` nicht zulässig. `percentage` nimmt bereits einen eigenen Zustand an.

</details>

<details>
<summary><code>LIMIT</code> — Anzahl der Zeilen oder Reihen</summary>

`LIMIT` benötigt eine positive Ganzzahl. Werte über 5.000 werden auf 5.000 beschränkt.

| Ergebnisform | Standard |
| ------------ | ---------- |
| Reklametafel | 1 |
| Facetteneffekt | 10 |
| Zeitreihe | 5.000 Zeilen |
| Ereignisse | 100 |

```sql
SELECT count(*) FROM Transaction FACET urlPath LIMIT 50
```

>[!TIP]
>
> Legen Sie immer eine `LIMIT` für abfragebasierte Variablen und Facetten mit hoher Kardinalität fest (`urlPath`, `traceId`, `jcr.path`, `url`).

</details>

<details>
<summary><code>ORDER BY</code> — Ergebnisse sortieren</summary>

`ORDER BY` sortiert nach einem **ausgewählten Alias oder ausgewählten Attribut**. Die Standardrichtung ist `DESC`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

| Ergebnisform | Standardreihenfolge |
| ----------------- | --------------------------------------------------------------------------- |
| Reklametafel/Facette | Erstes Aggregat, `DESC` |
| Zeitreihe | Zeit, `ASC` |
| Ereignisse | Erste ausgewählte Spalte, `DESC` (setzen Sie also `timestamp` für „Newest First“ an erste Stelle) |

Regeln:

- Wenn auf etwas verwiesen wird, das nicht ausgewählt ist, schlägt dies mit _ORDER BY &#39;x&#39; fehl und muss auf einen ausgewählten Wert oder Alias verweisen_.
- `ORDER BY` nicht zu einer `TIMESERIES` Abfrage hinzufügen. Sie überschreibt die für Diagramme erforderliche Zeitreihenfolge.

</details>

<details>
<summary><code>SINCE</code> — Start des Fensters</summary>

`SINCE n unit [AGO]` legt fest, wie weit das Fenster zurück beginnt. Einheiten: `second`, `minute`, `hour`, `day`, `week` (im Singular oder im Plural). `AGO` ist optional.

```sql
SELECT count(*) FROM Transaction SINCE 6 hours ago
```

- Der Standardwert lautet `SINCE 1 hour ago`.
- Das Fenster darf nicht länger **31 Tage** sein (_Zeitraum darf 31 Tage nicht überschreiten_).
- Absolute Datumswerte (`SINCE '2026-01-01'`) und `TODAY`/`YESTERDAY` werden nicht unterstützt.

>[!IMPORTANT]
>
> In Dashboards und im Explorer überschreibt **Zeitauswahl** `SINCE` und `UNTIL`. Siehe [Zeitfensterpriorität](#time-window-precedence).

</details>

<details>
<summary><code>TIMESERIES</code> — Diagramm im Zeitverlauf</summary>

`TIMESERIES` gibt eine Zeile pro Zeitbehälter (pro Facettenwert) zurück, die für Linien-, Flächen- oder Balkendiagramme bereit ist.

| Formular | Bucket-Größe |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `TIMESERIES 5 minutes` | Genau das, was Sie verlangen (beliebige Einheit) |
| `TIMESERIES AUTO` | Über Fenster ÷ 120, aufgerundet auf 1s / 5s / 10s / 15s / 30s / 1m / 5m / 10m / 15m / 30m / 1h / 3h / 6h / 12h / 1d. **Nie unter 2 Minuten.** |
| `TIMESERIES MAX` | 1-Sekunden-Buckets. Nur über wenige Minuten verwenden. |

Typische `AUTO`:

| Fenster | 1 h | 6 h | 24 Stunden | 7 T | 30 T |
| ------ | ----- | ----- | ------ | --- | ---- |
| Bucket | 2 Min | 5 Min | 15 Min | 3 Std | 6 h |

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction TIMESERIES AUTO
```

```sql
SELECT count(*) FROM Transaction TIMESERIES 1 hour SINCE 1 day ago
```

>[!TIP]
>
> Bevorzugte `AUTO` in Dashboards. Es hat ungefähr 120 Punkte, was auch immer die Zeitauswahl sagt. Außerdem werden „zersplitterte“ Zeilen pro Host vermieden, da Host-Metriken etwa alle 90 Sekunden abgekratzt werden.

</details>

<details>
<summary><code>UNTIL</code> — Ende des Fensters</summary>

`UNTIL NOW` (Standard) oder `UNTIL n unit [AGO]`. Kombinieren Sie es mit `SINCE`, um ein vergangenes Fenster zu betrachten:

```sql
SELECT count(*) AS yesterday FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

`SINCE` muss vor `UNTIL` liegen. Andernfalls schlägt die Abfrage mit fehl _SINCE muss vor UNTIL liegen_.

</details>

<details>
<summary><code>WHERE</code> — Zeilen filtern</summary>

`WHERE` speichert nur Zeilen, die einer Bedingung entsprechen. Kombinieren Sie Bedingungen mit `AND`, `OR`, `NOT` und Klammern. Siehe [Operatoren und Bedingungen](#45-operators-and-conditions).

```sql
SELECT count(*) FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR http.statusCode >= 500)
```

`WHERE` kann nur einmal angezeigt werden. Geben Sie alle Bedingungen hinein.

</details>

### Aggregator-Funktionen

| Funktion | Argumente | Rückgabe | Funktioniert am |
| -------------------------------------- | ----------------------------------- | ---------------------- | ------------------------------------- |
| [`apdex`](#fn-apdex) | Numerische Dauer, Schwellenwert `T` | 0-1 | Entitäten verfolgen |
| [`average` / `avg`](#fn-average) | Numerisch | Gleiche Einheit wie Eingang | Alle |
| [`count`](#fn-count) | `*` oder Attribut | Anzahl Zeilen | Alle |
| [`derivative`](#fn-derivative) | Numerisch | Änderung pro Sekunde | Beliebig (`TIMESERIES` erforderlich) |
| [`earliest`](#fn-latest) | Beliebige [, Rang] | Attributwert | Alle |
| [`latest`](#fn-latest) | Beliebige [, Rang] | Attributwert | Alle |
| [`max` / `min`](#fn-minmax) | Beliebig | Gleiche Einheit wie Eingang | Alle |
| [`percentage`](#fn-percentage) | `*` oder Attribut, `WHERE` Bedingung | % | Alle |
| [`percentile`](#fn-percentile) | Numerisch, Ebene `p` | Gleiche Einheit wie Eingang | Alle |
| [`percentOfTotal`](#fn-percentoftotal) | Numerisch | % | Beliebig (keine `TIMESERIES`) |
| [`rate`](#fn-rate) | numerischer Zähler | pro Sekunde | Zählerentitäten (benötigt `TIMESERIES`) |
| [`stddev`](#fn-stddev) | Numerisch | Gleiche Einheit wie Eingang | Alle |
| [`sum`](#fn-sum) | Numerisch | Gleiche Einheit wie Eingang | Alle |
| [`uniqueCount`](#fn-uniquecount) | Beliebig | Unterschiedliche Werte | Alle |

<a id="fn-apdex"></a>

<details>
<summary><code>apdex(attribute, T)</code></summary>

Berechnet einen [Apdex](https://www.apdex.org/)-Wert von 0 (frustriert) bis 1 (zufrieden) anhand des Schwellenwerts `T` der Einheit des Attributs (Millisekunden für `duration`).

| Bucket | Bedingung | Gewichtung |
| ---------- | ------------------------------------ | ------ |
| zufrieden | `duration ≤ T` und kein Fehler | 1 |
| erträglich | `T < duration ≤ 4T` und kein Fehler | 0.5 |
| frustriert | `duration > 4T` oder **Fehler** | 0 |

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service
```

- Funktioniert nur mit Ablaufverfolgungsentitäten (`Transaction`, `AppRequest`, `Span`).
- `T` muss größer als 0 sein. `FILTER` ist nicht erlaubt.
- `apdex(duration, 500)` auf `Transaction` wird über lange Zeitfenster hinweg rollup-beschleunigt.

</details>

<a id="fn-average"></a>

<details>
<summary><code>average(attribute)</code> / <code>avg(attribute)</code></summary>

Das arithmetische Mittel eines numerischen Attributs.

```sql
SELECT average(duration) AS avgMs FROM Transaction FACET service TIMESERIES AUTO
```

</details>

<a id="fn-count"></a>

<details>
<summary><code>count(*)</code> / <code>count(attribute)</code></summary>

- `count(*)` zählt Zeilen.
- `count(attribute)` zählt Zeilen, bei denen das Attribut **NOT NULL** ist. Bei Zeichenfolgenattributen sind fehlende Werte leer (`''`) und nicht NULL, sodass sie weiterhin gezählt werden (siehe [Typenverarbeitung](#46-type-handling-blank-vs-null)).

```sql
SELECT count(*) AS requests, count(http.statusCode) AS withStatusCode FROM Transaction
```

</details>

<a id="fn-derivative"></a>

<details>
<summary><code>derivative(attribute)</code></summary>

Die Änderung der `average(attribute)` pro Sekunde zwischen aufeinander folgenden Buckets: (dieser Bucket − vorheriger Bucket) ÷ Bucket Sekunden. Das Ergebnis kann negativ sein. Verwenden Sie ihn auf Messgeräten, um zu sehen, ob ein Wert wächst oder schrumpft.

```sql
SELECT derivative(value) AS heapGrowthPerSec FROM JvmHeapUsed WHERE host = 'publish-1' TIMESERIES 5 minutes
```

- Erfordert `TIMESERIES` und muss das **Aggregat**. `FILTER` ist nicht erlaubt.
- Der erste Bucket hat keinen Vorgänger.

</details>

<a id="fn-latest"></a>

<details>
<summary><code>latest(attribute [, rankAttribute])</code> / <code>earliest(…)</code></summary>

- `latest(attr)` Gibt den Wert aus der letzten Zeile zurück. `earliest(attr)` gibt den Wert aus der ältesten Zeile zurück.
- Bei einem zweiten Argument wird die Zeile durch dieses Attribut anstelle von Zeit ausgewählt. `latest(traceId, duration)` gibt die Ablaufverfolgungs-ID der **langsamsten)** zurück. `earliest(traceId, duration)` gibt die schnellste zurück.

```sql
SELECT latest(value) AS current FROM MemoryUtilization FACET host
```

```sql
SELECT max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 10
```

</details>

<a id="fn-minmax"></a>

<details>
<summary><code>max(attribute)</code> / <code>min(attribute)</code></summary>

Der größte oder kleinste Wert. Funktioniert mit Zahlen, Zeichenfolgen und Zeitstempeln.

```sql
SELECT min(duration) AS fastestMs, max(duration) AS slowestMs, max(timestamp) AS lastSeen FROM Transaction FACET service
```

</details>

<a id="fn-percentage"></a>

<details>
<summary><code>percentage(*, WHERE condition)</code></summary>

Der Anteil der Zeilen, die `condition` entsprechen, von 0 bis 100.

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO
```

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction
```

`percentage` hat einen eigenen Zustand, es braucht also keine `FILTER`.

</details>

<a id="fn-percentile"></a>

<details>
<summary><code>percentile(attribute, p)</code></summary>

Das ungefähre p-te Perzentil, wobei `0 < p ≤ 100`. Verwenden Sie eine `percentile` pro Ebene.

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

P50, P95 und P99 auf `Transaction` sind rollup-beschleunigt über lange Fenster.

</details>

<a id="fn-percentoftotal"></a>

<details>
<summary><code>percentOfTotal(attribute)</code></summary>

Der Anteil jeder Facettenzeile an der **Gesamtsumme** eines numerischen Attributs. Die Zeilen summieren sich zu 100 %.

```sql
SELECT sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfTime FROM Transaction FACET name LIMIT 15
```

Nicht zulässig mit `TIMESERIES`.

</details>

<a id="fn-rate"></a>

<details>
<summary><code>rate(attribute)</code></summary>

Die Rate pro Sekunde eines **kumulativen Zählers**. In jedem Bucket nimmt `rate` den höchsten Zählerstand pro Serie und vergleicht ihn mit dem vorherigen Bucket. Zählerzurücksetzungen werden bei 0 gesetzt, und der erste Bucket ist 0.

```sql
SELECT rate(value) AS bytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

- Funktioniert nur bei Zählerentitäten (✔ im [Datenwörterbuch) &#x200B;](#53-curated-metric-entities).
- Erfordert `TIMESERIES` und muss das **Aggregat**. `FILTER` ist nicht erlaubt.

>[!WARNING]
>
> Immer `FACET host` oder auf einen einzelnen Host filtern. Eine facettenlose `rate()` über viele Hosts hinweg vergleicht die Zähler verschiedener Hosts und erzeugt bedeutungslose Zahlen. Datenträgerzähler sind pro Gerät, daher ist `FACET host, device` am genauesten.

</details>

<a id="fn-stddev"></a>

<details>
<summary><code>stddev(attribute)</code></summary>

Die Stichprobenstandardabweichung. Verwenden Sie sie mit `average`, um zu beurteilen, wie konsistent die Latenz ist.

```sql
SELECT average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name LIMIT 20
```

</details>

<a id="fn-sum"></a>

<details>
<summary><code>sum(attribute)</code></summary>

Die Summe eines numerischen Attributs.

```sql
SELECT sum(callCount) AS calls FROM SpanBreakdown FACET segment LIMIT 20
```

</details>

<a id="fn-uniquecount"></a>

<details>
<summary><code>uniqueCount(attribute)</code></summary>

Die **exakte** Anzahl der eindeutigen Werte.

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

</details>

Funktionsbeschränkungen auf einen Blick:

| Beschränkung | Funktionen |
| ------------------------------------- | ----------------------------------------------------------------------------------------- |
| Numerisches Attribut erforderlich | `average`, `sum`, `percentile`, `stddev`, `percentOfTotal`, `rate`, `derivative`, `apdex` |
| Keine `FILTER (WHERE …)` | `apdex`, `rate`, `derivative`, `percentage` |
| Erfordert `TIMESERIES`, nur Aggregat | `rate`, `derivative` |
| Nicht mit `TIMESERIES` | `percentOfTotal` |
| Nur Entitäten verfolgen | `apdex` |
| Nur Zählerentitäten | `rate` |

### Benutzer und Bedingungen

| Operator | Bedeutung | Beispiel |
| ------------------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| `=` | Gleich | `status = 'Error'` |
| `!=`, `<>` | Ungleich | `kind != 'Internal'` |
| `<`, `<=`, `>`, `>=` | Vergleich | `http.statusCode >= 500` |
| `LIKE` | Wildcard match: `%` = beliebige Zeichen, `_` = ein Zeichen. Von Schreibweise abhängig. | `urlPath LIKE '/content/%'` |
| `NOT LIKE` | Negierter Platzhalter | `urlPath NOT LIKE '%.css'` |
| `IN (…)` | Entspricht einem der Werte 1-100 | `service IN ('checkout', 'cart')` |
| `NOT IN (…)` | Entspricht keinem der Werte | `http.method NOT IN ('OPTIONS', 'HEAD')` |
| `IS NULL` / `IS NOT NULL` | Null-Prüfung | `http.statusCode IS NOT NULL` |
| `AND`, `OR`, `NOT` | Logik | `NOT (status = 'Ok')` |
| `( … )` | Gruppierung | `(a = 1 OR b = 2) AND c = 3` |

```sql
SELECT count(*) FROM Transaction WHERE (status = 'Error' OR duration > 2000) AND NOT host LIKE '%dispatcher%' AND http.method IN ('GET', 'POST')
```

>[!WARNING]
>
> `attr = NULL` Fehler. Verwenden von `attr IS NULL`.

### Typ-Handling: leer vs. NULL

Jedes Attribut hat einen Typ im Katalog: `string`, `number`, `boolean` oder `timestamp`. Der SQL-Typ vergleicht Funktionen (z. B. wird `average(service)` zurückgewiesen). Es sind keine Typkonvertierungsfunktionen erforderlich oder verfügbar.

>[!IMPORTANT]
>
> **Leer vs. NULL.** Die meisten Zeichenfolgenattribute stammen aus OpenTelemetry-Attributzuordnungen, bei denen ein _fehlender_-Schlüssel als leere `''` und nicht als `NULL` gelesen wird. Um nur Bereiche beizubehalten, die ein -Attribut tragen, verwenden Sie **`attr != ''`** statt `IS NOT NULL`. Numerische Attribute wie `http.statusCode` werden in der Abwesenheit wirklich `NULL`, sodass `IS NOT NULL` für sie funktioniert.

```sql
SELECT count(*) AS jcrCalls FROM Span WHERE jcr.operation != '' FACET jcr.operation
```

### SQL in SQL übersetzen

| SQL | SQL |
| ------------------------------------------------------------ | ------------------------------------------------------------------- |
| `WHERE ts > now() - INTERVAL 1 HOUR` | Nichts. Die Zeitauswahl (oder `SINCE 1 hour ago`) legt das Fenster fest. |
| `GROUP BY service` | `FACET service` |
| `GROUP BY toStartOfInterval(ts, INTERVAL 5 MINUTE), service` | `FACET service TIMESERIES 5 minutes` |
| `quantile(0.95)(duration)` | `percentile(duration, 95)` |
| `countIf(status = 'Error')` | `count(*) FILTER (WHERE status = 'Error')` |
| `100 * countIf(x) / count()` | `percentage(*, WHERE x)` |
| `count(DISTINCT host)` | `uniqueCount(host)` |
| `argMax(traceId, duration)` | `latest(traceId, duration)` |
| `x / sum(x) OVER ()` | `percentOfTotal(x)` |
| `ORDER BY p95 DESC LIMIT 20` | `ORDER BY p95 DESC LIMIT 20` |
| `HAVING count() > 100` | _Nicht unterstützt_. Stattdessen sortieren und `LIMIT`. |
| `JOIN`/Unterabfrage | _Nicht unterstützt_. Verwenden Sie ein **Formel**-Widget. |
| `CASE WHEN` | _Nicht unterstützt_. Verwenden Sie `FILTER` Spalten oder `WHERE … LIKE` + `FACET`. |

### Importieren vorhandener Dashboards in Insights

Wenn Sie die vorhandene Abfrage kennen, finden Sie im Folgenden die Migrationsdetails

| Vorhandene Abfrage | SQL | Anmerkungen |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------ |
| `FROM Transaction SELECT count(*)` | Gleich | `FROM` erste Reihenfolge wird unterstützt |
| `SINCE 1 day ago` | Gleich | In Dashboards ignoriert; die Auswahl gewinnt |
| `TIMESERIES AUTO` / `MAX` / `5 minutes` | Gleich | `AUTO` ist 2 Minuten |
| `FACET appName` | `FACET appName` / `FACET service` |                                                              |
| `percentage(count(*), WHERE error IS true)` | `percentage(*, WHERE status = 'Error')` |                                                              |
| `filter(count(*), WHERE …)` | `count(*) FILTER (WHERE …)` | SQL-Standardformular |
| `apdex(duration, t: 0.5)` | `apdex(duration, 500)` | `duration` ist in **ms** |
| `latest(x)` / `earliest(x)` | Gleich | Plus eine Rangform: `latest(x, rankAttr)` |
| `uniqueCount(x)` | Gleich | Exakt |
| `rate(count(*), 1 minute)` | `count(*)` mit `TIMESERIES 1 minute` | SQL `rate()` ist nur für Zähler |
| `derivative(x, 1 minute)` | `derivative(x)` + `TIMESERIES` | pro Sekunde |
| `uniques(x)` | `count(*) … FACET x LIMIT n` |                                                              |
| `keyset()` / `SHOW EVENT TYPES` | [Discovery-](#101-data-discovery) oder das Bedienfeld „Anleitung zur Abfrage“ |                                                              |
| `histogram(duration, …)` | `FILTER` Buckets | Siehe [Tutorial 3](#84-build-a-latency-histogram) |
| `FACET CASES (…)` | Spalten `FILTER` | Siehe [Tutorial 3](#83-group-values-into-cases-with-filter) |
| `COMPARE WITH 1 week ago` | Nicht unterstützt | Ändern Sie die Zeitauswahl oder verwenden Sie `SINCE … UNTIL …` über die API |
| `RLIKE` | `LIKE` / `IN` |                                                              |
| Mathematische `SELECT`/verschachtelte Aggregation | **Formel**-Widget |                                                              |
| `WITH TIMEZONE`, `EXTRAPOLATE`, `SLIDE BY` | Nicht unterstützt |                                                              |
