# Lastenheft
## Projektübersicht
Das System „Rechnung Digital“ ist eine Softwarelösung zur automatisierten Erstellung, Validierung und Versendung elektronischer Rechnungen (E-Rechnungen) gemäß der europäischen Norm EN 16931 (ZUGFeRD/XRechnung). Ziel ist es, den manuellen Aufwand in der Buchhaltung durch eine „No-Touch“-Automatisierung um bis zu 80 % zu reduzieren. Das System integriert sich als Middleware zwischen bestehenden ERP/CRM-Systemen und dem Versandkanal (E-Mail/Peppol) und überwacht den gesamten Zahlungszyklus bis hin zum Mahnwesen.

##Funktionale Anforderungen
### Use Cases
Das System unterscheidet zwischen vollautomatisierten Prozessen (Regelfall) und manuellen Eingriffen durch Mitarbeiter im Fehlerfall („Discovery Layer“).
![Use-Case-Diagramm](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Use%20Case.png)

### Prozessabläufe
Die Prozesslogik folgt einem „Management by Exception“-Ansatz. Der Standardprozess läuft ohne menschliche Interaktion ab.
![Swimlane-Diagramm](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Swim%20Lanes.png)

## Technische Anforderungen
### Datenstruktur
Die Datenmodellierung muss zwingend den Vorgaben der EN 16931 entsprechen. Folgende Kern-Objekte sind zu implementieren:
![Klassendiagramm](https://github.com/finnfllr/SWE-Projekt/blob/main/Images/Klassendiagramm%20Allgemein.png)

Wichtige Datenfelder :
-Stammdaten: Name, Anschrift, USt-ID (Absender & Empfänger).
-Bewegungsdaten: Rechnungsnummer, Lieferdatum, Währung.
-Positionsdaten: Menge, Art, Einzelpreis, Steuersatz.
-Summen: Netto, Brutto, Steuerbetrag, Entgeltminderungen (Skonto).

### Systemarchitektur
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

### Sequenzdiagramme
Um die zeitliche Abfolge der Erstellung sicherzustellen, ist folgender Ablauf definiert:
![Seqzendiagramm-Erstellung](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/Sequenz%20Diagramm%20Rechnungserstellung.png)

Um die zeitliche Abfolge der Mahnung sicherzustellen, ist folgender Ablauf definiert:
![Sequenzdiagramm-Mahnung](https://github.com/finnfllr/SWE-Projekt-E-Rechnungen/blob/main/Images/SequenzDiagramm%20Mahnung.png)

