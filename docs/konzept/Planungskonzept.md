# OpenGewerk Kanzlei: Planungskonzept (Kanzlei-Hub für Steuerberater) · v1.2

2026-09-17 · Eigenständiges Projekt, Repository `opengewerk-kanzlei` in der GitHub-Organisation `opengewerk` · v1.1 trägt den Projektnamen ein · v1.2 (18.09.2026) zieht den entschiedenen Tech-Stack, die Regel-Engine und den Stand der Spezifikation nach

Zentrales, self-hosted System für Steuerberaterkanzleien, das beliebig viele Mandanten-Instanzen der Handwerkersoftware anbindet. Die Kanzlei arbeitet aus einer Anwendung heraus; die Daten bleiben beim Mandanten.

**Legende**
- ★ = eigene Idee, über vergleichbare Lösungen (DATEV Unternehmen online / Kanzlei-Rechnungswesen, sevdesk-Steuerberaterzugang, Lexware-Office-Steuerberatermodus, Agenda, Addison) hinaus
- ⚖ = rechtlich/berufsrechtlich erforderlich
- ⏳ = im Plan, bewusst später

---

## 0. Leitentscheidungen

1. **Föderation statt Zentralisierung.** Der Hub speichert keine Buchhaltung. Er greift per API live auf die Mandanten-Instanzen zu und hält nur einen Cache für Übersichten. Datenhoheit bleibt beim Mandanten, das ist das Verkaufsargument gegenüber DATEV Unternehmen online.
2. **Read-only als Default.** Schreibende Rechte (Buchungsvorschläge, Kontenzuordnungen, Rückfragen) werden vom Mandanten explizit pro Scope freigegeben und sind jederzeit widerrufbar.
3. **Der Mandant lädt ein, nicht die Kanzlei.** Verbindungsaufbau immer vom Mandantensystem aus (Einladungscode), damit kein Mandant ohne sein Wissen angebunden werden kann.
4. **Eigenes Repository (`opengewerk-kanzlei`), eigene Releases**, aber ein gemeinsam versioniertes API-Vertrags-Paket (`opengewerk-api-spec`), damit Hub und Handwerkersoftware kompatibel bleiben.
5. **Berufsrecht zuerst.** Verschwiegenheit (§203 StGB, §57 StBerG), Mandantentrennung und Zugriffsprotokollierung sind Architekturanforderungen, keine Features.
6. **Offen für weitere Quellen.** Die Handwerkersoftware OpenGewerk (Repo `opengewerk`) ist die erste Quelle; die Adapter-Schicht erlaubt später andere Systeme (sevdesk-API, Lexware-Office-API, CSV-Uploads) ⏳.
7. **Derselbe Tech-Stack wie die Handwerkersoftware.** Entschieden am 18.09.2026 in den ADRs 0002 bis 0008 des Repositories `opengewerk`: TypeScript mit NestJS, PostgreSQL mit Row-Level Security und Drizzle, React mit Vite. Die ADRs gelten ausdrücklich für beide Anwendungen, damit Domänenpakete, Auth-Muster und Betriebsweise geteilt werden können. Ein eigener Stack für den Hub hätte zwei Ökosysteme für eine Person bedeutet.

---

## 1. Zielgruppe & Nutzungsszenarien

**Nutzer**
- Steuerberater/-in (Kanzleiinhaber, Berufsträger)
- Steuerfachangestellte / Buchhaltungsmitarbeiter (bearbeiten einzelne Mandanten)
- Kanzlei-Admin (Verbindungen, Rollen, Backup)
- Prüfer/Revisor (temporär, read-only, z. B. Wirtschaftsprüfer, Betriebsprüfer mit Datenzugriff)

