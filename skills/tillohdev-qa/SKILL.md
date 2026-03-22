# /tillohdev-qa — Qualitätssicherung für den aktuellen Branch (tilloh.dev)

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


Führt alle QA-Schritte für den aktuellen Feature-Branch durch.
Kann unabhängig oder als Teil des `/tillohdev-feature` Workflows genutzt werden.

> **Voraussetzung**: Auf einem Feature-Branch befinden, nicht auf `main`.

---

## 0. Branch-Check

```bash
BRANCH=$(git branch --show-current)
[ "$BRANCH" = "main" ] && echo "FEHLER: QA nicht auf main ausführen" && exit 1
echo "QA für Branch: $BRANCH"
git diff main...HEAD --stat
```

---

## 1. TypeScript Type Check

```bash
cd frontend && npm run check 2>&1
```

Jeden Typ-Fehler sofort beheben, dann erneut bis sauber.
Besonders achten auf: kein `any`, keine blinden Type Assertions.

---

## 2. Linting

```bash
npm run lint 2>&1
```

`npm run lint -- --fix` für automatisch fixbare Fehler.
Prettier-Regel: `singleQuote: true` — alle Strings mit einfachen Anführungszeichen.

---

## 3. Backend Unit Tests

```bash
cd backend && nx run-many -t test 2>&1
```

Bei Fehlern: Implementierungsfehler oder veralteter Test? → Beheben.

---

## 4. Frontend Unit Tests

```bash
cd frontend && nx run-many -t test 2>&1
```

---

## 5. Build-Validierung

```bash
cd backend && npm run build 2>&1
cd ../frontend && npm run build 2>&1
```

---

## 6. i18n Vollständigkeitsprüfung — PFLICHT

i18n ist in tilloh.dev immer Pflicht. Jeder Key in `de.json` muss in `en.json` existieren und umgekehrt.

```bash
# Feature-Name aus Branch ermitteln
FEATURE=$(git branch --show-current | sed 's/feature\///')

# Keys vergleichen
node -e "
const de = require('./frontend/src/lib/config/de.json');
const en = require('./frontend/src/lib/config/en.json');
const feature = '$FEATURE';

if (!de[feature] && !en[feature]) {
  console.log('ℹ️  Kein Feature-spezifischer i18n-Key-Block gefunden');
  console.log('Falls das Feature i18n-Keys hat, prüfe manuell.');
  process.exit(0);
}

const deKeys = de[feature] ? Object.keys(de[feature]) : [];
const enKeys = en[feature] ? Object.keys(en[feature]) : [];

const missingInEn = deKeys.filter(k => !enKeys.includes(k));
const missingInDe = enKeys.filter(k => !deKeys.includes(k));

if (missingInEn.length > 0) console.error('❌ Fehlende EN Keys:', missingInEn);
if (missingInDe.length > 0) console.error('❌ Fehlende DE Keys:', missingInDe);
if (missingInEn.length === 0 && missingInDe.length === 0) {
  console.log('✅ i18n vollständig:', deKeys.length, 'Keys in de + en');
}
"
```

Zusätzlich manuell prüfen:
- Kein hartcodierter Deutsch/Englisch-String in `.svelte`-Dateien des Features
- Fehler-Keys vorhanden: `[feature].error.*`
- Leerer-Zustand-Key vorhanden: `[feature].empty` o.ä.

---

## 7. Svelte 5 Rune-Audit

```bash
# Prüfe auf $: Syntax (Svelte 4 — sollte nicht mehr vorkommen)
git diff main...HEAD -- '*.svelte' | grep -n '^\+.*\$:' | grep -v '^\+\+\+' \
  && echo "⚠️  Svelte 4 reaktive Deklarationen gefunden" \
  || echo "✅ Keine $: Syntax gefunden"

# Prüfe auf console.log
git diff main...HEAD | grep -n '^\+.*console\.log' | grep -v '^\+\+\+' \
  && echo "⚠️  console.log gefunden — bitte Logger nutzen" \
  || echo "✅ Kein console.log"

# Prüfe auf any
git diff main...HEAD -- '*.ts' '*.svelte' | grep -n '^\+.*: any' | grep -v '^\+\+\+' \
  && echo "⚠️  'any' Typ gefunden" \
  || echo "✅ Kein any gefunden"
```

---

## 8. Diff-Analyse

```bash
git diff main...HEAD
```

### Svelte 5 / SvelteKit
- [ ] Keine `$:` Reaktivität?
- [ ] `$effect` NICHT für Datenfetching (→ `onMount`)?
- [ ] `$effect` NICHT für Berechnungen (→ `$derived`)?
- [ ] Arrays direkt mutiert statt re-assigned?
- [ ] Props mit Interface typisiert?
- [ ] `<svelte:head>` mit `<title>` in jeder +page.svelte?
- [ ] Error States implementiert?
- [ ] Loading States implementiert?
- [ ] Alle Funktionen als Lambda Const?
- [ ] Rune-Reihenfolge: imports→props→const→state→derived→effects→lifecycle→functions?
- [ ] API-Calls nur in `/lib/api/`?
- [ ] Types nur in `/lib/types/`?

### i18n
- [ ] Kein einziger hartcodierter String in .svelte-Dateien?
- [ ] Alle Keys in de.json UND en.json?
- [ ] Fehler-/Ladetext-Keys vorhanden?

### Backend
- [ ] DTOs mit class-validator?
- [ ] Swagger vollständig (inkl. Error-Responses)?
- [ ] NestJS Logger statt console.log?
- [ ] HttpException für Fehler (keine internen Details)?
- [ ] Auth korrekt (@Public oder Guard)?
- [ ] Fastify-kompatibel (`@Res({ passthrough: true })`)?
- [ ] MongoDB Schema mit `{ timestamps: true }`?
- [ ] `lean()` bei Read-Only Queries?
- [ ] Naming Conventions eingehalten?

### HTML/CSS
- [ ] Semantische Elemente?
- [ ] Buttons mit Text oder aria-label?
- [ ] alt-Attribute bei Bildern?
- [ ] CSS scoped?
- [ ] Responsives Design?

### Sicherheit
- [ ] Keine Secrets im Code?
- [ ] Env Vars in env.validation.ts?
- [ ] Input-Validierung im Backend?

### tilloh.dev-spezifisch
- [ ] Toggle-Eintrag gesetzt?
- [ ] Route-Konfiguration ergänzt?
- [ ] Navigation aktualisiert?
- [ ] CHANGELOG.md aktualisiert?
- [ ] docs/features/[name].md erstellt?
- [ ] NX Path Alias in tsconfig.base.json?
- [ ] Module in AppModule registriert?

---

## 9. QA-Bericht

```
QA ERGEBNIS — [Branch] — [Datum]
══════════════════════════════════════════════
TypeScript Check:    ✅ / ❌ [Details]
Linting:             ✅ / ❌ [Details]
Backend Tests:       ✅ X/X / ❌ [Details]
Frontend Tests:      ✅ X/X / ❌ [Details]
Backend Build:       ✅ / ❌ [Details]
Frontend Build:      ✅ / ❌ [Details]
i18n Vollständigkeit: ✅ X Keys de+en / ❌ [fehlende Keys]
Svelte 5 Audit:      ✅ / ⚠️  [Details]
Code Conventions:    ✅ / ⚠️  [Abweichungen]
──────────────────────────────────────────────
GESAMT: BESTANDEN ✅ / FEHLGESCHLAGEN ❌
══════════════════════════════════════════════
```

Nur bei BESTANDEN → Weiter mit Playwright-Validierung (`/tillohdev-validate`).
