# ADR-001: Einführung des Saga-Orchestrator-Patterns für Vertragsänderungen

**Status:** Akzeptiert  
**Datum:** 19.11.2025  
**Autor(en):** Software Architekt Team

## Kontext
Der Geschäftsprozess zur Erhöhung der Deckungssumme erfordert Interaktionen über mehrere verteilte Systeme: Das Portal (Frontend), neue Microservices und monolithische Legacy-Systeme (Produkt- und Bestandssystem).

[cite_start]Die Transaktion darf fachlich nur erfolgreich sein, wenn alle Schritte (Kalkulation, Bestätigung durch Kunden, Bestandsführung) konsistent abgeschlossen sind [cite: 4-6]. Da die beteiligten Legacy-Systeme keine modernen verteilten Transaktionen (wie 2PC/XA) über REST oder Events unterstützen und über Adapter angebunden werden müssen, ist eine atomare Datenbank-Transaktion über alle Systeme hinweg technisch unmöglich. [cite_start]Zudem erfordert der Prozess eine Nutzerinteraktion (Angebot annehmen) [cite: 15-16], was den Vorgang zu einer "langlebigen Transaktion" macht.

## Entscheidung
Wir entscheiden uns für die Implementierung des **Saga-Patterns mit zentraler Orchestrierung**.

Ein dedizierter Service („Saga Service“) steuert den gesamten Ablauf der Vertragsänderung. Er ruft die beteiligten Services (Kalkulation, Bestand) sequenziell über Commands auf, verwaltet den Status des Antrags und wartet auf die notwendigen Rückmeldungen.

## Begründung
Diese Entscheidung basiert auf folgenden Faktoren:

1.  **Datenkonsistenz & Rollback:** Der Orchestrator garantiert, dass der Prozess einen definierten Endzustand erreicht. Schlägt ein Schritt fehl (z. B. das Legacy-Bestandssystem lehnt die Änderung ab), kennt der Orchestrator den aktuellen Zustand und kann Kompensationsmaßnahmen einleiten (z. B. Nutzer informieren, Status zurücksetzen).
2.  **Legacy-Integration:** Die Adapter (Strangler Facades) der Legacy-Systeme können so einfach gehalten werden. Sie müssen nur Commands ("Kalkuliere", "Aktualisiere") ausführen und das Ergebnis melden. Sie müssen keine komplexe Prozesslogik kennen.
3.  **State Management:** Da zwischen der Angebotslegung und der Annahme durch den Kunden Zeit vergeht, muss der Zustand persistiert werden. Der Orchestrator ist dafür prädestiniert.
4.  **Wartbarkeit:** Die Geschäftslogik (Reihenfolge: Erst Kalkulation, dann Angebot, dann Bestand) ist zentral an einem Ort definiert und nicht über viele Services verstreut.

## Alternativen

* **Choreografie (Event-Chain):** Services reagieren nur auf Events (Service A feuert Event -> Service B reagiert).
    * *Nicht gewählt, weil:* Der Prozessfluss wird unübersichtlich. Die Fehlerbehandlung (Komplexes Rollback über Legacy-Systeme hinweg) ist ohne zentralen Koordinator schwer zu überwachen.
* **Verteilter Monolith (Synchrone Aufrufe):** Portal ruft Service A, dieser ruft Service B, etc.
    * *Nicht gewählt, weil:* Es entsteht eine zu hohe Kopplung. Ein Timeout im Legacy-System würde den gesamten Thread im Portal blockieren.

## Konsequenzen

**Positiv:**
* Klare Trennung von Prozessfluss (Orchestrator) und fachlicher Ausführung (Services/Legacy).
* Der Status jeder Vertragsänderung ist jederzeit zentral abfragbar (wichtig für Support/Portal).
* Fehlgeschlagene Transaktionen können gezielt behandelt werden.

**Negativ:**
* Der Saga Orchestrator wird zu einem "Single Point of Failure" (erfordert Hochverfügbarkeit).
* Erhöhte Komplexität bei der Implementierung der Kompensationslogik (Was tun, wenn das Bestands-Update technisch fehlschlägt, nachdem der Kunde schon "Ja" gesagt hat?).
