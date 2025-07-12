# Erweiterte IP-Konzepte und Netzwerkplanung

## Gesamtnetzwerk-Übersicht

### Basis-Netzwerkstruktur
```
Gesamt-Netzwerk: 192.168.0.0/16
├── Management: 192.168.10.0/24
├── Produktion: 192.168.20.0/24
├── Entwicklung: 192.168.30.0/24
├── Backup: 192.168.40.0/24
├── DMZ: 192.168.50.0/24
├── Storage: 192.168.60.0/24
└── Gäste: 192.168.90.0/24
```

## VLAN-Konfiguration

### VLAN 10 - Management Network
```
Netzwerk: 192.168.10.0/24
Gateway: 192.168.10.1 (pfSense)
DNS: 192.168.10.2, 192.168.10.3
DHCP-Range: 192.168.10.100-192.168.10.199

Statische IPs:
192.168.10.1    - pfSense Management Interface
192.168.10.2    - Primary DNS Server
192.168.10.3    - Secondary DNS Server
192.168.10.10   - Proxmox Node 1 (pve01)
192.168.10.11   - Proxmox Node 2 (pve02)
192.168.10.12   - Proxmox Node 3 (pve03)
192.168.10.20   - Switch Management
192.168.10.21   - NAS Management
192.168.10.30   - Monitoring Server (Grafana)
192.168.10.31   - Prometheus Server
192.168.10.40   - Backup Server
192.168.10.50   - GitLab CI/CD Server
```

### VLAN 20 - Produktions-Server
```
Netzwerk: 192.168.20.0/24
Gateway: 192.168.20.1 (pfSense)
DNS: 192.168.10.2, 192.168.10.3
DHCP: Deaktiviert (nur statische IPs)

Statische IPs:
192.168.20.1    - pfSense Production Interface
192.168.20.10   - Domain Controller 1 (AD01)
192.168.20.11   - Domain Controller 2 (AD02)
192.168.20.15   - Certificate Authority
192.168.20.20   - Kubernetes Master
192.168.20.21   - Kubernetes Worker 1
192.168.20.22   - Kubernetes Worker 2
192.168.20.30   - Web Server 1 (nginx)
192.168.20.31   - Web Server 2 (nginx)
192.168.20.40   - Database Server (PostgreSQL)
192.168.20.41   - Database Server (MySQL)
192.168.20.50   - File Server (Nextcloud)
192.168.20.60   - Mail Server
192.168.20.70   - Load Balancer (HAProxy)
```

### VLAN 30 - Entwicklungsumgebung
```
Netzwerk: 192.168.30.0/24
Gateway: 192.168.30.1 (pfSense)
DNS: 192.168.10.2, 192.168.10.3
DHCP-Range: 192.168.30.100-192.168.30.199

Statische IPs:
192.168.30.1    - pfSense Development Interface
192.168.30.10   - Child Domain Controller (entwicklung.firma.local)
192.168.30.20   - Development Kubernetes Master
192.168.30.21   - Development Kubernetes Worker 1
192.168.30.22   - Development Kubernetes Worker 2
192.168.30.30   - Development Web Server
192.168.30.40   - Development Database Server
192.168.30.50   - Jenkins Build Server
192.168.30.60   - Staging Environment
192.168.30.70   - Test Automation Server
```

### VLAN 40 - Backup-Netzwerk
```
Netzwerk: 192.168.40.0/24
Gateway: 192.168.40.1 (pfSense)
DNS: 192.168.10.2, 192.168.10.3
DHCP: Deaktiviert

Statische IPs:
192.168.40.1    - pfSense Backup Interface
192.168.40.10   - Proxmox Node 1 (Backup Interface)
192.168.40.11   - Proxmox Node 2 (Backup Interface)
192.168.40.12   - Proxmox Node 3 (Backup Interface)
192.168.40.20   - Veeam Backup Server
192.168.40.21   - Proxmox Backup Server
192.168.40.30   - NAS Backup Target
192.168.40.40   - Offsite Backup Gateway
192.168.40.50   - Backup Verification Server
```

