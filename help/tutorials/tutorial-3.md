---
title: 'Tutorial 3: Erweitern des Dashboards'
description: Erstellen Sie Dashboards mit Variablen, Formeln und mehreren Widgets in Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '902'
ht-degree: 3%
---

## Tutorial 3: Erweitern des Dashboards {#tutorial-3-advance-your-dashboarding}

In diesem Tutorial erstellen Sie die Komponenten eines produktionsfähigen Dashboards: Benutzererlebnisbewertungen, Zählerraten, Fallgruppierung, Histogramme, KPI-Kacheln mit Schwellenwerten, Abschnitten, Formel-Widgets und Variablen.

**In diesem Tutorial:** [8.1 apdex](#81-score-user-experience-with-apdex) ・ [8.2 rate](#82-turn-counters-into-rates-with-rate) ・ [8.3 Cases](#83-group-values-into-cases-with-filter) ・ [8.4 Histogramme](#84-build-a-latency-histogram) ・ [8.5 KPI Kacheln und Abschnitte](#85-build-kpi-tiles-and-sections) ・ [8.6 Formeln](#86-combine-queries-with-formula-widgets) ・ [8.7 Variablen](#87-make-dashboards-reusable-with-variables) ・ [8.8](#88-put-it-together)

### Bewerten des Benutzererlebnisses mit `apdex`

Apdex wandelt Latenz und Fehler in einen Score von 0 bis 1 um. Bei einem Schwellenwert von 500 ms wird eine Anfrage _erfüllt_ mit oder unter 500 ms, _toleriert_ bis zu 2.000 ms und _frustriert_ über diesem **oder wenn sie fehlschlägt**:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

Trend pro Service:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

Platzieren Sie sie in einem **Zahl**-Widget mit Schwellenwerten `0 → Red`, `0.7 → Amber`, `0.85 → Green`. Die höchste passende Band gewinnt, sodass niedrige Werte rot bleiben.

### 8.2 Zähler in Tarife mit `rate` umwandeln

Zählerentitäten (✔ im [Datenwörterbuch](#53-curated-metric-entities)) speichern immer höhere Summen. `rate(value)` konvertiert sie in Werte pro Sekunde. Es muss `TIMESERIES` sein, und es muss das einzige Aggregat sein:

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host, device TIMESERIES AUTO LIMIT 10
```

>[!WARNING]
>
> Facetten immer nach `host` (und bei Festplatten nach `device`) `rate()` oder nach einem Host filtern. Andernfalls werden die Messwerte von verschiedenen Hosts gemischt und das Ergebnis ist bedeutungslos.

Um eine **Anfrage**-Rate aus Traces abzurufen, zählen Sie Anfragen stattdessen in festen Buckets. Dies ergibt Anfragen pro Minute:

```sql
SELECT count(*) AS requestsPerMinute FROM Transaction TIMESERIES 1 minute
```

### Werte mit `FILTER` in Fällen gruppieren

Die `FACET CASES` von NRQL gruppiert Zeilen in benannten Buckets. Schreiben Sie in SQL eine `FILTER` Spalte pro Groß-/Kleinschreibung. Im Folgenden finden Sie Statusklassen pro Service:

```sql
SELECT count(*) FILTER (WHERE http.statusCode < 400) AS ok, count(*) FILTER (WHERE http.statusCode >= 400 AND http.statusCode < 500) AS clientErrors, count(*) FILTER (WHERE http.statusCode >= 500) AS serverErrors FROM Transaction FACET service LIMIT 20
```

Die gleiche Idee funktioniert im Laufe der Zeit und gibt eine Zeile pro Fall:

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/graphql/%') AS graphql FROM Transaction TIMESERIES AUTO
```

### Erstellen eines Latenzhistogramms

SQL hat keine `histogram()`. Erstellen Sie die Buckets selbst mit `FILTER` und rendern Sie das Ergebnis als **Balken** oder **Tabellen**-Widget:

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS 'le 100ms', count(*) FILTER (WHERE duration > 100 AND duration <= 250) AS '100-250ms', count(*) FILTER (WHERE duration > 250 AND duration <= 500) AS '250-500ms', count(*) FILTER (WHERE duration > 500 AND duration <= 1000) AS '0.5-1s', count(*) FILTER (WHERE duration > 1000 AND duration <= 2000) AS '1-2s', count(*) FILTER (WHERE duration > 2000) AS 'over 2s' FROM Transaction
```

Fügen Sie `FACET service` hinzu, um die Verteilungen mehrerer Services nebeneinander zu vergleichen.

### Erstellen von KPI-Kacheln und -Abschnitten

Ein gutes Dashboard beginnt mit einer Reihe von KPI-Kacheln und gruppiert dann Detaildiagramme in Abschnitten.

1. Fügen Sie vier **Zahl**-Widgets hinzu, die jeweils drei Spalten breit sind:

   | Titel | Abfrage | Einheit | Schwellenwerte |
   | -------- | --------------------------------------------------------------------------- | ---- | ---------------------------------------- |
   | Anfragen | `SELECT count(*) AS requests FROM Transaction` |      |                                          |
   | Fehler % | `SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction` | `%` | `0 → Green`, `1 → Amber`, `5 → Red` |
   | P95 | `SELECT percentile(duration, 95) AS p95 FROM Transaction` | `ms` | `0 → Green`, `800 → Amber`, `2000 → Red` |
   | APDEX | `SELECT apdex(duration, 500) AS apdex FROM Transaction` |      | `0 → Red`, `0.7 → Amber`, `0.85 → Green` |

2. Fügen Sie ein **Text**-Widget hinzu, markieren Sie _Als Abschnittskopfzeile festlegen_ und nennen Sie es **LATENZ**. Fügen Sie darunter Latenzdiagramme hinzu.
3. Fügen Sie eine weitere Abschnittsüberschrift **FEHLER** hinzu, gefolgt von Fehlerdiagrammen. Einen Abschnitt mit dem Pfeil reduzieren, um den Fokus auf den anderen zu legen.
4. Verwenden Sie **Registerkarten** für verschiedene Zielgruppen, z. B. _Übersicht_, _AEM_, _Infrastruktur_. Nur die sichtbare Registerkarte führt ihre Abfragen aus.

### Kombinieren von Abfragen mit Formel-Widgets

SQL kann in `SELECT` nicht rechnen und hat keine Unterabfragen. **Formel-Widgets** füllen diese Lücke: Sie führen 2-6 benannte Abfragen aus und kombinieren ihre Ergebnisse mit `+ - * / ( )`.

**Beispiel: Anfragen pro Host im Zeitverlauf** (Formel, Zeile)

1. **Widget → Formel**.
2. Benennen Sie die erste `requests` und die zweite `hosts`.
3. Geben Sie die Abfragen und die Formel ein und klicken Sie dann auf **Formel ausführen**.

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

**Beispiel: Anteil der in der Datenbank verbrachten Zeit** (Formel, Zahl, `%`)

`db`:

```sql
SELECT sum(duration) AS v FROM Span WHERE db.system != ''
```

`server`:

```sql
SELECT sum(duration) AS v FROM Transaction
```

Formel:

```text
(db / server) * 100
```

Regeln für Formelabfragen:

| Regel | Details |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| Namen | Buchstaben, Ziffern und `_`, beginnend mit einem Buchstaben oder `_`; bis zu 40 Zeichen; eindeutig |
| Ausdruck | `+ - * / ( )`, Zahlen und Abfragenamen; bis zu 500 Zeichen |
| Gleiche Form | Alle Abfragen sind einfache Summen (Zahl) **oder** alle werden mit der **gleichen Behälterngröße)** Zeile / Bereich / Balken) `TIMESERIES` |
| Jeweils eine Metrik | Jede Abfrage gibt genau ein Aggregat und keine `FACET` zurück |
| Fehlende Daten | Ein fehlender Wert oder eine Division durch null erzeugt einen leeren Punkt, nie `Infinity` oder `NaN` |

>[!TIP]
>
> Verwenden Sie `TIMESERIES AUTO` in jeder Abfrage einer Zeitreihenformel. Da die Abfragen ein Fenster gemeinsam haben, werden sie immer auf dieselbe Bucket-Größe aufgelöst.

### Wiederverwendbarkeit von Dashboards mit Variablen

Mit Variablen kann ein Dashboard zahlreiche Services, Ebenen oder Schwellenwerte bereitstellen. Jede Variable ist eine Auswahl in der Variablenleiste. Referenzieren Sie es als `{{name}}` in Widget-Titeln und Text-Widgets (und, sobald die unten stehende Einschränkung behoben ist, in Widget-SQL).

**Variable erstellen** (_org und Superadmins_): Klicken Sie auf **+ Variable**.

| Feld | Anmerkungen |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name** | Buchstaben, Ziffern und `_`, beginnend mit einem Buchstaben oder `_`. Sie müssen eindeutig sein. Wird als `{{name}}` verwendet. |
| **label** | Was die Auswahl anzeigt (optional). |
| **Typ** | **Benutzerdefinierte Liste** (ein Wert pro Zeile), **Aus einer Abfrage** (eine SQL-Abfrage mit einem `FACET`; ihre unterschiedlichen Werte werden zu Optionen) oder **Textfeld** (freier Text). |
| **Auswahl mehrerer Werte zulassen** | Wird zu einer kommagetrennten Liste erweitert. Verwenden Sie es mit `IN (…)`. |
| **Werte sind Zahlen** | Fügt Werte ohne Anführungszeichen ein (`500` statt `'500'`). |

**Funktionsweise der Substitution**

| Auswahl | `{{svc}}` wird |
| ---------------------------------- | ------------------------------------------------------- |
| Einzelne `checkout` | `'checkout'` |
| Mehrere Werte `checkout`, `cart` | `'checkout', 'cart'` |
| Zahl `500` | `500` |
| Wert, der `'` enthält | Als `''` maskiert |
| Nichts ausgewählt | Das Widget wartet und zeigt _Wert für{{svc}}_ auswählen |

```text
WHERE service = {{svc}}            single select
WHERE service IN ({{svc}})         multiple select
WHERE duration > {{slowMs}}        numeric text box
```

Fügen Sie für eine abfragebasierte Variable **immer eine`LIMIT`** hinzu. Eine Facettenabfrage gibt standardmäßig nur die 10 wichtigsten Werte zurück. Hier sind bis zu 200 Services aufgeführt:

```sql
SELECT count(*) FROM Transaction FACET service LIMIT 200
```

Optionen werden aktualisiert, wenn sich der Mandant oder die Zeitauswahl ändert und sie alphabetisch sortiert werden. Wenn die aktuelle Auswahl nicht mehr gültig ist, werden der erste Wert (Einzelauswahl) oder alle Werte (Mehrfachauswahl) automatisch ausgewählt. Die Abfrage einer Variablen kann nicht auf andere Variablen verweisen.

>[!WARNING]
>
> **Aktuelle Build-Einschränkung.** Die Ersetzung erfolgt im Dashboard-Raster, aber die Vorschau **Widget hinzufügen** und die **Server-seitige Speicherprüfung** erhalten das Roh-`{{name}}`-Token und lehnen es mit `Unexpected character '{'` ab. In der Praxis:
>
> - `{{variables}}` in **Widget-** und **Text-Widgets** arbeiten von Anfang bis Ende.
> - `{{variables}}` innerhalb **Widget SQL** werden nach einem **Import** korrekt dargestellt, aber das Dashboard kann nicht **gespeichert** solange solche Widgets vorhanden sind.
>
> Bis dies behoben ist, erstellen Sie Widgets pro Kontext mit Literalfiltern (z. B. `WHERE service = 'checkout'`) und verwenden Sie **Duplizieren** um sie zu klonen.

### Bauen Sie es zusammen

Jetzt werden alle Bausteine im [Dashboard-Blueprints“ verwendet](#dashboard-blueprints) KPI-Kacheln mit Schwellenwerten, Abschnitten, Perzentilen, Apdex, Raten, Fallspalten, Histogrammen, Formeln und Variablen. Die Blueprints zeigen, wie sie zu vollständigen Golden-Signal-, Fehler-Triage-, AEM-, Infrastruktur- und JVM-Dashboards kombiniert werden können.
