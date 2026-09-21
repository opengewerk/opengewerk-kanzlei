<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/opengewerk/.github/main/brand/opengewerk-logo-dark.svg">
    <img alt="OpenGewerk" src="https://raw.githubusercontent.com/opengewerk/.github/main/brand/opengewerk-logo.svg" width="420">
  </picture>
</p>

<p align="center"><strong>Self-hosted Hub für Steuerberater</strong></p>

## Was ist OpenGewerk Kanzlei?

OpenGewerk Kanzlei ist ein zentrales, self-hosted System für Steuerberaterkanzleien, das beliebig viele Mandanten-Instanzen der Handwerkersoftware anbindet. Der Leitsatz lautet **Föderation statt Zentralisierung**: Der Hub speichert keine Buchhaltung, er greift per API live auf die Mandantensysteme zu und hält nur einen Cache für Übersichten. Die Datenhoheit bleibt beim Mandanten, und genau das ist das Argument gegenüber DATEV Unternehmen online. Lesender Zugriff ist der Default, schreibende Rechte gibt der Mandant einzeln pro Scope frei und kann sie jederzeit widerrufen. Die Verbindung wird immer vom Mandanten aus aufgebaut, damit kein Betrieb ohne sein Wissen angebunden werden kann. Verschwiegenheit nach §203 StGB und §57 StBerG, Mandantentrennung und Zugriffsprotokollierung sind Architekturanforderungen, keine Features.

## Abgrenzung zu bestehenden Lösungen

| Bestehende Lösung | Schwäche | Kanzlei-Hub |
| --- | --- | --- |
| DATEV Unternehmen online | Daten liegen zentral bei DATEV, laufende Kosten je Mandant, Handwerksprozesse fehlen | Föderiert, self-hosted, Handwerkersoftware liefert Fachkontext (Projekt, Anlage, Regiebericht am Beleg) |
| sevdesk / Lexware Steuerberaterzugang | Ein Login pro Mandant, keine Kanzleiübersicht, keine Fristen über Mandanten hinweg | Eine Anwendung, alle Mandanten, Gesundheitsindex, Fristenkalender |
| Kanzleisoftware (Agenda, Addison) | Import-zentriert, Rückfragen per Mail/Telefon | Rückfragen am Beleg, Vorschläge mit Ein-Klick-Freigabe, Webhooks |
| Lesezeichen-Ordner mit 40 Mandanten-Links | Unübersichtlich, unsicher, keine Statusinfo | Zentrales Dashboard mit Verbindungs- und Bearbeitungsstatus |

## So funktioniert die Anbindung

1. Der Mandant erzeugt in seiner Handwerkersoftware unter *Einstellungen → Steuerberater* einen Einladungscode (einmalig, 24 Stunden gültig) und wählt die freizugebenden Scopes.
2. Die Kanzlei gibt Code und Mandanten-URL im Hub ein.
3. Der Hub tauscht den Code gegen einen langlebigen, rotierbaren Token (ähnlich OAuth 2.0 Device- beziehungsweise Authorization-Code-Flow). Die kryptografische Bindung des Tokens an die Hub-Instanz über mTLS oder DPoP kommt nach der ersten Fassung; so hat es ADR 0006 im Repository `opengewerk` entschieden.
4. Der Mandant sieht in seiner Instanz die verbundene Kanzlei, die Scopes und die letzten Zugriffe und kann die Verbindung jederzeit trennen.

Welche Scopes es gibt und was sie umfassen, steht in [`docs/konzept/Planungskonzept.md`](docs/konzept/Planungskonzept.md), Abschnitt 2.5, und in der Spezifikation unter [`opengewerk-api-spec`](https://github.com/opengewerk/opengewerk-api-spec).

## Status

OpenGewerk Kanzlei ist in der **Planungsphase**. Es gibt noch keinen lauffähigen Code, nur das ausgearbeitete Konzept und dieses Repository-Gerüst.

Der Hub ist kein paralleler Strang. Seine erste Phase verlangt den API-Vertrag in Version 1 und das Connector-Modul in der Handwerkersoftware, und das gehört dort zu Phase 3, der Buchhaltung. Die Handwerkersoftware steckt gerade in Phase 1, dem MVP für ihren Pilotbetrieb. Vorher gibt es hier nichts zu bauen, was nicht ins Leere liefe; vorziehen lässt sich allein der Vertrag in [`opengewerk-api-spec`](https://github.com/opengewerk/opengewerk-api-spec).

Das vollständige Konzept liegt unter [`docs/konzept/`](docs/konzept/).

## Roadmap

Der Fahrplan in sechs Phasen, vom Vertrag bis zum Vollausbau, steht in [Abschnitt 9 des Planungskonzepts](docs/konzept/Planungskonzept.md#9-roadmap) und bewusst nur dort. Eine Abschrift daneben läuft irgendwann auseinander.

## Projektfamilie

- [`opengewerk`](https://github.com/opengewerk/opengewerk): die Handwerkersoftware beim Mandanten, erste und bisher einzige Quelle für den Hub.
- [`opengewerk-kanzlei`](https://github.com/opengewerk/opengewerk-kanzlei): dieser Hub, mit dem eine Kanzlei alle Mandanten aus einer Anwendung heraus bearbeitet.
- [`opengewerk-api-spec`](https://github.com/opengewerk/opengewerk-api-spec): der gemeinsame API-Vertrag, den Hub und Handwerkersoftware beide implementieren und dessen unterstützte Version beide Seiten deklarieren.

## Mitmachen

Besonders wertvoll sind Rückmeldungen aus dem Kanzleialltag: welche Kennzahl am Montagmorgen wirklich zählt, wo eine Rückfrage heute noch per Telefon läuft, welche Berufsrechtsfrage im Konzept fehlt.

- Fragen, Ideen und alles ohne konkreten Vorschlag gehören in die [Discussions](https://github.com/opengewerk/opengewerk-kanzlei/discussions).
- Für kurze Fragen und zum Mitreden gibt es einen [Discord-Server](https://discord.gg/NRrEvbQdxz). Er ersetzt die Discussions nicht: ein Chatverlauf ist nicht durchsuchbar, und was dort geklärt wird und für andere zählt, gehört hinterher in eine Discussion oder ein Issue.
- Konkrete Fehler und Wünsche laufen über die [Issue-Vorlagen](https://github.com/opengewerk/opengewerk-kanzlei/issues/new/choose).
- Die Beitragsregeln stehen in [CONTRIBUTING.md](https://github.com/opengewerk/.github/blob/main/CONTRIBUTING.md), der Verhaltenskodex in [CODE_OF_CONDUCT.md](https://github.com/opengewerk/.github/blob/main/CODE_OF_CONDUCT.md).

## Lizenz

[GNU Affero General Public License v3.0](LICENSE). Wer den Hub als Dienst für andere betreibt, gibt seine Änderungen zurück.