**Kernszenarien**
1. Montagmorgen: Welche der 40 Mandanten haben unverbuchte Belege, offene Rückfragen, fehlende Bankumsätze, anstehende USt-VA?
2. Monatsabschluss eines Mandanten: Belege prüfen, Kontierung korrigieren, Rückfragen stellen, USt-VA-Werte abnehmen, Export erzeugen.
3. Jahresabschluss: Anlagenverzeichnis prüfen, Abgrenzungen buchen (Vorschlag), Checkliste abarbeiten, Bilanzwerte übernehmen.
4. Betriebsprüfung: Z1-Z3-Export eines Mandanten aus dem Hub anstoßen, Prüferzugang zeitlich befristet einrichten.
5. Onboarding: Neuer Mandant schickt Einladungscode; Kanzlei richtet Kontenrahmen-Profil und Fristen ein.

---

## 2. Architektur

### 2.1 Komponenten

```
┌────────────────────────────┐        ┌────────────────────────────┐
│ Kanzlei-Hub (self-hosted)  │        │ Mandant A: Handwerkersoftw.│
│  · Web-App (Kanzlei-UI)    │◄──────►│  · Kanzlei-Connector-API    │
│  · Föderations-Gateway     │ HTTPS  │  · Scoped Token, Webhooks   │
│  · Adapter (HWS, sevdesk…) │        └────────────────────────────┘
│  · Cache / Index (Postgres)│        ┌────────────────────────────┐
│  · Fristen-Engine (Kanzlei)│◄──────►│ Mandant B: Handwerkersoftw.│
│  · Aufgaben/Rückfragen     │        └────────────────────────────┘
│  · Audit-Log               │        ┌────────────────────────────┐
│  · Export-Service (DATEV…) │◄──────►│ Mandant C: andere Quelle ⏳│
└────────────────────────────┘        └────────────────────────────┘
```

### 2.2 Föderations-Gateway

- Ein Adapter pro Quellsystem; der HWS-Adapter spricht die `opengewerk-api-spec`
- Verbindungsdaten pro Mandant: Basis-URL, Token (verschlüsselt im Hub, KMS/Age/libsodium), Scopes, Status, letzte Erreichbarkeit
- Live-Abfragen für Detailansichten; periodischer Sync (konfigurierbar, z. B. stündlich) für Übersichts-Kennzahlen in den Cache
- Webhooks vom Mandanten → Hub (neuer Beleg, Rückfrage beantwortet, Periode festgeschrieben) für Echtzeit-Aktualisierung
- Ausfallverhalten: Mandant nicht erreichbar → Übersicht zeigt letzten Stand mit Zeitstempel und Warnung; keine Blockade der anderen Mandanten

### 2.3 Cache-Regeln

- Gecacht werden nur Aggregate und Indexdaten (Anzahl offener Belege, Salden, Fristen, Belegköpfe)
- **Belegbilder und Buchungsdetails werden nie persistent im Hub gespeichert**: nur im Speicher/temporär zur Anzeige (Datenhoheit + Verschwiegenheit)
- Cache je Mandant löschbar; wird bei Verbindungstrennung automatisch geleert

### 2.4 Verbindungsaufbau (Handshake)

1. Mandant erzeugt in seiner Handwerkersoftware unter *Einstellungen → Steuerberater* einen Einladungscode (einmalig, 24 h gültig) und wählt die freizugebenden Scopes
2. Kanzlei gibt Code + Mandanten-URL im Hub ein
3. Hub tauscht Code gegen langlebigen, rotierbaren Token (OAuth 2.0 Device-/Authorization-Code-ähnlich); Token ist an Hub-Instanz gebunden (mTLS-Zertifikat oder DPoP) ⚖
4. Mandant sieht in seiner Instanz: verbundene Kanzlei, Scopes, letzte Zugriffe; er kann die Verbindung jederzeit trennen

### 2.5 Scopes (vom Mandanten vergeben)

| Scope | Inhalt | Default |
| --- | --- | --- |
| `read:ledger` | Journal, Konten, Salden, OP-Listen | ja |
| `read:documents` | Belegbilder, E-Rechnungs-XML | ja |
| `read:master` | Stammdaten Kunden/Lieferanten/Anlagen | ja |
| `read:periods` | Festschreibungsstatus, USt-VA-Werte | ja |
| `write:comments` | Rückfragen und Kommentare an Belegen | ja |
| `write:proposals` | Buchungs-/Kontierungsvorschläge (Mandant bestätigt) | optional |
| `write:coa` | Kontenrahmen-Profil, Automatikkonten pushen | optional |
| `write:closing` | Abschlussbuchungen direkt buchen | optional, nur Berufsträger |
| `export:audit` | Z1-Z3/GDPdU-Export auslösen | optional |

