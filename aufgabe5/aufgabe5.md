# Bearbeitung Fallstudie: Aufgabe 5 (Kommunikationsstile)

## Aufgabe 5.1 - Analyse der Kommunikationsanforderungen

### a. Identifikation der Kommunikationspartner

Gemäß dem Geschäftsprozess und der Architektur müssen folgende Systeme kommunizieren:

**Neue Architektur (Interne Systeme/Services):**
* **Antrags-Service:** (Erste Prüfung der Zulässigkeit)
* **Kalkulations-Service:** (Strangler Facade für Prämienkalkulation)
* **Bestands-Service:** (Strangler Facade für Policen-Aktualisierung)
* **Saga Orchestrator Service:** (Steuert den übergreifenden Prozess)
* **Event Bus:** (Message Broker für die asynchrone Kopplung)
* **DWH-Batch-Adapter:** (Notwendig, da DWH per Annahme keine Events verarbeiten kann. Sammelt Events und startet nächtliche Batch-Läufe)
* **SAP-Adapter:** (Notwendig, um Events vom Bus in ein SAP-Format, z.B. IDoc, zu übersetzen)

**Legacy & Externe Systeme (und deren notwendige Adapter):**
* **Portal / Vertriebssystem:** (Frontend/Eingabemaske für Nutzer)
* **Produktsystem (Alt):** (Beinhaltet die Tariflogik)
* **Bestandssystem (Alt):** (Führt die Policen)
* **SAP (Abrechnung):** (Wird über Änderungen informiert)
* **DWH (Reporting):** (Empfängt Daten für Auswertungen)

---

### b. Analyse ausgewählter Schnittstellen

**1. Schnittstelle: Portal -> Antrags-Service** (Synchrone Antrags-Vorprüfung)

* **Datenvolumen:** Gering. (Ein JSON-Payload mit Vertrags-ID und gewünschter neuer Deckungssumme).
* **Latenzanforderungen:** Sehr niedrig (z.B. < 500ms). Synchron. Der Nutzer wartet aktiv auf die Rückmeldung, ob der Vertrag überhaupt änderbar ist.
* **Kommunikationshäufigkeit:** Mittel. Tritt bei jeder initiierten Vertragsänderung auf.
* **Flexibilität der Datenabfrage:** Gering. Die Anfrage ist statisch; es wird immer dieselbe Prüfung ("Ist dieser Vertrag änderbar?") durchgeführt.
* **Sicherheitsanforderungen:** Hoch. Muss sicherstellen, dass der authentifizierte Nutzer (Kunde/Vertriebler) nur seine eigenen bzw. ihm zugewiesenen Verträge ändern darf (Autorisierung).

**2. Schnittstelle: Kalkulations-Service -> Produktsystem (Alt)** (Synchrone Prämienkalkulation)

* **Datenvolumen:** Mittel. (Request: Vertragsdaten, alte/neue Summe. Response: Neue Prämie, Tarifdetails, ggf. neue Bausteine).
* **Latenzanforderungen:** Mittel (z.B. < 3 Sekunden). Das Altsystem ist wahrscheinlich langsam. Da die Gesamtanfrage (Saga) asynchron ist, blockiert dies nicht den Nutzer.
* **Kommunikationshäufigkeit:** Mittel. (Immer, wenn ein Antrag die Vorprüfung (1) bestanden hat).
* **Flexibilität der Datenabfrage:** Sehr Gering. Das Altsystem hat eine starre, bestehende Schnittstelle (z.B. SOAP oder Mainframe-Transaktion), die der (Strangler-)Adapter exakt bedienen muss.
* **Sicherheitsanforderungen:** Mittel. Die Kommunikation findet im internen, als vertrauenswürdig erachteten Netz statt. Eine technische Authentifizierung (z.B. mTLS oder fester Service-User) ist ausreichend.

**3. Schnittstelle: Event Bus -> DWH-Batch-Adapter** (Asynchrones Reporting)

