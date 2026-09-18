# Änderungsprotokoll

Alle nennenswerten Änderungen an diesem Projekt werden in dieser Datei festgehalten.

Das Format orientiert sich an [Keep a Changelog](https://keepachangelog.com/de/1.1.0/),
die Versionsnummern folgen der [Semantischen Versionierung](https://semver.org/lang/de/).

## [Unreleased]

### Hinzugefügt

- Initiales Repository-Gerüst
- CI-Job "Schreibweise", der Gedankenstriche im gesamten Repository meldet

### Geändert

- CodeQL ermittelt die zu prüfenden Sprachen aus dem Dateibestand, statt sie in einer
  Liste zu führen. Dort stand bisher nur `actions`, mit einer Notiz, sie beim ersten
  Code zu ergänzen. Wer den ersten TypeScript-Code einspielt, denkt aber nicht an diese
  Datei und hätte danach ein Scanning, das nichts scannt
- Planungskonzept auf v1.2: Leitentscheidung 7 zum gemeinsamen Tech-Stack, neuer
  Abschnitt 3.3a mit der Regel-Engine für steuerliche Parameter, Stand der
  Spezifikation und Token-Eigenschaften nach ADR 0006 nachgezogen, Verzahnung mit
  der neu geschnittenen Roadmap der Handwerkersoftware ausgeschrieben
- Tech-Stack und Lizenz sind keine offenen Entscheidungen mehr