---

## 3. Funktionsumfang

### 3.1 Mandantenübersicht (Kanzlei-Dashboard)

- Liste aller Mandanten mit Status-Ampel: Verbindung, unverbuchte Belege, offene Rückfragen, unabgeglichene Bankumsätze, nächste Frist, letzte Festschreibung
- **Buchhaltungs-Gesundheitsindex je Mandant ★**: gewichtete Kennzahl aus Belegrückstand, offenen Rückfragen, fehlenden Belegen (Bankumsatz ohne Beleg), überfälligen Fristen, sortierbar, damit die Kanzlei die richtigen Mandanten zuerst bearbeitet
- Filter: Sachbearbeiter, Mandantengruppe, Rechtsform, Fristtyp
- Zuständigkeiten: Sachbearbeiter je Mandant (Vertretungsregel)
- Mandantengruppen (z. B. "Handwerk Rhein-Neckar", "Bilanzierer", "EÜR")

### 3.2 Mandanten-Arbeitsplatz

- Wechsel in einen Mandanten ohne neuen Login; Mandantenkontext immer sichtbar (Farbcode, Name) ⚖; das verhindert Verwechslungen
- Belegprüfung: Belegbild + Buchungsvorschlag nebeneinander, Kontierung korrigieren, Steuer-Schlüssel prüfen, Freigabe
- Journal, Kontenblätter, Saldenlisten, OP-Listen, BWA, USt-VA-Vorschau, live aus dem Mandantensystem
- Rückfragen an den Mandanten direkt am Beleg ("Was war das für ein Kauf?") mit Fälligkeit; Mandant antwortet in seiner Instanz oder App
- Fehlende-Belege-Liste: Bankumsätze ohne Beleg werden automatisch als Rückfrage vorgeschlagen ★
- Buchungsvorschläge (Scope `write:proposals`): Kanzlei schlägt vor, Mandant bestätigt mit einem Klick, oder die Kanzlei bucht direkt (Scope `write:closing`)
- Periodenfestschreibung anstoßen/anfordern

### 3.3 Fristen-Engine (Kanzleiebene)

Eigene Fristen-Engine im Hub (gleiche Grundidee wie in der Handwerkersoftware), gespeist aus Mandantendaten und Kanzleiregeln:

- USt-VA (monatlich/vierteljährlich, Dauerfristverlängerung), Zusammenfassende Meldung, Jahressteuererklärungen, Jahresabschluss-/Offenlegungsfristen, Lohnsteuer-Anmeldung
- Mandantenspezifische Fristen aus der Handwerkersoftware (z. B. Sicherheitseinbehalt-Auszahlung, Freistellungsbescheinigung §48 EStG läuft ab)
- Kanzlei-Fristenkalender über alle Mandanten, Ampel, Zuweisung an Sachbearbeiter, Erinnerung
- ELSTER-Abgabe erfolgt weiterhin aus der Kanzleisoftware/ELSTER, der Hub liefert die Werte und dokumentiert "abgegeben am" ⏳ (Direktübermittlung wie in der HWS bewusst ausgeklammert)
- Die Fristen selbst werden nicht im Code berechnet, sondern aus Regeldatensätzen abgeleitet, siehe 3.3a

### 3.3a Regel-Engine für steuerliche Parameter ★ ⚖

Gegenstück zu Abschnitt 1.7 der Feature-Gliederung, auf Kanzleiebene. Die Fristen- und Schwellenwerte des Steuerrechts stehen **nicht im Code**, sondern in versionierten Regeldatensätzen mit Gültigkeitszeitraum, Fundstelle und, wo nötig, Mandantenbezug:

