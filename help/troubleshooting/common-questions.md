---
title: Häufig gestellte Fragen
description: Häufige Fragen und Ausgangspunkte für Untersuchungen zu Observability Insights in AEM Managed Services.
feature: Operations
role: Admin
source-git-commit: 3e9cd3734665dc06a4b90902b229dffb8f5421df
workflow-type: tm+mt
source-wordcount: '216'
ht-degree: 0%

---


# Häufig gestellte Fragen {#faq}

Verwenden Sie diese Seite als Ausgangspunkt, wenn Sie sich nicht sicher sind, wo Sie beginnen sollen, oder wenn Sie während einer aktiven Untersuchung eine schnelle Antwort benötigen.

## Warum kann ich nicht auf Observability Insights zugreifen? {#cannot-access-observability-insights}

Beginnen Sie mit [Zugriff und Kontoverwaltung](../get-started/access-and-accounts.md). Wenn Ihre Bereitstellung unvollständig oder veraltet ist, wenden Sie sich an Ihren Customer Success Engineer (CSE), um Zugriff oder eine Aktualisierung anzufordern.

## Warum wird „Berechtigungen werden geladen“ angezeigt, wenn ich versuche, mich anzumelden? {#loading-permissions-error}

Dies weist normalerweise auf ein Problem bei der Benutzerbereitstellung hin. Wenden Sie sich an Ihren Customer Success Engineer (CSE), der das Zugriffsproblem mit den entsprechenden Teams lösen kann.

## Wie kann ich feststellen, ob ein Problem mit der Anwendung oder der Infrastruktur zusammenhängt? {#application-or-infrastructure}

Beginnen Sie mit [Anwendungsleistungsüberwachung](/help/applications.md), um Anfrageraten, Fehlerquoten und Latenzen bei der Autoren- oder Veröffentlichungsinstanz zu überprüfen. Wenn die Anwendungssignale erhöht sind, verwenden Sie [Hosts](/help/hosts.md), um zu überprüfen, ob der Ressourcendruck auf Host-Ebene - CPU, Arbeitsspeicher, Festplatte oder Netzwerk - erklärt oder erweitert, was Sie sehen.

## Wie sollte ich ein bestimmtes Diagramm oder eine bestimmte Metrik verstehen? {#understand-graph-or-metric}

Verwenden Sie die Dashboard-Referenzseiten für Bereichsbeschreibungen, Metriknamen, Einheiten und Screenshots:

- [APM-Dashboard-Referenz](../reference/apm-dashboard-reference.md)
- [Referenz zum Infrastruktur-Dashboard](../reference/infrastructure-dashboard-reference.md)

## Welche Daten werden von Observability Insights tatsächlich erfasst? {#what-data-is-collected}

Siehe [Abdeckung, Umgebungen und Datenaufbewahrung](../get-started/coverage-and-data.md) für den Überwachungsumfang, die Anwendungsdarstellung, die Aufbewahrungsfristen und die betrieblichen Auswirkungen.
