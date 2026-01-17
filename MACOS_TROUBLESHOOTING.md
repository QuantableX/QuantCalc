# macOS Installation Troubleshooting

## ❌ Fehler: "Application is not supported on this Mac"

### Mögliche Ursachen:

#### 1. **macOS Version zu alt**
**Mindestanforderung:** macOS 10.13 (High Sierra) oder neuer

**Prüfen:**
```bash
sw_vers
```

**Lösung:**
- macOS auf mindestens 10.13 updaten
- Oder neueren Mac verwenden

---

#### 2. **Architektur-Problem (Intel vs. Apple Silicon)**

**Prüfen:**
```bash
uname -m
```

**Ergebnis:**
- `x86_64` = Intel Mac
- `arm64` = Apple Silicon (M1/M2/M3/M4)

**Lösung für Apple Silicon:**
Rosetta 2 installieren (für Intel-Apps):
```bash
softwareupdate --install-rosetta
```

---

#### 3. **Gatekeeper blockiert die App**

**Symptom:** "App kann nicht geöffnet werden, da von unbekanntem Entwickler"

**Lösung:**
1. **Rechtsklick** auf `QuantCalc.app`
2. **"Öffnen"** wählen
3. Im Dialog nochmal **"Öffnen"** bestätigen

**Alternative (Terminal):**
```bash
xattr -cr /Applications/QuantCalc.app
```

---

#### 4. **App ist beschädigt**

**Symptom:** "App ist beschädigt und kann nicht geöffnet werden"

**Lösung:**
```bash
# Quarantäne-Attribut entfernen
xattr -d com.apple.quarantine /Applications/QuantCalc.app

# Oder alle Attribute entfernen
xattr -cr /Applications/QuantCalc.app
```

---

## 🔍 Diagnose-Script ausführen

Lade das Diagnose-Script herunter und führe es aus:

```bash
# Script ausführbar machen
chmod +x scripts/check-macos-compatibility.sh

# Script ausführen
./scripts/check-macos-compatibility.sh
```

**Sende die Ausgabe an den Entwickler!**

---

## 📋 Manuelle Prüfung

### System-Informationen sammeln:

```bash
# macOS Version
sw_vers

# CPU Architektur
uname -m

# Gatekeeper Status
spctl --status

# App-Informationen
file /Applications/QuantCalc.app/Contents/MacOS/QuantCalc

# App-Attribute prüfen
xattr -l /Applications/QuantCalc.app
```

---

## ✅ Erfolgreiche Installation

Nach erfolgreicher Installation sollte die App:
1. Im **Programme-Ordner** sein
2. Beim ersten Start eine **Sicherheitswarnung** zeigen (normal!)
3. Nach Bestätigung **normal starten**

---

## 🆘 Weitere Hilfe

Wenn nichts funktioniert, sende diese Informationen:

```bash
# Alle Infos in eine Datei schreiben
{
  echo "=== System Info ==="
  sw_vers
  echo ""
  echo "=== Architecture ==="
  uname -m
  echo ""
  echo "=== App File Info ==="
  file /Applications/QuantCalc.app/Contents/MacOS/QuantCalc
  echo ""
  echo "=== App Attributes ==="
  xattr -l /Applications/QuantCalc.app
  echo ""
  echo "=== Gatekeeper ==="
  spctl --status
} > ~/Desktop/quantcalc-debug.txt

echo "Debug-Info gespeichert: ~/Desktop/quantcalc-debug.txt"
```

Dann die Datei `quantcalc-debug.txt` vom Desktop senden.

