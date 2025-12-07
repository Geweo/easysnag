# Feuerwehr-Kassensystem

Offline-fähiges Kassensystem für Feuerwehr-Events. Ziel ist eine robuste Desktop-App mit klarer Touch-Bedienung, die Verkäufe pro Event erfasst, Bons druckt (wenn verfügbar) und historische Daten verlässlich speichert.

## Plattform & Laufzeit
- Portable Desktop-App auf Electron (Node.js + React)
- Zielsystem: Windows-EXE
- Datenbank: SQLite (lokal, offline-fähig)
- Drucker: optionaler ESC/POS-Bondrucker
- Updates: neue Versionen werden heruntergeladen, Installation erfolgt erst nach aktiver Bestätigung durch den Benutzer

## UI-Grundstruktur
Navigation als linke Sidebar:
```
---------------------------------
|  [Anzeige aktuelles Event]   |   <— oben
|------------------------------|
| [Kasse]                     |   Tab 1
| [Produkte / Artikel]        |   Tab 2
| [Events]                    |   Tab 3
---------------------------------
```

## Kernprinzipien
- Vollständig offline bedienbar
- Große, fingerfreundliche UI-Elemente, klare Farben
- Bedienung per Touchscreen, Maus oder Handschuhen möglich
- Fehlbedienungen vermeiden (Sicherheitsbestätigungen, keine kleinen Dropdowns)

## Tab 1 — Kassensystem (Event-abhängig)
### Voraussetzungen & Inhalt
- Ein Event ist aktiv ausgewählt (z. B. "Adventsmarkt 2025").
- Linke Seite: große Touch-Buttons für Produkte aus der eventabhängigen Artikelauswahl (z. B. Bier, Rote Wurst, Cola).
- Mitte/Rechts: aktueller Warenkorb mit Mengensteuerung und Summenanzeige.
- Aktionen: "Beleg drucken" und "Storno".

### Interaktionen
- Produkt hinzufügen: Klick = +1, Doppelklick = +2.
- Produkt reduzieren: Klick auf Eintrag im Warenkorb = −1.
- Produkt entfernen: Klick auf "Storno" neben Produkt oder Rechtsklick → "Artikel entfernen".
- Bon drucken: sofortiger ESC/POS-Druck, ansonsten Fehlermeldung "Kein Drucker gefunden"; Verkauf wird trotzdem verbucht.
- Bestandsänderungen nachträglich im Event-Tab: Artikelanzahl korrigieren, Storno nachträglich, Minusverkauf.

### Sicherheitsfunktion
- Existieren bereits Verkäufe eines Artikels im Event, darf der Artikel nur mit Admin-Bestätigung entfernt werden.
  Hinweis: "Dieser Artikel wurde bereits verkauft. Löschen erfordert Admin-Bestätigung. Sind Sie sicher?" mit Optionen [Ja, löschen] und [Abbrechen].

## Tab 2 — Artikel / Produkte (eventunabhängig)
- Generische Artikelliste mit Feldern: `id`, `name`, `price`, `category`, `tax` (optional später), `active`.
- Beispiele: Rote Wurst (4,00 €), Bier (3,00 €), Cola (2,50 €), Glühwein (3,50 €).
- Funktionen: neuen Artikel anlegen, bestehende bearbeiten, Artikel archivieren (nicht löschen), Kategorien verwalten (optional).
- Artikel sind nicht automatisch in Events verfügbar.

## Tab 3 — Events
- Beispiele: Adventsmarkt 2025, Frühlingsfest 2026, Johannisfeuer 2026, Weihnachtsmarkt 2026.
- Felder: `id`, `name`, `date_start`, `date_end`, `active` (bool), `selected_products` (Produkt-IDs), `sales` (Verkäufe pro Produkt).
- Funktionen: Event erstellen, Event archivieren, Event löschen (nur ohne Verkäufe), Produkte hinzufügen (aus globaler Artikelliste) und entfernen (nur ohne Verkauf; sonst Sicherheitswarnung).

## Datenmodell (vorläufig)
### Produkte (global)
`products`
- `id`
- `name`
- `price`
- `category`
- `created_at`

### Events
`events`
- `id`
- `name`
- `created_at`
- `startDate`
- `endDate`

### Event-Produkte
`event_products`
- `event_id`
- `product_id`

### Verkäufe
`sales`
- `id`
- `event_id`
- `product_id`
- `quantity`
- `price_at_time`
- `timestamp`

> `price_at_time` fixiert den Verkaufspreis, damit spätere Preisänderungen historische Verkäufe nicht verändern.

## Betriebsverhalten
- Verkauf legt Eintrag in der `sales`-Tabelle an.
- Drucker vorhanden: sofortiger ESC/POS-Bon.
- Kein Drucker: Verkauf wird verarbeitet, Fehlermeldung "Kein Drucker gefunden", kein erneuter Druck (außer optional "Beleg erneut drucken").

## Erweiterbarkeit / zukünftige Features
- Auswertung & Statistiken (Umsatz pro Produkt/Event, Top-Artikel)
- Export (CSV, PDF)
- Admin-PIN, Benutzerprofil "Kassierer"
- Mehrwertsteuergruppen
- Rückerstattungen, Inventur
- Einkaufspreise vs. Verkaufspreise, Gewinnermittlung

## Implementierungsfahrplan
1. **Grundgerüst aufsetzen**
   - Electron-Container mit React-Frontend und lokaler SQLite-Datenbank initialisieren.
   - Basale Navigations-Shell (Sidebar + Tabs) mit Platzhaltern erstellen.
2. **Datenmodell & Persistenz**
   - SQLite-Schema für `products`, `events`, `event_products`, `sales` anlegen.
   - Data-Access-Layer mit Migrationen implementieren; Testdaten (Seed) hinzufügen.
3. **Tab 2 — Produkte**
   - CRUD-UI für Produkte mit Archivieren-Funktion bauen.
   - Optional: Kategorienverwaltung und Filterung ergänzen.
4. **Tab 3 — Events**
   - Event-CRUD inkl. Archivieren/Löschen-Validierung (keine Verkäufe) umsetzen.
   - Produktzuordnung zum Event mit Sicherheitswarnung bei bestehenden Verkäufen.
5. **Tab 1 — Kasse**
   - Event-Auswahl erzwingen; produktspezifische Buttons dynamisch aus `event_products` laden.
   - Warenkorb-Logik (Hinzufügen/Reduzieren/Entfernen) mit Mengensteuerung implementieren.
   - Verkauf speichern (`sales`) mit `price_at_time` Snapshots.
6. **Drucker-Integration**
   - ESC/POS-Druck via optionalem Drucker-Adapter; Fehlermeldung bei Nichtverfügbarkeit.
   - Funktion "Beleg erneut drucken" als optionale Erweiterung vorsehen.
7. **Updates & Distribution**
   - Updater, der neue Versionen herunterlädt, Installation aber nur nach Bestätigung startet.
   - Windows-Build-Pipeline (EXE) einrichten und offline-Funktionalität verifizieren.
8. **Qualität & Sicherheit**
   - Admin-PIN für geschützte Aktionen (Löschen bei Verkäufen, Minusverkäufe) vorsehen.
   - End-to-End-Tests für zentrale Flows (Verkauf, Druck, Event-Produkt-Restriktionen) einplanen.
