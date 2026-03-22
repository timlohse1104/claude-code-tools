---
model: claude-opus-4-6
---

# /tillohdev-review-pr — Pull Request Review für tilloh.dev

⚠️  TILLOH.DEV ONLY — Dieser Command ist ausschließlich für das Repository
> **github.com/timlohse1104/tilloh.dev** und darf nicht in anderen Projekten verwendet werden.
> Vor jeder Ausführung prüfen: `test -f CLAUDE.md && grep -q "tilloh.dev" CLAUDE.md`

---

## 🔒 Repository-Guard (immer zuerst ausführen)

```bash
# Sicherstellen dass wir im tilloh.dev Repository sind
if ! (test -f CLAUDE.md && grep -q "tilloh.dev" CLAUDE.md); then
  echo "❌ ABBRUCH: Dieser Command darf nur im tilloh.dev Repository verwendet werden."
  echo "   Aktuelles Verzeichnis: $(pwd)"
  exit 1
fi
echo "✅ tilloh.dev Repository bestätigt"
```

Falls der Check fehlschlägt → Command sofort beenden und Nutzer darauf hinweisen.

---


Analysiert einen offenen oder lokalen Pull Request gegen allgemeine und tilloh.dev-spezifische
Best Practices und schreibt die Findings direkt als PR-Kommentare.

> **Ausschließlich für das tilloh.dev Repository.**
> Playwright MCP und GitHub CLI oder GitHub MCP müssen verfügbar sein.

---

## 0. Vorbereitung

```bash
# Aktuellen Branch und PR-Info ermitteln
BRANCH=$(git branch --show-current)
echo "Review für Branch: $BRANCH"

# Diff gegen main laden
git fetch origin main
git diff origin/main...HEAD --stat
git diff origin/main...HEAD
```

Falls der Nutzer eine PR-Nummer oder URL nennt → direkt diesen PR analysieren.
Falls auf Feature-Branch → aktuellen Diff reviewen.

---

## 1. ANALYSE-DURCHLAUF: Allgemeine Best Practices

### 1.1 TypeScript

Prüfe den gesamten Diff auf:

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| Kein `any` Typ | `unknown`, konkrete Typen | `as any`, `: any` |
| Keine Type Assertions ohne Check | narrowing mit `if` | `value as MyType` blind |
| Union Types für Zustände | `'idle' \| 'loading' \| 'error'` | `boolean isLoading + boolean isError` |
| Optional Chaining | `obj?.prop` | `obj && obj.prop` |
| Nullish Coalescing | `value ?? 'default'` | `value \|\| 'default'` (unsafe für 0/'') |
| Alle Exports typisiert | explizite Return-Types | implizites `any` Return |
| Generics sinnvoll genutzt | `Array<T>` | `Array<any>` |
| Enums statt Magic Strings | `Status.Active` | `'active'` hardcoded |

### 1.2 Svelte 5 Runes

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| Keine `$:` Syntax | `$derived`, `$effect` | `$: computed = ...` |
| `$effect` nicht für Fetching | `onMount` für Datenfetching | `$effect(() => fetchData())` |
| `$effect` nicht für Berechnungen | `$derived` nutzen | `$effect(() => { x = a + b })` |
| Array/Object direkt mutieren | `items.push(x)` | `items = [...items, x]` |
| Props mit Interface typisiert | `let { prop }: Props = $props()` | `let { prop } = $props()` ohne Typ |
| `$bindable` nur wenn nötig | sparsam eingesetzt | überall `$bindable` |
| Rune-Reihenfolge eingehalten | imports→props→const→state→derived→effects→lifecycle→functions | beliebige Reihenfolge |

### 1.3 SvelteKit

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| `<svelte:head>` mit Title | unique, beschreibender Titel | kein Title-Tag |
| Error States implementiert | Fehlermeldung sichtbar | leere Seite bei Fehler |
| Loading States implementiert | Skeleton/Spinner | keine Loading-Anzeige |
| Client-only korrekt gesetzt | `export const ssr = false` wo nötig | SSR-Fehler wegen Browser-APIs |
| Keine hardcodierten URLs | Route-Konstanten aus config | `/hardcoded/path` |
| API-Calls nur in `/lib/api/` | sauber getrennt | fetch direkt in `.svelte` |
| Types nur in `/lib/types/` | sauber getrennt | Interface inline definiert |
| Funktionen als Lambda Const | `const fn = () => {}` | `function fn() {}` |

