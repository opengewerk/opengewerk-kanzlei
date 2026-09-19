# Änderungsprotokoll

Alle nennenswerten Änderungen an diesem Projekt werden in dieser Datei festgehalten.

Das Format orientiert sich an [Keep a Changelog](https://keepachangelog.com/de/1.1.0/),
die Versionsnummern folgen der [Semantischen Versionierung](https://semver.org/lang/de/).

## [Unreleased]

### Hinzugefügt

- Initiales Repository-Gerüst
- CI-Job "Schreibweise", der Gedankenstriche im gesamten Repository meldet

### Geändert

- Das Planungskonzept sagt die Bindung des Tokens an die Hub-Instanz nicht mehr für die
  erste Fassung zu. ADR 0006 hat am 18.09.2026 anders entschieden: rotierende
  Bearer-Token in Phase 3, mTLS oder DPoP danach. Die Zusage stand an zwei Stellen, eine
  davon mit dem Zeichen für berufsrechtliche Relevanz, und die Richtigstellung stand
  vierzig Zeilen weiter unten in einem anderen Abschnitt. Jetzt steht sie an den beiden
  Stellen selbst, und die Kopie darunter ist weg

- Die Workflow-Dateien folgen der Regel "Code ist immer Englisch": Job-Kennungen,
  Variablen und Kommentare in den eingebetteten Skripten sind englisch. Deutsch bleibt,
  was ein Mensch liest, also die Job- und Schrittnamen in der Actions-Oberfläche und die
  Meldungen, die eine Prüfung ausgibt
- CodeQL ermittelt die zu prüfenden Sprachen aus dem Dateibestand, statt sie in einer
  Liste zu führen. Dort stand bisher nur `actions`, mit einer Notiz, sie beim ersten
  Code zu ergänzen. Wer den ersten TypeScript-Code einspielt, denkt aber nicht an diese
  Datei und hätte danach ein Scanning, das nichts scannt
- Planungskonzept auf v1.2: Leitentscheidung 7 zum gemeinsamen Tech-Stack, neuer
  Abschnitt 3.3a mit der Regel-Engine für steuerliche Parameter, Stand der
  Spezifikation und Token-Eigenschaften nach ADR 0006 nachgezogen, Verzahnung mit
  der neu geschnittenen Roadmap der Handwerkersoftware ausgeschrieben
- Tech-Stack und Lizenz sind keine offenen Entscheidungen mehr