- Abgabefristen: USt-Voranmeldung monatlich oder vierteljährlich, Dauerfristverlängerung mit Sondervorauszahlung, Zusammenfassende Meldung, Lohnsteuer-Anmeldung
- Schwellen, die den Rhythmus bestimmen: Grenzen für den Voranmeldungszeitraum, Kleinunternehmergrenze §19 UStG, Buchführungspflicht §141 AO
- Jahresfristen: Steuererklärungen mit und ohne Steuerberater, Offenlegung, Fristverlängerungen
- Aufbewahrung und Löschung: handels- und steuerrechtliche Fristen, DSGVO-Löschfristen für Hub-Daten

Drei Eigenschaften, die dieselben sind wie beim Mandanten:

1. **Ein Gesetzesupdate ist ein neuer Regeldatensatz mit Gültigkeitsbeginn, kein Release.** Eine Kanzlei kann nicht auf das nächste Softwareupdate warten, wenn sich zum Jahreswechsel eine Frist verschiebt.
2. **Regeln werden historisch angewendet.** Eine Voranmeldung für 2027 wird auch 2030 nach den Regeln von 2027 beurteilt, sonst stimmen Nachschauen nicht mehr.
3. **Der Anwender überschreibt keine Regeln**, er setzt nur mandantenbezogene Parameter, etwa Voranmeldungszeitraum, Dauerfristverlängerung ja oder nein, Kleinunternehmer ja oder nein.

Die Regelpakete des Hubs sind eigene Pakete und nicht dieselben wie beim Mandanten: Der Mandant braucht Regeln zum Rechnungstellen, die Kanzlei Regeln zum Abgeben. Gemeinsam ist das Format, damit die Werkzeuge zum Pflegen und Prüfen für beide Seiten dieselben sind.

### 3.4 Aufgaben & Kommunikation

- Kanzleiinterne Aufgaben (Mandant, Fälligkeit, Sachbearbeiter, Status)
- Rückfragen-Postfach über alle Mandanten (offen / beantwortet / erledigt)
- Nachrichtenkanal Kanzlei ↔ Mandant innerhalb der Systeme (kein E-Mail-Versand von Belegdaten) ⚖
- Wiederkehrende Aufgaben (Monatsabschluss-Checkliste je Mandant, automatisch erzeugt)

### 3.5 Abschlussarbeiten

- Jahresabschluss-Checkliste je Mandant (EÜR- oder Bilanz-Variante): Anlagenverzeichnis, AfA, Abgrenzungen, Rückstellungen, Forderungsbewertung, USt-Verprobung
- USt-Verprobung: Umsätze laut Journal vs. gemeldete USt-VA-Werte, Differenzen markiert ★
- Kontenrahmen-Profile: Kanzlei pflegt Standard-Kontierungen (Automatikkonten, Steuerschlüssel) einmal und pusht sie an alle Handwerks-Mandanten (Scope `write:coa`) ★
- Abschlussbuchungen als Vorschlagspaket an den Mandanten oder direkt gebucht

### 3.6 Exporte & Schnittstellen

- DATEV-Buchungsstapel und Belegbilder je Mandant oder gesammelt für einen Zeitraum
- Sammelexport für mehrere Mandanten in einem Lauf ★
- Betriebsprüfung: Z1-Z3/GDPdU-Export auslösen; Prüferzugang (read-only, befristet, protokolliert) ⚖
- Export zu Kanzleisoftware (DATEV Kanzlei-Rechnungswesen, Agenda, Addison) über deren Importformate ⏳
- Offene REST-API des Hubs (z. B. für Kanzlei-eigene Auswertungen)

### 3.7 Auswertungen (Kanzlei)

- Bearbeitungsstand je Sachbearbeiter, Rückstände, Durchlaufzeiten von Rückfragen
- Mandanten-Benchmark (anonymisiert, nur mit Einwilligung): Rohertrag, Materialquote, Zahlungsmoral im Handwerksvergleich ★ ⏳
- Kanzlei-Kapazitätsplanung nach Fristenlage

