---
source-git-commit: e5523081fcd68500602e5d1bf853694d1f6c3980
workflow-type: tm+mt
source-wordcount: '1104'
ht-degree: 7%

---
# Öffentliche Observability Insights-API

Mit der öffentlichen Observability Insights-API können Sie Ihre eigenen Observability-Daten - Anfrageübersichten, Service-Kataloge, Traces und Metriken - direkt in Ihre eigenen Tools, Skripte und Dashboards ziehen.

- **API-Basis-URL (API_BASE_URL):** `https://insights.adobecqms.net/`
- **Format:** JSON über HTTPS
- **Authentifizierung:** API-Schlüssel (Bearer-Token)

> Ersetzen Sie `{{API_BASE_URL}}` in diesem Dokument durch den API-Host Ihrer Observability Insights-Instanz, z. B. `https://insights.adobecqms.net/`.

&#x200B;---

## &#x200B;1. Abrufen eines API-Schlüssels

API-Schlüssel sind persönliche Anmeldeinformationen, die an Ihr Konto gebunden sind und sich auf eine einzelne Organisation beziehen. Mit einem Schlüssel können nur Daten für Mandanten gelesen werden, die zu der Organisation gehören, für die er erstellt wurde. Die Daten einer anderen Organisation können nicht angezeigt werden.

### Generieren eines Schlüssels

1. Melden Sie sich beim Dashboard [Observability Insights](https://insights.adobecqms.net/) an.
2. Öffnen Sie das Profilmenü (oben rechts) →API **Schlüssel**.
   ![Menü API-Schlüssel](v2-assets/api-key.png)
3. Klicken Sie auf **Registerkarte** API-Schlüssel **auf Schlüssel**.
   ![API-Schlüssel generieren](v2-assets/api-key-gen.png)
4. Geben Sie ihm einen beschreibenden Namen (z. B. `CI pipeline`, `Grafana datasource`), wählen Sie die Organisation aus, für die er gelten soll, und legen Sie optional ein Ablaufdatum fest.
5. Klicken Sie **Generate key**. Ihr Schlüssel wird **einmal** im folgenden Format angezeigt:

   ```
   synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
   ```

   **Kopieren Sie es sofort und speichern Sie es an einem sicheren Ort** (einen Secrets Manager, einen CI Secret Store usw.) — Das Dashboard kann Ihnen nicht erneut angezeigt werden. Wenn Sie sie verlieren, widerrufen Sie sie und generieren Sie eine neue.

### Verwalten vorhandener Schlüssel

Im Abschnitt API-Schlüssel werden alle von Ihnen erstellten Schlüssel aufgelistet, einschließlich Organisation, Erstellungsdatum, Ablauf und Zeitstempel der letzten Verwendung. Klicken Sie auf das Papierkorbsymbol neben einem Schlüssel, um ihn **widerrufen** - die Sperrung ist sofort und kann nicht rückgängig gemacht werden.

### Schlüsselsicherheit

- Behandeln eines API-Schlüssels genau wie ein Kennwort. Jeder mit dem -Schlüssel kann alle Beobachtbarkeitsdaten für jeden Mandanten in der Organisation lesen, auf die sie sich bezieht, bis sie widerrufen werden oder ablaufen.
- Übertragen Sie niemals einen Schlüssel in die Quell-Code-Verwaltung oder geben Sie ihn im Klartext (Chat, E-Mail, Tickets) frei.
- Drehen Sie Schlüssel regelmäßig und widerrufen Sie alle Schlüssel, die nicht mehr verwendet werden.
- Wenn ein Schlüssel kompromittiert ist, widerrufen Sie ihn sofort aus den **Org-Einstellungen → API-**) und generieren Sie einen Ersatz.

&#x200B;---

## &#x200B;2. Authentifizieren von Anforderungen

Jede Anfrage an die öffentliche API muss Ihren Schlüssel in der `Authorization`-Kopfzeile enthalten:

```
Authorization: Bearer synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
```

Anfragen ohne gültigen Schlüssel oder mit abgelaufenem/widerrufenem Schlüssel erhalten `401 Unauthorized`. Sitzungsanmeldungen (Browser-Cookies/Token) werden **nicht** auf dieser API akzeptiert.

