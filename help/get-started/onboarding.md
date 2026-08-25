---
title: Erste Schritte mit Observability Insights
description: Erfahren Sie in diesem Handbuch , wie Sie auf Observability Insights zugreifen, was Adobe in Ihrem Namen überwacht und wo Sie finden können, was Sie benötigen.
feature: Operations
role: Admin
source-git-commit: cc405e8b70973c33ecc6137114315998e8f9af50
workflow-type: tm+mt
source-wordcount: '533'
ht-degree: 0%

---


# Erste Schritte mit Observability Insights {#get-started}

Dieser Abschnitt behandelt die Grundlagen für neue Benutzende: wie Sie auf Ihr Observability Insights-Konto zugreifen, welche Umgebungen und Daten Adobe in Ihrem Namen überwacht und wie Sie durch den Rest dieser Dokumentation navigieren.

## Die Benutzeroberfläche von Observability Insights {#observability-insights-interface}

Wenn Sie sich unter [insights.adobecqms.net](https://insights.adobecqms.net) anmelden, erhalten Sie auf dem Startbildschirm einen Einstieg in alle Überwachungsbereiche für Ihre AEM Managed Services-Umgebungen.

![Öffner-Bildschirm von Observability Insights mit Einstiegspunkten zur APM- und Infrastrukturüberwachung](../v2-assets/observability-catalog-listing.png)

Die Benutzeroberfläche ist in zwei zentrale Überwachungsbereiche unterteilt:

- **Anwendungen** - Zeigt Anwendungsleistungsdaten für Ihre Autoren- und Veröffentlichungsebenen an. Verwenden Sie diese Option, um den Anfragedurchsatz, die Fehlerquoten, die Latenz, das JVM-Verhalten und die Ausführungsdetails auf Trace-Ebene zu untersuchen. Siehe [Programme](../applications.md).
- **Hosts** - Zeigt Integritätsdaten auf Host-Ebene in der verwalteten Topologie an. Verwenden Sie diesen Parameter, um CPU-, Speicher-, Datenträger-, Netzwerk- und Speichersignale auf den einzelnen Servern zu bewerten. Siehe [Hosts](../hosts.md).

Beide Bereiche sind für Kunden-Benutzer schreibgeschützt. Adobe Managed Services verwaltet die Kontobereitstellung, Instrumentierung und administrative Kontrolle.

## Zugriff und Kontoverwaltung {#access-overview}

Der Zugriff auf Observability Insights wird über Adobe IMS verwaltet. Adobe stellt das Konto Ihres Unternehmens bereit und verwaltet es. Kunden-Teams erhalten schreibgeschützten Zugriff auf alle überwachten Daten.

Wichtigste Punkte:

- Das Observability Insights-Konto Ihres Unternehmens ist mit einem einzigen Adobe-Masterkonto verknüpft.
- Alle Umgebungen in Ihrem Managed Services-Vertrag - Autor und Veröffentlichung, Produktion und produktionsfremd - melden Berichte in diesem Konto.
- Der Benutzerzugriff wird von Ihrem Customer Success Engineer (CSE) bereitgestellt und verwaltet.

Informationen zu Bereitstellungsschritten, Benutzerrollen und dazu, was Kundenbenutzer tun und nicht tun können, finden Sie unter [Zugriff und Kontoverwaltung](access-and-accounts.md).

## Abdeckung, Umgebungen und Datenaufbewahrung {#coverage-overview}

Adobe überwacht Ihre AEM-Autoren- und Veröffentlichungsebenen mit dem Observability Insights-APM-Java-Plug-in und alle gehosteten Server mit dem Observability Insights Infrastructure Agent. Die Überwachung ist sowohl in Nicht-Produktions- als auch in Produktionsumgebungen aktiviert.

Wichtigste Punkte:

- Jede AEM Managed Services-Umgebung enthält eine APM-Anwendung für Autoren- und eine für Veröffentlichungsumgebungen.
- APM-Metriken, Infrastrukturmetriken und Ereignisse werden bis zu **30 Tage lang**.
- Observability Insights eignet sich für operative Analysen und Trendvergleiche der letzten Zeit; es ist kein Archivierungs- oder langfristiges Reporting-Tool. Erfassen Sie Screenshots oder exportierte Beweise, bevor die Daten auslaufen.

Details zur vollständigen Abdeckung, einschließlich der Darstellung von Anwendungen in Ihrem Konto und der betrieblichen Auswirkungen des Aufbewahrungsfensters, finden Sie unter [Abdeckung, Umgebungen und Datenaufbewahrung](coverage-and-data.md).

## Strukturierung dieses Handbuchs {#how-this-guide-is-structured}

Die Dokumentation gliedert sich in vier Bereiche. Verwenden Sie die folgenden Beschreibungen, um direkt zu dem zu gelangen, was Sie benötigen.

**Erste Schritte** - Dieser Abschnitt. Behandelt Zugriff, Kontobereitstellung, Überwachungsumfang und Datenspeicherung.

**[Verwenden von Observability Insights](../use-observability-insights.md)** - Aufgabenorientierte Anleitung für die tägliche Untersuchung. Verwenden Sie [Anwendungen](../applications.md) wenn das Symptom anwendungsbezogen ist: langsame Seiten, Fehlerspitzen oder instabile Transaktionen. Verwenden Sie [Hosts](../hosts.md), wenn Sie feststellen müssen, ob der Ressourcendruck auf Host-Ebene - CPU, Arbeitsspeicher, Festplatte oder Netzwerk - erklärt, was Sie in Anwendungen sehen. Eine schrittweise Anleitung zur Untersuchung von [&#x200B; finden Sie unter &quot;](../use-cases/investigate-application-issues.md) untersuchen“ und [Infrastrukturprobleme untersuchen](../use-cases/investigate-infrastructure-issues.md).

**[Häufig gestellte Fragen](../troubleshooting/common-questions.md)** - Häufig gestellte Fragen und Support-orientierte Einstiegspunkte für den Fall, dass Sie sich nicht sicher sind, wo Sie anfangen sollen, oder während eines aktiven Vorfalls schnelle Antworten benötigen.
