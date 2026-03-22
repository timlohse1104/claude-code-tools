# /tillohdev-feature — Ganzheitlicher Feature-Entwicklungsprozess für tilloh.dev

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


Dieser Command orchestriert den vollständigen Lifecycle einer neuen Feature-Idee:
Discovery → Planung → Implementierung → QA → Playwright-Validierung → Walkthrough → Review → Merge.

> **Ausschließlich für das tilloh.dev Repository.**
> Vor dem Start sicherstellen, dass `npm run dev` läuft.

---

## ⚙️ Voraussetzungen prüfen

```bash
test -f CLAUDE.md && grep -q "tilloh.dev" CLAUDE.md || echo "FEHLER: Nicht im tilloh.dev Root"
git branch --show-current
git status --short
```

Falls uncommittete Änderungen auf `main` → Nutzer darauf hinweisen und abbrechen.
Falls bereits auf einem Feature-Branch → Fragen ob weitergemacht werden soll.

---

## 🔍 PHASE 1: Discovery — Informationen sammeln

Stelle die Fragen **einzeln nacheinander** und warte jeweils auf die Antwort.

### Frage 1 — Feature-Titel
```
Wie soll das neue Feature heißen?
(Kurzname, wird als Branch-Name und Dateiname verwendet, z.B. "weather", "recipe-box")
```

### Frage 2 — Feature-Beschreibung
```
Beschreibe das Feature in 2-3 Sätzen: Was macht es? Wer nutzt es?
```

### Schritt 3 — Architektur-Scope (automatisch ableiten)

Leite den Scope direkt aus der Beschreibung ab und begründe kurz:
- Werden Daten dauerhaft gespeichert? → MongoDB → Scope 3
- Gibt es einen API-Proxy oder externe Dienste? → Backend nötig → Scope 2 oder 3
- Rein UI/interaktiv ohne Persistenz? → Scope 1

Teile dem Nutzer mit welchen Scope du abgeleitet hast und warum.
Nur bei echter Unklarheit nachfragen.

### Schritt 4 — Feature-Detailklärung (EINZELN nachfragen)

Analysiere die Beschreibung und identifiziere alle offenen Detailfragen.
Stelle sie **eine nach der anderen** — warte jeweils auf die Antwort bevor du die nächste stellst.
Behalte intern eine Liste welche Fragen schon beantwortet sind.

Typische Detailfragen je nach Feature-Typ:

**API/Extern:**
- Welche konkrete API soll genutzt werden? (ggf. Empfehlung machen)
- Ist ein API-Key vorhanden oder wird einer ohne Key bevorzugt?

**Datenspeicherung:**
- Wie wird zwischen Cloud und localStorage unterschieden? (Login-Status, expliziter Toggle?)
- Eigene Collection oder bestehende Infrastruktur (z.B. Keystore)?

**UI/Interaktion:**
- Wie wird die Hauptinteraktion ausgelöst? (Klick, Hover, Auto?)
- Was genau wird angezeigt? (Felder, Format, Einheiten?)
- Verhalten bei fehlendem/noch nicht gesetztem Zustand?

**Integration:**
- In welche bestehende Route wird integriert oder ist es eine neue Route?
- Gibt es Abhängigkeiten zu bestehenden Features (Settings, Auth, Keystore)?

Fahre erst mit Frage 6 fort wenn alle Detailfragen beantwortet sind.

### Frage 5 — Externe Abhängigkeiten
```
Benötigt das Feature externe APIs oder Dienste?
Falls ja: Welche? Hast du bereits einen API Key?
```

### Frage 6 — Authentifizierung
```
Soll das Feature:
  [1] Öffentlich zugänglich sein (@Public decorator im Backend)
  [2] Nur für eingeloggte Nutzer (Identifier-basierte Auth)
  [3] Nur im Admin-Bereich
```

### Frage 7 — i18n Keys (PFLICHT)
```
i18n ist in tilloh.dev immer Pflicht (de + en).
Nenne die wichtigsten UI-Texte des Features, damit ich die Keys schon planen kann.
Beispiel: Seitentitel, Button-Labels, Fehlermeldungen, leere Zustände.
```

