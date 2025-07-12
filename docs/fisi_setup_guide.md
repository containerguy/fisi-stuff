# Vorbereitung für FiSi-Stuff Repository - MacBook Setup Guide

## Über dieses Repository

Dieses Repository enthält eine umfassende Sammlung von Aufgaben und Vorlagen zur Vorbereitung auf die Externenprüfung zum **Fachinformatiker Systemintegration (FiSi)**. Der Fokus liegt auf praktischen Übungen mit:

- **Virtualisierung** mit Proxmox VE
- **Cluster-Setup** und Hochverfügbarkeit
- **Windows Active Directory** Administration
- **Linux-Server** Konfiguration
- **Backup-Strategien** mit Veeam
- **Netzwerktechnik** und IP-Konzepte

Die Aufgaben sind so konzipiert, dass sie auf 3 Minisforum HM90 mit Proxmox VE ausgeführt werden können und den Aufbau eines produktiven Clusters simulieren.

## Systemanforderungen

- **Hardware**: MacBook (Intel oder Apple Silicon)
- **Betriebssystem**: macOS 12.0 oder höher
- **RAM**: Mindestens 8GB (16GB empfohlen)
- **Speicherplatz**: Mindestens 2GB freier Speicherplatz
- **Internetverbindung**: Für Downloads und Repository-Zugriff

## Voraussetzungen installieren

### 1. Homebrew Package Manager installieren

Homebrew ist der Standard-Paketmanager für macOS und vereinfacht die Installation von Entwicklungstools.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Nach der Installation den Terminal neu starten oder folgendes ausführen:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### 2. Git installieren

```bash
brew install git
```

Git-Konfiguration einrichten:

```bash
git config --global user.name "Dein Name"
git config --global user.email "deine-email@example.com"
```

### 3. Visual Studio Code installieren

#### Option A: Über Homebrew (empfohlen)

```bash
brew install --cask visual-studio-code
```

#### Option B: Direkt von der Website

1. Besuche [code.visualstudio.com](https://code.visualstudio.com)
2. Lade die macOS-Version herunter
3. Installiere die .app-Datei in den Programme-Ordner

### 4. Nützliche VS Code Extensions installieren

Öffne VS Code und installiere folgende Extensions über die Extension-Suchleiste (`Cmd+Shift+X`):

- **Markdown All in One** - Für bessere Markdown-Bearbeitung
- **GitLens** - Erweiterte Git-Integration
- **Remote - SSH** - Für SSH-Verbindungen zu Servern
- **YAML** - Für YAML-Dateien
- **PowerShell** - Falls PowerShell-Skripte bearbeitet werden
- **Excel Viewer** - Für Excel-Dateien im Repository

### 5. Zusätzliche Tools installieren

```bash
# SSH-Client (meist bereits vorhanden)
brew install openssh

# Nützliche Netzwerk-Tools
brew install nmap
brew install wget
brew install curl

# Virtualisierung (optional für lokale Tests)
brew install --cask virtualbox
brew install --cask vagrant
```

## Repository klonen und einrichten

### 1. Repository klonen

```bash
# Wechsle in dein gewünschtes Arbeitsverzeichnis
cd ~/Documents

# Klone das Repository
git clone https://github.com/containerguy/fisi-stuff.git

# Wechsle in das Repository-Verzeichnis
cd fisi-stuff
```

### 2. Repository-Struktur verstehen

```
fisi-stuff/
├── README.md                 # Projekt-Übersicht
├── docs/
│   └── aufgaben.md          # Ausführliche Aufgabenstellungen
├── templates/
│   ├── ip-konzept.xlsx      # IP-Konzept Vorlage
│   ├── ad-ou-struktur.txt   # Active Directory OU-Struktur
│   └── gpo-richtlinien.md   # Group Policy Richtlinien
├── vms/
│   ├── nextcloud01/         # Nextcloud VM-Konfiguration
│   ├── nas01/               # NAS VM-Konfiguration
│   └── ad01/                # Active Directory VM-Konfiguration
└── backup/
    ├── veeam-linux-notes.md # Veeam Backup für Linux
    └── veeam-windows-notes.md # Veeam Backup für Windows
```

### 3. VS Code für das Projekt einrichten

```bash
# Öffne das Repository in VS Code
code .
```

## Arbeitsweise und Tipps

### Wichtige Hinweise

- **Alle IP-Adressen müssen manuell vergeben werden** - Verwende die Vorlagen im `templates/` Ordner
- **Aufgaben sind ausführlich beschrieben** - Starte mit `docs/aufgaben.md`
- **Verwende die Vorlagen** - Nutze die Excel- und Markdown-Vorlagen für eine strukturierte Herangehensweise

### Empfohlener Workflow

1. **Aufgaben lesen**: Beginne mit `docs/aufgaben.md`
2. **IP-Konzept erstellen**: Verwende `templates/ip-konzept.xlsx`
3. **AD-Struktur planen**: Nutze `templates/ad-ou-struktur.txt`
4. **VMs konfigurieren**: Arbeite mit den Vorlagen im `vms/` Ordner
5. **Backup einrichten**: Folge den Anleitungen im `backup/` Ordner

### Git-Workflow für eigene Änderungen

```bash
# Aktuellen Stand abrufen
git pull origin main

# Eigenen Branch erstellen
git checkout -b meine-loesungen

# Änderungen committen
git add .
git commit -m "Meine Lösungen für Aufgabe XY"

# Branch pushen (falls du Fork erstellst)
git push origin meine-loesungen
```

## Fehlerbehebung

### Permission Denied bei Git

```bash
# SSH-Key generieren (falls noch nicht vorhanden)
ssh-keygen -t rsa -b 4096 -C "deine-email@example.com"

# SSH-Agent starten
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

### VS Code startet nicht

```bash
# PATH für VS Code setzen
echo 'export PATH="/Applications/Visual Studio Code.app/Contents/Resources/app/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Homebrew Probleme

```bash
# Homebrew reparieren
brew doctor
brew update
brew upgrade
```

## Nächste Schritte

1. **Dokumentation lesen**: Starte mit der `README.md` und `docs/aufgaben.md`
2. **Testumgebung vorbereiten**: Stelle sicher, dass du Zugang zu Proxmox VE hast
3. **IP-Konzept erstellen**: Plane deine Netzwerkstruktur
4. **Mit einfachen Aufgaben beginnen**: Arbeite dich schrittweise durch die Aufgaben

## Support und Hilfe

- **Repository Issues**: Bei Problemen mit dem Repository erstelle ein Issue auf GitHub
- **Dokumentation**: Alle wichtigen Informationen findest du im `docs/` Ordner
- **Community**: Suche nach weiteren FiSi-Ressourcen in der Community

---

**Viel Erfolg bei deiner Vorbereitung auf die Fachinformatiker Systemintegration Prüfung!** 🚀