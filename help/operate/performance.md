---
title: Leistung
description: Verbessern Sie die Leistung von Abfragen, Diagrammen und Dashboards in Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '227'
ht-degree: 3%
---

## Leistung {#performance}

Jede Abfrage wird mit einem 15-Sekunden-Timeout und einem Byte-Read-Budget ausgeführt. In diesem Dokument wird erläutert, wie Observability Insights Abfragen automatisch beschleunigt und wie Dashboards erstellt werden, die zu jedem Zeitpunkt schnell bleiben.

### Rollup-Beschleunigung (automatisch)

`Transaction`-Diagramme lesen voraggregierte Tabellen, wenn **alle** diese enthalten:

| Bedingung | Anforderung |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Form | Abfrage hat `TIMESERIES` (Zahlen-/Tabellensummen werden immer auf Genauigkeit geprüft) |
| Fenster | ≥ 3 h (1-Min.-Stufe) ・ ≥ 1 d (5-Min.-Stufe) ・ ≥ 7 d (1-H-Stufe) |
| Aggregate | Nur `count(*)`, `percentage(*, WHERE status = 'Error')`, `apdex(duration, 500)`, `percentile(duration, 50 \| 95 \| 99)` |
| `FACET` / `WHERE` | Nur auf `service`, `appInstance`, `kind` |

`CpuUtilization` und `MemoryUtilization` (`average(value)`) und `JvmCpuRecentUtilization` (`average(value)`) Diagramme mit `FACET host`/`WHERE host …` über ≥ 3 Stunden 1-minütige Datenaggregationen lesen.

**Design-Tipp:** halten Sie sich bei 7-31-tägigen Trend-Widgets an das rollup-freundliche Set (p95, Fehler %, Apdex @ 500, count, FACET-Service) - sie bleiben in jedem Bereich schnell. Speichern Sie `FACET host`/`name`-, `average(duration)`- oder `urlPath` für kürzere Fenster oder Tabellen.

### Allgemeine Tipps

- Verwenden Sie `LIMIT` für jede facettierte Abfrage - insbesondere `FACET urlPath`, `traceId`, `jcr.path` (hohe Kardinalität).
- Frühzeitiges Filtern mit `WHERE` auf Attribute mit niedriger Kardinalität (`service`, `kind`, `status`).
- Wenn Sie nur eingehende Anfragen benötigen, sollten Sie `Transaction` vor `Span` bevorzugen - viel weniger Zeilen.
- `uniqueCount` ist exakt; bei Attributen mit hoher Kardinalität über lange Zeitfenster ist dies das teuerste Aggregat.
- Vermeiden Sie `TIMESERIES MAX`, außer bei Fenstern von einigen Minuten.
- Dashboard-Widgets speichern die Ergebnisse 30 s lang. Die Schaltfläche „Aktualisieren“ erzwingt eine erneute Ausführung.
- Ein Widget, das die 15-Sekunden-Zeitüberschreitung oder die Byte-Lesen-Budgetfehler erreicht - Grenzen Sie das Fenster ein, fügen Sie Filter hinzu oder reduzieren Sie Facetten.
