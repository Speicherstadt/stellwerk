# Hinweise für Coding-Agenten

Stellwerk: eine zentrale Weboberfläche, die Syncthing auf allen Geräten verwaltet, selbst gehostet. Rust-Workspace in
`crates/` (`protocol`, `engine`, `agent`, `zentrale` mit axum und SQLite), React-Oberfläche in `web/`, eingebettet
in die Zentrale, Playwright in `e2e/`.

Die Regeln stehen einmal: Beitragen in [`docs/mitwirken.md`](docs/mitwirken.md), Bauen in
[`docs/development.md`](docs/development.md), beides auf Deutsch, für Menschen und Agenten. Diese Datei sagt, wo was
steht und was nie passieren darf. Sie gilt für jeden Agenten; `CLAUDE.md` verweist nur hierher. Vor dem Start die
Abschnitte lesen, die die Änderung berühren.

## Doku

Die Doku liegt in `docs/` (<https://speicherstadt.org/stellwerk/>). Vor dem Start die Seiten lesen, die die Änderung
berühren: `architecture.md` für das Zusammenspiel von Zentrale, Hub, Agent und Engine und für die Roadmap. Links in
README und Doku zeigen auf die Website, nicht auf `docs/*.md`.

## Skills

Die Schritte für wiederkehrende Aufgaben stehen in [`.agents/skills`](.agents/skills); jeder Agent folgt denselben
Dateien. Aus der [Werkbank](https://github.com/Speicherstadt/werkbank) (gleich in allen Projekten, Änderungen dort):
`beitrag` (Branch, Commits, Pull Request als Entwurf, dann Selbstprüfung und Review), `beitrag-pruefen` (gewollt,
Quellen, einfachste Lösung, Regeln), `tdd-review` (unabhängiges Review, ein Befund braucht einen fehlschlagenden
Test), `doku-pruefen` (Review des Markdowns), `fork-pushen` (Push auf den Fork einer anderen Person),
`prs-abarbeiten` (alle offenen Pull Requests, vom einfachsten Merge an), `issue-schreiben`, `board-pflegen` (das
Board auf GitHub), `release`, `fehler-eingrenzen` (Ursache eines Fehlers über eine rote Prüfung finden) und
`vorhaben-klaeren` (einen Plan vor dem Bauen mit der Person durchfragen). Aus diesem Projekt: `checks` (alle
Prüfungen: Rust, Oberfläche, E2E, Doku).

## Referenzimplementierungen

Details stützen sich auf die Quellen, nicht auf das Gedächtnis: die
[REST-API von Syncthing](https://docs.syncthing.net/dev/rest.html) und der Quellcode von Syncthing, vor allem
`lib/config` (Typen der Konfiguration, Standards, `prepare`) und `lib/api` (`confighandler.go` dafür, was `PUT`,
`PATCH` und `DELETE` auf `/rest/config/...` tun). Flach klonen (`git clone --depth 1
https://github.com/syncthing/syncthing`) in ein temporäres Verzeichnis, den Tag aus `crates/engine/src/pin.rs`
auschecken und die Quelldatei in Kommentaren und Commit-Nachrichten nennen, wo sie ein Detail entscheidet.

## Nie

- Auf `main` pushen (Ausnahme: Reparatur eines roten `main` durch oder auf Anweisung des Maintainers), Commits von
  Beitragenden umschreiben, mergen (nur Maintainer oder Collaborator mit dessen Zustimmung), eigenen Code reviewen.
- Einen Test überspringen, abschwächen oder löschen, um grün zu werden; einen übersprungenen Test als bestanden melden.
- Issues und Pull Requests ohne Auftrag anlegen oder kommentieren.
- Eine Abhängigkeit hinzufügen, ohne zuerst eigenen Code zu versuchen und den Grund je Paket unter
  „## Neue Abhängigkeiten“ im Pull Request zu nennen.
- Befehle als root (`sudo`, `pkexec`) ausführen oder umgehen; die führt die Nutzerin oder der Nutzer aus. Laufende
  Prozesse von ihnen stoppen; für Tests andere Ports nehmen.
- **Ein Test-Syncthing auf den Standard-Ports 8384 (GUI) oder 22000 (Sync) starten.** Immer
  `engine::process::SpawnOptions::isolated`: zufällige Ports auf 127.0.0.1 vom System, Discovery, Relays und NAT aus.
- **Ein Syncthing anfassen, das schon läuft** (das eigene der Nutzerin oder des Nutzers oder eines aus einem anderen
  Test): kein `pkill syncthing`, kein Zugriff auf sein `--home` oder seine API. Gestoppt werden nur selbst gestartete
  Instanzen, über `Instance::stop`.

## Bei jeder Änderung

- Jede berührte Datei sauberer hinterlassen, ohne das Verhalten zu ändern, und unter „Aufgeräumt“ nennen.
- Tests für das, was sich ändert, wie in `docs/mitwirken.md` beschrieben; ein Bugfix beginnt mit einem
  fehlschlagenden Test.
- Doku in `docs/` im selben Pull Request, kurz und für Menschen; README und Screenshots nach `docs/mitwirken.md`.
- Issues mit `Umgesetzt #…` oder `Refs #…`, nie `Closes`/`Fixes`.
- Prüfungen grün vor jedem Push (Skill `checks`), übersprungene Tests nennen; danach die CI prüfen.
- Die Beschreibung des Pull Requests wird die Squash-Nachricht: bei jedem Push nachziehen, der etwas daran ändert.
- Vor dem Merge ein unabhängiges Review durch einen eigenen Agenten: `tdd-review` für Code, `doku-pruefen` für Markdown.

## Arbeitsweise

- Deutsch für Kommentare, Doku, Commits, Pull Requests und alles, was Nutzer lesen; Bezeichner englisch. Ältere
  Kommentare und Commits sind englisch; was man anfasst, darf deutsch werden. Die Oberfläche auf Deutsch und
  Englisch (`docs/mitwirken.md`, Texte der Oberfläche).
- Technik nach `tech-stack.md` in firsttris/workflows, mit einer Abweichung: Die Zentrale ist in **Rust (axum)**
  geschrieben, nicht in TanStack Start, weil Agent und Zentrale die Crates `protocol` (Nachrichten, Soll-Zustand) und
  `engine` (Steuerung von Syncthing; die Zentrale steuert ihr Hub-Syncthing mit demselben Code wie die Agenten)
  teilen. Ein Crate für beide heißt weniger Code und keine auseinanderlaufenden Typen. Die Oberfläche in `web/` folgt
  dem Tech-Stack (React, TanStack, Tailwind, shadcn-Stil).
- Clean Code, funktional: klare Namen, kleine Funktionen, die eine Sache tun, Abhängigkeiten werden übergeben, keine
  Doppelungen, kein toter Code, Kommentare sagen warum. SOLID und Entwurfsmuster als Funktionen, Daten und Closures,
  nie als Klassen, Vererbung oder Klassenhierarchien. Reine Funktionen, die Daten nehmen und Daten zurückgeben,
  Seiteneffekte am Rand; Funktionskomponenten und Hooks; Fehler als Werte. Eine Datei tut eine Sache: über ~400
  Zeilen wird sie geteilt, eine Funktion über ~80. Ein Refactor ändert kein Verhalten.
- `engine` ist das einzige Crate, das Syncthing kennt; `protocol` beschreibt nur Geräte, Ordner, Modi und Pfade, damit
  später eine andere Engine `engine` ersetzen kann.
- Der Reconciler (`engine::reconcile::plan`) bleibt eine reine Funktion: Ist-Konfiguration und Soll-Zustand → Liste
  von API-Aufrufen. Seiteneffekte nur in `engine::apply`. Er ändert die Konfiguration, nie Dateien.
- Die Version von Syncthing steht mit SHA-256 je Plattform in `crates/engine/src/pin.rs`; sie anzuheben heißt neue
  Prüfsummen und ein grüner Integrationstest (`docs/development.md`, Die Syncthing-Version ändern).
- Fremde, nicht committete Änderungen gehören jemand anderem (oft einem anderen Agenten): beim eigenen Umfang bleiben.
