# tilloh.dev — Feature Pipeline Commands

> 🔒 **Ausschließlich für [github.com/timlohse1104/tilloh.dev](https://github.com/timlohse1104/tilloh.dev)** — nicht für andere Projekte verwenden.

Ausschließlich für das **tilloh.dev Repository**.
Alle Commands gehören in `.claude/commands/` (das `claude-code-tools` Submodul).

---

## Commands

| Command | Beschreibung | Standalone |
|---------|-------------|-----------|
| `/tillohdev-feature` | Vollständiger Feature-Lifecycle (alle Phasen) | ✅ |
| `/tillohdev-qa` | Qualitätssicherung | ✅ |
| `/tillohdev-validate` | Playwright E2E-Validierung | ✅ |
| `/tillohdev-review-pr` | PR-Review gegen Best Practices + GitHub-Kommentar | ✅ |
| `/tillohdev-commit-push` | Commit + Push mit Qualitätschecks | ✅ |

---

## Der vollständige Feature-Lifecycle

```
/feature
   │
   ├── 🔍 PHASE 1: Discovery
   │       Fragt: Name, Beschreibung, Scope, Auth,
   │              i18n-Keys (PFLICHT), Akzeptanzkriterien
   │       Erstellt: Feature-Steckbrief zur Bestätigung
   │
   ├── 📋 PHASE 2: Planung
   │       Liest: CLAUDE.md + Referenz-Docs aus docs/features/ + docs/shared/
   │       Erstellt: Implementierungsplan zur Bestätigung
   │
   ├── 🛠️  PHASE 3: Implementierung
   │       Branch erstellen
   │       Backend: NX Scaffold + DTOs + Service + Controller + Tests
   │       Frontend: Types + API + i18n (de+en gleichzeitig) + Stores + Components + Route
   │       Docs: docs/features/[name].md + CHANGELOG.md
   │
   ├── 🔬 PHASE 4: QA  ←── auch standalone: /qa
   │       TypeScript → Linting → Tests → Build
   │       i18n Vollständigkeitsprüfung (PFLICHT)
   │       Svelte 5 Audit + Diff-Analyse
   │
   ├── 🎭 PHASE 5: Playwright  ←── auch standalone: /validate
   │       S1 Route, S2 Navigation, S3 i18n de+en (PFLICHT),
   │       S4 Loading, S5 Mobile, S6 Error + Akzeptanzkriterien
   │
   ├── 🗺️  PHASE 6: Feature Walkthrough  ←── NEU
   │       Was wurde gebaut? (nicht-technische Übersicht)
   │       Technische Umsetzung erklären
   │       Schritt-für-Schritt Testanleitung für manuelles Testen
   │       → Wartet auf Bestätigung: "lokal getestet, OK?"
   │
   ├── 🔍 PHASE 7: Code Review
   │       Security / QA / Code-Qualität
   │       Verdict: APPROVED oder CHANGES REQUESTED
   │
   └── 🚀 PHASE 8: Commit & Push
           Commit (Pre-commit: Lint+Tests) → Push → PR-Template
           → Anschließend: /review-pr für GitHub PR-Kommentar
```

---

## Best Practices Katalog (tilloh.dev)

Dieser Katalog wird von `/tillohdev-qa` und `/tillohdev-review-pr` als Prüfgrundlage genutzt.

### Svelte 5 Runes
- **`$state`** für reaktiven lokalen State — Arrays/Objekte direkt mutieren
- **`$derived`** für berechnete Werte — nicht `$effect` missbrauchen
- **`$effect`** nur für Side Effects mit Cleanup — nicht für Fetching (→ `onMount`)
- **`$props()`** immer mit Interface typisieren
- **`$bindable()`** sparsam — nur wenn bidirektionale Bindung wirklich nötig
- Keine `$:` Syntax mehr — vollständig auf Runes migrieren
- Rune-Reihenfolge: imports → props → const → state → derived → effects → lifecycle → functions

### SvelteKit
- Jede Route hat `<svelte:head><title>[Feature] | tilloh.dev</title></svelte:head>`
- `export const ssr = false` explizit bei Client-Only Features
- Error States und Loading States sind Pflicht — nie leere Seite
- API-Calls ausschließlich in `/lib/api/` — nie direkt in `.svelte`
- Types ausschließlich in `/lib/types/`
- Alle Funktionen als Lambda Const

### i18n — IMMER PFLICHT
- `de.json` und `en.json` **gleichzeitig** pflegen
- Key-Schema: `[feature].[kontext].[bezeichnung]`
- Kein einziger hartcodierter String in Svelte-Dateien
- Fehler- und Leer-Zustands-Keys immer mitführen
- `[missing: ...]` Placeholder = Fehler

### NestJS + Fastify
- DTOs mit `class-validator` Dekoratoren validieren
- Swagger vollständig: `@ApiTags`, `@ApiOperation`, `@ApiResponse` inkl. 4xx/5xx
- NestJS `Logger` statt `console.log`
- `HttpException` für Fehler — nie interne Details zurückgeben
- Fastify: Return-Wert nutzen statt `res.send()` — `@Res({ passthrough: true })` wenn nötig
- Service Naming: `listX`, `createX`, `updateX`, `removeX`
- MongoDB Service Naming: `findAll`, `findById`, `create`, `update`, `remove`

### MongoDB / Mongoose
- Schema immer mit `{ timestamps: true }`
- `@Prop({ required: true })` für Pflichtfelder
- `lean()` bei Read-Only Queries
- Indexes für häufig abgefragte Felder
- Try/Catch in allen DB-Service-Methoden

### TypeScript
- Kein `any` — `unknown` + narrowing wenn Typ wirklich unbekannt
- Keine blinden Type Assertions (`as MyType`)
- Union Types für Zustände: `'idle' | 'loading' | 'success' | 'error'`
- Optional Chaining (`?.`) und Nullish Coalescing (`??`)

### HTML / CSS / Accessibility
- Semantische Elemente: `<button>`, `<main>`, `<section>`, `<article>`
- Jeder Button mit Text-Content oder `aria-label`
- `alt`-Attribut bei allen Bildern
- CSS scoped in Svelte-Komponenten
- Mobile-First, keine hardcodierten Pixel-Breiten
- Carbon Design System Komponenten nutzen wo möglich

---

## Setup

### 1. Commands ins Submodul

```bash
# Im claude-code-tools Repo:
cp tilloh-feature.md commands/tillohdev-feature.md
cp tilloh-qa.md commands/tillohdev-qa.md
cp tilloh-validate.md commands/tillohdev-validate.md
cp tilloh-review-pr.md commands/tillohdev-review-pr.md
git add commands/ && git commit -m "add tilloh.dev feature pipeline v2" && git push

# Im tilloh.dev Repo Submodul updaten:
cd .claude && git pull origin main && cd ..
git add .claude && git commit -m "update claude-code-tools submodule"
```

### 2. Playwright MCP in settings.local.json

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

### 3. GitHub CLI für /review-pr (optional aber empfohlen)

```bash
brew install gh
gh auth login
```

### 4. Branch-Schutz auf GitHub

Repository Settings → Branches → main → Add rule:
- [x] Require a pull request before merging
- [x] Do not allow bypassing

---

## Checkpoints im /feature Workflow

| Phase | Wartet auf Bestätigung |
|-------|------------------------|
| Nach Phase 1 | Feature-Steckbrief: "Fortfahren?" |
| Nach Phase 2 | Implementierungsplan: "Beginnen?" |
| Nach Phase 6 | Walkthrough: "Lokal getestet und OK?" |
| Nach Phase 7 | Review-Verdict: "APPROVED?" |
| Phase 8 | Explizit "commit" sagen |
