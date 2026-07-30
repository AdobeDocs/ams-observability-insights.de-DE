---
title: Infrastrukturüberwachung mit Synoptryx
description: Verwenden Sie die Synoptryx-Infrastrukturüberwachung, um System-, Netzwerk-, Prozess- und Speichermetriken auf Host-Ebene für Ihren gesamten AEM Managed Services-Platzbedarf zu überprüfen.
feature: Operations
role: Admin
source-git-commit: 261f6fac681c000ea6cbbdf403b144f00ab98326
workflow-type: tm+mt
source-wordcount: '1110'
ht-degree: 7%

---


# Dashboard zur Überwachung der Host-Infrastruktur

In diesem Abschnitt werden die einzelnen Diagramme zur Überwachung der Infrastruktur auf Host-Ebene beschrieben, die im Dashboard zur Überwachung der Infrastruktur angezeigt werden. In jedem Abschnitt werden der Zweck der Metrik, die erfassten Daten, die Maßeinheiten und die in der Visualisierung dargestellten Informationen erläutert.

## Dashboard-Übersicht

Das Dashboard zur Überwachung der Host-Infrastruktur bietet Echtzeiteinblicke in die Nutzung und Leistung des zugrunde liegenden Hosts. Diese Metriken unterstützen Benutzende bei der Überwachung von Rechen-, Speicher-, Speicher- und Netzwerkressourcen und identifizieren gleichzeitig potenzielle Ressourcenengpässe.

Das Dashboard umfasst die folgenden Überwachungs-Panels:

- Host-Nutzung von CPU
- Host-Datenträger-E/A
- Host-Netzwerk-E/A
- CPU I/O-Wartezeit
- Speicherverwendung
- Speichernutzung
- Durchschnittliche Host-CPU-Last
- Host-Speicherauslastung

## &#x200B;1. Host-Nutzung von CPU

![Host-Nutzung von CPU](assets/host-monitoring/host_cpu_utilization.png)

### Beschreibung

Das Bedienfeld **Host-CPU-Nutzung** zeigt den Prozentsatz der CPU-Ressourcen an, die derzeit vom Betriebssystem und allen ausgeführten Prozessen im Zeitverlauf verbraucht werden.

Diese Metrik stellt die gesamte CPU-Nutzung auf dem Host dar und bietet eine Zeitreihenvisualisierung der Prozessoraktivität.

Das Diagramm ermöglicht es Benutzenden, zu überwachen, wie sich der CPU-Verbrauch während des ausgewählten Beobachtungsfensters ändert.

### Metrik

| Metrik | Beschreibung |
| --------- | ---------------------------------------- |
| `cpu_pct` | Prozentsatz der gesamten aktuell verwendeten CPU |

### Einheiten

- Prozentsatz (%)

### Angezeigte Statistiken

Das Bedienfeld fasst die CPU-Auslastung anhand von drei Werten zusammen:

| Statistik | Beschreibung |
| --------- | --------------------------------------------------------------- |
| Mean | Durchschnittliche CPU-Auslastung im ausgewählten Zeitraum |
| Letzte | Zuletzt erfasster Wert der CPU-Nutzung |
| Max | Höchste im ausgewählten Zeitraum beobachtete CPU-Auslastung |

### Diagrammkomponenten

- Zeitreihenzeile, die die CPU-Auslastung darstellt.
- Prozentbasierte Y-Achse von **0 % bis 100 %**.
- Zusammenfassungsstatistiken werden unterhalb des Diagramms angezeigt.
- Historischer Trend im ausgewählten Überwachungsintervall.

## &#x200B;2. Host-Datenträger-E/A

![Host-Datenträger-E/A](assets/host-monitoring/host_disk_io.png)

### Beschreibung

Das **Host Disk I/O**-Bedienfeld zeigt den Speicherdurchsatz für vom Host durchgeführte Festplatten-Lese- und -Schreibvorgänge an.

Das Diagramm zeigt zwei unabhängige Zeitreihen, die die zwischen dem Betriebssystem und den Speichergeräten übertragenen Daten darstellen.

Diese Visualisierung hilft, die Speicheraktivität im Laufe der Zeit zu überwachen und liefert insight in das Datenvolumen, das von Festplatten gelesen und auf Festplatten geschrieben wird.

### Metrik

| Metrik | Beschreibung |
| ------------ | --------------------------------- |
| `disk_read` | Aus dem Speicher gelesene Datenmenge |
| `disk_write` | In den Speicher geschriebene Datenmenge |

Intern werden diese Metriken mithilfe von Werten für den geglätteten Durchsatz angezeigt.

### Einheiten

- Bytes pro Sekunde (B/s)
- Kilobyte pro Sekunde (KB/s)
- Megabyte pro Sekunde (MB/s)
- Gigabyte pro Sekunde (GB/s)