### Frage 8 — Akzeptanzkriterien
```
Beschreibe 3-5 konkrete Akzeptanzkriterien — was muss funktionieren,
damit das Feature "fertig" ist? Diese werden als Playwright-Testszenarien verwendet.

Beispiel:
- Die Seite ist unter /weather erreichbar
- Der aktuelle Standort wird automatisch ermittelt
- Temperatur und Wetterlage werden korrekt angezeigt
- Bei API-Fehler erscheint eine Fehlermeldung
```

### Feature-Steckbrief zur Bestätigung

```
╔══════════════════════════════════════════════════════╗
║          FEATURE STECKBRIEF — [FEATURE_NAME]         ║
╠══════════════════════════════════════════════════════╣
║ Name:          [feature-name]                        ║
║ Branch:        feature/[feature-name]                ║
║ Scope:         [Frontend only / +Backend / +MongoDB] ║
║ Auth:          [Public / Identifier / Admin]         ║
║ Extern:        [Ja: API-Name / Nein]                 ║
║ i18n:          Ja (de + en) — Pflicht                ║
╠══════════════════════════════════════════════════════╣
║ BESCHREIBUNG:                                        ║
║ [Beschreibung aus Frage 2]                           ║
╠══════════════════════════════════════════════════════╣
║ GEPLANTE i18n-KEYS ([feature-name].*):               ║
║ · [feature-name].title                               ║
║ · [feature-name].error.loading                       ║
║ · [feature-name].empty                               ║
║ · [weitere Keys aus Frage 6]                         ║
╠══════════════════════════════════════════════════════╣
║ AKZEPTANZKRITERIEN:                                  ║
║ ✓ [Kriterium 1]                                      ║
║ ✓ [Kriterium 2]                                      ║
║ ✓ [Kriterium 3]                                      ║
╚══════════════════════════════════════════════════════╝
```

Frage: **"Soll ich mit diesem Plan fortfahren? (ja / anpassen)"**

---

## 📋 PHASE 2: Planung — Implementierungsplan erstellen

### 2a. Referenz-Docs laden

Lese immer folgende Docs als Kontext:
- `CLAUDE.md` (Coding Conventions, zwingend)
- `docs/shared/toggle-system.md`
- `docs/shared/route-page-pattern.md`
- `docs/shared/i18n.md` (i18n ist immer Pflicht)

Zusätzlich nach Scope:
- Scope 1: `docs/features/uno-sort.md`
- Scope 2: `docs/features/food-scan.md` + `docs/shared/nx-library-scaffold.md`
- Scope 3: `docs/features/jokes.md` + `docs/shared/nx-library-scaffold.md` + `docs/shared/keystore-persistence.md`

Falls localStorage: `docs/shared/stores.md`
Falls Auth: `docs/shared/auth-guard.md`

### 2b. Implementierungsplan

