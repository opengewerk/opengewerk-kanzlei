# Konzept

Hier liegen die Planungsdokumente von OpenGewerk Kanzlei. Sie sind die verbindliche Quelle für den Funktionsumfang, solange es noch keinen Code gibt.

| Datei | Inhalt |
| --- | --- |
| [`Planungskonzept.md`](Planungskonzept.md) | Vollständiges Planungskonzept des Kanzlei-Hubs: Leitentscheidungen, Architektur und Föderation, Scopes, Funktionsumfang, Rollen, Berufsrecht, Schnittstellenvertrag, Roadmap |

Das Konzept der Handwerkersoftware liegt im Repository [`opengewerk`](https://github.com/opengewerk/opengewerk) unter `docs/konzept/`. Abschnitt 6 des Planungskonzepts beschreibt, was dort ergänzt werden muss, damit der Hub überhaupt anbinden kann.

## Wie diese Dokumente geändert werden

Änderungen laufen wie Codeänderungen über einen Pull Request, nicht über direkte Pushes auf `main`. Damit bleibt nachvollziehbar, wann eine Entscheidung gefallen ist und warum.

Für einen Pull Request an diesen Dokumenten gilt:

- Die Versionsnummer in der Kopfzeile des Dokuments anheben und das Änderungsprotokoll am Dateiende ergänzen.
- Widersprüche zu anderen Abschnitten mit auflösen, statt eine Korrektur an einer zweiten Stelle danebenzuschreiben.
- Betrifft die Änderung die Endpunkte, die Scopes oder die Webhooks, gehört sie zusätzlich in [`opengewerk-api-spec`](https://github.com/opengewerk/opengewerk-api-spec). Der Vertrag und das Konzept dürfen nicht auseinanderlaufen.
- Berufsrechtliche Aussagen brauchen eine Fundstelle, also Paragraf und Gesetz, keine allgemeine Einschätzung.

Wer erst einmal nur eine Frage oder eine Idee hat, ist in den [Discussions](https://github.com/opengewerk/opengewerk-kanzlei/discussions) besser aufgehoben als in einem Pull Request.
