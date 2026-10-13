---
title: Anhang
description: Referenz-Zusatzinformationen für SQL-Abfragen und Dashboards zu Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '145'
ht-degree: 7%
---

## Anhang {#appendix}

**In diesem Dokument:** [14.1 Beschränkungen](#141-limits) ・ [14.2 Dateiformat exportieren](#142-export-file-format) ・ [14.3 One-Page Cheat Sheet](#143-one-page-cheat-sheet)

### Beschränkungen

| Element | Beschränkung |
| -------------------------------- | ----------------------------------------------------------------------- |
| Abfragetext | 4 KB |
| `FACET`-Attribute | 5 |
| `LIMIT` | 1 - 5 000 (Standard: 1, Facette 10, Ereignisse 100, Zeitreihe 5 000) |
| `IN (…)` Werte | 1 - 100 |
| Zeitfenster | 31 Tage |
| Zeitüberschreitung der Abfrage | 15 s (plus ein Byte-Lesebudget) |
| `TIMESERIES AUTO` minimaler Bucket | 2 Minuten |
| Formel | 2 - 6 Abfragen; Ausdruck ≤ 500 Zeichen; Namen ≤ 40 Zeichen |
| Dashboard | ≤ 40 Registerkarten ・ ≤ 200 Widgets pro Registerkarte ・ ≤ 50 Variablen |
| Widget | Titel ≤ 200 ・ Untertitel ≤ 500 ・ Einheit ≤ 40 ・ ≤ 10 Schwellenwerte |
| Text-Widget | ≤ 4 000 Zeichen |
| Variable | Name ≤ 64 Zeichen ・ ≤ 500 Optionen/Werte ・ Abfrage ≤ 4 KB |

### Dateiformat exportieren

Verwenden Sie **Export**, um dies zu erstellen **Import**, um es zu laden. Nützlich für Dashboards zur Versionskontrolle oder zur Weiterleitung zwischen Mandanten.

```json
{
  "name": "Golden Signals",
  "templateVariables": [
    {
      "id": "var-1",
      "name": "svc",
      "label": "Service",
      "kind": "query",
      "multi": true,
      "query": "SELECT count(*) FROM Transaction FACET service LIMIT 200",
      "value": null
    }
  ],
  "pages": [
    {
      "id": "page-1",
      "name": "Overview",
      "widgets": [
        {
          "id": "sec-kpis",
          "title": "SLO KPIs",
          "queryLanguage": "text",
          "text": "",
          "isSection": true,
          "collapsed": false,
          "layout": { "x": 0, "y": 0, "w": 12, "h": 1 }
        },
        {
          "id": "w-error-rate",
          "title": "Error %",
          "queryLanguage": "SQL",
          "query": "SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction",
          "vizType": "stat",
          "unit": "%",
          "thresholds": [
            { "value": 0, "status": "ok" },
            { "value": 1, "status": "warning" },
            { "value": 5, "status": "error" }
          ],
          "layout": { "x": 0, "y": 1, "w": 3, "h": 2 }
        },
        {
          "id": "w-throughput",
          "title": "Throughput by service",
          "queryLanguage": "SQL",
          "query": "SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8",
          "vizType": "area",
          "layout": { "x": 3, "y": 1, "w": 9, "h": 4 }
        },
        {
          "id": "w-availability",
          "title": "Availability %",
          "queryLanguage": "SQL-formula",
          "formulaQueries": [
            {
              "name": "ok",
              "query": "SELECT count(*) AS v FROM Transaction WHERE status != 'Error'"
            },
            {
              "name": "total",
              "query": "SELECT count(*) AS v FROM Transaction"
            }
          ],
          "formulaExpression": "(ok / total) * 100",
          "vizType": "stat",
          "unit": "%",
          "layout": { "x": 0, "y": 3, "w": 3, "h": 2 }
        }
      ]
    }
  ],
  "exportedAt": "2026-10-01T00:00:00.000Z"
}
```

`vizType` Werte: `timeseries` (Zeile), `area`, `bar`, `pie`, `stat` (Zahl), `table`. Schwellenwert `status`: `ok` (grün), `warning` (gelb), `error` (rot).

### Einseitige Spickzettel

```text
SHAPE        SELECT agg FROM Entity                         -> Number
             ... FACET a[, b..e] LIMIT n                     -> Bar / Pie / Table
             ... TIMESERIES AUTO                             -> Line / Area
             ... FACET a TIMESERIES AUTO LIMIT n             -> top-n series
             SELECT attr1, attr2 FROM Entity LIMIT n         -> event Table

AGGREGATES   count(*)  average(x)  sum(x)  min(x)  max(x)  uniqueCount(x)
             percentile(x, 95)  stddev(x)  latest(x[, rank])  earliest(x[, rank])
             percentage(*, WHERE c)  apdex(duration, 500)  percentOfTotal(x)
             rate(value)        -- counters, TIMESERIES only
             derivative(value)  -- gauges,   TIMESERIES only
             agg FILTER (WHERE c) AS alias

WHERE        = != < <= > >=   LIKE / NOT LIKE ('%', '_')   IN / NOT IN (...)
             IS [NOT] NULL   AND OR NOT ( )   strings in 'single quotes'
             string attr present?  ->  attr != ''

ORDER/LIMIT  ORDER BY alias [ASC|DESC]   LIMIT 1..5000

VARIABLES    = {{name}}   IN ({{name}})   numeric: > {{name}}

FORMULA      2-6 named queries, one metric each, no FACET, same shape
             expression: + - * / ( ) numbers names
```