### VLAN 50 - DMZ (Demilitarized Zone)
```
Netzwerk: 192.168.50.0/24
Gateway: 192.168.50.1 (pfSense)
DNS: 8.8.8.8, 8.8.4.4 (externe DNS)
DHCP: Deaktiviert

Statische IPs:
192.168.50.1    - pfSense DMZ Interface
192.168.50.10   - Reverse Proxy (nginx)
192.168.50.20   - Web Application Firewall
192.168.50.30   - Public Web Server
192.168.50.40   - Public API Gateway
192.168.50.50   - VPN Server (OpenVPN)
192.168.50.60   - Jump Host/Bastion
```

### VLAN 60 - Storage Network
```
Netzwerk: 192.168.60.0/24
Gateway: 192.168.60.1 (pfSense)
DNS: 192.168.10.2, 192.168.10.3
DHCP: Deaktiviert

Statische IPs:
192.168.60.1    - pfSense Storage Interface
192.168.60.10   - Proxmox Node 1 (Storage Interface)
192.168.60.11   - Proxmox Node 2 (Storage Interface)
192.168.60.12   - Proxmox Node 3 (Storage Interface)
192.168.60.20   - Ceph Monitor 1
192.168.60.21   - Ceph Monitor 2
192.168.60.22   - Ceph Monitor 3
192.168.60.30   - NAS Storage Head
192.168.60.40   - SAN Storage Controller
192.168.60.50   - iSCSI Target
```

### VLAN 90 - Gäste-Netzwerk
```
Netzwerk: 192.168.90.0/24
Gateway: 192.168.90.1 (pfSense)
DNS: 8.8.8.8, 8.8.4.4
DHCP-Range: 192.168.90.100-192.168.90.199

Statische IPs:
192.168.90.1    - pfSense Guest Interface
192.168.90.10   - Guest Portal Server
```

## Firewall-Regeln Matrix

### Inter-VLAN Kommunikation
```
From\To     | Management | Production | Development | Backup | DMZ | Storage | Guests
------------|------------|------------|-------------|--------|-----|---------|--------
Management  | Allow      | Allow      | Allow       | Allow  | Allow| Allow   | Block
Production  | Allow      | Allow      | Block       | Allow  | Block| Allow   | Block
Development | Allow      | Block      | Allow       | Allow  | Block| Allow   | Block
Backup      | Allow      | Allow      | Allow       | Allow  | Block| Allow   | Block
DMZ         | Block      | Block      | Block       | Block  | Allow| Block   | Block
Storage     | Allow      | Allow      | Allow       | Allow  | Block| Allow   | Block
Guests      | Block      | Block      | Block       | Block  | Block| Block   | Allow
```

### Internet-Zugriff
```
VLAN         | HTTP/HTTPS | SSH | DNS | NTP | SMTP | Other
-------------|------------|-----|-----|-----|------|-------
Management   | Allow      | Allow| Allow| Allow| Allow| Allow
Production   | Allow      | Block| Allow| Allow| Allow| Restrict
Development  | Allow      | Allow| Allow| Allow| Allow| Allow
Backup       | Allow      | Block| Allow| Allow| Block| Restrict
DMZ          | Allow      | Allow| Allow| Allow| Allow| Restrict
Storage      | Block      | Block| Allow| Allow| Block| Block
Guests       | Allow      | Block| Allow| Allow| Block| Block
```

## Service-Ports Übersicht

### Standard-Ports
```
Service                 | Port(s)    | Protocol | VLAN
------------------------|------------|----------|----------
SSH                     | 22         | TCP      | All
HTTP                    | 80         | TCP      | All
HTTPS                   | 443        | TCP      | All
DNS                     | 53         | TCP/UDP  | All
DHCP                    | 67/68      | UDP      | Client VLANs
NTP                     | 123        | UDP      | All
SNMP                    | 161/162    | UDP      | Management
```

### Proxmox-Ports
```
Service                 | Port(s)    | Protocol | VLAN
------------------------|------------|----------|----------
Proxmox Web Interface   | 8006       | TCP      | Management
Proxmox SSH             | 22         | TCP      | Management
Ceph Monitor            | 6789       | TCP      | Storage
Ceph OSD                | 6800-7300  | TCP      | Storage
Corosync                | 5404/5405  | UDP      | Management
```

