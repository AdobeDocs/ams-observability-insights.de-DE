---
title: Fehlerbehebung
description: Fehlerbehebung bei SQL-Abfragen, Diagrammen und Dashboards in Observability Insights.
source-git-commit: c45d62d2af62e6bd0c88e210b31c8954b75533be
workflow-type: tm+mt
source-wordcount: '335'
ht-degree: 6%
---

## Fehlerbehebung {#troubleshooting}

Fehlermeldungen werden unter dem Editor angezeigt, wenn Sie **Abfrage** oder **ausführen** und im Widget selbst in einem Dashboard. Suchen Sie die Nachricht in der ersten Spalte.

| Nachricht/Symptom | Ursache | Fehlerbehebung |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Unknown entity 'X'` | Typo- oder Nicht-Katalogname | Überprüfen Sie [Datenwörterbuch](#data-dictionary) / das Bedienfeld Abfragehandbuch . |
| `Attribute 'x' is not available on Entity` | Attribut nicht auf dieser Entität (z. B. `duration` auf `CpuUtilization`) | Trace-Attribute vs. Metrikattribute - siehe [Datenwörterbuch](#data-dictionary). |
| `Raw attributes cannot be mixed with aggregates; use FACET to group results` | `SELECT service, count(*)` | `SELECT count(*) … FACET service`. |
| `FACET and TIMESERIES require an aggregate query` | Ereignisabfrage mit FACET/TIMESERIES | Aggregat verwenden. |
| `FACET supports at most five attributes` | > 5 Facetten | Reduzieren. |
| `average requires a numeric attribute` | e.g. `average(service)` | Verwenden Sie `uniqueCount`/`latest` oder ein numerisches Attribut. |
| `rate requires TIMESERIES` / `rate is only available for counter measures` | `rate()` ohne TIMESERIES oder auf einem Tacho | `TIMESERIES AUTO` hinzufügen; ✔ Zählerentität verwenden ([Kuratierte Metrikentitäten](#53-curated-metric-entities)); für Messgeräte `derivative()` verwenden. |
| `rate cannot be combined with other aggregates` | Mehrere Elemente mit `rate()` | Ein `rate()` pro Widget; mit einer Formel kombinieren. |
| `percentOfTotal cannot be combined with TIMESERIES` | — | ZEITREIHEN entfernen. |
| `apdex requires an entity with a status attribute` | `apdex` einer Metrik-Entität | Verwenden Sie `Transaction`/`Span`/`AppRequest`. |
| `ORDER BY 'x' must reference a selected value or alias` | Sortierung nach etwas nicht ausgewähltem | Alias Das Aggregat und die Reihenfolge nach dem Alias. |
| `IN requires between 1 and 100 values` | — | Aufspaltung oder Verwendung von `LIKE`. |
| `Use IS NULL or IS NOT NULL for null comparisons` | `attr = NULL` | Verwenden von `IS NULL`. |
| `Unexpected character '{'` | `{{variable}}` in der Widget-Vorschau hinzufügen oder auf Speichern | Siehe [Tutorial 3](../tutorials/tutorial-3.md#make-dashboards-reusable-with-variables) Einschränkung. |
| `Unexpected character ';'` / `'"'` | Semikolons oder doppelte Anführungszeichen | `;` entfernen; einfache Anführungszeichen verwenden. |
| `Expected a positive duration such as 30 minutes` | `TIMESERIES 5m` | `TIMESERIES 5 minutes` schreiben. |
| `Time range cannot exceed 31 days` | Auswahl > 31 T | Verengt das Fenster. |
| `Select a customer before running a SQL query` | Kein Mandant/Demo-Mandant | Wählen Sie einen Kunden im Mandantenschalter aus. |
| `All queries in a formula must be the same kind…` | Mix aus Total und TIMESERIES oder einer FACETTE | Machen Sie alle Unterabfragen einfach oder alle `TIMESERIES AUTO`; entfernen Sie FACET. |
| `Query 'x' must select exactly one metric…` | Die Unterabfrage hat zwei Aggregate | Ein Aggregat pro Unterabfrage. |
| `Unknown query name 'x' - must be one of: …` | Formel verweist auf einen fehlenden Namen | Namen genau übereinstimmen (Groß- und Kleinschreibung berücksichtigen). |
| **Speichern fehlgeschlagen** | Eine Widget-Abfrage ist Server-seitig ungültig (oft `{{var}}`) | Öffnen Sie jedes kürzlich bearbeitete Widget und **Ausführen** beheben oder entfernen Sie das ungültige Widget. |
| _Wert für{{x}}_ auswählen | Variable hat keine Auswahl | Wählen Sie einen Wert in der Variablenleiste. |
| _Keine Aktivität in der letzten …_ | Keine Zeilen im Fenster | Fenster erweitern; Filter mit einer Erkennungsabfrage überprüfen ([Datenerkennung](../examples/query-cookbook.md#data-discovery)) |
| In Punkte zersplitterte Linien | Bucket kleiner als Datenkadenz | Verwenden Sie `TIMESERIES AUTO` (≥ 2 Min.) oder ein Balkendiagramm. |
| Facet zeigt eine Gruppe mit leeren Zeichenfolgen an | Attribut fehlt in einigen Zeilen | Fügen Sie `WHERE attr != ''` hinzu. |
| _Dieses alte SQL-Bedienfeld wird nicht mehr ausgeführt_ | Bedienfeld aus dem alten Raw-SQL-Builder | Bearbeiten → in SQL neu erstellen. |
