---
title: Erste Schritte mit SQL
description: Erfahren Sie, wie Sie die Observability Insights-Telemetrie mit SQL abfragen.
source-git-commit: 148bf6480461a0109b221b8e7c2f9ab2035a7d68
workflow-type: tm+mt
source-wordcount: '687'
ht-degree: 1%
---

# Erste Schritte

**Adobe Observability Insights Query Language (SQL)** - Abfrage, Diagramm und Dashboard Ihrer Telemetrie in Adobe Observability Insights.

|                |                                                                  |
| -------------- | ---------------------------------------------------------------- |
| **Gilt für** | Insights-Benutzeroberfläche → **Erkunden** und **Dashboards** (benutzerdefinierte Dashboards) |
| **Zielgruppe** | Ingenieure, SREs und Power-User, die SQL kennen |

> **Tipp**
>
> Jeder Code-Block mit der Kennzeichnung `sql` ist eine SQL-Abfrage, die Sie direkt in **Erkunden → SQL** oder in **Dashboards → Widget hinzufügen → SQL** einfügen können.

## Erste Schritte mit SQL: die Sprache Ihrer Telemetrie {#get-started-with-sql-the-language-of-your-telemetry}

Insights-Abfragesprache ist eine SQL-ähnliche Abfragesprache für Ihre Observability Insights-Daten. Es ist dieselbe Sprache hinter jedem benutzerdefinierten Dashboard-Widget und der Seite **Erkunden**, sodass Sie die Ansichten erstellen können, die Ihr Team benötigt.

SQL arbeitet mit **Entitäten** zusammen, die benutzerfreundliche logische Ansichten wie `Transaction`, `Span` oder `CpuUtilization` sind. Sie fragen Speichertabellen nie direkt ab. Observability Insights kompiliert jede SQL-Abfrage in einer sicheren, parametrisierten Warehouse-Abfrage. Diese Abfrage gilt immer für Ihren Mandanten und das ausgewählte Zeitfenster.

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

### Was Sie mit SQL tun können

SQL verwenden für:

- **Build-Diagramme** mit Durchsatz, Fehlerrate, Latenz, Apdex, CPU, Arbeitsspeicher, JVM-Heap, Festplatte und Netzwerk im Zeitverlauf.
- **Beantworten Sie spezifische**, z. B. „Welche URL-Pfade auf der Veröffentlichungsebene haben die schlechtesten p95?“ oder „Welche nachgelagerte Domain schlägt fehl?“
- **Untersuchen Sie Vorfälle** indem Sie von einem KPI bis hin zu einzelnen Anfragen und Trace-IDs bohren.
- **Dashboards erstellen** mit KPIs, Trends, Tabellen, Formeln, Anmerkungen, Abschnittsüberschriften und Variablen.
- **Profile AEM Workloads** (Adobe Experience Manager) mit den Attributen Sling, JSP, JCR, Dispatcher und GraphQL.

SQL unterstützt sowohl **Ereignisabfragen**, die einzelne Anfragezeilen zurückgeben, als auch **Aggregatabfragen**, die Gesamtsummen, Perzentile, Raten, Aufschlüsselungen und Zeitreihen zurückgeben.

> **Tipp**
>
> Die Vorlagen in **Dashboards → Dashboard erstellen** werden vollständig aus SQL erstellt. Öffnen Sie ein beliebiges Vorlagen-Widget mit dem Stiftsymbol, um die zugehörige Abfrage anzuzeigen und daraus zu lernen.

### Was Sie abfragen können

SQL kann vier Datenfamilien abfragen. Die vollständige Liste finden Sie unter [Datenwörterbuch](/help/references/data-dictionary.md).

| Familie | Entitäten | Typische Fragen |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| **TRACES (APM)** | `Transaction` (eingehende Anfragen), `AppRequest` (Anfragen + Nachrichtenkonsumenten), `Span` (alle Bereiche) | Durchsatz, Latenz, Fehler, APDEX, nachgelagerte Aufrufe, AEM-Codepfade |
| **Kuratierte Host- und Laufzeitmetriken** | `CpuUtilization`, `MemoryUtilization`, `LoadAverage5m`, `DiskUtilization`, `NetworkReceiveRate`, `JvmHeapUsed`, `JvmThreadCount`, `DispatcherRequestRate` und mehr | Sättigung, Kapazität, JVM-Status, Dispatcher-Auslastung |
| **Generische Metriken** | `MetricGauge`, `MetricSum` | Beliebige OpenTelemetry-Metriken nach Name, Metrikerkennung |
| **Voraggregierte span-Komposition** | `SpanBreakdown` | Welche untergeordneten Vorgänge eine Transaktion bilden |

