---
name: checks
description: "Prüfungen von Stellwerk ausführen (cargo fmt, clippy, Tests gegen das festgelegte Syncthing, Web-Checks, checks, Playwright, Doku). Vor jedem Push, im Review und wenn Rust, Bun, Syncthing oder ein Werkzeug für die Tests fehlt."
---

# Prüfungen

Vor jedem Push grün, danach die CI prüfen. Die CI führt Rust unter Linux, Windows und macOS aus und in einem
Pull Request nur die Jobs, die er kaputt machen kann (`scripts/ci-changes.sh`); lokal läuft, was die Änderung
berührt.

Voraussetzung: Rust stable (mindestens 1.94), Bun in der Version aus `packageManager` der `package.json`, für die
Doku Python mit `pip install -r requirements-docs.txt`.

```sh
bun install --frozen-lockfile
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo test                   # lädt das festgelegte Syncthing einmal nach .syncthing-cache/ (die CI: cargo nextest run)
bun run --filter web check   # Biome, tsc --noEmit, Vitest, vite build
bun checks                   # Dateigröße, Texte der Oberfläche, keine Ringe zwischen Modulen (checks.json, firsttris/workflows)
bun run e2e                  # Playwright (bun --bun playwright test) auf dem Demo-Build
zensical build --strict      # wenn docs/ oder mkdocs.yml geändert sind
```

- `cargo test` startet echte Syncthing-Instanzen, nur isoliert und ohne ein laufendes Syncthing anzufassen
  (`AGENTS.md`, Nie).
- `bun run e2e`: außerhalb des Playwright-Images `PW_CHROMIUM=/pfad/zu/chrome`; ist Port 4173 belegt, `PW_PORT`
  setzen, denn lokal nimmt Playwright einen laufenden Server auf dem Port statt des Demo-Builds.
- Die Screenshots (`bun run screenshots`) entstehen nicht lokal, sondern mit **Actions → Update screenshots**.
- Die Regeln am Pull Request selbst (Titel, `Umgesetzt`/`Refs`, neue Abhängigkeiten, lange Funktionen) prüft vor
  dem Push die Werkbank, siehe Skill `beitrag`.

Laufende Prozesse nicht stoppen, für Tests andere Ports nehmen. Was übersprungen wurde, gehört mit Grund in die
Beschreibung des Pull Requests, nie als bestanden.
