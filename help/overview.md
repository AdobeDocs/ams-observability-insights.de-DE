---
title: Überwachen Ihrer AEM Managed Services-Umgebung mit Observability Insights
description: Beginnen Sie hier, um zu verstehen, was Observability Insights in AEM Managed Services behandelt, für wen es sich handelt und wie Sie durch den Rest dieses Handbuchs navigieren können.
feature: Operations
role: Admin
source-git-commit: 90ca53475d23dd9b3100236d899d3941f717edbd
workflow-type: tm+mt
source-wordcount: '744'
ht-degree: 0%

---


# Überwachen Ihrer AEM Managed Services-Umgebung mit Observability Insights {#observability-insights-monitoring}

**Observability Insights** bietet Einblicke in die Anwendungsleistung, den Zustand der Infrastruktur und das Verhalten von Services in AEM Managed Services, ohne dass eine separate Überwachungsplattform erforderlich ist.

Wenn Sie für die Zuverlässigkeit von Services, die Reaktion auf Vorfälle oder Leistungsanalysen verantwortlich sind, hilft **Observability Insights** Ihnen, schnell von Symptomen zu Beweisen zu wechseln. Sie kombiniert Anwendungstelemetrie und Statussignale auf Host-Ebene, damit Kundenteams und Adobe Probleme aus einer gemeinsamen betrieblichen Sicht untersuchen können.

## Whitepaper zu Observability Insights

<iframe
  src="v2-assets/Observability_Insights_Overview.pdf"
  title="Whitepaper zu Observability Insights"
  width="100%"
  height="800"
  style="border: 0;"
></iframe>

[Whitepaper zu Observability Insights herunterladen](v2-assets/Observability_Insights_Overview.pdf)

## Warum verwenden Teams Observability Insights? {#why-teams-use-observability-insights}

Verwenden Sie Observability Insights zur Beantwortung betrieblicher Fragen, z. B.:

- Wirkt sich das Problem auf Autor, Veröffentlichung oder beides aus?
- Wird das Problem durch das Anwendungsverhalten, den Ressourcendruck auf dem Host oder eine Kombination aus beidem verursacht?
- Welche Transaktionen, Endpunkte oder Statusgruppen erklären die Spitze bei Fehlern oder Latenzen?
- Ist das Problem auf eine Umgebung beschränkt oder in der gesamten Topologie sichtbar?

Observability Insights wurde für die operative Analyse des aktuellen Verhaltens entwickelt. Auf diese Weise können Sie vor einer Eskalation oder Korrektur feststellen, was sich geändert hat, wo sich Änderungen ergeben und welche Signale am relevantesten sind.

## Was hilft Ihnen Observability Insights? {#what-observability-insights-helps-you-do}

Verwenden Sie Observability Insights für Folgendes:

- Erfahren Sie, wie sich die Autoren- und Veröffentlichungsebenen unter echtem Traffic verhalten.
- Korrelieren Sie Anwendungslatenz, Fehlerquoten und JVM-Status mit Signalen auf Host-Ebene.
- Prüfen, ob ein Problem auf eine Umgebung, eine Ebene oder einen Host beschränkt ist.
- Geben Sie Adobe Managed Services und Ihren internen Teams während der Untersuchung eine gemeinsame operative Ansicht.

Observability Insights ist in AEM Managed Services enthalten. Adobe stellt das Konto bereit und verwaltet die von Instrumenten unterstützten Umgebungen und stellt die resultierenden Dashboards Ihrem Team als schreibgeschützte operative Tools zur Verfügung.

Da Adobe die Plattformeinrichtung und -instrumentation verwaltet, können Sie sich auf die Untersuchung und Interpretation statt auf die Agentenbereitstellung, Kontoverwaltung oder Dashboard-Assembly konzentrieren.

## Auf einen Blick {#at-a-glance}

Als Teil von AEM Managed Services erhalten Sie:

