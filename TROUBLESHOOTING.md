# Troubleshooting Guide - GitHub Profile Setup

Häufige Probleme und Lösungen beim Setup des GitHub-Profils.

## 🔴 GitHub Actions Probleme

### Problem 1: "Permission denied" oder "403 Forbidden"

**Symptome:**
- Workflow schlägt fehl mit Permission-Fehler
- Kann nicht in `output` Branch schreiben

**Lösung:**
1. Gehe zu Repository **Settings** → **Actions** → **General**
2. Scrolle zu **Workflow permissions**
3. Wähle **"Read and write permissions"**
4. Aktiviere **"Allow GitHub Actions to create and approve pull requests"**
5. Klicke **Save**
6. Starte den Workflow erneut

### Problem 2: Workflow läuft nicht automatisch

**Symptome:**
- Cron-Job triggert nicht alle 12 Stunden
- Workflow erscheint nicht im Actions Tab

**Lösung:**
1. **GitHub Actions muss aktiviert sein:**
   - Settings → Actions → General
   - Wähle "Allow all actions and reusable workflows"
   
2. **Cron-Jobs laufen nur auf default Branch:**
   - Stelle sicher, dass `main.yml` auf dem `main` Branch liegt
   - Der Workflow muss mindestens einmal manuell gestartet werden
   
3. **Erste Ausführung manuell triggern:**
   - Actions Tab → "Generate Snake Animation" → "Run workflow"

### Problem 3: Output Branch `output` existiert nicht

**Symptome:**
- Snake SVG URL zeigt 404
- Branch `output` nicht in Branch-Liste

**Lösung:**
- Der `output` Branch wird **automatisch** beim ersten Workflow-Run erstellt
- Warte bis der Workflow erfolgreich durchgelaufen ist
- Überprüfe: Repository → Branches → Sollte `output` Branch zeigen
- Falls nicht: Workflow-Logs prüfen auf Fehler

### Problem 4: Snake Animation zeigt nicht an

**Symptome:**
- README zeigt kaputtes Bild-Icon
- SVG lädt nicht

**Lösung A - Cache-Problem:**
```
# Verwende Raw-URL mit Cache-Buster:
https://raw.githubusercontent.com/Praeto89/Praeto89/output/snake.svg?t=12345
```

**Lösung B - Branch-Name prüfen:**
```
# Überprüfe in .github/workflows/main.yml:
target_branch: output  # Muss mit URL übereinstimmen
```

**Lösung C - Warte auf erste Generation:**
- Snake wird erst nach erstem erfolgreichen Workflow generiert
- Kann 1-2 Minuten dauern
- Danach README-Kommentar entfernen und neu committen

### Problem 5: Workflow schlägt fehl mit "Error: HttpError: Resource not accessible by integration"

**Symptome:**
- Fehler beim Push zum `output` Branch
- Workflow zeigt roten X

**Lösung:**
1. Gehe zu **Settings** → **Actions** → **General**
2. Unter **Workflow permissions**:
   - ✅ Read and write permissions
   - ✅ Allow GitHub Actions to create and approve pull requests
3. **Wichtig**: Repository muss **öffentlich** sein für kostenlose Actions
4. Re-run Workflow

## 🔴 README Darstellungs-Probleme

### Problem 6: Canvas-Divider zeigen nicht an

**Symptome:**
- Leinwand-Textur fehlt zwischen Sections
- Broken Image Icons

**Lösung:**
```markdown
# Überprüfe Pfad (relativ zum Repository-Root):
<img src="./assets/canvas-bg.png" alt="" width="100%">

# NICHT:
<img src="assets/canvas-bg.png">  # Fehlt ./
<img src="/assets/canvas-bg.png"> # Absoluter Pfad funktioniert nicht
```

### Problem 7: Header-Bild zeigt nicht an

**Lösung:**
1. **Datei existiert**: Überprüfe ob `assets/header.png` committed wurde
2. **Dateiname korrekt**: Exakt `header.png` (case-sensitive auf Linux)
3. **Pfad korrekt**: `./assets/header.png` im README
4. **Dateigröße**: Nicht größer als 25MB (GitHub Limit)
5. **Format**: PNG oder JPG (kein WebP)

### Problem 8: Opacity bei Canvas-Divider funktioniert nicht

**Symptome:**
- `style="opacity: 0.8"` wird ignoriert
- Divider zu prominent

**Lösung:**
GitHub's Markdown-Renderer unterstützt limitiertes CSS. Alternativen:

**Option A - Bild extern mit Opacity vorbereiten:**
- Exportiere Canvas-Textur bereits mit 80% Opacity
- Speichere als `canvas-bg-subtle.png`

**Option B - HTML img mit inline style (limitiert):**
```html
<img src="./assets/canvas-bg.png" width="100%" style="opacity:0.8;">
```

**Option C - Picture Element mit Filter:**
```html
<picture>
  <img src="./assets/canvas-bg.png" width="100%" style="filter:opacity(0.8);">
</picture>
```

## 🔴 GitHub Stats Card Probleme

### Problem 9: Stats Cards zeigen nicht an