```
## IMPLEMENTIERUNGSPLAN: [Feature-Name]

### Branch: feature/[feature-name]

### Neue Dateien

#### Backend (Scope 2/3)
- [ ] backend/libs/[name]/src/lib/[name].module.ts
- [ ] backend/libs/[name]/src/lib/[name].controller.ts
- [ ] backend/libs/[name]/src/lib/[name].service.ts
- [ ] backend/libs/[name]/src/lib/[name]-mongodb.service.ts  (Scope 3)
- [ ] backend/libs/[name]/src/lib/schema/[name].schema.ts    (Scope 3)
- [ ] backend/libs/[name]/src/lib/dto/[name].dto.ts
- [ ] backend/libs/[name]/src/index.ts
- [ ] backend/libs/[name]/project.json + tsconfig*.json + jest.config.ts + .eslintrc.json

#### Frontend
- [ ] frontend/src/routes/[name]/+page.svelte
- [ ] frontend/src/routes/[name]/+page.ts
- [ ] frontend/src/lib/api/[name].api.ts          (falls Backend)
- [ ] frontend/src/lib/types/[name].dto.ts
- [ ] frontend/src/lib/components/[Name]*.svelte
- [ ] frontend/src/lib/util/stores/store-[name].ts (falls localStorage)

### Zu modifizierende Dateien
- [ ] backend/apps/tilloh-dev/src/main.ts          (Module registrieren)
- [ ] backend/tsconfig.base.json                   (Path alias)
- [ ] backend/apps/tilloh-dev/src/env.validation.ts (Env vars)
- [ ] frontend/src/lib/config/de.json              (i18n — PFLICHT)
- [ ] frontend/src/lib/config/en.json              (i18n — PFLICHT)
- [ ] frontend/src/lib/util/toggles.ts
- [ ] frontend/src/lib/config/toggles.config.ts
- [ ] frontend/src/lib/config/routes.config.ts
- [ ] frontend/src/lib/components/Navigation.svelte
- [ ] CHANGELOG.md
- [ ] docs/features/[name].md

### Geplante i18n-Keys
de.json + en.json unter "[name].*":
[Alle Keys aus Phase 1, Frage 6 auflisten]

### Geplante API Endpoints (falls Backend)
| Method | Path | Auth | Beschreibung |
|--------|------|------|--------------|
| ...    | ...  | ...  | ...          |

### Implementierungsreihenfolge
1. Branch erstellen
2. Backend: NX Library Scaffold (Scope 2/3)
3. Backend: DTOs mit class-validator definieren
4. Backend: Schema + MongoDB Service (Scope 3)
5. Backend: Service + Controller + vollständige Swagger-Docs
6. Backend: AppModule registrieren + Env Vars
7. Backend: Unit Tests
8. Frontend: Types in /lib/types/ definieren
9. Frontend: API Client in /lib/api/
10. Frontend: i18n-Keys in de.json UND en.json — GLEICHZEITIG
11. Frontend: Stores in /lib/util/stores/ (falls localStorage)
12. Frontend: Svelte 5 Komponenten
13. Frontend: Route +page.svelte + +page.ts mit svelte:head title
14. Frontend: Toggle + Navigation
15. Dokumentation: docs/features/[name].md
16. CHANGELOG.md aktualisieren
```

Frage: **"Soll ich mit der Implementierung beginnen? (ja / plan anpassen)"**

---

## 🛠️ PHASE 3: Implementierung

### 3a. Branch erstellen
```bash
git checkout main && git pull origin main
git checkout -b feature/[feature-name]
```

### 3b. Backend (Scope 2/3)

Folge exakt `docs/shared/nx-library-scaffold.md`. Referenz: `backend/libs/jokes/`.

**NestJS/Fastify Best Practices:**
- DTOs immer mit `class-validator` Dekoratoren (`@IsString()`, `@IsNotEmpty()`, `@IsOptional()` etc.)
- Response-Typen immer als DTO-Klasse typisieren, nie `any`
- Jeden Endpunkt mit vollständigen Swagger-Dekoratoren: `@ApiTags`, `@ApiOperation`, `@ApiResponse` (auch für 4xx/5xx)
- Fastify-kompatibel: `@Res({ passthrough: true })` wenn `reply` nötig, sonst Return-Wert nutzen
- Fehler nie mit internen Details zurückgeben — NestJS HttpException verwenden
- Cron-Jobs (`@Cron`) für regelmäßige Hintergrundaufgaben statt Polling
- Rate Limiting bereits vorhanden (500 req/5min) — bei sensiblen Endpunkten prüfen ob reicht
- Pino Logger für strukturiertes Logging nutzen (`this.logger = new Logger(ClassName.name)`)
- Keine `console.log` Statements — ausschließlich NestJS Logger

**MongoDB/Mongoose Best Practices:**
- Schema immer mit `{ timestamps: true }` definieren (`createdAt`, `updatedAt` automatisch)
- `@Prop({ required: true })` für Pflichtfelder — nie stumme Fehlschläge
- Indexes für häufig abgefragte Felder: `@Prop({ index: true })`
- Unique Constraints direkt im Schema: `@Prop({ unique: true })`
- `lean()` bei Read-Only Queries nutzen für bessere Performance
- Keine ObjectId-Strings manuell bauen — `new Types.ObjectId()` oder Mongoose übernehmen lassen
- Fehlerfall DB-Ausfall: immer try/catch in MongoDB-Services mit sinnvollem Fallback

**Naming Conventions:**
- Service (Business Logic): `listItems`, `createItem`, `updateItem`, `removeItem`
- MongoDB Service (DB Layer): `findAll`, `findById`, `create`, `update`, `remove`
- DTOs: `CreateItemDto`, `UpdateItemDto`, `ItemResponseDto`

### 3c. Frontend