&#x200B;---

## &#x200B;3. Grundlegende Konzepte

### Mandanten

Jeder Endpunkt benötigt einen `tenant_id` Abfrageparameter, der angibt, welche Mandantendaten gelesen werden sollen. Mit einem Schlüssel können nur Mandanten abgefragt werden, die zu der Organisation gehören, für die er erstellt wurde. Wenn ein Mandant außerhalb dieser Organisation angefordert wird, wird `403 Forbidden` zurückgegeben. Es gibt keinen Modus „Alle Mandanten“ auf dieser API - übergeben Sie immer eine bestimmte `tenant_id`.

Sie sind sich nicht sicher, welche `tenant_id` Werte Ihr Schlüssel verwenden kann? [`GET /public/v1/tenants`](#get-publicv1tenants) aufrufen - Es werden genau die Mandanten aufgelistet, die Ihr Schlüssel abfragen darf.

### Zeitbereiche

Endpunkte, die `from`-/`to`-Parameter akzeptieren, benötigen Unix-Zeitstempel (Sekunden), Millisekunden-Zeitstempel oder ISO 8601-Datums-Zeit-Zeichenfolgen, z. B.:

```
from=1735689600
from=2025-01-01T00:00:00Z
```

Wenn sie weggelassen wird, wird für die meisten Endpunkte standardmäßig ein rollierendes Fenster verwendet (siehe die einzelnen Endpunkte unten).

### Ratenbeschränkungen

Anfragen sind ratenbegrenzt pro API-Schlüssel. Wenn Sie das Limit überschreiten, erhalten Sie:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{ "error": "Too Many Requests", "message": "Rate limit of 300 requests/60s exceeded" }
```

Nach der Anzahl der Sekunden in der `Retry-After`-Kopfzeile muss der Vorgang abgebrochen und erneut versucht werden. Wenden Sie sich an den Support, wenn Ihr Anwendungsfall eine höhere Beschränkung erfordert.

### Fehler

Fehler werden als JSON mit einem `error` Feld und normalerweise einer für Menschen lesbaren `message` zurückgegeben:

```json
{ "error": "Bad Request", "message": "tenant_id is required" }
```

| Status | Bedeutung |
| ------------------------- | ------------------------------------------------------------------ |
| `400 Bad Request` | Fehlender oder ungültiger Parameter (z. B. kein `tenant_id`, ungültiger Zeitraum) |
| `401 Unauthorized` | Fehlender, ungültiger, abgelaufener oder widerrufener API-Schlüssel |
| `403 Forbidden` | Der Schlüssel ist für den angeforderten Mandanten nicht autorisiert |
| `429 Too Many Requests` | Ratenlimit überschritten - siehe `Retry-After` |
| `502 Bad Gateway` | Upstream-Abfrage fehlgeschlagen - sicherer Versuch |
| `503 Service Unavailable` | Daten-Backend vorübergehend nicht verfügbar |

&#x200B;---

## &#x200B;4. Endpunkte

### `GET /public/v1/tenants`

Listet die Mandanten-IDs auf, die Ihr Schlüssel abfragen darf. Rufen Sie dies zuerst auf. Jeder andere Endpunkt benötigt `tenant_id` einen dieser Werte.

```bash
curl -s "{{API_BASE_URL}}/public/v1/tenants" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{ "tenants": ["tenant1", "tenant2"] }
```

### `GET /public/v1/overview`

Allgemeine KPIs zur Konsistenz für einen Mandanten über ein Zeitfenster: Anfragevolumen, Fehlerrate und Latenzperzentile.

| Parameter | Erforderlich | Beschreibung |
| ------------ | -------- | ------------------------------------------------------------------------- |
| `tenant_id` | Ja | Zu abfragender Mandant |
| `from`, `to` | Nein | Zeitraum (siehe [Zeitbereiche](#time-ranges)) |
| `minutes` | Nein | Kurzschreibweise für „Letzte N Minuten“, wenn `from`/`to` nicht angegeben sind (`15`) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/overview?tenant_id=<tenant_id>&minutes=30" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "total_spans": 48213,
  "errors": 112,
  "error_rate_pct": 0.23,
  "p50_ms": 34,
  "p95_ms": 210,
  "p99_ms": 480,
  "service_count": 12,
  "trace_count": 9021
}
```

### `GET /public/v1/services`

Listet verschiedene Berichtsnamen von Diensten für einen Mandanten auf.

| Parameter | Erforderlich | Beschreibung |
| ------------ | -------- | --------------------------------------------------------------------- |
| `tenant_id` | Ja | Zu abfragender Mandant |
| `from`, `to` | Nein | Auf in diesem Fenster angezeigte Services beschränken; standardmäßig sind es die letzten 7 Tage |

```bash
curl -s "{{API_BASE_URL}}/public/v1/services?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "services": ["checkout-api", "payments-worker", "web-frontend"]
}
```

### `GET /public/v1/traces`

Durchsucht aktuelle Spuren nach einem Mandanten mit optionalen Filtern.

| Parameter | Erforderlich | Beschreibung |
| ----------------- | -------- | ------------------------------------------------- |
| `tenant_id` | Ja | Zu abfragender Mandant |
| `from`, `to` | Nein | Zeitraum; standardmäßig auf „Letzte 24 Stunden“ eingestellt |
| `limit` | Nein | Max. zurückzugebende Zeilen (1-200, Standard 100) |
| `offset` | Nein | Paginierungsversatz (Standard 0) |
| `service` | Nein | Nach Service-Namen filtern |
| `app_name` | Nein | Nach Programm-/Instanzname filtern |
| `status` | Nein | Filtern nach Ablaufverfolgungsstatus: `ok`, `error` oder `unset` |
| `search` | Nein | Freitextsuche über span-/Vorgangsnamen hinweg |
| `min_duration_ms` | Nein | Nur Spuren zu oder über dieser Dauer |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces?tenant_id=<tenant_id>&status=error&limit=25" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TraceId": "4bf92f3577b34da6a3ce929d0e0e4736",
      "ServiceName": "checkout-api",
      "DurationMs": 812,
      "StatusCode": "Error",
      "Timestamp": "2026-08-30T09:12:44Z"
    }
  ],
  "rows": 137,
  "limit": 25,
  "offset": 0
}
```

Verwenden Sie `rows` (die Gesamtzahl der Übereinstimmungen) zusammen mit `limit`/`offset`, um die Ergebnisse durchzublättern.

### `GET /public/v1/traces/:traceId`

Gibt den gesamten span-Wasserfall für eine einzelne Spur zurück.

| Parameter | Erforderlich | Beschreibung |
| ----------- | -------- | ---------------------------------------- |
| `tenant_id` | Ja | Mandant, zu dem die Ablaufverfolgung gehört |
| `limit` | Nein | Max. Spannen für Rückgabe (1-500, Standard 500) |
| `offset` | Nein | Paginierungsversatz für sehr große Spuren |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces/4bf92f3577b34da6a3ce929d0e0e4736?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "spans": [
    {
      "SpanId": "00f067aa0ba902b7",
      "Name": "POST /checkout",
      "DurationMs": 812,
      "children": []
    }
  ],
  "totalDurationMs": 812,
  "spanCount": 14,
  "limit": 500,
  "offset": 0
}
```

