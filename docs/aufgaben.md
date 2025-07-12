# Aufgabenstellung: Proxmox Cluster & VM Setup

## 1. Installation Proxmox VE auf Minisforum HM90

- Installiere Proxmox VE auf der kleinen 250 GB SATA SSD.
- Nutze die 512 GB NVMe M.2 SSD später für VM-Datenspeicher.
- Verbinde die 2.5 Gbit/s Netzwerkschnittstellen der MiniPCs mit dem dedizierten 2.5 Gbit/s Managed Switch.
- Verbinde die 1 Gbit/s Netzwerkschnittstellen der MiniPCs mit dem 1 Gbit/s Managed Switch.
- Konfiguriere für das Cluster (Corosync etc.) die 2.5 Gbit/s Schnittstellen über den dedizierten 2.5 Gbit/s Switch.
- Verwalte den VM- und Management-Traffic über die 1 Gbit/s Schnittstellen.
- Weise allen Netzwerkschnittstellen manuell statische IP-Adressen zu.

## 2. Cluster Einrichtung

- Erstelle einen Proxmox VE Cluster mit den 3 Minisforum HM90.
- Vergewissere dich, dass Corosync den Cluster Traffic über den 2.5 Gbit/s Switch nutzt.
- Prüfe die Cluster-Verbindung und den Status aller Nodes.

## 3. Einrichtung des verteilten VM-Datenspeichers

- Formatiere die 512 GB NVMe M.2 SSD auf jedem Node als LVM oder ZFS.
- Erstelle einen gemeinsamen Storage-Pool (z.B. via Ceph oder Proxmox Cluster Filesystem).
- Richte den Speicher so ein, dass VMs von allen Nodes darauf zugreifen können.
- Definiere den Storage als Datastore für VM-Images, Snapshots und ISOs.
- Lege einen separaten Ordner für ISOs an, in dem später Installationsdateien abgelegt werden.

## 4. VM Netzwerk Setup

- Richte ein separates virtuelles Netzwerk ein, das ausschließlich vom Router VM verwaltet wird.
- Die VMs sollen im neuen Netzwerk nur über die Router VM ins Internet kommen.
- Verknüpfe alle VM-Netzwerke mit der 1 Gbit/s Management-Schnittstelle.

## 5. Installation und Konfiguration der VMs

- Erstelle folgende VMs:

  - **Router & Firewall VM** (z.B. pfSense oder OPNsense)
  - **Windows Server VM** mit Active Directory Domain Controller
  - **2x Windows 11 Clients**, die der Domain beitreten
  - **Ubuntu Server VM für Nextcloud**
  - **Ubuntu Server VM für NAS System** (z.B. mit OpenMediaVault)

- Die ISOs für die Installation liegen im gemeinsamen ISO-Datastore.

## 6. Active Directory Einrichtung (Windows Server VM)

- Installiere die Active Directory Domain Services.
- Erstelle eine sinnvolle OU-Struktur (z.B. `Clients`, `Server`, `Benutzer`).
- Lege Benutzerkonten und Gruppen an.
- Setze GPO-Richtlinien zur Basisabsicherung und Standardkonfiguration.
- Sorge für DNS-Integration im AD.

## 7. Veeam Backup Einrichtung

- Installiere und konfiguriere Veeam Backup & Replication.
- Erstelle Backup Jobs für den Windows Server mit AD und die beiden Ubuntu Server.
- Stelle sicher, dass Backups regelmäßig und automatisiert durchgeführt werden.
- Teste eine Wiederherstellung einer VM aus einem Backup.

---

## Weitere Hinweise

- Nutze die mitgelieferten Managed Switches:
  - 1x 2.5 Gbit/s Switch für Cluster Traffic
  - 1x 1 Gbit/s Switch für VM und Management Traffic

- Alle IP-Adressen sind manuell zu vergeben und dokumentieren.