* **Datenvolumen:** Sehr Hoch (aggregiert). Der Adapter sammelt potenziell tausende "VertragGeändert"-Events über den Tag.
* **Latenzanforderungen:** Sehr Hoch (Batch-fähig). Reporting-Daten sind (per Annahme) nicht zeitkritisch. Eine nächtliche Verarbeitung (Latenz bis 24h) ist akzeptabel.
* **Kommunikationshäufigkeit:** Sehr Gering (1x pro Nacht). Der Adapter konsumiert zwar ständig Events vom Bus, kommuniziert aber nur 1x (z.B. um 02:00 Uhr) mit dem eigentlichen DWH.
* **Flexibilität der Datenabfrage:** Keine. Dies ist eine "Push"-Schnittstelle. Der Adapter konsumiert die Events in einem festen Format und schreibt sie in eine Batch-Import-Datei (z.B. CSV).
* **Sicherheitsanforderungen:** Mittel. Der Adapter benötigt sichere Credentials für den Event Bus (Lesen) und den DWH-Import-Server (Schreiben).

---

## Aufgabe 5.2 – Auswahl und detaillierte Modellierung von Schnittstellen

Gemäß der Aufgabenstellung werden zwei unterschiedliche Schnittstellen modelliert, reduziert auf das Wesentliche.

### 1. Schnittstelle: Portal -> Antrags-Service (Stil: REST)

**Begründung:** REST (mit JSON) ist der Standard für die Kommunikation zwischen Web-Frontends und Backend-Services. Es ist flexibel und nutzt etablierte Web-Technologien.

**Definitionen:**

* **Ressource:** `/vertragsaenderungsantraege`
* **Methode:** `POST /vertragsaenderungsantraege`
* **Zweck:** Stößt die Vorprüfung für eine Vertragsänderung an. Da der Gesamtprozess asynchron ist, erstellt dieser Aufruf nur den *Antrag* und gibt eine `202 (Accepted)` Antwort zurück.

**JSON-Datenformat (Skizze):**

**Request:** `POST /vertragsaenderungsantraege`
```json
{
  "vertragsnummer": "H-456.789.001",
  "aenderungsart": "DECKUNGSSUMME_ERHOEHEN",
  "gewuenschteDeckungssumme": 100000000.00
}
```
**Response (202 Accepted):** (Antrag zur Prüfung angenommen)
```json
{
  "antragsId": "A-2025-1A9F4",
  "status": "IN_PRUEFUNG",
  "message": "Antrag zur Vorprüfung angenommen. Status wird aktualisiert."
}
```
**Response (422 Unprocessable Entity):** (z.B. Vertrag gekündigt)
```json
{ 
	"vertragsnummer": "H-456.789.001", 
	"fehlercode": "VERTRAG_GEKUENDIGT", 
	"message": "Vertragsänderung nicht möglich: Vertrag hat Status 'GEKUENDIGT'." 
}
```
### 2. Schnittstelle: Saga Orchestrator -> Kalkulations-Service (Stil: gRPC)

**Begründung:** gRPC ist performanter als REST und erzwingt durch die `.proto`-Definition einen klaren "Contract-First"-Ansatz, was dem Befehlscharakter ("Command") einer Saga-Orchestrierung entspricht.

**Definitionen (`kalkulation.proto`-Datei):**
``` protobuf
syntax = "proto3";

// Definition des Service-Pakets
package kalkulation;

// Der Service, den der Saga Orchestrator aufruft
service KalkulationsService {
  // RPC-Methode, um die Prämie basierend auf dem Antrag zu kalkulieren
  rpc BerechnePraemie (KalkulationsAnfrage) returns (KalkulationsAntwort);
}

// Request-Struktur
message KalkulationsAnfrage {
  string antragsId = 1;         // Zur Nachverfolgung
  string vertragsnummer = 2;
  double alteDeckungssumme = 3;
  double neueDeckungssumme = 4; // Die gewünschte Summe
}

// Response-Struktur
message KalkulationsAntwort {
  string antragsId = 1;
  bool fachlichZulaessig = 2; // Prüfung der Tarifregeln
  
  // Optionale Felder, je nach Ergebnis
  oneof ergebnis {
    PraemieDetails positiveAntwort = 3;
    Ablehnungsgrund negativeAntwort = 4;
  }
}

// Details bei positiver Prüfung
message PraemieDetails {
  double neuePraemie = 1;
  string hinweisText = 2; // z.B. Info über neue Wartezeiten
}

// Details bei negativer Prüfung
message Ablehnungsgrund {
  string grundCode = 1; // z.B. "MAX_SUMME_UEBERSCHRITTEN"
  string begruendung = 2; // "Die Deckungssumme übersteigt das Maximum."
}
```
