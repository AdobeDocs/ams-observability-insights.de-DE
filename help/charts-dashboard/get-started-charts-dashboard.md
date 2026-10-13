---
title: Erste Schritte mit Diagrammen und Dashboards
description: Erfahren Sie, wie Sie Observability Insights-Daten mit Diagrammen und Dashboards visualisieren können.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 3%
---

## Diagramme und Dashboards: Visualisieren Ihrer Daten {#charts-and-dashboards-visualizing-your-data}

Bei Abfragen werden die benötigten Informationen gefunden. Diagramme und Dashboards machen diese Informationen für Ihr Unternehmen nützlich. Dieses Dokument zeigt, wie SQL-Ergebnisse zu Diagrammen werden und wie Dashboards erstellt, organisiert und freigegeben werden.

### Diagramme

Jedes SQL-Ergebnis kann als einer von sechs Diagrammtypen dargestellt werden.

| Diagrammtyp | Verwenden Sie ihn für | Form der Abfrage |
| --------------- | --------------------------------------------------- | ----------------------------------------------- |
| **line** | Trends im Zeitverlauf, eine Zeile pro Facettenwert | Aggregat + `TIMESERIES` |
| **Bereich** | Volumen im Zeitverlauf, z. B. Durchsatz nach Service | Aggregat + `TIMESERIES` |
| **bar** | Kategorien vergleichen; Zeitreihen mit geringer Dichte | Aggregat + `FACET` (± `TIMESERIES`) |
| **Torte/Ringdiagramm** | Anteil eines Ganzen (Status-Code-Mix, Traffic nach Ebene) | Ein Aggregat + `FACET`, wenige Werte |
| **Zahl** | Ein einzelner KPI mit optionalen Farbschwellenwerten | Reklametafel (nur Aggregate) |
| **Tabelle** | Mehrere Aggregate pro Gruppe oder Ereignislisten | Mehrere Aggregate + `FACET` oder Rohattribute |

Nachdem sich ein Widget in einem Dashboard befindet, haben Sie folgende Möglichkeiten:

- **Bearbeiten** die Abfrage, den Titel, den Diagrammtyp und die Einheit (Stiftsymbol).
- **Duplizieren** um eine Variante zu erstellen, z. B. dasselbe Diagramm für eine andere Ebene.
- **CSV herunterladen** der aktuellen Zeilen oder **PNG** mit 2× Auflösung.
- **Ziehen Sie** am Griffpunkt und **ändern** Sie ihn von der rechten unteren Ecke.

### Dashboards

Diagramme funktionieren für sich genommen gut, aber Dashboards bringen sie zusammen. Ein Dashboard ist eine benannte, für Mandanten bestimmte Sammlung von **Registerkarten**. Jede Registerkarte ist ein 12-spaltiges Raster aus **Widgets**.

| Konzept | Bedeutung |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Dashboard** | Eine benannte Sammlung von 1-40 Registerkarten, die Server-seitig für den ausgewählten Mandanten gespeichert werden. |
| **Registerkarte** | Ein Raster von Widgets (bis zu 200). Benennen Sie ihn mit dem Stift um, löschen Sie ihn mit × und fügen Sie einen mit **+** hinzu. Mindestens eine Registerkarte muss bestehen bleiben. |
| **Widget** | Eine **einzelne Abfrage**, eine **Formel** oder **Text** (eine Anmerkung oder eine ausblendbare Abschnittskopfzeile). |
| **Variable** | Eine Auswahl über dem Raster. Sein Wert wird als `{{name}}` eingefügt. |
| **Zeitauswahl** | Das Fenster für jedes Widget im Dashboard. |