**Svelte 5 Rune-Struktur** (strikt einhalten):
```svelte
<script lang="ts">
  // 1. IMPORTS
  import { onMount } from 'svelte';
  import type { MyType } from '$lib/types/my.dto';

  // 2. PROPS
  let { prop1, prop2 = 'default', bindable = $bindable() } = $props();

  // 3. CONST (non-reactive)
  const MAX_ITEMS = 50;

  // 4. STATE
  let items = $state<MyType[]>([]);
  let loading = $state(false);
  let error = $state<string | null>(null);

  // 5. DERIVED
  const hasItems = $derived(items.length > 0);
  const filteredItems = $derived(items.filter(i => i.active));

  // 6. EFFECTS ($effect nur wenn wirklich nötig — nicht für Datenfetching)
  $effect(() => {
    // Cleanup zurückgeben wenn nötig
    return () => { /* cleanup */ };
  });

  // 7. LIFECYCLE
  onMount(async () => {
    await loadData();
  });

  // 8. FUNCTIONS (immer Lambda Const)
  const loadData = async () => { ... };
  const handleSubmit = () => { ... };
</script>
```

**Svelte 5 Best Practices:**
- `$effect` NICHT für Datenfetching nutzen → `onMount` verwenden
- `$effect` NICHT für Berechnungen → `$derived` verwenden
- Arrays/Objekte in `$state` direkt mutieren (kein Re-Assign nötig): `items.push(newItem)`
- `$derived` ist lazy — nur berechnen wenn abhängige States sich ändern
- Keine `$:` Reaktivität mehr — ausschließlich Svelte 5 Runes
- Keine Stores für lokale Komponenten-State → `$state` nutzen
- Props mit Interface typisieren für TypeScript-Sicherheit:
  ```ts
  interface Props { label: string; count?: number; }
  let { label, count = 0 }: Props = $props();
  ```
- `$bindable()` nur wenn bidirektionale Datenbindung wirklich benötigt wird

**SvelteKit Best Practices:**
- Jede Route hat `<svelte:head><title>[Feature] | tilloh.dev</title></svelte:head>`
- `+page.ts` für Client-Side Data Loading (kein SSR in diesem Projekt)
- `export const ssr = false` explizit setzen bei Client-Only Features
- Keine hardcodierten URLs — `$page.url` oder Route-Konstanten aus `routes.config.ts`
- Error States immer abfangen und nutzerfreundlich anzeigen (nie leerer Bildschirm)
- Loading States immer implementieren (Carbon Skeleton oder Spinner)
- Lazy Loading für schwere Komponenten: `{#await import('./HeavyComponent.svelte')}...{/await}`

**i18n — IMMER PFLICHT:**
- Kein einziger hartcodierter deutscher oder englischer String in Svelte-Dateien
- Keys GLEICHZEITIG in `de.json` UND `en.json` anlegen — nie nur eine Sprache
- Key-Schema: `[featureName].[kontext].[bezeichnung]`
  - `jokes.title` = "Witze" / "Jokes"
  - `jokes.error.loading` = "Fehler beim Laden" / "Error loading"
  - `jokes.empty.state` = "Keine Witze gefunden" / "No jokes found"
  - `jokes.button.next` = "Nächster" / "Next"
- Fehler- und Ladetext-Keys immer mitführen, nicht nur Happy-Path-Texte
- Pluralisierung beachten wo nötig

**HTML/CSS Best Practices:**
- Semantisches HTML: `<main>`, `<section>`, `<article>`, `<header>`, `<nav>`, `<button>` statt generische `<div>`
- Jeder `<button>` hat entweder Text-Content oder `aria-label`
- Bilder immer mit `alt`-Attribut (leer `alt=""` bei dekorativen Bildern)
- Interaktive Elemente erreichbar via Keyboard (Tab-Reihenfolge, Enter/Space)
- Keine `tabindex` > 0 — natürliche DOM-Reihenfolge nutzen
- Carbon Design System Komponenten nutzen wo möglich (bereits integriert)
- CSS Scoped in Svelte-Komponenten — kein globales CSS außer in `app.css`
- CSS Custom Properties für wiederverwendbare Werte
- Responsive Design: Mobile-First Ansatz, keine hardcodierten Pixel-Breiten

