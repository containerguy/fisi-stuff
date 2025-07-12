# FISI-Stuff - Virtualisierung & Cluster Setup für IHK Externenprüfung

Dieses Repository enthält eine Sammlung von Aufgaben und Vorlagen zur Vorbereitung auf die Externenprüfung zum Fachinformatiker Systemintegration. 
Der Fokus liegt auf Virtualisierung mit Proxmox VE, Cluster-Setup, Windows AD, Linux-Servern, Backup mit Veeam und Netzwerktechnik.

## Inhalt

- `docs/aufgaben.md` - Ausführliche Aufgabenstellungen zum Cluster-Setup und zur VM-Installation
- `templates/` - Hilfsdateien wie IP-Konzept Vorlage und AD OU-Struktur
- `backup/` - Hinweise zum Einrichten von Veeam Backup für Windows- und Linux-VMs
- `vms/` - Ordnerstruktur für VM-Installationen und ISO-Dateien

## Ziel

Die Aufgaben sind so konzipiert, dass sie auf 3 Minisforum HM90 mit Proxmox VE ausgeführt werden können und den Aufbau eines produktiven Clusters simulieren.

---

## Git Struktur

```
fisi-stuff/
├── README.md
├── docs/
│   └── aufgaben.md
├── templates/
│   ├── ip-konzept.xlsx
│   ├── ad-ou-struktur.txt
│   └── gpo-richtlinien.md
├── vms/        
│   ├── nextcloud01/  
│   ├── nas01/        
│   └── ad01/         
└── backup/
    ├── veeam-linux-notes.md
    └── veeam-windows-notes.md
```

## Hinweise

- Alle IP-Adressen müssen manuell vergeben werden.
- Die Aufgaben in `docs/aufgaben.md` sind ausführlich beschrieben.

---
