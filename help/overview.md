---
title: Überwachen Sie Ihre AEM Managed Services-Umgebung mit [!DNL Synoptryx]
description: Ein Überblick über  [!DNL Synoptryx]  Überwachung in Adobe [!DNL Experience Manager] Managed Services - was Adobe überwacht, wie Ihr Konto eingerichtet ist und wie Ihr Team Zugriff erhält.
feature: Operations
role: Admin
source-git-commit: c79ae46b8ab4f6aab02821bc4446e04a94670aef
workflow-type: tm+mt
source-wordcount: '618'
ht-degree: 0%

---


# Überwachen Ihrer AEM Managed Services-Umgebung mit [!DNL Synoptryx] {#synoptryx-monitoring}

[!DNL Synoptryx] bietet Ihrem Team Einblicke in die Anwendungsleistung, den Zustand der Infrastruktur und das Erlebnis der Endbenutzer, ohne dass eine separate Überwachungsplattform eingerichtet wird.

>[!NOTE]
>
> Ein [!DNL Synoptryx] Whitepaper mit der Produktübersicht ist für die vollständige Übersicht über die Beobachtbarkeit und Überwachung von AEM Managed Services verfügbar, das sich ideal für die Freigabe mit Stakeholdern oder die Offline-Überprüfung eignet.

## Überblick {#overview}

[!DNL Synoptryx] ist Adobes Beobachtbarkeitsplattform der nächsten Generation, die für einheitliche Sichtbarkeit der Anwendungsleistung, des Infrastrukturzustands und der synthetischen Überwachung sorgt. Sie ermöglicht die proaktive Überwachung kritischer Business-Services über ein einziges, integriertes Erlebnis. [!DNL Synoptryx] kombiniert Application Performance Monitoring (APM), Infrastructure Monitoring und synthetische User Journey-Überwachung, um Probleme zu identifizieren und zu lösen, bevor sie Endbenutzer betreffen. Die Plattform bietet tiefgehende Transaktionsnachverfolgung, JVM-Einblicke, Infrastrukturtelemetrie und erweiterte Diagnosen für eine schnellere Ursachenanalyse. Sie basiert auf modernen Observability-Technologien und bietet eine skalierbare und sichere Überwachung über komplexe Unternehmensumgebungen hinweg. [!DNL Synoptryx] bietet erweiterte Datenaufbewahrung, umfangreiche Dashboards und intelligente Analysen zur Unterstützung herausragender betrieblicher Leistung. Die nahtlose Anmeldung mit [!DNL Adobe IMS] gewährleistet sicheren Zugriff und sichere Governance. Die Plattform wurde entwickelt, um die Service-Zuverlässigkeit zu verbessern, die Fehlerbehebung zu beschleunigen und das Kundenerlebnis zu verbessern. Als strategische Observability-Lösung von Adobe bietet [!DNL Synoptryx] eine zukunftsfähige Grundlage für die Überwachung, Automatisierung und operative Einblicke in Managed Services-Umgebungen.

[!DNL Synoptryx] ist in Adobe [!DNL Experience Manager] Managed Services enthalten - es ist keine separate Überwachungsplattform oder Lizenz erforderlich. Adobe überwacht die Verfügbarkeit und Leistung Ihrer Umgebung als Teil unseres Standardangebots und [!DNL Synoptryx] ist die dedizierte Plattform, die Ihr Team verwenden kann, um zu verstehen, wie Ihre Adobe [!DNL Experience Manager] (AEM)-Anwendung und unterstützende Infrastruktur funktionieren.

In diesem Handbuch wird erläutert, was überwacht wird, wie Ihr [!DNL Synoptryx] eingerichtet wird und wie Sie durch die Dashboards navigieren, die Sie für die tägliche Analyse und Fehlerbehebung verwenden.

## Auf einen Blick {#at-a-glance}

Als Teil von AEM Managed Services erhalten Sie:

- **Dediziertes [!DNL Synoptryx]-** - Bereitstellung und Überwachung durch Adobe Managed Services mit schreibgeschütztem Zugriff für Ihr Team.
- **Deep AEM-Transaktionsüberwachung** - Der [!DNL Synoptryx] APM-Agent verfolgt aussagekräftige Transaktionen bis hin zu Methodenaufrufen (einschließlich Zeilennummern), externen Abhängigkeiten und Repository-Vorgängen.
- **Einheitliche Anwendungs- und Infrastrukturansicht** - Kombinieren Sie APM- und Host-Metriken, um die Leistung ganzheitlich zu optimieren.

## Was Adobe mit [!DNL Synoptryx] überwacht {#what-we-monitor}

Adobe überwacht die Ebenen **Autor** und **Veröffentlichung** von AEM mit dem [!DNL Synoptryx] APM Java-Plug-in. Alle gehosteten Server in Ihrer Topologie werden mit dem [!DNL Synoptryx] Infrastructure Agent überwacht. Die benutzerdefinierte Überwachung von APM und Infrastruktur ist sowohl in Nicht-Produktions- als auch in Produktionsumgebungen von Managed Services aktiviert.

![Abbildung mit der Synoptryx-APM- und -Infrastrukturüberwachung auf AEM-Autoren-, Veröffentlichungs- und gehosteten Servern](assets/image6.png)

### Anwendungen in Ihrem Konto {#applications-in-your-account}

Ihr [!DNL Synoptryx]-Konto ist mit einem einzigen Adobe-Masterkonto verknüpft und kann Daten von mehreren Programmen empfangen, darunter:

- Eine APM-Anwendung für die **Author**-Ebene pro AEM Managed Services-Umgebung
- Eine APM-Anwendung für die **Veröffentlichungs**-Ebene pro AEM Managed Services-Umgebung

Jede Anwendung verfügt über einen eigenen Lizenzschlüssel. Alle Topologien in Ihrem Managed Services-Vertragsbericht in einem [!DNL Synoptryx]. APM- und Infrastrukturmetriken und -ereignisse werden bis zu **30 Tage lang**.

## Zugriff auf und Konto {#access}

Die Überwachungsdaten werden in einem [!DNL Synoptryx] konsolidiert, das von Adobe bereitgestellt und verwaltet wird. Ihr Team erhält **vollen schreibgeschützten Zugriff** auf alle APM- und Infrastrukturmetriken, die von den Agenten erfasst wurden. Adobe Managed Services behält das Eigentum und die administrative Kontrolle über das Konto.

>[!NOTE]
>
> **Zugriff erhalten:** Zugriff auf [!DNL Synoptryx] erfordert [!DNL Adobe IMS] Bereitstellung. Ihr Customer Success Engineer (CSE) kann den Benutzerzugriff für Ihr Unternehmen bereitstellen und verwalten.

Nachdem der CSE das Konto bereitgestellt hat, können Sie sich unter [synoptryx.adobecqms.net](https://synoptryx.adobecqms.net) anmelden.

## Wie geht es weiter {#whats-next}

Fahren Sie mit den Überwachungs-Dashboards fort, die Ihr Team täglich verwendet:

- [Application Performance Monitoring (APM)](application-performance-monitoring.md) - Verfolgen Sie AEM-Transaktionen, analysieren Sie das JVM-Verhalten und untersuchen Sie externe Services.
- [Infrastrukturüberwachung](infrastructure-monitoring.md) - Überprüfen Sie System-, Netzwerk-, Prozess- und Speichermetriken auf Host-Ebene.
