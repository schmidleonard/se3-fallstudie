# Bearbeitung Fallstudie: Aufgabe 4

## Aufgabe 4.1 – Auswahl eines Architektur-Stils

### a. Entscheidung: Kombination aus (1) Event-Driven Architecture (EDA) & (2) Microservices / SCS

**Begründung (EDA):**

* **Hohe Integrationsanforderung:** Prozess erfordert 5+ Systeme (Portal, Produkt, Bestand, SAP, DWH).
* **Entkopplung:** EDA entkoppelt diese Systeme ideal.
* **Asynchronität:** Systeme reagieren *asynchron* auf Events (z.B. "VertragGeändert"). SAP (Abrechnung) und DWH (Reporting) müssen nicht direkt/synchron gerufen werden.
* **Erfüllt QAs:** Steigert **Integrationsfähigkeit** und **Änderbarkeit**.

**Begründung (Microservices / SCS):**

* **Separierbare Fachlichkeit:** Prozess hat klare, abgrenzbare Domänen:
    1.  Antragsprüfung (Sperren, Kündigungsstatus).
    2.  Kalkulation (Tarifregeln, Prämien).
    3.  Bestandsführung (Policierung, Archivierung).
* **Kapselung:** Altsysteme (`Produktsystem`, `Bestandssystem`) werden durch Services gekapselt.
* **Erfüllt QAs:**
    * **Änderbarkeit:** Tarifregeln im `Kalkulations-Service` änderbar, ohne `Bestands-Service` zu beeinflussen.
    * **Skalierbarkeit:** Rechenintensive Kalkulation kann unabhängig skaliert werden.

---

## Aufgabe 4.2 – Auswahl von Architekturmustern

### a. Entscheidung: (1) Saga Pattern & (2) Strangler (Fig) Pattern

**Begründung (Saga Pattern):**

* **Verteilte Transaktion:** Der Prozess ist eine fachliche Transaktion über mehrere Services/Systeme (Antrag -> Kalkulation -> Bestand -> SAP).
* **Konsistenz:** Stellt die fachliche Korrektheit und Datenkonsistenz über alle Schritte sicher.
* **Fehlerbehandlung:** Ermöglicht Kompensationsaktionen (Rollbacks), falls ein späterer Schritt fehlschlägt. Notwendig, da eine technische ACID-Transaktion unmöglich ist.

**Begründung (Strangler (Fig) Pattern):**

* **Migrationsanforderung:** Expliziter Hinweis auf Migration des Alt-Systems.
* **Risikoarme Migration:** Erlaubt die schrittweise Ablösung der Legacy-Systeme (`Produktsystem`, `Bestandssystem`).
* **Umsetzung:**
    1.  Neue Microservices (`Kalkulations-Service`) werden als Fassade *vor* die Altsysteme gebaut.
    2.  Zunächst "proxied" der Service Anfragen an das Altsystem.
    3.  Logik wird schrittweise aus dem Altsystem in den neuen Service extrahiert.

---

## Aufgabe 4.3 – Darstellung und Begründung

### a. Architekturübersichtsdiagramm


---

### b. Begründung der Eignung (Fachlich & Technisch)

**Fachliche Passung:**

* Prozess ist naturgemäß *asynchron* (z.B. Warten auf Kundenbestätigung) und *verteilt* (verschiedene Systeme/Abteilungen).
* EDA + Saga bilden diese Realität perfekt ab.

**Technische Passung (Erfüllung der Qualitätsziele):**

* **Integrationsfähigkeit:** EDA als robustester Stil zur Kopplung heterogener Systeme (Legacy, SAP, DWH). Systeme müssen nur Events verstehen, nicht sich gegenseitig.
* **Änderbarkeit:** Komplexe Tariflogik ist im `Kalkulations-Service` gekapselt (via Strangler Pattern) und kann isoliert modernisiert werden.
* **Nachvollziehbarkeit:** Event-Log dient als revisionssicheres Protokoll aller Änderungen.

---

### c. Diskussion verworfener Alternativen

**Alternative 1: Monolith**

* **Ansatz:** Eine einzige, große Anwendung für den gesamten Prozess.
* **Verworfen, weil:**
    * Führt zu extremer Kopplung (Spaghetti-Architektur).
    * Integration von 5+ Systemen wäre starr und fehleranfällig.
    * Qualitätsziele (Änderbarkeit, Skalierbarkeit) werden massiv verletzt.

**Alternative 2: Synchrone Microservices (Reine REST-Orchestrierung)**

* **Ansatz:** Services rufen sich gegenseitig synchron via REST auf (Antrag -> ruft Kalkulation -> ruft Bestand -> ruft SAP...).
* **Verworfen, weil:**
    * **"Cascading Failures":** Fällt ein System (z.B. SAP) aus, schlägt die *gesamte* Kette fehl.
    * **Schlechte Performance:** Nutzer im Portal müsste warten, bis *alle* Systeme (inkl. DWH) geantwortet haben.

    * **Passt nicht:** Ignoriert die asynchrone Natur des fachlichen Prozesses.
