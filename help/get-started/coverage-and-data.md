---
title: Abdeckung, Umgebungen und Datenaufbewahrung
description: Hier erfahren Sie, was Observability Insights in AEM Managed Services überwacht, wie Programme dargestellt werden und wie lange Überwachungsdaten aufbewahrt werden.
feature: Operations
role: Admin
source-git-commit: 1d54a6a398360b040221db5b2780d301722894bf
workflow-type: tm+mt
source-wordcount: '267'
ht-degree: 2%

---


# Abdeckung, Umgebungen und Datenaufbewahrung {#coverage-environments-and-data-retention}

Auf dieser Seite wird zusammengefasst, welche Daten in Observability Insights für AEM Managed Services erfasst werden und wie diese Daten organisiert sind.

## Überwachungsbereich {#monitoring-coverage}

Adobe Monitore:

- AEM-Autorenebenen mit dem Observability Insights-APM-Java-Plug-in
- AEM-Veröffentlichungsebenen mit dem Observability Insights-APM-Java-Plug-in
- Gehostete Server in der verwalteten Topologie mit dem Observability Insights-Infrastrukturagenten

Die benutzerdefinierte Überwachung von APM und Infrastruktur ist sowohl in Nicht-Produktions- als auch in Produktionsumgebungen von Managed Services aktiviert.

## Darstellung der Anwendungen {#how-applications-are-represented}

Jede AEM Managed Services-Umgebung umfasst in der Regel:

- Eine APM-Anwendung für Autor
- Eine APM-Anwendung zum Veröffentlichen

Alle Topologien in einem Managed Services-Vertragsbericht in ein Observability Insights-Konto.

## Datenaufbewahrung {#data-retention}

APM-Metriken, Infrastrukturmetriken und zugehörige Ereignisse werden bis zu **30 Tage lang**.

## Zusammenfassungstabellen {#summary-tables}

| Versorgungsbereich | Was überwacht wird |
| -------------- | ------------------------------------------ |
| APM | AEM-Autoren- und -Veröffentlichungsanwendungen |
| Infrastruktur | Alle gehosteten Server in der verwalteten Topologie |

| Element | Darstellung |
| ------------------------------ | ------------------------------------------------------------- |
| AEM-Umgebung | Eine Authoring-APM-Anwendung und eine Publish-APM-Anwendung |
| Observability Insights-Konto | Ein von Adobe verwaltetes Konto pro Managed Services-Kundenbereich |

| Datentyp | Treue |
| --------------------------------- | ------------- |
| APM-Metriken und -Ereignisse | Bis zu 30 Tage |
| Infrastrukturmetriken und Ereignisse | Bis zu 30 Tage |

## Was dies operativ bedeutet {#what-this-means-operationally}

- Observability Insights ist für operative Analysen, aktive Incidents und aktuelle Trendvergleiche geeignet.
- Historische Analysen, die über das Aufbewahrungsfenster hinausgehen, sollten bei Bedarf durch andere Reporting- oder Archivierungsprozesse verarbeitet werden.
- Erfassen Sie bei der Untersuchung wiederkehrender Probleme Screenshots oder exportierte Beweise, bevor die Daten auslaufen.