- **Dediziertes Observability Insights-Konto** - von Adobe Managed Services bereitgestellt und überwacht, mit schreibgeschütztem Zugriff für Ihr Team.
- **Deep AEM-Transaktionsüberwachung** - Der APM-Agent von Observability Insights verfolgt aussagekräftige Transaktionen bis hin zu Methodenaufrufen (einschließlich Zeilennummern), externen Abhängigkeiten und Repository-Vorgängen.
- **Ansicht „Einheitliche Anwendungen und Hosts** - Kombinieren Sie Anwendungen und Metriken auf Host-Ebene, um die Leistung ganzheitlich zu optimieren.

## Für wen diese Dokumentation gedacht ist {#who-this-documentation-is-for}

Diese Dokumentation richtet sich in erster Linie an:

- AEM Managed Services-Administratoren, die Einblick in überwachte Umgebungen benötigen
- Betriebs- und Support-Teams, die mit Vorfällen, Trendanalysen und Service-Prüfungen umgehen
- Customer-Engineering-Teams arbeiten bei Untersuchungen mit Adobe Managed Services zusammen
- Stakeholder, die den Überwachungsbereich und die betrieblichen Zuständigkeiten verstehen müssen

## Was Adobe mit Observability Insights überwacht {#what-we-monitor}

Adobe überwacht die AEM **Autoren** und **Veröffentlichungs**-Ebenen mit dem Observability Insights APM Java-Plug-in. Alle gehosteten Server in Ihrer Topologie werden mit dem Observability Insights Infrastructure Agent überwacht. Die benutzerdefinierte Überwachung von APM und Infrastruktur ist sowohl in Nicht-Produktions- als auch in Produktionsumgebungen von Managed Services aktiviert.

![Abbildung mit Observability Insights-APM- und Infrastrukturüberwachung auf AEM-Autoren-, Veröffentlichungs- und gehosteten Servern](v2-assets/login-screen.png)

### Anwendungen in Ihrem Konto {#applications-in-your-account}

Ihr Observability Insights-Konto ist mit einem einzigen Adobe-Masterkonto verknüpft und kann Daten von mehreren Programmen empfangen, darunter:

- Eine APM-Anwendung für die **Author**-Ebene pro AEM Managed Services-Umgebung
- Eine APM-Anwendung für die **Veröffentlichungs**-Ebene pro AEM Managed Services-Umgebung

Jede Anwendung verfügt über einen eigenen Lizenzschlüssel. Alle Topologien in Ihrem Managed Services-Vertragsbericht in einem Observability Insights-Konto zusammengefasst. APM- und Infrastrukturmetriken und -ereignisse werden bis zu **30 Tage lang**.

## Zugriff auf Ihr Konto {#access}

Die Überwachungsdaten werden in einem Observability Insights-Konto zusammengefasst, das von Adobe bereitgestellt und verwaltet wird. Kundenbenutzer erhalten **schreibgeschützten Zugriff** auf APM- und Infrastrukturdaten, die von den Agenten erfasst werden. Adobe Managed Services behält Kontoeigentum und administrative Kontrolle.

### Voraussetzungen {#access-prerequisites}

Bevor Sie sich anmelden können, bestätigen Sie Folgendes:

- Ihr Unternehmen verfügt über ein aktives **AEM Managed Services**-Abonnement. Observability Insights ist ohne zusätzliche Kosten enthalten.
- Ihr Customer Success Engineer (CSE) hat Ihr Adobe IMS-Konto bereitgestellt und Ihnen Zugriff auf das Observability Insights-Konto für Ihr Unternehmen gewährt.

>[!NOTE]
>
> **Zugriff erhalten:** Zugriff auf Observability Insights erfordert die Bereitstellung von Adobe IMS. Wenden Sie sich an Ihren Customer Success Engineer (CSE), um den Benutzerzugriff für Ihr Unternehmen bereitzustellen und zu verwalten.

Nachdem der CSE das Konto bereitgestellt hat, melden Sie sich unter &quot;[.adobecqms.net“ ](https://insights.adobecqms.net). Diese URL ist für alle AEM Managed Services-Kunden identisch. Die Umgebungen und Dashboards Ihres Unternehmens werden auf Ihr bereitgestelltes Konto angewendet.