### `GET /public/v1/metrics`

Gibt rohe Metrikdatenpunkte für einen Mandanten zurück.

| Parameter | Erforderlich | Beschreibung |
| ---------------------------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `tenant_id` | Ja | Zu abfragender Mandant |
| `metric` | Eines von `metric`/`like` | Exakter Metrikname |
| `like` | Eines von `metric`/`like` | SQL-`LIKE`, um mehrere Metriknamen abzugleichen |
| `type` | Nein | `gauge` (Standard) oder `sum` |
| `from`, `to` | Nein | Zeitraum; standardmäßig auf „Letzte 24 Stunden“ eingestellt |
| `service` | Nein | Nach Service-Namen filtern |
| `host` | Nein | Nach Hostnamen filtern. Erforderlich für die unten stehenden Metriken des Infrastruktur-Hosts - ohne diese werden die Messwerte von jedem Host im Mandanten gemischt |
| `attribute_key`, `attribute_value` | Nein | Nach einem bestimmten Metrikattribut filtern (muss zusammen verwendet werden) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=jvm.memory.used&type=gauge" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TimeUnix": "2026-08-30T09:00:00Z",
      "MetricName": "jvm.memory.used",
      "Value": 512482816,
      "ServiceName": "checkout-api",
      "host": ""
    }
  ],
  "rows": 1
}
```

#### Infrastruktur-Host-Metriken

Derselbe Endpunkt dient auch den Metriken auf Host-Ebene, die auf dem Infrastruktur-Dashboard angezeigt werden (CPU, Arbeitsspeicher, Lastdurchschnitt, Datenträger-E/A, Netzwerk-E/A). Verwenden Sie diese exakten `metric` / `attribute_key` / `attribute_value` Kombinationen, immer mit einem `host`:

| Dashboard-Widget | `metric` | `attribute_key` | `attribute_value` |
| --------------------- | ------------------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| CPU % | `system.cpu.utilization` | `state` | `idle` (ziehen Sie von 1 für „In Verwendung“ ab) oder fragen Sie `user`/`system`/`iowait` separat ab und addieren Sie |
| Speicherauslastung % | `system.memory.utilization` | `state` | `used` |
| Belastungsdurchschnitt (1m) | `system.cpu.load_average.1m` | — | — |
| Datenträger-Lese-E/A | `system.disk.io` (`type=sum`) | `direction` | `read` |
| Datenträger-Schreib-E/A | `system.disk.io` (`type=sum`) | `direction` | `write` |
| Disc-Lesevorgänge | `system.disk.operations` (`type=sum`) | `direction` | `read` |
| Schreibvorgänge auf der Festplatte | `system.disk.operations` (`type=sum`) | `direction` | `write` |
| Netzwerk in | `system.network.io` (`type=sum`) | `direction` | `receive` |
| Netzwerk-Ausgang | `system.network.io` (`type=sum`) | `direction` | `transmit` |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=system.cpu.utilization&type=gauge&attribute_key=state&attribute_value=idle&host=<host_name>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

**Wichtig: Datenträger- und Netzwerkwerte sind rohe, ständig steigende Zähler, keine Raten.** Die Diagramme „Bytes/Sek.“ und „Vorgänge/Sek.“ des Dashboards werden berechnet, indem zwei aufeinander folgende Zählerstände genommen und durch die verstrichene Zeit dividiert werden:

```
rate = (value_at_t2 - value_at_t1) / (t2 - t1_in_seconds)
```

### `GET /public/v1/pages`

Top der angeforderten Inhaltsseiten (`.html`) pro Dispatcher-Instanz, sortiert nach Anzahl der Anfragen. Unterstützt durch die `dispatcher.httpd.requests`-Metrik - dieser Endpunkt ist spezifisch für Zugriffsprotokolle im AEM Dispatcher/CDN-Stil und kein allgemeines Seitenanalysetool.

| Parameter | Erforderlich | Beschreibung |
| ------------ | -------- | -------------------------------------- |
| `tenant_id` | Ja | Zu abfragender Mandant |
| `from`, `to` | Nein | Zeitraum; standardmäßig auf „Letzte 24 Stunden“ eingestellt |
| `limit` | Nein | Max. zurückzugebende Zeilen (1-500, Standard 50) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/pages?tenant_id=<tenant_id>&limit=50" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "data": [
    {
      "instance": "<instance_name>",
      "domain": "www.abc.com",
      "path": "/join-us/insights.html",
      "full_url": "https://www.abc.com/join-us/insights.html",
      "requests": 7
    }
  ],
  "rows": 1
}
```

&#x200B;---

## &#x200B;5. Was diese API nicht tut

- **Kein unformatierter SQL-Zugriff.** Alle Endpunkte geben kuratierte, speziell entwickelte Daten-Shapes zurück - Sie können den zugrunde liegenden Datenspeicher nicht direkt abfragen.
- **Keine mandantenübergreifenden Abfragen.** Jede Anfrage ist auf genau eine `tenant_id` beschränkt.
- **Kein Schreibzugriff.** Die öffentliche API ist schreibgeschützt.

&#x200B;---

## 6. Support

Wenn unerwartete Fehler auftreten oder ein Anwendungsfall nicht durch diese Endpunkte abgedeckt ist, wenden Sie sich an Ihren Customer Success / Enablement Engineer, um weitere Hilfe zu erhalten.