### 3.8 Hilfe & Dokumentation

- Kontextsensitive Hilfe, Kanzlei-Wissensdatenbank (wie in der Handwerkersoftware)
- Onboarding-Anleitung für Mandanten ("So verbinden Sie Ihre Kanzlei"), öffentlich, aus Mandantensicht
- Administratorhandbuch (Installation, Backup, Token-Rotation)

---

## 4. Rollen & Rechte (Kanzlei)

| Rolle | Rechte |
| --- | --- |
| Kanzlei-Admin | Verbindungen, Nutzer, Rollen, Backup, Audit-Log; kein Standard-Fachzugriff |
| Berufsträger | Alle Mandanten der Kanzlei, alle Scopes inkl. `write:closing` |
| Sachbearbeiter | Zugewiesene Mandanten, Prüfung/Rückfragen/Vorschläge; keine Abschlussbuchungen |
| Vertretung | Zeitlich befristete Übernahme der Mandanten eines Sachbearbeiters |
| Prüfer | Ein Mandant, read-only, befristet, jeder Zugriff protokolliert |

- Zwei-Faktor-Authentifizierung Pflicht für alle Kanzleinutzer ⚖
- SSO optional (OIDC), z. B. bestehender Kanzlei-IdP
- Mandantentrennung auf Datenbank- und UI-Ebene; Sachbearbeiter sehen keine anderen Mandanten in Listen, Suche oder Exporten

---

## 5. Sicherheit & Berufsrecht ⚖

- **Verschwiegenheit** (§203 StGB, §57 StBerG): Hub steht in der Kanzlei oder bei einem Hoster mit Auftragsverarbeitungsvertrag; Belegdaten werden nicht persistent im Hub abgelegt; Nachrichten nur innerhalb des Systems
- **Zugriffsprotokoll**: jeder Mandantenzugriff (wer, wann, welcher Beleg/Report) wird im Hub **und** im Mandantensystem protokolliert; Mandant kann sein Zugriffslog jederzeit einsehen ★
- **Token-Sicherheit**: verschlüsselte Ablage, Rotation, Bindung an Hub-Instanz (mTLS/DPoP), automatischer Ablauf bei Inaktivität, Sofort-Sperre durch Mandant
- **Transport**: TLS 1.3, optional WireGuard/Tailscale-Tunnel zwischen Kanzlei und Mandanten (Empfehlung im Admin-Handbuch)
- **DSGVO**: Kanzlei ist Verantwortlicher für ihre Nutzerdaten; für Mandantendaten bleibt der Mandant Verantwortlicher; AV-Vertrag nur bei externem Hub-Hosting; Verarbeitungsverzeichnis für den Hub generiert
- **GoBD**: der Hub verändert keine festgeschriebenen Daten; alle schreibenden Aktionen laufen als Vorschlag/Buchung im Mandantensystem und dessen Journal
- **Notfall**: Backup des Hubs enthält keine Belegdaten → Wiederherstellung ist unkritisch; Verbindungen werden bei Restore neu bestätigt

---

## 6. Auswirkungen auf die Handwerkersoftware

Damit der Hub funktioniert, braucht die Handwerkersoftware (Hauptplan v2) folgende Ergänzungen:

- **Kanzlei-Connector-Modul**: Einstellungsseite *Steuerberater*, Einladungscode, Scope-Auswahl, verbundene Kanzleien, Zugriffslog, Trennen-Button
- **`opengewerk-api-spec` implementieren**: Endpunkte für Journal, Konten, Salden, OP, Belege (Bild + XML), Perioden, Rückfragen, Buchungsvorschläge, Kontenrahmen-Profil, Audit-Export
- **Webhooks**: neuer Beleg, Beleg geändert, Rückfrage beantwortet, Periode festgeschrieben
- **Rückfragen-Postfach** in der Mandanten-App (mobil beantwortbar mit Foto/Kommentar)
- **Vorschlags-Freigabe**: Buchungs-/Kontierungsvorschläge der Kanzlei mit einem Klick bestätigen
- Die bereits geplante **Steuerberater-Rolle** (read-only Login) bleibt als Fallback für Kanzleien ohne Hub erhalten