### 1.4 NestJS / Fastify

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| DTOs mit class-validator | `@IsString()`, `@IsNotEmpty()` | keine Validierungsdekoratoren |
| Vollständige Swagger-Docs | `@ApiTags`, `@ApiOperation`, `@ApiResponse` inkl. 4xx | nur Happy-Path dokumentiert |
| NestJS Logger | `new Logger(ClassName.name)` | `console.log(...)` |
| HttpException für Fehler | `throw new NotFoundException(...)` | `return { error: 'message' }` |
| Fastify-kompatibel | `@Res({ passthrough: true })` wenn nötig | `res.send()` direkt |
| Separation of Concerns | Service für Logic, Controller für HTTP | Business-Logic im Controller |
| Service-Naming descriptiv | `listJokes`, `createJoke` | `getAll`, `post` |
| MongoDB-Naming generisch | `findAll`, `findById`, `create` | `listJokes` im DB-Layer |

### 1.5 MongoDB / Mongoose

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| `{ timestamps: true }` im Schema | automatisch `createdAt`, `updatedAt` | manuell gepflegte Timestamps |
| `@Prop({ required: true })` | kein stummes Fehlschlagen | fehlende required-Dekoratoren |
| `lean()` bei Read-Only | bessere Performance | immer volle Mongoose-Dokumente |
| Index-Dekoratoren | `@Prop({ index: true })` | keine Indexes bei häufigen Queries |
| Keine rohen ObjectId-Strings | `new Types.ObjectId()` | String-Manipulation |
| Try/Catch in DB-Services | Fehler sauber gefangen | unkontrollierte DB-Fehler |

### 1.6 HTML / CSS / Accessibility

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| Semantische Elemente | `<button>`, `<main>`, `<section>` | `<div onclick=...>` |
| `aria-label` oder Text bei Buttons | zugänglich | Icons ohne Beschriftung |
| `alt`-Attribut bei Bildern | vorhanden (auch leer für dekorativ) | fehlt |
| CSS scoped | in `.svelte`-Komponente | globales CSS ohne Grund |
| Responsive Design | keine fixen Pixel-Breiten | `width: 800px` hardcoded |
| Keine `tabindex > 0` | natürliche DOM-Reihenfolge | `tabindex="5"` |

### 1.7 Tests — Neu und Bestehend

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| Neue Services haben Unit Tests | Happy Path + Fehlerfall abgedeckt | Kein einziger Test |
| Bestehende Tests angepasst | Geänderte Signaturen/Felder reflektiert | Tests kompilieren nicht mehr |
| Geänderte Routen in E2E abgedeckt | Playwright-Tests aktualisiert | Bestehende E2E-Tests schlagen fehl |
| Keine auskommentierten Tests | Tests aktiv und grün | `it.skip` oder `xit` ohne Kommentar |
| Test-Fixtures aktuell | Passen zu aktuellem Schema | Veraltete Mock-Daten |

### 1.8 Sicherheit

| Check | Gut ✅ | Schlecht ❌ |
|-------|--------|------------|
| Keine Secrets im Code | Env Vars genutzt | API Keys hardcoded |
| Env Vars validiert | `env.validation.ts` aktualisiert | nicht validiert |
| Fehler-Responses sicher | nur Nutzer-freundlicher Text | Stack-Trace im Response |
| Input-Validierung | class-validator DTOs | rohe User-Inputs verarbeitet |
| Auth korrekt gesetzt | `@Public()` bewusst oder Guard aktiv | vergessen |

---

## 2. ANALYSE-DURCHLAUF: tilloh.dev-spezifische Best Practices

### 2.1 i18n — PFLICHT

```bash
# Prüfe ob [feature-name] Keys in beiden Sprachen existieren
```

| Check | Anforderung |
|-------|------------|
| de.json + en.json GLEICHZEITIG | Kein Feature-Key nur in einer Sprache |
| Key-Schema: `[feature].[kontext].[bezeichnung]` | Konsistentes Schema wie bestehende Features |
| Kein hartcodierter String in .svelte | Ausnahmslos alle UI-Texte über i18n |
| Fehler-Keys vorhanden | `[feature].error.loading`, `[feature].error.notFound` |
| Leerer-Zustand-Key vorhanden | `[feature].empty` oder äquivalent |
| Kein `[missing: ...]` Placeholder | Keys müssen in beiden Sprachen existieren |

### 2.2 NX Monorepo Struktur

