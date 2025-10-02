# Qualitätsziele

1. Sicherheit / Compliance
Gesetzliche Anforderungen: **DSGVO, Solvency II, BaFin** → Pflicht für Betriebserlaubnis.
Audit Trail notwendig → alle Änderungen müssen revisionssicher nachvollziehbar sein.
Sensible Kundendaten (z. B. Verträge, Schadensmeldungen) erfordern hohen Zugriffsschutz.
Ohne Einhaltung keine Zulassung → Projekt scheitert bei Verstößen.

---

2. Zuverlässigkeit / Verfügbarkeit
Bestandssystem ist **geschäftskritisch**: ohne Verfügbarkeit Stillstand im Tagesgeschäft.
Arbeitszeiten 8–18 Uhr = Kernzeiten mit hoher Nutzung (Fachbereich fordert Hochverfügbarkeit).
Batchläufe müssen automatisiert laufen, manuelle Überwachung soll entfallen (Dr. Adams).
Wiederherstellbarkeit im Störfall entscheidend, da Kundendaten und Verträge sensibel sind.

---

3. Usability / UX
Fachbereich arbeitet **täglich** mit dem System → hohe Relevanz für Akzeptanz und Produktivität.
Neue Mitarbeiter:innen sollen innerhalb 1 Woche die Hauptprozesse bedienen können (Dr. Adams im Interview).
Reduzierte Schulungszeiten senken Kosten (bisher 9 Monate Einarbeitung → viel zu hoch).
Fehlertoleranz nötig, damit Prozesse bei Eingabefehlern nicht abbrechen
(UX für Externe)

---

4. Performance / Skalierbarkeit
Verarbeitung von Millionen Verträgen (z. B. Beitragsläufe, Prämienberechnung).
Fachbereich fordert kurze Ladezeiten (Adressänderung, Vertragsdaten → max. 2–3 Sekunden).
Schadenmeldungen bei Unwetter erzeugen Spitzenlasten, System muss diese abfangen.
Zukünftig zusätzliche Sparten (Hausrat, Unfall) → wachsende Daten- und Nutzerlast.

---

5. Wartbarkeit / Erweiterbarkeit
Modernisierung langfristig → weitere Sparten sollen integriert werden.
Budget limitiert → Architektur muss schrittweise erweiterbar sein.
Legacy-Probleme (monolithisch, schlecht dokumentiert) sollen künftig vermieden werden.
Modularität und klare Trennung (z. B. Domain-Driven Design) erleichtern spätere Erweiterungen

---