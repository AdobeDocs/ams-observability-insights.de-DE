---
title: Infrastrukturüberwachung mit Observability Insights
description: Erfahren Sie, wann die Infrastruktur-Dashboards verwendet werden, welche Signale zuerst überprüft werden müssen und wo die vollständige Referenz zur Host-Metrik zu finden ist.
feature: Operations
role: Admin
source-git-commit: 825334e003ae814af1b0845c6de1a533b4b5f47b
workflow-type: tm+mt
source-wordcount: '580'
ht-degree: 0%

---


# Hosts {#hosts}

Verwenden Sie Hosts in Observability Insights, um Zustand, Leistung und Ressourcenauslastung der Infrastruktur zu überwachen, die Ihre Programme und Services unterstützt. Verwenden Sie die Infrastruktur-Dashboards, um Probleme im Zusammenhang mit der Host-Kapazität, dem Speicherdruck, dem Netzwerkdurchsatz oder dem Ressourcenkonflikt des Betriebssystems zu identifizieren.

## Welche Infrastrukturüberwachung hilft Ihnen zu beantworten? {#what-infrastructure-monitoring-helps-you-answer}

Die Überwachung der Infrastruktur ist am nützlichsten, wenn Sie Fragen beantworten müssen, z. B.:

- Ist die verlangsamte Anwendungsperiode von CPU, Arbeitsspeicher oder I/O-Druck begleitet?
- Verhält sich ein Host anders als andere in derselben Umgebung?
- Ändern sich Netzwerk- oder Festplattenmuster im selben Intervall wie bei einem kundenorientierten Vorfall?
- Deuten die Trends bei der Speichernutzung auf ein bevorstehendes Kapazitätsproblem hin?

## Zugreifen auf Infrastruktur-Hosts {#infrastructure-host-overview}

Die Infrastrukturüberwachung bietet Einblicke auf Host-Ebene in den Zustand und die Leistung der Infrastruktur, die Ihre verwalteten AEM-Umgebungen unterstützt. Aus dem **Observability Catalog** können Sie Infrastruktur-Hosts durchsuchen und einen einzelnen Host aufrufen, um CPU, Speicher, Netzwerk, Speicher und andere Signale auf Systemebene zu untersuchen.

## Zugreifen auf Infrastruktur-Hosts

Wählen Sie **Katalog** die Registerkarte **Hosts** aus, um die mit dem ausgewählten Konto verknüpfte Infrastruktur anzuzeigen.

![Infrastruktur-Hosts](v2-assets/1_host.png)

Die **Infrastruktur-Hosts** bietet eine Bestandsaufnahme der überwachten Hosts und umfasst:

- **Hostname** - Name des überwachten Infrastruktur-Hosts.
- **Konto** - Konto, das mit dem Host verknüpft ist.
- **Umgebung** - Umgebungsklassifizierung, wie `DEV` oder `STAGE`.
- **Health** - Aktueller Gesundheitsstatus des Hosts.
- **Zuletzt gesehen** — Wie kürzlich die Telemetrie vom Host empfangen wurde.

![HostsOverview](v2-assets/2_hostOverview.png)

## Empfohlener Untersuchungsfluss {#suggested-investigation-flow}

Überprüfen Sie für die meisten Vorfälle das Host-Dashboard in dieser Reihenfolge:

1. Überprüfen Sie die CPU-Auslastung, den Lastdurchschnitt und die Speichernutzung auf eine offensichtliche Sättigung.
2. Überprüfen Sie die CPU I/O-Wartezeit und den Datenträgerdurchsatz, wenn die Antwortzeiten ohne eine entsprechende CPU-Spitze ansteigen.
3. Vergleichen Sie den Netzwerkdurchsatz mit dem Anwendungs-Traffic, um lastbedingte Verschiebungen zu identifizieren.
4. Überprüfen Sie die Speicherauslastung und die Festplattenauslastung auf Dateisystemebene auf ein persistentes Kapazitätsrisiko.
5. Vergleichen Sie mehrere Hosts, um festzustellen, ob das Problem lokalisiert oder systemisch ist.

## Was zuerst zu überprüfen ist {#what-to-review-first}

- **CPU und Speicher** wenn eine Anwendung über einen längeren Zeitraum langsam oder instabil erscheint.
- **Festplatten-E/A und CPU-E/A warten** wenn Anforderungen unerwartet anhalten oder in die Warteschlange gestellt werden.
- **Netzwerk-E/A**, wenn sich die Traffic-Merkmale ändern oder nachgelagerte Abhängigkeiten vermutet werden.
- **Speicherauslastung** wenn es bei Vorfällen zu Bereitstellungsfehlern, Indizierungsdruck oder langfristigen Kapazitätsproblemen kommt.

Verwenden Sie das Feld **Name enthält** und den **Tier**-Filter, um die Liste der Hosts einzugrenzen. Wählen Sie einen Host-Namen aus, um die Details zur Infrastrukturüberwachung zu öffnen.

## Host-Überwachung

Nach Auswahl eines Hosts stellt die Ansicht **Infrastruktur** dedizierte Überwachungsseiten für diesen Host bereit.

Die Host-Navigation umfasst:

- **Übersicht** - Konsolidierte Übersicht über die Signale zur Zustandsänderung und Auslastung der wichtigsten Infrastruktur.
- **Metriken** - Detaillierte Metriken zur Host-Leistung, einschließlich CPU, Arbeitsspeicher, Auslastung, Datenträger-E/A und Netzwerkdurchsatz.
- **Netzwerk** - Netzwerk-Traffic, Schnittstellenaktivität und Sende-/Empfangsfehler.
- **Process** - Überwachung auf Host-Prozessebene.
- **Speicher** - Festplattenauslastung, Festplatten-E/A und Dateisystemnutzung.
- **System** - Metriken zu Kernsystemressourcen wie CPU, Arbeitsspeicher und Lastdurchschnitt.

## Fragen, die während der Untersuchung zu beantworten sind {#questions-to-answer}

- Ist das Problem auf einen einzelnen Host beschränkt oder in der gesamten Umgebung sichtbar?
- Werden CPU-, Speicher- oder Festplattensignale im selben Zeitfenster wie der Vorfall erhöht?
- Geht der Trend zum Speicherwachstum auf einen Schwellenwert zu, der sich auf den Betrieb auswirken könnte?
- Erklären Infrastruktursymptome das Anwendungsverhalten oder treten sie nur als nachgelagerter Effekt auf?

## Bei Eskalation zu erfassende Beweise {#evidence-to-capture}

- Betroffene Umgebung und Hosts
- Zeitfenster des Problems
- Screenshots zu CPU, Arbeitsspeicher, Festplatte und Netzwerk
- Ob die Anomalie isoliert oder systemisch ist
- Verwandte Anwendungssymptome von APM