---

## 7. Schnittstellenvertrag `opengewerk-api-spec` (Eckpunkte)

- Eigenes Repo mit OpenAPI-Definition, JSON-Schemas und Konformitätstests; SemVer; Hub und HWS deklarieren unterstützte Versionen
- Stand 18.09.2026: Die Spezifikation steht auf 0.2.0. Pfade, Methoden und Scopes sind festgelegt, die Scopes seit 0.2.0 maschinenlesbar als OAuth-2.0-Schema mit dem Flow `clientCredentials`. Die Nutzlasten sind noch Platzhalter, die JSON-Schemas und die Konformitätstests kommen als Nächstes.
- Der Token trägt in der ersten Fassung keine kryptografische Bindung an die Hub-Instanz. Er rotiert, läuft bei Inaktivität ab und ist sofort widerrufbar; DPoP oder mTLS kommen später, siehe ADR 0006 im Repo `opengewerk`.
- Ressourcen: `/periods`, `/journal`, `/accounts`, `/balances`, `/open-items`, `/documents/{id}` (Bild, XML), `/inquiries`, `/proposals`, `/coa-profile`, `/audit-export`, `/access-log`
- Paginierung, ETags/If-None-Match für effizienten Sync, Idempotenz-Keys bei schreibenden Aufrufen
- Alle Beträge als Integer-Cent, Datumsangaben ISO 8601, Steuerschlüssel nach DATEV-Konvention (für den späteren Export)

---

## 8. Plattform & Betrieb

- Webanwendung (Desktop-fokussiert; Kanzleiarbeit ist Bildschirmarbeit), responsive für Tablet
- Self-hosted: Docker-Compose-Referenz, Postgres, Reverse Proxy; Update-Mechanismus mit Migrationen
- Backup/Restore (ohne Belegdaten, siehe 5.)
- Health-Dashboard: Erreichbarkeit aller Mandanten, Token-Ablauf, Sync-Fehler
- Mehrere Kanzleien auf einer Hub-Instanz bewusst **nicht** vorgesehen (eine Kanzlei = eine Instanz), um die Mandantentrennung einfach zu halten; Kanzleiverbünde ⏳

---

## 9. Roadmap

| Phase | Inhalt | Ergebnis |
| --- | --- | --- |
| 0: Vertrag | `opengewerk-api-spec` v1, Connector-Modul in der HWS, Handshake, Scopes, Audit-Log beidseitig | Verbindung steht, read-only |
| 1: Übersicht | Mandantenliste, Status-Ampel, Gesundheitsindex, Cache/Sync, Webhooks | Kanzlei sieht alle Mandanten auf einen Blick |
| 2: Arbeitsplatz | Belegprüfung, Journal/Konten/OP live, Rückfragen, Fehlende-Belege-Liste | Monatsarbeit aus dem Hub |
| 3: Fristen & Aufgaben | Kanzlei-Fristen-Engine, Aufgaben, Checklisten, Sachbearbeiter-Zuweisung | Kanzleisteuerung |
| 4: Schreiben & Export | Buchungsvorschläge, Kontenrahmen-Profile, DATEV-Sammelexport, Prüfer-Zugang, Z1-Z3 | Vollständiger Buchhaltungsprozess |
| 5: Abschluss & Erweiterung | Jahresabschluss-Checkliste, USt-Verprobung, Abschlussbuchungen, weitere Adapter (sevdesk/Lexware/CSV), Benchmark | Vollausbau |

**Der Hub ist kein paralleler Strang.** Seine Phase 0 verlangt `opengewerk-api-spec` v1 und das Connector-Modul im Mandantensystem, und das ist Inhalt von Phase 3 der Handwerkersoftware. Deren Roadmap wurde mit v2.3 auf ein MVP umgeschnitten: Phase 1 ist der Pilotbetrieb, Phase 3 die Buchhaltung. Vor Phase 3 der Handwerkersoftware gibt es hier nichts zu bauen, was nicht ins Leere liefe. Was sich sinnvoll vorziehen lässt, ist allein der Vertrag.