Die angezeigte Einheit skaliert automatisch anhand des Durchsatzes.

### Diagrammkomponenten

- Grüne Linie für den Datenträgerlesedurchsatz.
- Orangefarbene Linie für den Schreibdurchsatz der Festplatte.
- Zeitreihenvisualisierung.
- Separate Legende für jede Metrik.
- Aktuelle Metrikwerte, die neben jeder Serie angezeigt werden.

## &#x200B;3. Host-Netzwerk-E/A

![Host-Netzwerk-E/A](assets/host-monitoring/host_network_io.png)

### Beschreibung

Das **Host Network I/O**-Bedienfeld zeigt das Volumen des Netzwerk-Traffics an, der vom Host im Zeitverlauf übertragen und empfangen wird.

Das Diagramm misst die Rate, mit der Daten durch die Netzwerkschnittstellen fließen, und bietet Einblick in den Bandbreitenverbrauch des Netzwerks.
Diese Metrik stellt den aggregierten Netzwerkdurchsatz dar.

### Metrik

| Metrik | Beschreibung |
| --------------- | --------------------------------------------------------------------- |
| `bytes_per_sec` | Aggregierter Netzwerkdurchsatz, gemessen als pro Sekunde übertragene Bytes |

### Einheiten

Das Diagramm wird automatisch skaliert zwischen:

- Bytes/Sek
- KB/s
- MB/s
- GB/s

abhängig vom beobachteten Traffic-Volumen.

### Angezeigte Statistiken

| Statistik | Beschreibung |
| --------- | ---------------------------------- |
| Mean | Durchschnittlicher Netzwerkdurchsatz |
| Letzte | Letzte Durchsatzmessung |
| Max | Höchster festgestellter Durchsatz |

### Diagrammkomponenten

- Einzelne Durchsatzleitung.
- Zeitreihenvisualisierung.
- Dynamische Bandbreitenskalierung.
- Unter dem Diagramm angezeigte Zusammenfassungsstatistiken.

## &#x200B;4. CPU I/O-Wartezeit

![CPU I/O-Wartezeit](assets/host-monitoring/cpu_io_wait.png)

### Beschreibung

Das Bedienfeld **CPU-E/A** Wartezeit“ zeigt den Prozentsatz der CPU-Zeit an, die damit verbracht wurde, auf den Abschluss von Eingabe-/Ausgabevorgängen zu warten.

Diese Metrik stellt die Inaktivitätsdauer des Prozessors dar, die auftritt, weil aktive Prozesse beim Warten auf Speichergeräte oder andere I/O-Vorgänge blockiert werden.

Das Diagramm zeigt, wie sich die I/O-Wartezeit im Laufe der Zeit ändert.

### Metrik

| Metrik | Beschreibung |
| --------- | ------------------------------------------------------ |
| `cpu_pct` | Prozentsatz der CPU-Zeit, die mit dem Warten auf E/A-Vorgänge verbracht wurde |

### Einheiten

- Prozentsatz (%)

### Angezeigte Statistiken

| Statistik | Beschreibung |
| --------- | ------------------------------- |
| Mean | Durchschnittlicher CPU I/O-Warteprozentsatz |
| Letzte | Zuletzt erfasster Wert |
| Max | Höchster erfasster Wert |

### Diagrammkomponenten

- Zeitreihen-Zeile.
- Prozentbasierte Y-Achse.
- Zusammenfassungsstatistiken.
- Visualisierung historischer Trends.

## &#x200B;5. Speicherverwendung

![Speicherauslastung](assets/host-monitoring/storage_disk_usage.png)

### Beschreibung

Das **Speicherauslastung** zeigt den Gesamtanteil der aktuell auf dem überwachten Host ausgelasteten Speicherkapazität an.

Das Diagramm bietet eine historische Ansicht der Kapazitätsauslastung des Dateisystems im ausgewählten Zeitintervall.

### Metrik

| Metrik | Beschreibung |
| --------------- | -------------------------------------------------- |
| Speicherauslastung % | Prozentsatz des aktuell belegten zugewiesenen Speichers |

### Einheiten

- Prozentsatz (%)

### Diagrammkomponenten

- Diagramm zur Zeitreihenauslastung.
- Prozentuale Skalierung.
- Historischer Trend der Speichernutzung.

## &#x200B;6. Speichernutzung

![Festplattenauslastung](assets/host-monitoring/storage_disk_usage.png)

### Beschreibung

Das **Festplattenauslastung** zeigt die Speicherauslastung für jedes bereitgestellte Dateisystem oder Speichergerät an.

Jede Zeile entspricht einem bestimmten Blockgerät oder einer installierten Partition und zeigt den Prozentsatz des aktuell verwendeten Platzes an.