| Check | Anforderung |
|-------|------------|
| Path Alias in `backend/tsconfig.base.json` | `@backend/[name]` eingetragen |
| Barrel Export in `src/index.ts` | Öffentliche API sauber exportiert |
| `project.json` korrekt | NX Library korrekt registriert |
| Module in `AppModule` registriert | Nicht vergessen in `main.ts` |

### 2.3 Toggle-System

| Check | Anforderung |
|-------|------------|
| Toggle in `toggles.ts` Enum | `[featureName] = 'TOGGLE_NAV_[NAME]'` |
| Toggle-Seed in `toggles.config.ts` | Standardwert gesetzt |
| Toggle in Navigation geprüft | Feature erst sichtbar wenn Toggle aktiv |

### 2.4 Dokumentation

| Check | Anforderung |
|-------|------------|
| `docs/features/[name].md` existiert | Nach Schema von `hitstar.md` |
| Alle Endpunkte dokumentiert | Backend-Sektion vollständig |
| Stores dokumentiert | Falls localStorage genutzt |
| Key Implementation Notes | Fallstricke und Entscheidungen notiert |
| CHANGELOG.md unter `[Unreleased]` | Eintrag vorhanden |

### 2.5 Code-Konventionen (aus CLAUDE.md)

| Check | Anforderung |
|-------|------------|
| Alle Funktionen Lambda Const | `const fn = () => {}` nicht `function fn() {}` |
| Service-Naming descriptiv | `listItems`, `createItem` etc. |
| MongoDB-Naming generisch | `findAll`, `findById` etc. |
| Prettier: singleQuote | `'string'` nicht `"string"` |

---

## 3. FINDINGS DOKUMENTIEREN

Erstelle nach dem Analyse-Durchlauf eine strukturierte Findings-Liste:

```
PR REVIEW FINDINGS — [Feature-Name] — [Datum]
══════════════════════════════════════════════════════════

ALLGEMEINE BEST PRACTICES
─────────────────────────
🚫 BLOCKER [Datei:Zeile]
   Was: [Beschreibung]
   Warum: [Begründung]
   Fix: [Konkreter Vorschlag]

⚠️  SUGGESTION [Datei:Zeile]
   Was: [Beschreibung]
   Warum: [Begründung]
   Fix: [Konkreter Vorschlag]

✅ GUT GELÖST
   · [Was positiv aufgefallen ist]

TILLOH.DEV-SPEZIFISCH
──────────────────────
🚫 BLOCKER
   ···

⚠️  SUGGESTION
   ···

✅ GUT GELÖST
   ···

──────────────────────────────────────────────────────────
STATISTIK:
  🚫 Blocker:    [Anzahl]
  ⚠️  Suggestion: [Anzahl]
  ✅ Gut:         [Anzahl]

VERDICT: APPROVED ✅ / APPROVED WITH SUGGESTIONS ⚠️ / CHANGES REQUESTED 🚫
══════════════════════════════════════════════════════════
```

---

## 4. FINDINGS IN PR EINTRAGEN

### Option A: GitHub CLI (falls verfügbar)
```bash
gh pr review [PR-Nummer] --comment --body "[Findings als Markdown]"

# Für spezifische Zeilen-Kommentare:
gh pr review [PR-Nummer] --comment \
  --body "$(cat <<'EOF'
## Review Findings

### 🚫 Blocker
[Details]

### ⚠️ Suggestions
[Details]

### ✅ Gut gelöst
[Details]

**Verdict:** CHANGES REQUESTED / APPROVED
EOF
)"
```

### Option B: GitHub MCP (falls konfiguriert)
Nutze das GitHub MCP um einen Review-Kommentar am PR zu erstellen mit den vollständigen Findings.

### Option C: Manuelle Ausgabe
Falls weder CLI noch MCP verfügbar: Findings als Markdown ausgeben mit dem Hinweis, den Text manuell als PR-Kommentar einzufügen.

---

## 5. NACH DEM REVIEW

**Bei APPROVED oder APPROVED WITH SUGGESTIONS:**
```
Review abgeschlossen.
PR kann gemergt werden: https://github.com/timlohse1104/tilloh.dev/compare/main...feature/[branch]

Nach dem Merge:
  git checkout main
  git pull origin main
  git branch -d feature/[branch]
```

**Bei CHANGES REQUESTED:**
```
X Blocker müssen behoben werden:
[Liste der Blocker]

Nach den Fixes: /review-pr erneut ausführen.
```

---

## Verwendung

```
# Review für aktuellen Branch
/review-pr

# Review für spezifischen Branch
/review-pr feature/my-feature

# Review mit PR-Nummer
/review-pr 42
```