### Application-Ports
```
Service                 | Port(s)    | Protocol | VLAN
------------------------|------------|----------|----------
Active Directory        | 389/636    | TCP      | Production
DNS (AD)                | 53         | TCP/UDP  | Production
Kerberos                | 88         | TCP/UDP  | Production
LDAP SSL                | 636        | TCP      | Production
Global Catalog          | 3268/3269  | TCP      | Production
PostgreSQL              | 5432       | TCP      | Production
MySQL                   | 3306       | TCP      | Production
Kubernetes API          | 6443       | TCP      | Production
etcd                    | 2379/2380  | TCP      | Production
Grafana                 | 3000       | TCP      | Management
Prometheus              | 9090       | TCP      | Management
Elasticsearch           | 9200       | TCP      | Management
Kibana                  | 5601       | TCP      | Management
```

## QoS-Konfiguration

### Traffic-Prioritäten
```
Priority | Traffic Type        | Bandwidth | Latency
---------|-------------------- |-----------|----------
1        | Management (ICMP)   | 10%       | < 10ms
2        | VoIP                | 20%       | < 20ms
3        | Video Conferencing  | 15%       | < 50ms
4        | Business Critical   | 30%       | < 100ms
5        | General Web         | 20%       | < 200ms
6        | File Transfer       | 5%        | Best Effort
```

### Bandwidth-Limits
```
VLAN         | Upload Limit | Download Limit | Priority
-------------|--------------|----------------|----------
Management   | 100 Mbps     | 100 Mbps       | High
Production   | 500 Mbps     | 500 Mbps       | High
Development  | 200 Mbps     | 200 Mbps       | Medium
Backup       | 1000 Mbps    | 1000 Mbps      | Low
DMZ          | 100 Mbps     | 100 Mbps       | Medium
Storage      | 1000 Mbps    | 1000 Mbps      | High
Guests       | 50 Mbps      | 50 Mbps        | Low
```

## Monitoring-Konzept

### SNMP-Überwachung
```
Device Type     | Community | Version | OIDs
----------------|-----------|---------|------------------
Proxmox Nodes   | public    | v2c     | 1.3.6.1.2.1.1.1
Switches        | public    | v2c     | 1.3.6.1.2.1.2.2
pfSense         | public    | v2c     | 1.3.6.1.2.1.1.1
VMs             | public    | v2c     | 1.3.6.1.2.1.25.1
```

### Syslog-Konfiguration
```
Facility    | Severity | Destination
------------|----------|------------------
kern        | info     | 192.168.10.30
mail        | info     | 192.168.10.30
daemon      | info     | 192.168.10.30
auth        | info     | 192.168.10.30
syslog      | info     | 192.168.10.30
user        | info     | 192.168.10.30
local0      | info     | 192.168.10.30
```

## Backup-Netzwerk-Strategie

### Backup-Datenflüsse
```
Source VLAN    | Destination    | Method    | Schedule
---------------|----------------|-----------|----------
Production     | Backup VLAN    | Veeam     | Daily
Development    | Backup VLAN    | Veeam     | Weekly
Management     | Backup VLAN    | rsync     | Daily
Storage        | Backup VLAN    | Ceph      | Continuous
DMZ            | Backup VLAN    | Veeam     | Daily
```

### Backup-Traffic-Isolation
```
Time Window    | Allowed Traffic              | Bandwidth Limit
---------------|------------------------------|------------------
22:00-06:00    | Backup Traffic Only          | Unlimited
06:00-22:00    | Normal Traffic Priority      | 100 Mbps Max
Emergency      | Manual Override              | Administrator
```

## Sicherheits-Zonen

### Zone-Definitionen
```
Zone Name      | Trust Level | Access Control | Monitoring
---------------|-------------|----------------|------------
Trusted        | High        | Minimal        | Basic
Internal       | Medium      | Standard       | Enhanced
DMZ            | Low         | Strict         | Comprehensive
Untrusted      | Very Low    | Deny All       | Full Logging
```

### Micro-Segmentierung
```
Application    | Allowed Sources           | Allowed Destinations
---------------|---------------------------|------------------------
Web Servers    | Load Balancer, Users     | Database, File Storage
Database       | Web Servers, Admins      | Backup, Monitoring
File Storage   | All Internal             | Backup, Replication
Monitoring     | All Sources              | Internet (Alerts)
```

Diese erweiterte Netzwerkplanung bietet eine professionelle Grundlage für komplexe Infrastrukturen und berücksichtigt Sicherheit, Performance und Wartbarkeit.