---

## 10. Abgrenzung zu bestehenden Lösungen

| Bestehende Lösung | Schwäche | Kanzlei-Hub |
| --- | --- | --- |
| DATEV Unternehmen online | Daten liegen zentral bei DATEV, laufende Kosten je Mandant, Handwerksprozesse fehlen | Föderiert, self-hosted, Handwerkersoftware liefert Fachkontext (Projekt, Anlage, Regiebericht am Beleg) |
| sevdesk / Lexware Steuerberaterzugang | Ein Login pro Mandant, keine Kanzleiübersicht, keine Fristen über Mandanten hinweg | Eine Anwendung, alle Mandanten, Gesundheitsindex, Fristenkalender |
| Kanzleisoftware (Agenda, Addison) | Import-zentriert, Rückfragen per Mail/Telefon | Rückfragen am Beleg, Vorschläge mit Ein-Klick-Freigabe, Webhooks |
| Lesezeichen-Ordner mit 40 Mandanten-Links | Unübersichtlich, unsicher, keine Statusinfo | Zentrales Dashboard mit Verbindungs- und Bearbeitungsstatus |

---

## 11. Bewusst ausgeklammert / später

- ELSTER-Direktübermittlung aus dem Hub (Werteübergabe an die Kanzleisoftware reicht) ⏳
- Lohnbuchhaltung (bleibt in der Kanzleisoftware; Hub zeigt nur Lohnexport-Status) ⏳
- Mehrkanzlei-Instanzen / Kanzleiverbünde ⏳
- Adapter für Fremdsysteme (sevdesk, Lexware Office, CSV) ⏳, erst nach stabiler `opengewerk-api-spec` v1
- Mandanten-Benchmark ⏳, nur mit Einwilligung und ausreichender Mandantenzahl
- Native Apps: Kanzleiarbeit ist Desktop; Mandanten antworten über die Handwerkersoftware-App

---

## 12. Änderungsprotokoll

### v1.1 → v1.2

- Neu: Leitentscheidung 7, derselbe Tech-Stack wie die Handwerkersoftware, entschieden in den ADRs 0002 bis 0008
- Neu: 3.3a Regel-Engine für steuerliche Parameter, Gegenstück zu Abschnitt 1.7 der Feature-Gliederung
- Präzisiert: Abschnitt 7 nennt den Stand der Spezifikation (0.2.0, Scopes maschinenlesbar, Nutzlasten offen) und die Token-Eigenschaften nach ADR 0006
- Präzisiert: Abschnitt 9 sagt ausdrücklich, dass der Hub hinter Phase 3 der Handwerkersoftware hängt, deren Roadmap mit v2.3 neu geschnitten wurde
- Erledigt: Tech-Stack und Lizenz sind keine offenen Entscheidungen mehr

### v1 → v1.1

- Produktname **OpenGewerk Kanzlei** und Repo `opengewerk-kanzlei` eingetragen; Paketname `kanzlei-api-spec` durch `opengewerk-api-spec` ersetzt

## 13. Offene Entscheidungen

- Hosting-Empfehlung für Kanzleien ohne eigene IT (Referenz-Hoster mit AV-Vertrag vs. reine Anleitung)
- Ob der Hub auch für **Bürogemeinschaften/Buchhaltungsbüros** (nicht Steuerberater) freigegeben wird, berufsrechtliche Grenzen (§6 StBerG) beachten
- Ob Kanzleien ein kommerzielles Support-Modell brauchen. Die Lizenzfrage selbst ist entschieden: AGPL-3.0 wie die Handwerkersoftware, der Schnittstellenvertrag unter Apache-2.0

Zwei Punkte sind seit dem 18.09.2026 entschieden und stehen deshalb nicht mehr hier: der Tech-Stack (ADRs 0002 bis 0008, siehe Leitentscheidung 7) und die Lizenz.
