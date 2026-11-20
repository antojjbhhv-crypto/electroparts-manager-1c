# ElectroParts Manager

## 1. Projektübersicht

ElectroParts Manager ist eine kleine Lernkonfiguration auf Basis von 1C:Enterprise 8.3. Sie bildet die Lagerhaltung elektronischer Bauteile für eine kleine Servicewerkstatt ab: Bauteile, Lieferanten, Wareneingang, Verbrauch und zwei Auswertungen.

Das Projekt entstand als selbstständiges Übungsprojekt, um den Umgang mit Metadaten, Dokumentenbuchungen, Akkumulationsregistern und Datenkompositionssystem (DKS) zu üben. Der Umfang ist bewusst klein gehalten.

## 2. Funktionen

- Katalog der elektronischen Bauteile mit Artikel, Typ, Einheit, Beschreibung, Mindestbestand und optionalem Lieferanten.
- Katalog der Lieferanten mit Kürzel, Ansprechpartner, Kontaktdaten, Lieferzeit, Mindestbestellwert und Aktiv-Kennzeichen.
- Wareneingang: Buchung erzeugt Zugangsbewegungen im Bestandsregister.
- Verbrauch: Buchung prüft den Mindestbestand und erzeugt Abgangsbewegungen.
- Mehrere Positionen im Verbrauch werden je Bauteil zusammengefasst, bevor geprüft wird.
- Wiederholte Buchung erzeugt keine doppelten Bewegungen, Stornierung entfernt die Bewegungen.
- Bericht Lagerbestand mit Bestand, Mindestbestand und Status je Bauteil.
- Bericht Lieferantenübersicht mit Anzahl Bauteile, Summe Bestand und Anzahl der Bauteile unter Mindestbestand.

## 3. Verwendete Technologien

- 1C:Enterprise 8.3, Testversion 8.3.27.1508, Dateibasis.
- Verwaltete Anwendung, Schnittstelle Taxi.
- Akkumulationsregister vom Typ Saldo.
- Datenkompositionssystem für beide Berichte.
- Module in eingebauter Sprache (Basis-Syntax von 1C).

## 4. Datenmodell

- Katalog **Komponenten**: Artikel, KomponentenTyp, Einheit, Beschreibung, Mindestbestand (Zahl, 15 Stellen, 3 Nachkommastellen, nicht negativ, Standardwert 0), Lieferant (Verweis auf Katalog Lieferanten, optional).
- Katalog **Lieferanten**: Kuerzel (Zeichenkette, 10 Zeichen, Pflichtfeld, eindeutig), Ansprechpartner, Email, Telefon, Land, Lieferzeit (Zahl, nicht negativ), Mindestbestellwert (Zahl, 15.2, nicht negativ, Betrag in EUR), Aktiv (Wahrheitswert, Standardwert Ja).
- Akkumulationsregister **Bestaende**: Typ Saldo, Dimension Komponenten, Ressource Menge.
- Dokument **Wareneingang**: Attribute Komponenten und Menge, registriert Bewegungen im Register Bestaende.
- Dokument **Verbrauch**: tabellarischer Teil Komponenten mit den Spalten Komponente und Menge, registriert Bewegungen im Register Bestaende.
- Berichte **Lagerbestand** und **Lieferantenuebersicht**.
- Subsystem **Lager** mit beiden Katalogen, beiden Dokumenten und beiden Berichten.

## 5. Lagerverwaltung

Der Wareneingang erfasst genau ein Bauteil pro Beleg mit Menge. Beim Buchen schreibt das Dokument eine Zugangsbewegung mit Datum des Belegs.

Der Verbrauch erfasst mehrere Positionen. Beim Buchen prüft das Dokument zuerst jede Zeile (Bauteil vorhanden, Menge positiv), summiert danach die Mengen je Bauteil und vergleicht das Ergebnis mit dem verfügbaren Bestand und dem Mindestbestand. Erst wenn alle Positionen zulässig sind, werden die Abgangsbewegungen geschrieben.

Der Bestand wird ausschließlich über Bewegungen im Register geführt. Es gibt keine manuelle Bestandskorrektur.

## 6. Mindestbestandsprüfung

Jedes Bauteil hat einen Mindestbestand. Beim Verbrauch gilt:

- Unterschreitet der Bestand nach dem Verbrauch den Mindestbestand, wird der Beleg nicht gebucht.
- Erreicht der Bestand nach dem Verbrauch genau den Mindestbestand, wird der Beleg gebucht.
- Die Meldung nennt Artikel, verfügbare Menge nach Verbrauch und Mindestbestand, zum Beispiel: `Bestand unter Mindestbestand: C-100N. Verfügbar nach Verbrauch: 4,000 Stk. Mindestbestand: 5,000 Stk.`