**TypeScript Best Practices:**
- Kein `any` — `unknown` wenn Typ wirklich unbekannt, dann narrowen
- Alle API-Response-Typen in `/lib/types/` definieren
- Union Types für Zustände: `'idle' | 'loading' | 'success' | 'error'`
- Keine Type Assertions (`as MyType`) außer nach echten Checks
- Optional Chaining (`?.`) und Nullish Coalescing (`??`) statt `if`-Kaskaden

### 3d. Feature-Dokumentation

Erstelle `docs/features/[name].md` nach dem Schema von `hitstar.md`:
- Architecture Pattern (1-2 Sätze)
- Backend-Sektion: Module Structure, API Endpoints, Env Variables
- Frontend-Sektion: Route, State Machine (falls vorhanden), API Client, Types, Stores
- i18n Key Prefix
- Toggle Configuration
- Shared Infrastructure Dependencies
- Key Implementation Notes (Fallstricke, wichtige Entscheidungen, Migration Notes)

### 3e. CHANGELOG.md aktualisieren

```markdown
## [Unreleased]
### Added
- [feature-name] [Beschreibung der neuen Funktionalität]
```

---

## 🔬 PHASE 4: QA

Führe `/tillohdev-qa` aus. Alle Checks müssen bestanden sein bevor weitergemacht wird.

Erst bei `GESAMT: BESTANDEN ✅` → weiter mit Phase 5.

---

## 🎭 PHASE 5: Playwright MCP Validierung

Führe `/tillohdev-validate` aus. Die Akzeptanzkriterien aus Phase 1 sind die Grundlage für die Szenarien A1-An.

Erst bei `GESAMT: X/Y Szenarien bestanden` (alle ✅) → weiter mit Phase 6.

---

## 🗺️ PHASE 6: Feature Walkthrough — Vorzeigen vor dem PR

**Bevor der PR erstellt wird**, präsentiere das implementierte Feature strukturiert.
Ziel: Du kannst es lokal nachvollziehen und gezielt testen.

### 6a. Zusammenfassung was umgesetzt wurde

Gib eine klare, nicht-technische Übersicht:
```
FEATURE WALKTHROUGH — [Feature-Name]
══════════════════════════════════════════════════════════

WAS WURDE GEBAUT?
  [2-3 Sätze: Was tut das Feature aus Nutzersicht]

WO FINDEST DU ES?
  → http://localhost:5173/[feature-name]
  → Auch über Navigation: [Navigationsmenü-Name]

WAS KANNST DU TESTEN?
  ✓ [Konkrete Aktion 1 die du ausführen kannst]
  ✓ [Konkrete Aktion 2]
  ✓ [Edge Case den du testen solltest]
  ✓ [Fehlerfall: Was passiert wenn X nicht funktioniert?]
```

### 6b. Technische Umsetzung erklären

Erkläre die wesentlichen Implementierungsentscheidungen:

```
TECHNISCHE UMSETZUNG
══════════════════════════════════════════════════════════

SCOPE: [Frontend only / Frontend+Backend / Frontend+Backend+MongoDB]

NEUE DATEIEN ([Anzahl]):
  Backend:
    · backend/libs/[name]/... → [Was macht dieses Modul?]
  Frontend:
    · frontend/src/routes/[name]/+page.svelte → [Hauptseite]
    · frontend/src/lib/api/[name].api.ts → [API-Calls zu: ...]
    · frontend/src/lib/types/[name].dto.ts → [Typen für: ...]
    · [weitere relevante Dateien]

WICHTIGE ENTSCHEIDUNGEN:
  · [Entscheidung 1 + kurze Begründung]
  · [Entscheidung 2 + kurze Begründung]

i18n:
  · [X] Keys in de.json, [X] Keys in en.json
  · Key-Prefix: [feature-name].*

BEKANNTE EINSCHRÄNKUNGEN / OFFENE PUNKTE:
  · [Falls etwas bewusst nicht umgesetzt wurde]
  · [Falls etwas noch verbessert werden könnte]
```

### 6c. Lokales Testen — Schritt-für-Schritt Anleitung