**Symptome:**
- Broken Image bei GitHub Stats
- "Something went wrong" Fehlermeldung

**Lösung:**
1. **Rate Limiting**: Zu viele Requests
   - Warte 1 Stunde
   - Verwende `&cache_seconds=3600` Parameter
   
2. **Username falsch**:
   ```
   # Korrekt:
   ?username=Praeto89
   
   # NICHT:
   ?username=@Praeto89
   ?username=github.com/Praeto89
   ```

3. **Vercel Service Down**:
   - Prüfe: https://github-readme-stats.vercel.app/
   - Falls down: Warte oder verwende Alternative

### Problem 10: Custom Colors funktionieren nicht

**Symptome:**
- Stats Cards haben Standard-Farben
- Pastel-Farben werden ignoriert

**Lösung:**
Überprüfe URL-Encoding und Format:

```
# KORREKT (ohne #):
bg_color=fdfbf7
title_color=8B9DC3

# FALSCH:
bg_color=#fdfbf7  ❌ Kein # Symbol
title_color=8b9dc3  ✅ Case-insensitive, aber konsistent bleiben
```

## 🔴 Git & Push Probleme

### Problem 11: "fatal: repository not found"

**Lösung:**
```powershell
# Repository-Name prüfen (muss EXAKT Username sein):
git remote -v

# Falls falsch, neu setzen:
git remote remove origin
git remote add origin https://github.com/Praeto89/Praeto89.git
```

### Problem 12: "Authentication failed"

**Lösung:**
1. **Personal Access Token verwenden** (nicht Passwort):
   - GitHub → Settings → Developer settings → Personal access tokens
   - Generate new token (classic)
   - Scopes: `repo` (Full control)
   - Token kopieren
   
2. **Bei Push verwenden**:
   ```powershell
   git push https://TOKEN@github.com/Praeto89/Praeto89.git main
   ```

3. **Oder Git Credential Manager**:
   ```powershell
   git config --global credential.helper manager-core
   git push
   # Browser öffnet sich für OAuth
   ```

### Problem 13: Assets zu groß / Push schlägt fehl

**Symptome:**
- "file exceeds GitHub's file size limit"
- Push wird abgelehnt

**Lösung:**
1. **Bilder komprimieren**:
   - Header: Max 500KB (1200x300px)
   - Canvas-Textur: Max 200KB
   - Screenshot: Max 500KB
   
2. **Tools**:
   - TinyPNG.com für PNG-Kompression
   - Squoosh.app für moderne Kompression
   
3. **Git History bereinigen** (falls bereits committed):
   ```powershell
   git rm --cached assets/huge-file.png
   git commit -m "Remove large file"
   # Kleinere Version hinzufügen
   git add assets/huge-file.png
   git commit -m "Add compressed version"
   ```

## 🔴 DevIcon Probleme

### Problem 14: DevIcons laden nicht

**Symptome:**
- Broken images bei Tech Stack Icons

**Lösung:**
```markdown
# Aktualisiere CDN-URL (DevIcon v2):
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/javascript/javascript-original.svg">

# Oder verwende SimpleIcons:
<img src="https://cdn.simpleicons.org/javascript/F7DF1E">
```

## 🔴 Snake Animation Custom Colors

### Problem 15: Snake hat nicht Pastel-Farben

**Symptome:**
- Snake zeigt Standard GitHub-Grün
- Pastel-Palette wird nicht angewendet

**Lösung:**
Die aktuelle `Platane/snk@v3` Version unterstützt limitierte Color-Customization.

**Workaround - Custom Palette in Workflow:**

Ersetze in `.github/workflows/main.yml`:

```yaml
- name: Generate snake.svg
  uses: Platane/snk/svg-only@v3
  with:
    github_user_name: Praeto89
    outputs: |
      dist/snake.svg?palette=pastel
    
    # Custom color scheme (falls unterstützt)
    color_snake: "#C7CEEA"
    color_dots: "#B5EAD7,#FFD6BA,#B8D4E8,#C7CEEA"
```

**Alternative**: Verwende `github-dark` oder `github-light` Palette:
```yaml
outputs: |
  dist/snake.svg
  dist/snake-dark.svg?palette=github-dark
```

## 📞 Weitere Hilfe

### Workflow-Logs prüfen:
1. Repository → Actions Tab
2. Klicke auf fehlgeschlagenen Workflow-Run
3. Klicke auf Job-Name
4. Expandiere jeden Step für detaillierte Logs

### GitHub Community:
- [GitHub Community Forum](https://github.community/)
- [Platane/snk Issues](https://github.com/Platane/snk/issues)
- [GitHub Readme Stats Issues](https://github.com/anuraghazra/github-readme-stats/issues)

### Nützliche Links:
- [Shields.io Badge Generator](https://shields.io/)
- [DevIcon Icon List](https://devicon.dev/)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Markdown Guide](https://www.markdownguide.org/)

---

**Tipp**: Bei Problemen immer zuerst:
1. ✅ Browser-Cache leeren (Strg + F5)
2. ✅ Workflow-Logs prüfen
3. ✅ Repository public & Actions aktiviert
4. ✅ Permissions korrekt gesetzt
