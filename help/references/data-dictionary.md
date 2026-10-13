---
source-git-commit: 4949fcad799260aa73354a8c0b9d79e6394cb257
workflow-type: tm+mt
source-wordcount: '585'
ht-degree: 21%
---
## Datenwörterbuch {#data-dictionary}

Dies sind alle Entitäten und Attribute, die Sie abfragen können. Der Live-Katalog mit Beschreibungen befindet sich auch im Bedienfeld **Handbuch zur Abfrage** unter „Erkunden“ und im Dialogfeld Widget hinzufügen .

### Entitäten verfolgen

| Entität | Enthält | Fester Umfang | Datenaggregationen |
| ------------- | --------------------------------------------------------- | ---------------------------- | ------------------------------------------ |
| `Transaction` | Eingehende Server-Anfragen | `kind = Server` | 1 m / 5 m / 1 h ([Performance](#performance)) |
| `AppRequest` | Eingehende Server **und** Nachrichten-Privatkunden-Anfragen | `kind IN (Server, Consumer)` | — |
| `Span` | Alle Bereiche (Server, Client, Produzent, Verbraucher, intern) | — | — |

<details open>
<summary><strong>Trace-Attribute</strong> (identisch in allen drei Entitäten)</summary>

| Attribut | Typ | Beschreibung |
| --------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------- |
| `timestamp` | Zeitstempel | Span Start Time |
| `traceId`, `spanId`, `parentSpanId` | Zeichenfolge | Kennungen (Stammbereiche haben `parentSpanId = ''`) |
| `name` | Zeichenfolge | Name des Vorgangs oder der Transaktion |
| `kind` | Zeichenfolge | `Server`, `Client`, `Producer`, `Consumer`, `Internal` |
| `service` | Zeichenfolge | Name des sendenden Dienstes |
| `serviceKey` | Zeichenfolge | Effektive Service-Identität (fällt auf `appName` zurück, wenn `service` leer oder ein grobes übergeordnetes Element ist) |
| `appName` | Zeichenfolge | Observability Insights-Anwendungsname |
| `appInstance` | Zeichenfolge | Identität pro Instanz: Host-Präfix für Autor/Veröffentlichung, App-Name, dann Service |
| `hostTierPrefix` | Zeichenfolge | Hostname an seinem `author` / `publish` / `dispatcher` Rollensuffix abgeschnitten |
| `duration` | Anzahl (ms) | Dauer der Spanne in Millisekunden |
| `status` | Zeichenfolge | `Ok`, `Error`, `Unset` |
| `statusMessage` | Zeichenfolge | Statusbeschreibung |
| `host` | Zeichenfolge | Host-Name |
| `environment` | Zeichenfolge | Bereitstellungsumgebung |
| `region` | Zeichenfolge | Cloud-Region |
| `cloud.provider` | Zeichenfolge | AWS / AZURE / GCP |
| `http.method` | Zeichenfolge | `GET`, `POST`, … |
| `http.statusCode` | number | HTTP-Antwortstatus |
| `http.route` | Zeichenfolge | Normalisierte Routenvorlage |
| `url` | Zeichenfolge | Vollständige URL |
| `urlPath` | Zeichenfolge | Pfad - Teil der URL (`/content/site/en.html`) |
| `externalDomain` | Zeichenfolge | Domain einer ausgehenden URL (wird auf Client-Spannen verwendet) |
| `peer.service` | Zeichenfolge | Nachgelagerter Dienst namens |
| `db.system`, `db.operation`, `db.namespace` | Zeichenfolge | Datenbanktechnologie, -betrieb und -schema |
| `messaging.system`, `messaging.destination` | Zeichenfolge | Kafka, RabbitMQ, … und Thema oder Warteschlange |
| `rpc.system`, `rpc.method` | Zeichenfolge | gRPC und Ähnliches |
| `error.type` | Zeichenfolge | Ausnahme- oder Fehlertyp |
| `sling.script`, `sling.servlet`, `sling.requestUri` | Zeichenfolge | AEM Sling-Details |
| `jsp.path` | Zeichenfolge | AEM JSP ausgeführt |
| `jcr.operation`, `jcr.path` | Zeichenfolge | Vorgang und Pfad des AEM JCR-Repositorys |
| `graphql.operation` | Zeichenfolge | GraphQL-Vorgangsname |

</details>

### Generische Metrik-Entitäten

| Entität | Enthält |
| ------------- | -------------------------------------------------- |
| `MetricGauge` | Jedes Point-in-Time-Beispiel einer OpenTelemetrie-Anzeige |
| `MetricSum` | Jedes kumulative oder Delta-OpenTelemetry-Summenbeispiel |

Verwenden Sie sie für **Erkennung** oder für Metriken, die keine kuratierte Entität haben: `WHERE metric = 'jvm.gc.duration'`.

<details open>
<summary><strong>Metrikattribute</strong> (für alle Metrikentitäten freigegeben)</summary>

| Attribut | Typ | Beschreibung |
| ----------------------------------------------------------------------- | --------- | ------------------------------------------------------- |
| `timestamp` | Zeitstempel | Abtastzeit |
| `metric` | Zeichenfolge | OpenTelemetry-Metrikname |
| `value` | number | Beispielwert (konvertiert in kuratierte Entitäten, siehe unten) |
| `unit` | Zeichenfolge | Metrische Einheit |
| `service`, `appName`, `host`, `environment`, `region`, `cloud.provider` | Zeichenfolge | Ressourcenkennung |
| `state` | Zeichenfolge | CPU-Modus oder Speicherstatus |
| `cpu`, `device`, `direction`, `mountpoint` | Zeichenfolge | Pro CPU, pro Gerät, I/O-Richtung, Dateisystem |
| `pool` | Zeichenfolge | JVM-Speicherpool oder Verbindungspool |
| `topic` | Zeichenfolge | Kafka-Thema |
| `process.pid`, `process.name`, `process.owner` | Zeichenfolge | Prozessidentität |

</details>

### Kuratierte Metrik-Entitäten

Jede kuratierte Entität umschließt eine OpenTelemetry-Metrik mit dem richtigen Filter und der richtigen Einheitenkonvertierung, sodass `value` bereits bedeutungsvoll ist. **Zähler** Entitäten (✔) enthalten kumulative Gesamtwerte. Abfragen mit `rate(value)` und `TIMESERIES`.

| Entität | `value` Mittel | Einheit | Zähler | Anmerkungen |
| ------------------------------------------------------------------------------ | ----------------------------------- | --------- | ------- | -------------------------------------------- |
| `CpuUtilization` | CPU beschäftigt | % |         | 1-minütiger Rollup für `average(value)` nach Host |
| `CpuUserPercent` / `CpuSystemPercent` / `CpuIowaitPercent` / `CpuStealPercent` | CPU-Zeit nach Modus | % |         |                                              |
| `MemoryUtilization` | Verwendeter Speicher | % |         | 1-minütiger Rollup für `average(value)` |
| `LoadAverage1m` / `LoadAverage5m` / `LoadAverage15m` | Lastdurchschnitt | — |         |                                              |
| `LoadAverageWithFallback` | 5m Last, Rückfall auf 1m | — |         |                                              |
| `DiskUtilization` | Verwendete Festplatte pro Mount | % |         | Facette nach `mountpoint` |
| `DiskFreeBytes` | Root-Dateisystem frei | Byte |         |                                              |
| `DiskReadRate` / `DiskWriteRate` | Datenträgerdurchsatz | Byte/s | ✔ |                                              |
| `DiskReadOpsRate` / `DiskWriteOpsRate` | Festplatten-IOPS | Ops/s | ✔ |                                              |
| `NetworkReceiveRate` / `NetworkTransmitRate` | eth0-Durchsatz | Byte/s | ✔ |                                              |
| `NetworkReceiveRateAllDevices` / `NetworkTransmitRateAllDevices` | Alle Geräte | Byte/s | ✔ |                                              |
| `NetworkIoRate` | Alle Richtungen und Geräte kombiniert | Byte/s | ✔ |                                              |
| `NetworkReceiveErrorRate` / `NetworkTransmitErrorRate` | Netzwerkfehler | Fehler/s | ✔ |                                              |
| `NetworkReceivePacketRate` / `NetworkTransmitPacketRate` | Pakete | Pakete/s | ✔ |                                              |
| `ProcessCpuUtilization` | CPU pro Prozess | % |         | Facette nach `process.name` |
| `ProcessMemoryUsage` | Speicher pro Prozess | MB |         |                                              |
| `JvmHeapUsed` / `JvmHeapLimit` | JVM-Heap | MB |         | Gemeldet pro Speicherpool |
| `JvmThreadCount` | JVM-Threads | — |         |                                              |
| `JvmCpuRecentUtilization` | JVM-Prozess-CPU | % |         | 1-minütiger Rollup |
| `DispatcherRequestRate` | AEM Dispatcher-HTTP-Anfragen | Erf/Sek | ✔ |                                              |

&#x200B;> [!NOTE]
> JVM-Speichermetriken werden **pro Speicherpool** gemeldet, sodass die `JvmHeapUsed`-/`JvmHeapLimit`-Durchschnittswerte der Pools `average(value)` werden. Verwenden Sie `FACET host, pool` für eine exakte Ansicht pro Pool.

### `SpanBreakdown`: voraggregierte Transaktionszusammensetzung

`SpanBreakdown` enthält die pro Minute erforderliche untergeordnete Komposition jeder Transaktion. Es wird bei der Aufnahme berechnet, daher ist es billig, Abfragen durchzuführen.

| Attribut | Typ | Beschreibung |
| --------------- | --------- | ---------------------------------------------------------------------- |
| `timestamp` | Zeitstempel | Ein-Minuten-Eimer |
| `service` | Zeichenfolge | Dienst |
| `rootOperation` | Zeichenfolge | Name der übergeordneten Transaktion |
| `segment` | Zeichenfolge | Name des untergeordneten Bereichs |
| `callCount` | number | Aufrufe dieses Segments im Bucket |
| `avgDurationMs` | number | Durchschnittliche Segmentdauer im Bucket (stellen Sie die Einheit manuell auf `ms` ein) |
