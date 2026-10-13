---
source-git-commit: 05ed98c07602488843c52cd262af2d8cc204fef1
workflow-type: tm+mt
source-wordcount: '1117'
ht-degree: 2%
---
### Schritt für Schritt ein Dashboard erstellen

#### Schritt 1: Bevor Sie beginnen

- **Wählen Sie im Mandanten-** einen einzelnen Kunden aus. **Dashboard erstellen** ist deaktiviert, während **Alle Konten** ausgewählt ist. SQL kann nicht für den Demo-Mandanten ausgeführt werden.
- **ROLES.** Jeder Benutzer mit Dashboard-Zugriff kann Widgets hinzufügen, bearbeiten und speichern. Das Erstellen oder Bearbeiten **Variablen** und das Überprüfen der Abfrage eines Widgets ist auf **Organisations-** und **Superadmins** beschränkt.

#### Schritt 2: Dashboard erstellen

1. Öffnen Sie **Dashboards** im linken Navigationsbereich.
2. Klicken Sie **Dashboard erstellen**.
3. Startpunkt wählen:

   | Option | Was Sie erhalten |
   | ---------------------------- | -------------------------------------------------------------------------------------------------------- |
   | **Erstellen eines benutzerdefinierten Dashboards** | Ein leeres Dashboard mit einer Registerkarte. |
   | **Grundlagen zu CPU** | Host und Prozess-CPU (user/system/iowait/steal), Durchsatz, Last, Apdex. |
   | **All-in-One-Übersicht** | KPIs (Volumen, Fehler %, Apdex, p95), Fehler-/Traffic-Pies, CPU-/Speichertabellen, JVM-Threads, langsame Vorgänge. |
   | **Top-Transaktionen** | Apdex, p95 nach Vorgang, Host-seitige Last/CPU, langsamste Transaktionen mit Trace-IDs, HTTP-Fehleraufschlüsselung. |

4. Geben Sie einen **Namen** ein und bestätigen Sie. Die Dashboard-Arbeitsfläche wird geöffnet.

#### Schritt 3: Die Arbeitsfläche kennenlernen

| Bereich | Steuerung |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Obere Leiste** | **Widget hinzufügen**, **Anleitung** (In-App-Spickzettel), **Importieren**, **Exportieren**, **Speichern**, Zeitauswahl und aktualisieren |
| **Variablenleiste** | Mandanten-Badge, eine Auswahl pro Variable, **+ Variable** (Administratoren) |
| **Registerkartenleiste** | Registerkarten wechseln; umbenennen (Bleistift), löschen (×), hinzufügen (+) |
| **Raster** | 12 Spalten. Widgets am Griff ziehen, Größe von der Ecke ändern (mindestens 2×2). |
| **Widget-Kopfzeile** (beim Bewegen des Mauszeigers) | CSV herunterladen, PNG herunterladen, bearbeiten, duplizieren, entfernen, Griff ziehen |

