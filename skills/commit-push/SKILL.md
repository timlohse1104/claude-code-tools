---
name: commit-push
description: Führt einen vollständigen Commit-und-Push-Workflow durch, inklusive Feature-Branch-Erstellung, Qualitätschecks (Build, Lint, Tests), CHANGELOG- und Docs-Update. Auslösen bei Anfragen wie "committe meine Änderungen", "push das", "erstelle einen Commit" oder dem Befehl /commit-push.
disable-model-invocation: true
---

Führe einen vollständigen Commit-und-Push-Workflow durch. Falls `$ARGUMENTS` übergeben wurde, verwende es als Commit-Nachricht. Andernfalls leite eine kurze Nachricht (max. 1 Satz) aus den Änderungen ab.

## Schritte

### 1. Status analysieren
Führe folgende Befehle parallel aus:
- `git status` — zeigt untracked und geänderte Dateien
- `git diff` — zeigt staged und unstaged Änderungen
- `git log --oneline -5` — zeigt die letzten 5 Commits für Stil-Konsistenz

### 2. Feature Branch erstellen
Leite aus den Änderungen einen kurzen Branch-Namen ab (Format: `feature/<kebab-case-beschreibung>`).
- Erstelle den Branch von `main`: `git checkout -b feature/<name>`
- Der Branch-Name soll die Änderungen knapp beschreiben, z.B. `feature/memorandum-skeleton-loading`

### 3. Qualitätschecks (Frontend & Backend)
Führe **parallel** aus:
- `cd frontend && npm run build` — Frontend-Build
- `cd backend && npm run build` — Backend-Build

Führe danach **parallel** aus:
- `cd frontend && nx run-many -t lint` — Frontend-Lint
- `cd backend && nx run-many -t lint` — Backend-Lint

Führe danach **parallel** aus:
- `cd frontend && nx run-many -t test` — Frontend-Tests
- `cd backend && nx run-many -t test` — Backend-Tests

**Bei jedem Fehler in Build, Lint oder Tests: Sofort abbrechen, zurück zu `main` wechseln (`git checkout main`) und den Fehler dem User melden. Keinen Commit erstellen.**

### 4. CHANGELOG.md prüfen und aktualisieren
Prüfe, ob CHANGELOG.md bereits geändert wurde (`git status`). Falls **nicht**:
- Analysiere die Änderungen und erstelle passende Einträge unter `## [Unreleased]`
- Nutze das bestehende Format: `### Added / Changed / Fixed / Removed` mit `[modul]`-Prefix
- Beispiel: `- [memorandum] Neue Funktion XY hinzugefügt.`

Falls CHANGELOG.md bereits geändert wurde: Prüfe, ob die aktuellen Änderungen darin bereits vollständig abgedeckt sind. Falls nicht, ergänze fehlende Einträge.

### 5. Docs aktualisieren

**Nur wenn ein `docs/`-Verzeichnis im Repository existiert** — andernfalls diesen Schritt überspringen.

**Schritt 1: Betroffene Docs ermitteln**

Führe `find docs -name "*.md" | sort` aus, um alle vorhandenen Doc-Dateien zu kennen. Gleiche dann die geänderten Dateien (aus `git diff --name-only`) semantisch gegen die Doc-Dateinamen ab:
- Der Stem eines Doc-Dateinamens (z.B. `hitstar` aus `docs/features/hitstar.md`) entspricht typischerweise einem Verzeichnis- oder Modulnamen im Quellcode.
- Eine geänderte Datei ist relevant für eine Doc, wenn ihr Pfad den Doc-Stem enthält (z.B. `src/routes/hitstar/+page.svelte` → `docs/features/hitstar.md`) oder wenn der Doc-Stem inhaltlich beschreibt, was die Datei tut (z.B. `store-hitstar.ts` → `hitstar.md`).
- Docs unter `docs/shared/` betreffen querschnittliche Themen (z.B. Auth, i18n, Stores) — prüfe, ob geänderte Dateien eines dieser Themen berühren.

**Schritt 2: Jede betroffene Doc aktualisieren**

Für jede identifizierte Doc:
1. Lese die Doc-Datei.
2. Lese die geänderten Quelldateien aus dem Diff.
3. Prüfe, ob Fakten in der Doc veraltet sind: API-Endpunkte, Dateinamen, Env-Variablen, Typen, Toggle-Keys, Architektur-Beschreibungen.
4. Aktualisiere nur veraltete Abschnitte — kein komplettes Neuschreiben.
5. Falls alles noch stimmt: keine Änderung.

**Schritt 3: CLAUDE.md prüfen**

Nur wenn `CLAUDE.md` im Repo existiert und dort eine Docs-Referenz (z.B. eine Feature- oder Shared-Tabelle) enthalten ist:
- Neues Feature, das noch nicht in der Tabelle steht → Zeile ergänzen.
- Neue Shared-Infrastructure → Zeile ergänzen.
- Sonst: keine Änderung nötig.

### 6. Dateien stagen
Stage alle relevanten Dateien **gezielt per Name** (kein `git add -A` oder `git add .`):
- Starte mit allen geänderten Dateien aus `git status`
- Stage immer auch CHANGELOG.md mit
- Falls Docs geändert wurden: Stage diese ebenfalls mit
- Vermeide versehentliches Stagen von: `.env`, Credential-Dateien, Binaries, `node_modules/`

### 7. Commit erstellen
- Falls `$ARGUMENTS` übergeben: verwende exakt diesen Text als Commit-Nachricht
- Sonst: leite einen prägnanten Satz aus den Änderungen ab (Imperativ, Englisch, max. 1 Satz)
- **Kein Gitmoji** in der Nachricht — der Post-Commit Hook fügt es automatisch hinzu
- Erstelle den Commit im HEREDOC-Format mit Co-Authored-By-Zeile:

```
git commit -m "$(cat <<'EOF'
Commit-Nachricht hier.

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

### 8. Push
- Branch hat noch kein Upstream-Tracking → immer: `git push -u origin <branch-name>`

### 9. Bestätigung
- Führe `git status` aus
- Gib eine kurze Zusammenfassung aus: Commit-Hash, Nachricht, gepushter Branch

## Wichtige Regeln

- **Niemals** `.env`, Credential-Dateien oder Secrets committen
- **Kein** `--no-verify` oder `--force`
- **Kein** `git add -A` oder `git add .`
- **Kein Commit bei fehlgeschlagenem Build, Lint oder Test** — Fehler melden und abbrechen
- Bei Pre-Commit-Hook-Fehler: Problem beheben, dann **neuen** Commit erstellen (kein `--amend`)
- Commit-Nachricht: **maximal 1 Satz**, Imperativ, Englisch
- Immer CHANGELOG.md mit aktualisieren und auf Vollständigkeit prüfen
