# /tillohdev-validate — Playwright MCP Validierung (tilloh.dev)

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


E2E-Tests für das Feature auf dem aktuellen Branch.
Kann standalone oder als Teil des `/tillohdev-feature` Workflows genutzt werden.

> **Voraussetzungen**:
> - Dev-Server läuft: `npm run dev`
> - Playwright MCP konfiguriert

---

## 0. Vorbereitung

### Bestehende E2E-Tests prüfen

Bevor neue Szenarien geschrieben werden: prüfen ob bestehende Playwright-Tests durch
die Feature-Änderungen betroffen sind.

```bash
# Bestehende E2E-Tests finden
find . -name "*.spec.ts" -path "*/e2e/*" -o -name "*.spec.ts" -path "*/playwright/*" 2>/dev/null

# Geänderte Dateien analysieren
git diff main...HEAD --name-only
```

Für jede geänderte Route, Komponente oder API:
- Gibt es bestehende Playwright-Tests die diese Route/Komponente testen?
- Wurden Navigation oder Layout geändert? → Bestehende Navigations-Tests anpassen
- Wurden bestehende API-Endpunkte geändert? → Bestehende API-Szenarien prüfen
- Wurden i18n-Keys umbenannt? → Bestehende Text-Assertions aktualisieren

Bestehende Tests reparieren **bevor** neue Szenarien geschrieben werden.

### Setup

```bash
BRANCH=$(git branch --show-current)
FEATURE=$(echo $BRANCH | sed 's/feature\///')
echo "Validierung für: $FEATURE"

curl -s -o /dev/null -w "%{http_code}" http://localhost:5173 | grep -q "200" \
  && echo "✅ Frontend erreichbar" || echo "❌ Bitte npm run dev starten"

curl -s http://localhost:61154/v1/health 2>/dev/null \
  && echo "✅ Backend erreichbar" || echo "⚠️  Backend nicht erreichbar"
```

---

## S1: Route-Erreichbarkeit

```
navigate → http://localhost:5173/$FEATURE
screenshot → "s1-route"
```
Prüfe: Keine 404, kein JS-Error, Feature-Inhalt sichtbar.

---

## S2: Navigation

```
navigate → http://localhost:5173
screenshot → "s2-home"
Suche: Navigation-Link zu $FEATURE sichtbar?
click → Navigation-Link
screenshot → "s2-nav-clicked"
```
Prüfe: Feature im Navigationsmenü, Klick navigiert korrekt.

---

## S3: i18n — Deutsch und Englisch (PFLICHT)

```
navigate → http://localhost:5173/$FEATURE
screenshot → "s3-de"
```
Prüfe Deutsch: Alle UI-Texte sichtbar? Kein `[missing: ...]` Placeholder?

```
Sprache auf Englisch wechseln (Settings oder Sprachumschalter)
navigate → http://localhost:5173/$FEATURE
screenshot → "s3-en"
```
Prüfe Englisch: Texte übersetzt? Kein `[missing: ...]`?

Falls kein Sprachumschalter auf der Seite: Settings-Route aufrufen, Sprache wechseln, zurücknavigieren.

---

## S4: Loading State

```
navigate → http://localhost:5173/$FEATURE
screenshot direkt nach navigate → "s4-loading"
```
Prüfe: Loading-Indikator (Skeleton, Spinner) sichtbar während Daten laden?

---

## S5: Mobile Responsiveness

```
Viewport → 390x844 (iPhone 14)
navigate → http://localhost:5173/$FEATURE
screenshot → "s5-mobile"
```
Prüfe: Kein horizontaler Overflow, Inhalte lesbar, Buttons erreichbar.

```
Viewport zurück → 1280x800
```

---

## S6: Error State (Backend-Features)

```
navigate → http://localhost:5173/$FEATURE
Backend-Request blockieren / simulieren
Feature-Hauptaktion auslösen
screenshot → "s6-error"
```
Prüfe: Nutzerfreundliche Fehlermeldung sichtbar, kein weißer Bildschirm, kein JS-Error.

---

## S7-Sn: Akzeptanzkriterien

Für jedes Akzeptanzkriterium aus Phase 1:
```
navigate → http://localhost:5173/$FEATURE
[Interaktionen die das Kriterium testen]
screenshot → "an-[kurzname]"
Erwartung: [Was zu sehen sein soll]
```

---

## Validierungsbericht

```
PLAYWRIGHT VALIDIERUNG — [Feature]
══════════════════════════════════════════════════════════
S1 Route-Erreichbarkeit:    ✅ / ❌
S2 Navigation:              ✅ / ❌
S3 i18n (de + en):          ✅ / ❌  ← PFLICHT
S4 Loading State:           ✅ / ❌
S5 Mobile Responsiveness:   ✅ / ❌
S6 Error State:             ✅ / ❌ / ⏭️

AKZEPTANZKRITERIEN
A1 [Kriterium 1]:           ✅ / ❌
A2 [Kriterium 2]:           ✅ / ❌
A3 [Kriterium 3]:           ✅ / ❌
──────────────────────────────────────────────────────────
GESAMT: X/Y Szenarien bestanden
══════════════════════════════════════════════════════════
```

Bei ❌: Fehler beheben, betroffenes Szenario wiederholen.
Erst bei allen ✅ → Weiter mit Feature Walkthrough (Phase 6 in `/tillohdev-feature`) oder `/tillohdev-commit-push`.
