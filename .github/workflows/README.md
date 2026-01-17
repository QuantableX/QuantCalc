# GitHub Actions Build Workflow

Dieser Workflow baut QuantCalc automatisch für **Windows**, **macOS** und **Linux**.

## 🚀 Wie funktioniert es?

Der Build wird automatisch gestartet bei:
- **Push** auf `main` oder `master` Branch
- **Pull Requests** auf `main` oder `master`
- **Tags** die mit `v` beginnen (z.B. `v1.0.0`)
- **Manuell** über GitHub Actions UI

## 📦 Build-Outputs

Nach erfolgreichem Build findest du die Dateien unter **Actions** → **Build QuantCalc** → **Artifacts**:

### Windows
- `QuantCalc-Windows.zip` enthält:
  - `QuantCalc_1.0.0_x64-setup.exe` (NSIS Installer)
  - `quantcalc.exe` (Standalone)

### macOS
- `QuantCalc-macOS.zip` enthält:
  - `QuantCalc_1.0.0_x64.dmg` (DMG Installer)
  - `QuantCalc.app` (App Bundle)

### Linux
- `QuantCalc-Linux.zip` enthält:
  - `quantcalc_1.0.0_amd64.deb` (Debian/Ubuntu)
  - `quantcalc_1.0.0_amd64.AppImage` (Universal)
  - `quantcalc` (Binary)

## 🎯 Verwendung

### 1. Code pushen
```bash
git add .
git commit -m "Update"
git push
```

### 2. Artifacts herunterladen
1. Gehe zu GitHub → **Actions** Tab
2. Klicke auf den neuesten **Build QuantCalc** Workflow
3. Scrolle nach unten zu **Artifacts**
4. Lade die gewünschte Plattform herunter

### 3. Release erstellen (Optional)
Für einen offiziellen Release mit Tag:
```bash
git tag v1.0.0
git push origin v1.0.0
```

## ⚙️ Manuelle Builds

Du kannst auch manuell einen Build starten:
1. Gehe zu **Actions** Tab
2. Wähle **Build QuantCalc** Workflow
3. Klicke **Run workflow**
4. Wähle den Branch und klicke **Run workflow**

## 🔧 Troubleshooting

### Build schlägt fehl?
- Prüfe die Logs im Actions Tab
- Stelle sicher, dass `npm run tauri:build` lokal funktioniert
- Prüfe ob alle Dependencies in `package.json` und `Cargo.toml` korrekt sind

### Artifacts nicht gefunden?
- Warte bis der Build komplett durchgelaufen ist (✅ grüner Haken)
- Artifacts sind nur 90 Tage verfügbar
- Erstelle einen Release für permanente Downloads