```
SO TESTEST DU ES LOKAL
══════════════════════════════════════════════════════════

Voraussetzung: npm run dev läuft im Root

1. Gehe zu http://localhost:5173/[feature-name]
   → Du siehst: [Was du sehen solltest]

2. [Konkrete Testaktion]
   → Erwartetes Ergebnis: [Was passieren soll]

3. [Nächste Testaktion]
   → Erwartetes Ergebnis: [Was passieren soll]

4. Fehlerfall testen:
   → [Wie du den Fehlerfall auslösen kannst]
   → Erwartetes Ergebnis: [Fehlermeldung sichtbar]

5. i18n testen:
   → Sprache in Settings auf Englisch wechseln
   → Alle Texte sollten auf Englisch erscheinen
   → Kein "[missing: ...]" Placeholder sichtbar

Alle Playwright-Screenshots aus Phase 5 findest du im lokalen Verzeichnis.
```

Warte auf Bestätigung vom Nutzer:
**"Hast du das Feature lokal getestet und bist zufrieden? (ja / Änderungen nötig)"**

Bei Änderungen: zurück zu Phase 3, Änderungen implementieren, QA + Playwright + Walkthrough wiederholen.
Erst bei ausdrücklichem "ja" → weiter zu Phase 7.

---

## 🔍 PHASE 7: Code Review

Führe `/tillohdev-review-pr` auf dem lokalen Branch aus (vor dem Commit).

Erst bei `VERDICT: APPROVED` oder `APPROVED WITH SUGGESTIONS` → weiter mit Phase 8.
Bei `CHANGES REQUESTED`: Blocker beheben, dann Review wiederholen.

---

## 🚀 PHASE 8: Commit & Push + PR

```bash
# Nur geänderte und neue Dateien des Features hinzufügen (kein blindes -A)
git diff --name-only main...HEAD | xargs git add
git ls-files --others --exclude-standard | xargs git add
git status

# Pre-commit Hook läuft: Lint + Tests
git commit -m "add [feature-name] feature"
# Post-commit Hook: Gitmoji hinzufügen (✨ für "add")

git push origin feature/[feature-name]
```

PR als **Draft** anlegen — GitHub Actions Workflows laufen erst los wenn der Draft
manuell auf "Ready for Review" gesetzt wird:

```bash
gh pr create --draft --title "✨ Add [feature-name]" --base main
```

Anschließend `/tillohdev-review-pr` ausführen um Findings als Kommentar in den Draft-PR einzutragen.
Erst nach APPROVED → PR auf "Ready for Review" setzen:

```bash
gh pr ready
```

PR-Vorlage (als `--body` übergeben):
```
PULL REQUEST (DRAFT)
══════════════════════════════════════════════════════════
Titel: ✨ Add [Feature-Name]

## Was wurde gebaut?
[Feature-Beschreibung aus Phase 1]

## Akzeptanzkriterien
- [x] [Kriterium 1]
- [x] [Kriterium 2]
- [x] [Kriterium 3]

## i18n
- [x] de.json aktualisiert ([X] Keys)
- [x] en.json aktualisiert ([X] Keys)

## Testing
- [x] Backend Unit Tests: X/X
- [x] Frontend Unit Tests: X/X
- [x] Playwright E2E: X/X Szenarien
- [x] Manuell lokal getestet

## Screenshots
[Playwright-Screenshots aus Phase 5 einfügen]

## Dokumentation
- [x] docs/features/[name].md
- [x] CHANGELOG.md
══════════════════════════════════════════════════════════
PR URL: https://github.com/timlohse1104/tilloh.dev/compare/main...feature/[feature-name]
```

---

## 📊 Abschluss-Summary

```
╔══════════════════════════════════════════════════════╗
║     FEATURE FERTIG — [Feature-Name]                  ║
╠══════════════════════════════════════════════════════╣
║ Branch:         feature/[feature-name]               ║
║ Neue Dateien:   [Anzahl]                             ║
║ Geänderte:      [Anzahl]                             ║
║ Backend Tests:  [X] bestanden                        ║
║ Frontend Tests: [X] bestanden                        ║
║ Playwright:     [X/Y] Szenarien                      ║
║ i18n:           [X] Keys de + en                     ║
║ Manuell OK:     ✅ (Walkthrough bestätigt)            ║
╠══════════════════════════════════════════════════════╣
║ NÄCHSTE SCHRITTE                                     ║
║ → PR auf GitHub reviewen mit /review-pr              ║
║ → Nach Merge: git checkout main && git pull          ║
║ → Deployment: automatisch via GitHub Actions         ║
╚══════════════════════════════════════════════════════╝
```