&#x200B;> [!IMPORTANT]
> **Speichern ist explizit.** Layout-Änderungen, neue Widgets, Registerkarten und Variablen verbleiben im Browser, bis Sie auf &quot;**&quot;**. Die Schaltfläche _Speichern… → gespeichert_ oder _Speichern fehlgeschlagen_ (siehe [Fehlerbehebung](#troubleshooting)).

#### Schritt 4: Widget hinzufügen

Klicken Sie **Widget hinzufügen**. Das Dialogfeld verfügt über drei Modi: **Einzelabfrage**, **Formel** und **Text**. Es verfügt auch über eine eigene Zeitauswahl, sodass Sie eine Vorschau des Widgets für jedes Fenster anzeigen können.

**Option A: der Builder (kein Code)**

1. _(Optional)_ **Mit einem Helper beginnen**: Wählen Sie eine von 57 Vorgaben, gruppiert als Golden Signals, APM, AEM, Transactions, Traces &amp; Spans, Infrastructure, Publishers, JVM &amp; Runtime und Data Discovery. Eine Voreinstellung füllt den Titel, die Abfrage, den Diagrammtyp und die Einheit aus.
2. **Anzeigen**: Wählen Sie `Count`, `Average`, `Percentile` (und seine Ebene), `Sum`, `Min`, `Max` oder `Unique count` und dann ein Attribut aus.
3. **Von**: Wählen Sie die Entität aus.
4. **Filter**: Zeilen von `attribute operator value` hinzufügen (`=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`). Alle Zeilen werden mit `AND` kombiniert.
5. **Gruppieren nach**: Wählen Sie ein Facettenattribut und **Top N**, um nur die geschäftigste Serie zu behalten.
6. **Im Zeitverlauf**: Schaltet `TIMESERIES` ein, mit einer **alle N Minuten** Behältergröße.
7. Überprüfen Sie die generierte SQL unter dem Formular und klicken Sie dann auf **Abfrage ausführen**.

**Option B: der SQL-Editor**

1. Wechseln Sie den Umschalter von **Builder** zu **SQL**. Das Bearbeiten eines vorhandenen Widgets wird immer in SQL geöffnet.
2. Geben Sie eine Abfrage ein oder fügen Sie sie ein. Der Editor wird automatisch im Live-Katalog ausgefüllt.
3. Verwenden Sie das Bedienfeld **Handbuch zur Abfrage**, um Entitäten, Attribute und Beispiele zu durchsuchen.
4. Klicken Sie **Abfrage ausführen**. Sie sehen, _`<Entity>`Abfrage gültig ist_ eine Live-Vorschau und eine Zusammenfassung (`N rows · N time points · N groups`). Eine _Serie_ Warnung wird angezeigt, wenn weniger als drei Zeitpunkte vorliegen.

&#x200B;> [!TIP]
> Der Builder behandelt die häufigsten Abfragen. Zu SQL wechseln für `apdex`, `percentage`, `rate`, `FILTER`, mehrere Aggregate, `OR`, `IN`, mehrere Facetten oder `ORDER BY`.

#### Schritt 5: Konfigurieren des Erscheinungsbilds

| Feld | Anmerkungen |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Titel** | Bis zu 200 Zeichen. Kann `{{variable}}` Token enthalten. |
| **Diagrammtyp** | `Line`, `Area`, `Bar`, `Pie / Donut`, `Number`, `Table`. Nach jedem Durchlauf wird ein Diagrammtyp empfohlen, bis Sie selbst einen auswählen. |
| **Einheit** | Optional, bis zu 40 Zeichen (`ms`, `%`, `req/s`, `MB`, `bytes`). Lassen Sie es leer, um die abgeleitete Einheit zu verwenden (z. B. `ms` für `duration`, `%` für `CpuUtilization`). |
| **Farbschwellen** | Nur Zahl-Widgets. Bis zu 10 Bänder der Form _at oder höher `value` → Grün / Amber / Rot_. Die höchste passende Band gewinnt. |

#### Schritt 6: Speichern Sie zuerst das Widget und dann das Dashboard

**Widget speichern** wird erst aktiviert, nachdem eine Ausführung Daten zurückgibt. Neue Widgets bestehen aus 6 Spalten × 3 Zeilen und werden jeweils zwei pro Zeile gepackt. Ordnen Sie sie an und klicken Sie dann **Speichern** in der oberen Leiste.

### Wählen des richtigen Diagramms

| Form der Abfrage | Bestes Diagramm | Vorteile |
| ------------------------------------------------- | ------------------- | ---------------------------------------------- |
| Ein Aggregat, kein `FACET`, kein `TIMESERIES` | **Zahl** | Eine einzelne Kachel KPI; Hinzufügen von Schwellenwerten. |
| Aggregat + `TIMESERIES` | **LINE**/**AREA** | Ein Trend; eine Reihe pro Facettenwert. |
| Aggregat + `FACET` (wenige Werte) | **bar**/**pie** | Vergleicht Kategorien; Kuchen als Anteil eines Ganzen. |
| Mehrere Aggregate + `FACET` | **Tabelle** | Jedes Aggregat wird zu einer Spalte. |
| Rohattribute | **Tabelle** | Eine Ereignisliste. |
| Aggregat + `FACET` + `TIMESERIES` auf wenigen Daten | **bar** | Liest besser als glückliche Zeilen. |

Wenn die einzelne Zeile eines Zahlen-Widgets einen Textwert enthält (z. B. `latest(host)`), wird das Widget als Untertitel angezeigt.

### Text-Widgets: Anmerkungen und Abschnittsüberschriften

**Widget → Text hinzufügen**:

- **Hinweis**: Nur Text (bis zu 4.000 Zeichen, Leerzeilen werden beibehalten). Verwenden Sie es für Runbook-Links, Eigentümer und SLO-Definitionen. `{{variables}}` werden gerendert.
- **Bereichs-**: Häkchen _Erstellen Sie eine Bereichs-_. Es erstreckt sich über die gesamte Breite und sein Pfeil reduziert jedes Widget darunter bis zur nächsten Abschnittsüberschrift. Verwenden Sie sie für Gruppierungen wie **PUBLISH**, **AUTHOR** und **DISPATCHER**.

&#x200B;> [!TIP]
> Widget-Titel, die auf `- PUBLISH` oder `- AUTHOR` enden, erhalten einen subtilen Farbhinweis, der das Scannen von AEM-Dashboards nach Ebene erleichtert.

### Die Zeitauswahl

- Damit wird das Fenster für **alle**-Widget und für abfragebasierte Variablenoptionen festgelegt.
- In Widget-Abfragen wird `SINCE`/`UNTIL` überschrieben.
- Das maximale Fenster beträgt **31 Tage**.
- `TIMESERIES AUTO` passt seine Behältergröße an das Fenster an.
- Der Untertitel jedes Widgets zeigt das tatsächlich angewendete Fenster an, z. B. _Seit vor 1 Stunde_.
- Die Ergebnisse werden 30 Sekunden lang zwischengespeichert. Verwenden Sie die Schaltfläche „Aktualisieren“, um eine erneute Ausführung zu erzwingen.

### Variablen

Mit Variablen kann ein Dashboard mehrere Kontexte bereitstellen, z. B. jeweils einen Service, eine Umgebung oder einen Schwellenwert. Sie werden in ([&#x200B; 3) &#x200B;](#87-make-dashboards-reusable-with-variables).

&#x200B;> [!WARNING]
> **Bekannte Einschränkung im aktuellen Build:** `{{variables}}` in Widgets **Titel** und **Text-Widgets**. Innerhalb des Widgets **SQL** werden sie im Dashboard gerendert, aber die **Widget hinzufügen** -Vorschau und die Server-seitige **Speichern**-Prüfung lehnen das rohe `{{name}}`-Token mit _unerwartetem Zeichen &#39;&lbrace;&#39;_ ab. Bis dies behoben ist, verwenden Sie Literalwerte in Widget-Abfragen und verwenden Sie **Duplizieren** um Varianten zu erstellen.

### Formel-Widgets

Eine Formel kombiniert 2-6 benannte SQL-Abfragen arithmetisch, z. B. `(errors / total) * 100`. Verwenden Sie eine Option, wenn ein KPI mehr als eine Abfrage erfordert, z. B. ein Verhältnis, Reserven, eine Summe der Raten oder eine Brennrate. Siehe [Tutorial &#x200B;](#86-combine-queries-with-formula-widgets).

### Freigeben, Exportieren und Importieren

| Aktion | Funktion |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Speichern** | Behält den Namen, die Registerkarten, Widgets, das Layout und die Variablen bei. Jede Widget-Abfrage wird Server-seitig erneut validiert. Bei einem ungültigen Widget schlägt die gesamte Speicherung fehl. |
| **Export** | Downloads `<dashboard-name>.json` (Registerkarten, Widgets, Variablen). |
| **Import** | Lädt einen JSON-Export **in das geöffnete Dashboard** und ersetzt dessen Registerkarten und Variablen. Klicken Sie dann auf **Speichern**. Tipp: Erstellen Sie zuerst ein leeres Dashboard und importieren Sie es dann in das Dashboard. |
| **CSV/PNG** | Downloads pro Widget aus der Widget-Kopfzeile. |

&#x200B;> [!TIP]
> Mit dem Export und Import können Sie Dashboards zur Versionskontrolle in Git verwenden und ein Dashboard von einem Kunden zum anderen weiterleiten. Das Dateiformat wird im [Anhang](#142-export-file-format) beschrieben.

### Tipps für die Leistung von Dashboards und Diagrammen

- **Verwenden Sie Tabellen für große** anstelle von Balken oder Vielreihen-Linien.
- **Teilen Sie belegte Dashboards in Registerkarten auf.** Nur die Widgets der sichtbaren Registerkarte werden ausgeführt.
- **Immer Facettenabfragen `LIMIT`** insbesondere bei Attributen mit hoher Kardinalität wie `urlPath`, `traceId` und `jcr.path`.
- **Filtern Sie früh** nach Attributen mit niedriger Kardinalität (`service`, `kind`, `status`).
- **Bevorzugen Sie`TIMESERIES AUTO`**, um kleine Behälter zu fixieren, und vermeiden Sie `TIMESERIES MAX`, außer über ein paar Minuten.
- **Entwerfen Sie langfristige Trends für Datenaggregationen.** Siehe [Leistung](#performance).
