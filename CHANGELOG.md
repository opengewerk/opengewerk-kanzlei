# Änderungsprotokoll

Alle nennenswerten Änderungen an diesem Projekt werden in dieser Datei festgehalten.

Das Format orientiert sich an [Keep a Changelog](https://keepachangelog.com/de/1.1.0/),
die Versionsnummern folgen der [Semantischen Versionierung](https://semver.org/lang/de/).

## [Unreleased]

### Hinzugefügt

- Initiales Repository-Gerüst
- CI-Job "Schreibweise", der Gedankenstriche im gesamten Repository meldet
- Die README verlinkt das Subreddit r/OpenGewerk neben dem Discord-Server, als Ort für den
  Austausch, auch unter Kanzleien

### Geändert

- Die Vorlage für einen Fehlerbericht spricht vom Kanzlei-Hub statt vom ganzen Projekt. Das
  Projekt ist nicht mehr in der Planungsphase, die Handwerkersoftware läuft; ohne Code ist
  hier nur der Hub
- Die Prüfung "Schreibweise" kennt keine Ausnahme mehr für die zwei Schlüssel der
  Konventionen im API-Vertrag, die Umlaute umschrieben. Der Vertrag benennt sie seit
  opengewerk-api-spec#19 englisch, und der Job bleibt in allen Repositories derselbe
- Die Workflows nehmen die neuesten Hauptversionen der Actions: `actions/checkout` in
  v7, CodeQL in v4. Die alten Fassungen liefen noch auf Node 20, dessen Pflege im April
  2026 endete, und CodeQL v3 wird im Dezember 2026 abgekündigt

- Die README sagt nicht mehr, der Token sei über mTLS oder DPoP an die Hub-Instanz
  gebunden. Das Planungskonzept ist mit #7 richtiggestellt worden, die README war eine
  dritte Stelle mit derselben Zusage und blieb dabei stehen
- Der Status in der README nennt den Grund, warum hier noch nichts gebaut wird: der Hub
  hängt nicht nur am Vertrag, sondern auch am Connector-Modul aus Phase 3 der
  Handwerkersoftware. So steht es seit v1.2 im Planungskonzept
- Die Roadmap steht nur noch in Abschnitt 9 des Planungskonzepts, die README verweist
  darauf. Eine Abschrift daneben läuft irgendwann auseinander
- Unter Mitmachen nennt die README den Discord-Server, wie die Beitragsregeln der
  Organisation

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
- Das Security-Review jedes Pull Requests läuft in GitHub Actions, mit dem Workflow der
  Organisation (`opengewerk/.github#27`): für einen Branch dieses Repositorys von einem Mitglied
  voll, für einen PR von außen nur lesend und erst nach der Freigabe eines Maintainers. Geprüft
  wird der neueste Commit, ein neuer Push bricht den Lauf davor ab; das Ergebnis steht als
  Kommentar am PR und als Status `PR-Security-Review`. Der Aufruf heftet einen Commit des
  Workflows an. Bisher lief das Review auf dem Rechner von Moritz mit einem kostenlosen
  Modell, dessen Kontingent am 10.10.2026 aufgebraucht war
- Das Security-Review gibt einen PR frei oder verlangt Änderungen, wie früher der lokale Reviewer,
  und läuft mit einem zweiten Konto weiter, wenn das Guthaben des ersten aufgebraucht ist
  (`opengewerk/.github#32` bis `#34`, Aufruf angehoben). Ein PR von außen bekommt nie eine
  Freigabe, nur einen Kommentar: über ihn urteilt ein Modell, das Text aus dem PR lenken kann.
  Hebt ein PR den Aufruf an, liest das Review den Diff des Workflows aus dessen Repository; bisher
  blieb der neue Commit für das Review ein offener Punkt.
- Das Security-Review stoppt Docker, bevor es sudo abschaltet, und prüft beides; den Checkout, in
  dem der Agent schreiben kann, entfernt es gleich nach ihm, bevor der Post-Schritt von
  `actions/checkout` dort git aufruft. Lässt sich das Review am PR nicht ablegen, steht der Status
  auf `error` statt dauerhaft auf "läuft" (`opengewerk/.github#35` und `#36`, Aufruf angehoben).
  Vorher war Docker ein zweiter Weg zu root, und eine Git-Konfiguration aus der Sandbox hätte nach
  ihr außerhalb laufen können