Beim erneuten Buchen wird der eigene alte Bewegungsbestand herausgerechnet, damit keine doppelte Zählung entsteht.

## 7. Lieferantenverwaltung

Lieferanten werden im Katalog Lieferanten geführt. Das Kürzel ist Pflicht und muss eindeutig sein; die Prüfung ignoriert Groß- und Kleinschreibung.

Ein Bauteil kann einem Lieferanten zugeordnet werden. Ein inaktiver Lieferant kann keinem Bauteil neu zugeordnet werden; eine bereits bestehende Zuordnung bleibt beim Deaktivieren erhalten. Ein Lieferant mit zugeordneten Bauteilen kann nicht gelöscht werden.

## 8. Berichte und Auswertungen

**Lagerbestand** zeigt je Bauteil Einheit, Mindestbestand, Status und Bestand zum Stichtag. Bauteile ohne Bewegungen erscheinen mit Bestand 0. Der Status lautet `OK` oder `Unter Mindestbestand`. Negative Bestände werden nicht versteckt.

**Lieferantenuebersicht** zeigt je Lieferant Kürzel, Anzahl der zugeordneten Bauteile, Summe des Bestands und Anzahl der Bauteile unter Mindestbestand. Lieferanten ohne Bauteile erscheinen mit Nullen. Bauteile ohne Lieferant werden nicht in die Übersicht aufgenommen.

## 9. Installation und Start

1. 1C:Enterprise 8.3 starten.
2. Eine leere Dateibasis anlegen oder die mitgelieferte Demo-Basis öffnen.
3. Konfiguration aus der Datei `ElectroPartsManager_v1.0.cf` laden und die Datenbankkonfiguration aktualisieren.
4. Anwendung im Modus 1C:Enterprise starten.
5. Im Abschnitt Lager mit den Katalogen, Dokumenten und Berichten arbeiten.

## 10. Testdaten

Die Demo-Basis enthält ausschließlich erfundene Daten:

- Lieferanten: Elektronik Nord GmbH (ELN), Bauteile Sued AG (BSA), Muster Handel KG (MHK, ohne Bauteile).
- Ansprechpartner, E-Mail und Telefon sind frei erfunden. Die E-Mail-Adressen nutzen die reservierte Domain `example.com`, die Telefonnummern sind neutrale Platzhalter.
- Bauteile: Kondensator 100 nF (Mindestbestand 10, Bestand 10, Status OK), Widerstand 10 kOhm (Bestand 100, Status OK), LED rot 5 mm (ohne Bewegungen, Bestand 0, Status OK), Mikrocontroller ATmega328P (Bestand -1, Status Unter Mindestbestand als Defizitbeispiel).
- Belege: zwei Wareneingänge vom 05.11.2026.

## 11. Bekannte Einschränkungen

- Der Stichtag der Berichte ist im Schema als aktuelles Datum hinterlegt. Ein Ändern des Stichtags über die Berichtseinstellungen in der Form ist derzeit nicht möglich.
- Der Wareneingang erfasst genau eine Position pro Beleg, der Verbrauch mehrere Positionen.
- Es gibt keine Reservierungen, keine Bestellungen, keine Preise und keine Buchhaltung.
- Die Prüfung auf eindeutiges Kürzel erfolgt im Modul, nicht über einen Datenbankindex.
- Die Demo-Basis enthält bewusst einen negativen Bestand als Defizitbeispiel.

### Hinweis zur Plattformoberfläche

Die verwendete Lernversion der 1C:Enterprise-Plattform besitzt eine russischsprachige Systemoberfläche. Die fachlichen Bezeichnungen der Konfiguration, Stammdaten, Dokumente und Berichte sind auf Deutsch.

## 12. Screenshots

Im Paket liegen fünf Aufnahmen der Demo-Basis:

| Datei | Inhalt |
| --- | --- |
| `screenshot-01-start.png` | Startseite mit dem Abschnitt Lager und der Befehlsleiste |
| `screenshot-02-lieferanten-liste.png` | Liste der Lieferanten mit Kürzel, Ansprechpartner und Kontaktdaten |
| `screenshot-03-lieferant-karte.png` | Karte eines Lieferanten mit allen Feldern und dem Kennzeichen Aktiv |
| `screenshot-04-lagerbestand.png` | Bericht Lagerbestand mit Stichtag, Status und Gesamtsumme |
| `screenshot-05-lieferantenuebersicht.png` | Bericht Lieferantenübersicht je Lieferant |

## 13. Download

Die Konfigurationsdatei steht im Release bereit:

- [ElectroParts Manager v1.0 (GitHub Release)](https://github.com/antojjbhhv-crypto/electroparts-manager-1c/releases/tag/v1.0.0)