Diese Tabelle bietet eine Aufschlüsselung der Speicherauslastung auf Dateisystemebene.

### Angezeigte Informationen

Jeder Eintrag enthält:

| Feld | Beschreibung |
| --------------- | -------------------------------------------- |
| Gerät | Bereitgestelltes Speichergerät oder Dateisystem |
| Verwendet % | Prozentsatz der ausgelasteten Speicherkapazität |
| Nutzbarren | Visuelle Darstellung des Speicherverbrauchs |

### Einheiten

- Prozentsatz (%)

### Diagrammkomponenten

- Dateisystem-/Geräteliste
- Auslastungsprozentsatz.
- Farbkodierte Kapazitätsanzeige.
- Sortierte Nutzungswerte.

## &#x200B;7. Durchschnittliche Host-CPU-Last

![Host CPU Load Average](assets/host-monitoring/host_cpu_load_average.png)

### Beschreibung

Das **Host-CPU-Lastdurchschnitt** zeigt die Durchschnittswerte der Linux-Systemlast über drei rollierende Zeitfenster an.

Im Gegensatz zur CPU-Auslastung gibt der Lastdurchschnitt die durchschnittliche Anzahl der Prozesse an, die entweder aktiv ausgeführt werden oder auf den Abschluss der CPU-Planung oder der E/A warten.

Das Diagramm zeigt gleichzeitig drei rollierende Durchschnittswerte an, die kurzfristige und langfristige Workload-Trends bereitstellen.

### Metrik

| Metrik | Beschreibung |
| ---------- | -------------------------------------------- |
| `load_1m` | Durchschnittliche Systemlast in den letzten 1 Minute |
| `load_5m` | Durchschnittliche Systemlast in den letzten 5 Minuten |
| `load_15m` | Durchschnittliche Systemlast in den letzten 15 Minuten |

### Einheiten

- Lastdurchschnitt (dimensionsloser Wert)

### Angezeigte Statistiken

Für jede Metrik des Lastdurchschnitts:

| Statistik | Beschreibung |
| --------- | --------------------------------------- |
| Mean | Durchschnittliche Auslastung während des ausgewählten Zeitraums |
| Letzte | Letzter erfasster Lastwert |
| Max | Höchster beobachteter Lastwert |

### Diagrammkomponenten

- Drei unabhängige Trendlinien.
- Zeitreihenvisualisierung.
- Einzelne Legenden für jeden rollierenden Durchschnitt.
- Zusammenfassungsstatistiken für jede Metrik.

## &#x200B;8. Host-Speicherauslastung

![Host-Speichernutzung](assets/host-monitoring/host_memory_usage.png)

### Beschreibung

Das Bedienfeld **Host-Speichernutzung** zeigt den Prozentsatz des physischen Systemspeichers an, der derzeit vom Betriebssystem zugewiesen ist.

Diese Metrik stellt die allgemeine RAM-Auslastung für alle laufenden Prozesse, den Kernelspeicher, Puffer und Caches dar.

Das Diagramm bietet eine kontinuierliche Ansicht des Speicherverbrauchs während des ausgewählten Überwachungszeitraums.

### Metrik

| Metrik | Beschreibung |
| ------------ | ---------------------------------------------- |
| `memory_pct` | Prozentsatz des aktuell verwendeten physischen Speichers |

### Einheiten

- Prozentsatz (%)

### Angezeigte Statistiken

| Statistik | Beschreibung |
| --------- | ---------------------------------- |
| Mean | Durchschnittliche Speichernutzung |
| Letzte | Zuletzt aufgezeichnete Nutzung |
| Max | Höchste festgestellte Auslastung |

### Diagrammkomponenten

- Diagramm zur Zeitreihen-Speichernutzung.
- Prozentbasierte Y-Achse.
- Historischer Nutzungstrend.
- Zusammenfassungsstatistiken.

## Zusammenfassung der Dashboard-Metriken

| Dashboard-Bedienfeld | Primäre Metrik | Einheit |
| --------------------- | -------------------------------- | ------------ |
| Host-Nutzung von CPU | `cpu_pct` | % |
| Host-Datenträger-E/A | `disk_read`, `disk_write` | Bytes/Sek |
| Host-Netzwerk-E/A | `bytes_per_sec` | Bytes/Sek |
| CPU I/O-Wartezeit | `cpu_pct` | % |
| Speicherverwendung | Speicherauslastung % | % |
| Speichernutzung | Nutzung des Dateisystems | % |
| Durchschnittliche Host-CPU-Last | `load_1m`, `load_5m`, `load_15m` | Lastdurchschnitt |
| Host-Speicherauslastung | `memory_pct` | % |
