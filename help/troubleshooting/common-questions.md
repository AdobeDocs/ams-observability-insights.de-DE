---
title: Häufig gestellte Fragen
description: Häufige Fragen und Ausgangspunkte für Untersuchungen zu Observability Insights in AEM Managed Services.
feature: Operations
role: Admin
product_v2:
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
    internal-label: Experience Manager
feature_v2:
  - id: 8a70d214-ab7b-58c1-b001-2ed2e5d6303d
    internal-label: Operations
role_v2:
  - id: c66ffd68-0f65-42bb-aa23-b4020f12e0bd
    internal-label: Admin
source-git-commit: ba4f816187e77f3f7b708942b3f14535417c2fdc
workflow-type: tm+mt
source-wordcount: '253'
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

## Welche Daten werden von Observability Insights tatsächlich erfasst? {#what-data-is-collected}

Siehe [Abdeckung, Umgebungen und Datenaufbewahrung](../get-started/coverage-and-data.md) für den Überwachungsumfang, die Anwendungsdarstellung, die Aufbewahrungsfristen und die betrieblichen Auswirkungen.

## Warum kann ich mich mit anderen Browsern, aber nicht mit Google Chrome anmelden?

So lassen Sie Drittanbieter-Cookies für Observability Insights zu und versuchen Sie erneut, sich anzumelden:

1. Um auf Cookie-Einstellungen zuzugreifen, öffnen Sie `chrome://settings/cookies` in Google Chrome.
2. Um Cookies für Observability Insights zuzulassen, wählen **unter „Websites, die Cookies von Drittanbietern verwenden dürfen** die Option &quot;**&quot; aus** geben Sie `https://insights.adobecqms.net` ein.
3. Um zu überprüfen, ob das Problem behoben wurde, melden Sie sich bei Observability Insights in einem Inkognito-Fenster an.