# Lastenheft
## 1. Projektübersicht
Das System „Rechnung Digital“ ist eine Softwarelösung zur automatisierten Erstellung, Validierung und Versendung elektronischer Rechnungen (E-Rechnungen) gemäß der europäischen Norm EN 16931 (ZUGFeRD/XRechnung). Ziel ist es, den manuellen Aufwand in der Buchhaltung durch eine „No-Touch“-Automatisierung um bis zu 80 % zu reduzieren. Das System integriert sich als Middleware zwischen bestehenden ERP/CRM-Systemen und dem Versandkanal (E-Mail/Peppol) und überwacht den gesamten Zahlungszyklus bis hin zum Mahnwesen.

## 2. Funktionale Anforderungen
### 2.1. Use Cases
Das System unterscheidet zwischen vollautomatisierten Prozessen (Regelfall) und manuellen Eingriffen durch Mitarbeiter im Fehlerfall („Discovery Layer“).
![Use-Case-Diagramm](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Use%20Case.png)

### 2.2. Prozessabläufe
Die Prozesslogik folgt einem „Management by Exception“-Ansatz. Der Standardprozess läuft ohne menschliche Interaktion ab.
![Swimlane-Diagramm](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Swim%20Lanes.png)

## 3. Technische Anforderungen
### 3.1. Datenstruktur
Das System basiert auf einer serviceorientierten Architektur, die streng zwischen dem Domänen-Modell (Daten) und den Services (Logik) trennt.
![Klassendiagramm](https://github.com/finnfllr/SWE-Projekt/blob/main/Images/Klassendiagramm%20Allgemein.png)

Erläuterung der Komponenten:

- Domänen-Modell:

  - Zentrales Objekt ist die `Rechnung`, welche über eine eindeutige `rechnungsNr` (UUID) verfügt.

  - Der Lebenszyklus der Rechnung wird über das Enum `RechnungsStatus` gesteuert. Dieses bildet die logische Kette von der Erstellung (`ERSTELLT`) über die Prüfung (`VALIDIERUNG_OK` / `FEHLER`) bis hin zum Mahnwesen (`GEMAHNT_STUFE_1` bis `BEI_INKASSO`) ab .

  - Die Klasse `RechnungsFehler` speichert Validierungsprobleme, damit diese im Dashboard angezeigt werden können.

- Kern-Services (Core Engine):

  - Der `RechnungsManager` fungiert als Controller, der die Prozesse steuert.

  - Die Interfaces `IGenerator` und `IValidator` entkoppeln die konkrete Implementierung (ZUGFeRD, XML) von der Logik. Dies gewährleistet, dass später auch neue Standards (z. B. XRechnung 3.0) leicht integriert werden können.

  - Der `MahnService` prüft täglich die Fälligkeiten und eskaliert den Status der Rechnung bei Zahlungsverzug automatisch.

- Infrastruktur & Schnittstellen:

  - `ERPRepository`: Lädt die Rohdaten aus dem vorgeschalteten ERP-System.

  - `MailService`: Übernimmt den physischen Versand der generierten PDF/XML-Container via SMTP.

  -

### 3.2. Systemarchitektur
Das System wird als mehrschichtige Anwendung (Multi-Tier) realisiert :
1. Frontend (Dashboard): Webbasierte Oberfläche für Statusanzeige und Fehlerbehandlung.
2. Backend (Core Engine): Java-basierte Anwendung.
  -Nutzung von javax.xml.parsers für XML-Verarbeitung.
  -Validierung mittels javax.xml.validation gegen XRechnung-Schemata.
  -Währungsberechnungen präzise mittels java.math.BigDecimal.

3. Datenbank: Relationale Datenbank zur Speicherung von Rechnungen, Audit-Logs (Protokollierung) und Status.

4. Interfaces:
  -SMTP-Server (Jakarta Mail API) für den Versand.
  -REST/CSV-Schnittstellen zu ERP und Inkasso-Dienstleistern.

### 3.3. Sequenzdiagramme
Um die zeitliche Abfolge der Erstellung sicherzustellen, ist folgender Ablauf definiert:
![Seqzendiagramm-Erstellung](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Sequenz%20Diagramm%20Rechnungserstellung.png)

Um die zeitliche Abfolge der Mahnung sicherzustellen, ist folgender Ablauf definiert:
![Sequenzdiagramm-Mahnung](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/SequenzDiagramm%20Mahnung.png)

## 4. Nicht-funktionale Anforderungen (Qualitätssicherung)
- Performance: Die Validierung einer Rechnung muss in unter 150 ms erfolgen, um auch Massenverarbeitungen nicht zu verzögern.
- Rechtssicherheit & Compliance: Das System muss 100 % konform zur Norm EN 16931 sein. Änderungen am Validierungsstandard (z. B. XRechnung 3.x) müssen durch Updates der Schemata einpflegbar sein.
- Zuverlässigkeit: Bei SMTP-Fehlern (z. B. Bounces, Server nicht erreichbar) muss eine automatische Wiederholungslogik (Retry 4xx) greifen.
- Audit-Trail: Jede Änderung am Status einer Rechnung sowie manuelle Eingriffe müssen unveränderbar protokolliert werden (Logging für GoBD).

## 5. Testplan