> **Hinweis**
>
> Protokolle, Warnhinweisdefinitionen und Inventarbeziehungen sind nicht verfügbar.

### Verwendung von SQL

| Dabei gilt | Anleitung zum Öffnen | Wofür es ist |
| ----------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Erkunden** | Linke Navigation → **Erkunden** | Ad-hoc-Analyse. Verwenden Sie den **Visual Builder** oder die **SQL**-Registerkarte **Validieren** dann **Ausführen** (⌘/Strg+Eingabe). Enthält **Verlauf** und **Gespeicherte** Abfragen und ein Bedienfeld **Anleitung für Abfragen**. |
| **Dashboards** | Die linke Navigation → **Dashboards** → ein Dashboard öffnen oder erstellen → **Widget hinzufügen** | Beständige Diagramme, KPIs und Tabellen, kombiniert in Dashboards. |
| **Formel-Widgets** | **Widget → Formel hinzufügen** | Kombinieren Sie 2-6 SQL-Abfragen mit Arithmetik (Verhältnisse, Reserven, Abbrandraten). |
| **Dashboard-Variablen** | **+ Variable → aus einer Abfrage** | Füllen Sie eine Auswahl aus den unterschiedlichen Werten einer SQL `FACET`-Abfrage. |
| **Abfragenprüfung** | Bewegen Sie den Mauszeiger über ein Diagramm → überprüfen Sie die Abfrage | SQL hinter einem integrierten Diagramm oder Dashboard-Diagramm anzeigen. |

### SQL-Abfragestruktur

SQL folgt einer auf einer Klausel basierenden Syntax. Diese Abfrage zählt alle eingehenden Anfragen im ausgewählten Fenster:

```sql
SELECT count(*) FROM Transaction
```

Im Folgenden finden Sie alle verfügbaren Klauseln:

```text
SELECT  function(attribute) [FILTER (WHERE condition)] [AS alias] [, ...]
        | attribute [AS alias] [, ...]
FROM    Entity
  [WHERE      condition [AND | OR ...]]
  [FACET      attribute [, attribute ...]]        -- up to 5
  [SINCE      n unit [AGO]]
  [UNTIL      NOW | n unit [AGO]]
  [TIMESERIES AUTO | MAX | n unit]
  [ORDER BY   alias [ASC | DESC]]
  [LIMIT      number]                              -- 1 to 5000
```

Im Folgenden finden Sie die SQL-Regeln:

| Regel | Details |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Erforderliche Klauseln** | `SELECT` und `FROM` sind erforderlich. Jede andere Klausel ist optional. Sie können mit einem der folgenden Elemente beginnen: `FROM Transaction SELECT count(*)` ist auch gültig. |
| **Klauselreihenfolge** | Die Klauseln nach `FROM` können in beliebiger Reihenfolge angezeigt werden. Jede Klausel darf nur einmal angezeigt werden. |
| **Abfragengröße** | Eine Abfrage muss 4 KB oder weniger betragen. |
| **Groß-/Kleinschreibung** | Bei Schlüsselwörtern, Funktionsnamen, Entitätsnamen und Attributnamen wird **Groß-** Kleinschreibung unterschieden. Bei Zeichenfolgenwerten **wird zwischen** Groß- und Kleinschreibung unterschieden (`'Error'` ≠ `'error'`). |
| **Zeichenfolgen** | Verwenden Sie einfache Anführungszeichen: `WHERE status = 'Error'`. Um ein Zitat einzuschließen, verdoppeln Sie es: `'O''Brien'`. Doppelte Anführungszeichen werden nicht akzeptiert. |
| **Literale** | Zeichenfolgen, Zahlen (einschließlich Negativ- und Dezimalzahlen), `TRUE`, `FALSE`, `NULL`. |
| **Attributnamen** | Namen können Buchstaben, Ziffern, `_` und `.` enthalten (zum Beispiel `http.statusCode`). Es sind keine Backticks erforderlich. |
| **Aggregate vs. Attribute** | Bei einer Abfrage werden entweder Rohattribute (eine Ereignisliste) oder Aggregate ausgewählt, aber nie beides. Verwenden Sie zum Gruppieren `FACET`. |
| **Zeitfenster** | Der Standardwert ist die letzte Stunde. Die maximale Wartezeit beträgt 31 Tage. In Dashboards und im Explorer **die** das Fenster (siehe [Zeitfenster](how-to-query-sql.md#time-window-precedence)). |
| **Nicht unterstützt** | Semikolons, Kommentare, Unterabfragen, Joins, `SELECT *`, Mathematik zwischen Attributen in `SELECT`, `HAVING`, `COMPARE WITH`. Siehe [Was SQL nicht tut](sql-query-examples.md#what-sql-doesnt-do) für Alternativen. |
