---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '562'
ht-degree: 0%
---
## Tutorial 1: Erste Abfragen {#tutorial-1-your-first-queries}

Willkommen bei SQL. In diesem Tutorial schreiben Sie Ihre ersten Abfragen, steuern das Zeitfenster, zeichnen Trends auf, filtern und gruppieren die Ergebnisse und legen Ihr erstes Diagramm auf einem Dashboard ab. Jeder Schritt baut auf dem vorherigen auf.

**In diesem Tutorial:** [Bevor Sie beginnen](#before-you-begin) ・ [6.1 Abfrageereignisse](#61-query-events-with-select-from-and-limit) ・ [6.2 Aggregat](#62-aggregate-your-data) ・ [6.3 Mehrere Aggregate](#63-return-several-aggregates-at-once) ・ [6.4 Zeitfenster](#64-set-the-time-window) ・ [6.5 ZEITREIHEN](#65-plot-trends-with-timeseries) ・ [6.6 WO](#66-filter-with-where) ・ [6.7 FACET](#67-group-with-facet) ・ [6.8 In einem Dashboard speichern](#68-save-your-query-to-a-dashboard)

### Voraussetzungen

- Sie benötigen Zugriff auf Observability Insights und mindestens einen Kunden-Mandanten, der Traces oder Metriken sendet.
- Wählen Sie diesen **Kunden** im Mandantenumschalter aus. SQL wird nicht ausgeführt, während **Alle Konten** ausgewählt ist.
- Öffnen Sie **Erkunden** und wechseln Sie zur Registerkarte **SQL**. Verwenden **Ausführen** (⌘/Strg + Eingabetaste) nach jeder Abfrage.

### Abfragen von Ereignissen mit `SELECT`, `FROM` und `LIMIT`

Jede Abfrage benötigt zwei Klauseln: `SELECT` sagt, was zurückgegeben werden soll, und `FROM` sagt, welche Entität gelesen werden soll. Mit eingehenden Anfragen beginnen:

```sql
SELECT timestamp, service, name, duration FROM Transaction
```

Dies ist eine **Ereignisabfrage**. Jede Zeile ist eine Anfrage. Ohne `LIMIT` erhalten Sie bis zu 100 Zeilen. Weniger verlangen:

```sql
SELECT timestamp, service, name, duration FROM Transaction LIMIT 10
```

Die Zeilen werden nach der ersten Spalte in absteigender Reihenfolge sortiert. Wenn Sie also zuerst `timestamp`, erhalten Sie die neuesten Anfragen am Anfang.

&#x200B;> [!NOTE]
> SQL hat keine `SELECT *`. Durch die Auflistung von Attributen werden Ergebnisse klein und lesbar gehalten. Das [Datenwörterbuch](#data-dictionary) und die automatische Vervollständigung im Editor zeigen, was verfügbar ist.

### Daten aggregieren

Die meisten Dashboards zeigen keine einzelnen Ereignisse an. Sie fassen sie zusammen. Ersetzen Sie die Attribute durch eine **Aggregator-Funktion**:

```sql
SELECT count(*) FROM Transaction
```

Das ist die Anzahl der eingehenden Anfragen im Fenster. Versuchen Sie es noch einmal:

```sql
SELECT average(duration) FROM Transaction
```

```sql
SELECT max(duration) FROM Transaction
```

```sql
SELECT uniqueCount(host) FROM Transaction
```

### Mehrere Aggregate gleichzeitig zurückgeben

Trennt Aggregate durch Kommas und benennt jedes mit `AS`:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

&#x200B;> [!TIP]
> Aliase werden zu Spaltenüberschriften und Legendenbeschriftungen, auf die Sie in `ORDER BY` verweisen. Gewöhnen Sie sich an, jedes Aggregat zu benennen.

### Zeitfenster festlegen

Standardmäßig umfasst eine Abfrage die **letzte Stunde**. Verwenden Sie `SINCE`, um weiter zurückzublicken:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 1 day ago
```

Verwenden Sie `UNTIL`, um das Fenster in der Vergangenheit zu beenden. Dies zählt den gestrigen Traffic von vor 48 bis 24 Stunden:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

Einheiten können `second`, `minute`, `hour`, `day` oder `week` sein. Das Fenster kann höchstens 31 Tage betragen.

&#x200B;> [!IMPORTANT]
> In **Erkunden** und **Dashboards** steuert die **Zeitauswahl** oben auf der Seite das Fenster und überschreibt `SINCE`/`UNTIL`. Um einen anderen Punkt anzuzeigen, ändern Sie die Auswahl. Der Rest dieses Handbuchs lässt `SINCE` aus diesem Grund aus Dashboard-Abfragen aus.

### Trends mit `TIMESERIES` grafisch darstellen

Fügen Sie `TIMESERIES` hinzu, um eine einzelne Zahl im Laufe der Zeit in eine Zeile umzuwandeln:

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

`AUTO` wählt eine Bucket-Größe aus, sodass das Diagramm etwa 120 Punkte aufweist (2 Minuten für ein 1-Stunden-Fenster, 15 Minuten für 24 Stunden). Sie können den Bucket auch explizit festlegen:

```sql
SELECT average(duration) AS avgMs FROM Transaction TIMESERIES 5 minutes
```

### Mit `WHERE` filtern

`WHERE` behält nur die Zeilen, die Ihnen wichtig sind. Nur fehlgeschlagene Anfragen zählen:

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error'
```

Bedingungen mit `AND`, `OR` und Klammern kombinieren:

```sql
SELECT count(*) AS slowOrFailed FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR duration > 2000)
```

Verwenden Sie `LIKE` mit `%` für Teilübereinstimmungen:

```sql
SELECT count(*) AS contentRequests FROM Transaction WHERE urlPath LIKE '/content/%'
```

&#x200B;> [!NOTE]
> Bei Zeichenfolgenwerten wird zwischen Groß- und Kleinschreibung unterschieden und einfache Anführungszeichen verwendet: `'Error'`, nicht `"Error"` oder `'error'`. Ersetzen Sie `checkout` durch einen Dienst Ihres Mandanten. Im nächsten Schritt erfahren Sie, wie Sie sie auflisten.

### Mit `FACET` gruppieren

`FACET` gruppiert Ergebnisse, wie die `GROUP BY` von SQL. Listen Sie Ihre Services auf und geben Sie an, wie ausgelastet jeder ist:

```sql
SELECT count(*) AS requests FROM Transaction FACET service
```

Eine Facettenabfrage gibt standardmäßig die 10 wichtigsten Gruppen zurück. Verwenden Sie `LIMIT`, um mehr zu sehen:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs FROM Transaction FACET service LIMIT 50
```

Kombinieren Sie `FACET` und `TIMESERIES`, um eine Zeile pro Gruppe zu erhalten:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Hier hält `LIMIT 5` die fünf meistbeschäftigten Serien.

### Speichern der Abfrage in einem Dashboard

1. Öffnen Sie **Dashboards → Dashboard erstellen → ein benutzerdefiniertes Dashboard erstellen** und nennen Sie es _Mein erstes Dashboard_.
2. Klicken Sie auf **Widget hinzufügen**, wählen Sie **Einzelabfrage** aus und wechseln Sie zum Umschalter **SQL**.
3. Fügen Sie die Abfrage aus Schritt 6.7 ein und klicken Sie auf **Abfrage ausführen**.
4. Legen Sie **Titel** auf _Anfragen nach Service_ fest und wählen Sie den Diagrammtyp **Linie** aus.
5. Klicken Sie **Widget speichern** und dann **Speichern** in der oberen Leiste.

Sie haben Ihr erstes Dashboard erstellt.
