Schließe den aktuellen Feature-Branch ab: Prüfe ob der PR gemergt ist, wechsle zu main, aktualisiere ihn und lösche den Feature-Branch lokal.

## Schritte

### 1. Aktuellen Branch ermitteln
Führe `git branch --show-current` aus, um den Branch-Namen zu speichern.

Falls der aktuelle Branch `main` ist: Abbrechen und den User informieren, dass kein Feature-Branch aktiv ist.

### 2. PR-Status prüfen
Führe `gh pr view --json state,mergedAt --jq '"State: \(.state), MergedAt: \(.mergedAt)"'` aus.

- Falls der PR **nicht merged** ist (state ≠ `MERGED`): Abbrechen und den User informieren. Keinen Branch löschen.
- Falls kein PR existiert (Fehler): Den User fragen, ob der Branch trotzdem gelöscht werden soll. Ohne explizite Bestätigung abbrechen.

### 3. Zu main wechseln
```bash
git checkout main
```

### 4. Main aktualisieren
```bash
git pull
```

### 5. Feature-Branch lokal löschen
```bash
git branch -d <branch-name>
```

Falls `-d` fehlschlägt weil der Branch laut Git nicht vollständig gemergt ist (obwohl der PR merged war): `-D` verwenden und den User darauf hinweisen.

### 6. Stale Remote-Referenzen aufräumen
```bash
git remote prune origin
```

Gibt aus, welche verwaisten Tracking-Referenzen entfernt wurden.

### 7. Bestätigung
Gib eine kurze Zusammenfassung aus:
- Gelöschter Branch
- Aktueller Stand von main (letzter Commit-Hash + Nachricht)
- Entfernte Remote-Referenzen (falls vorhanden)

## Wichtige Regeln

- **Niemals** einen Branch löschen, dessen PR nicht gemergt ist — außer der User bestätigt es explizit
- **Kein** `git branch -D` ohne Hinweis an den User
- Immer zuerst zu `main` wechseln, bevor der Feature-Branch gelöscht wird
