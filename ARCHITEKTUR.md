# Architektur der Konfiguration

## Überblick

Die Konfiguration besteht aus zwei Katalogen, einem Akkumulationsregister, zwei Dokumenten, zwei Berichten und einem Subsystem. Die Datenhaltung ist bewusst schlank: Bestände entstehen ausschließlich aus Belegen.

## Objekte

| Objekt | Typ | Aufgabe |
| --- | --- | --- |
| Komponenten | Katalog | Stammdaten der Bauteile, Mindestbestand, Zuordnung zum Lieferanten |
| Lieferanten | Katalog | Stammdaten der Lieferanten, Kürzel, Konditionen, Aktiv-Kennzeichen |
| Bestaende | Akkumulationsregister (Saldo) | Bestand je Bauteil, Dimension Komponenten, Ressource Menge |
| Wareneingang | Dokument | Zugang einer Position, registriert im Register |
| Verbrauch | Dokument | Abgang mehrerer Positionen, prüft Mindestbestand |
| Lagerbestand | Bericht (DKS) | Bestand, Mindestbestand, Status zum Stichtag |
| Lieferantenuebersicht | Bericht (DKS) | Kennzahlen je Lieferant |
| Lager | Subsystem | Zusammenfassung der Objekte für die Oberfläche |

## Module und Logik

**Katalog Komponenten, Modul des Objekts**

- Vor dem Schreiben: Mindestbestand darf nicht negativ sein.
- Vor dem Schreiben: Ein inaktiver Lieferant kann einem Bauteil nicht neu zugeordnet werden; eine bestehende Zuordnung bleibt erhalten.

**Katalog Lieferanten, Modul des Objekts**

- Vor dem Schreiben: Name und Kürzel sind Pflicht, Lieferzeit und Mindestbestellwert dürfen nicht negativ sein.
- Vor dem Schreiben: Das Kürzel muss eindeutig sein, die Prüfung erfolgt über eine Abfrage ohne Beachtung der Groß- und Kleinschreibung.
- Vor dem Löschen: Ein Lieferant mit zugeordneten Bauteilen wird nicht gelöscht.

**Dokument Wareneingang, Modul des Objekts**

- Prüft Bauteil und Menge.
- Schreibt eine Zugangsbewegung über die Bewegungen des Dokuments, also ohne manuelles Schreiben des Registers.

**Dokument Verbrauch, Modul des Objekts**

- Sperrt das Register für die Dauer der Buchung.
- Prüft jede Zeile, summiert die Mengen je Bauteil.
- Liest den Bestand zum Dokumentdatum und rechnet den Saldo der eigenen Bewegungen heraus, damit eine erneute Buchung nicht doppelt zählt.
- Vergleicht das Ergebnis mit dem Mindestbestand und verweigert die Buchung mit deutscher Meldung, wenn der Mindestbestand unterschritten wird.
- Schreibt die Abgangsbewegungen gesammelt je Bauteil.

**Bericht Lagerbestand**

- Datenquelle: Katalog Komponenten, links verknüpft mit der virtuellen Tabelle Bestaende.Остатки zum Stichtag.
- Fehlende Bewegungen werden über ЕСТЬNULL auf 0 gesetzt, der Status wird im Abfrageausdruck gebildet.
- Gruppierung nach Bauteil, Einheit, Mindestbestand und Status, Summe über den Bestand.
- Darstellung der Nullwerte über bedingte Formatierung mit dem Format `ЧН=0`.

**Bericht Lieferantenuebersicht**

- Datenquelle: Katalog Lieferanten, links verknüpft mit einer Unterabfrage, die je Lieferant die Anzahl Bauteile, die Summe des Bestands und die Anzahl der Bauteile unter Mindestbestand bildet.
- Lieferanten ohne Bauteile erscheinen mit Nullen, Bauteile ohne Lieferant werden nicht berücksichtigt.
- Jedes Bauteil wird genau einmal gezählt.

## Zusammenspiel

1. Bauteile und Lieferanten werden gepflegt.
2. Wareneingang bucht Zugänge, Verbrauch bucht Abgänge.
3. Das Register Bestaende enthält den aktuellen Bestand je Bauteil.
4. Beide Berichte lesen ausschließlich das Register und die Stammdaten und verändern keine Daten.

## Bewusste Grenzen

- Keine Reservierungen, Bestellungen, Preise oder Buchhaltung.
- Kein eigener Register für Lieferungen oder Preise.
- Der Stichtag der Berichte ist im Schema hinterlegt und nicht über die Form änderbar.
