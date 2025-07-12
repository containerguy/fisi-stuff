# Praktische Übungen - Schritt-für-Schritt Anleitungen

## Übersicht

Diese Datei enthält detaillierte Schritt-für-Schritt Anleitungen für die praktische Umsetzung der erweiterten Aufgaben.

## 1. Proxmox VE Cluster Setup

### 1.1 Cluster-Initialisierung

**Schritt 1: Grundinstallation**
```bash
# Auf Node 1 (pve01)
pvecm create mycluster

# Auf Node 2 und 3 (pve02, pve03)
pvecm add <IP-von-pve01>
```

**Schritt 2: Cluster-Status prüfen**
```bash
pvecm status
pvecm nodes
```

**Schritt 3: Ceph Storage einrichten**
```bash
# Auf allen Nodes
pveceph install

# Auf Node 1
pveceph init --network 192.168.40.0/24

# Auf allen Nodes
pveceph mon create

# OSDs erstellen (auf jedem Node)
pveceph osd create /dev/sdb
```

### 1.2 Hochverfügbarkeit konfigurieren

**HA-Gruppen erstellen:**
```bash
# Erstelle HA-Gruppe
ha-manager groupadd important-vms --nodes pve01:2,pve02:1,pve03:1

# VM zu HA-Gruppe hinzufügen
ha-manager add vm:100 --group important-vms
```

## 2. Netzwerk-Segmentierung

### 2.1 VLAN-Konfiguration auf Proxmox

**Schritt 1: Bridge mit VLAN-Unterstützung**
```bash
# In /etc/network/interfaces
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    bridge-ports ens18
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

**Schritt 2: VLAN-Interfaces erstellen**
```bash
# Management VLAN
auto vmbr0.10
iface vmbr0.10 inet static
    address 192.168.10.10/24

# Produktions VLAN
auto vmbr0.20
iface vmbr0.20 inet static
    address 192.168.20.10/24
```

### 2.2 pfSense VLAN-Konfiguration

**Schritt 1: VLAN-Interfaces in pfSense**
- Interfaces → Assignments → VLANs
- Neue VLAN-Interfaces hinzufügen:
  - VLAN 10: Management
  - VLAN 20: Produktion
  - VLAN 30: Entwicklung
  - VLAN 40: Backup
  - VLAN 50: DMZ

**Schritt 2: Firewall-Rules erstellen**
```
# Beispiel-Rules für Management VLAN
Management → LAN: Allow
Management → WAN: Allow (für Updates)
Management → DMZ: Block
```

## 3. Active Directory Multi-Domain Setup

### 3.1 Forest Root Domain erstellen

**Schritt 1: Windows Server 2019/2022 VM erstellen**
- CPU: 2 Cores
- RAM: 4GB
- Disk: 60GB
- Network: VLAN 20

**Schritt 2: Domain Controller installieren**
```powershell
# PowerShell als Administrator
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Domain erstellen
Install-ADDSForest -DomainName "firma.local" -DomainNetbiosName "FIRMA" -InstallDNS
```

### 3.2 Child Domain erstellen

**Schritt 1: Neue Windows Server VM**
- Gleiche Spezifikationen wie Root DC
- Network: VLAN 30 (für Entwicklung)

**Schritt 2: Child Domain Controller installieren**
```powershell
# Domain Controller Rolle installieren
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Child Domain erstellen
Install-ADDSDomain -NewDomainName "entwicklung" -ParentDomainName "firma.local" -DomainType "ChildDomain"
```

### 3.3 Erweiterte OU-Struktur

**PowerShell-Script für OU-Erstellung:**
```powershell
# OU-Struktur erstellen
Import-Module ActiveDirectory

$OUs = @(
    "OU=Firma,DC=firma,DC=local",
    "OU=Abteilungen,OU=Firma,DC=firma,DC=local",
    "OU=IT,OU=Abteilungen,OU=Firma,DC=firma,DC=local",
    "OU=Buchhaltung,OU=Abteilungen,OU=Firma,DC=firma,DC=local",
    "OU=Vertrieb,OU=Abteilungen,OU=Firma,DC=firma,DC=local",
    "OU=Server,OU=Firma,DC=firma,DC=local",
    "OU=Workstations,OU=Firma,DC=firma,DC=local"
)

foreach ($OU in $OUs) {
    $Parts = $OU.Split(",")
    $Name = $Parts[0].Replace("OU=", "")
    $Path = $Parts[1..($Parts.Count-1)] -join ","
    
    try {
        New-ADOrganizationalUnit -Name $Name -Path $Path
        Write-Host "OU $Name erstellt" -ForegroundColor Green
    }
    catch {
        Write-Host "OU $Name existiert bereits" -ForegroundColor Yellow
    }
}
```

## 4. Kubernetes Cluster Setup

### 4.1 Node-Vorbereitung

**Schritt 1: Ubuntu VMs erstellen**
- Master: 4 CPU, 8GB RAM, 60GB Disk
- Worker1: 2 CPU, 4GB RAM, 40GB Disk
- Worker2: 2 CPU, 4GB RAM, 40GB Disk

**Schritt 2: Docker installieren**
```bash
# Auf allen Nodes
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
usermod -aG docker $USER

# Containerd konfigurieren
cat > /etc/containerd/config.toml <<EOF
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
EOF

systemctl restart containerd
```

### 4.2 Kubernetes installieren

**Schritt 1: kubeadm, kubelet, kubectl installieren**
```bash
# Repository hinzufügen
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

# Pakete installieren
apt-get update
apt-get install -y kubeadm kubelet kubectl
apt-mark hold kubeadm kubelet kubectl
```

**Schritt 2: Cluster initialisieren (Master)**
```bash
# Cluster initialisieren
kubeadm init --pod-network-cidr=10.244.0.0/16

# kubectl konfigurieren
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Flannel CNI installieren
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

**Schritt 3: Worker Nodes hinzufügen**
```bash
# Auf Worker Nodes (Token vom Master verwenden)
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```

## 5. Monitoring Stack Setup

### 5.1 Prometheus und Grafana mit Docker Compose

**docker-compose.yml:**
```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'

volumes:
  prometheus_data:
  grafana_data:
```

**prometheus.yml:**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'proxmox'
    static_configs:
      - targets: ['192.168.10.10:9100', '192.168.10.11:9100', '192.168.10.12:9100']
```

### 5.2 ELK Stack Setup

**docker-compose-elk.yml:**
```yaml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=docker-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    container_name: logstash
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  esdata1:
```

## 6. Backup-Strategien

### 6.1 Veeam Backup Setup

**Schritt 1: Veeam Backup & Replication installieren**
- Windows Server VM erstellen (4 CPU, 8GB RAM)
- Veeam B&R Community Edition installieren
- Proxmox VE als Infrastruktur hinzufügen

**Schritt 2: Backup-Jobs konfigurieren**
```
Job Name: Daily-VM-Backup
VMs: Alle kritischen VMs
Schedule: Täglich 22:00
Retention: 7 Tage
Backup Repository: Lokaler Storage + NAS
```

### 6.2 Proxmox Backup Server

**Schritt 1: PBS VM erstellen**
- 2 CPU, 4GB RAM, 100GB System + 500GB Backup Storage
- Proxmox Backup Server ISO installieren

**Schritt 2: Backup-Jobs in Proxmox konfigurieren**
```bash
# Backup-Job über Web-Interface oder CLI
vzdump --mode snapshot --compress gzip --storage pbs-backup --node pve01 --vmid 100,101,102
```

## 7. Automatisierung mit Ansible

### 7.1 Ansible Setup

**Schritt 1: Ansible Control Node einrichten**
```bash
# Ubuntu VM erstellen
apt update && apt install -y ansible

# SSH-Keys generieren
ssh-keygen -t rsa -b 4096
```

**Schritt 2: Inventory-Datei erstellen**
```ini
[proxmox]
pve01 ansible_host=192.168.10.10
pve02 ansible_host=192.168.10.11
pve03 ansible_host=192.168.10.12

[windows]
dc01 ansible_host=192.168.20.10
dc02 ansible_host=192.168.30.10

[linux]
k8s-master ansible_host=192.168.20.20
k8s-worker1 ansible_host=192.168.20.21
k8s-worker2 ansible_host=192.168.20.22
```

### 7.2 Playbook-Beispiele

**Linux-Updates Playbook:**
```yaml
---
- name: Update Linux Systems
  hosts: linux
  become: yes
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Upgrade all packages
      apt:
        upgrade: dist
        autoremove: yes
        autoclean: yes

    - name: Check if reboot required
      stat:
        path: /var/run/reboot-required
      register: reboot_required

    - name: Reboot if required
      reboot:
        reboot_timeout: 600
      when: reboot_required.stat.exists
```

**VM-Deployment Playbook:**
```yaml
---
- name: Deploy VMs on Proxmox
  hosts: proxmox
  tasks:
    - name: Create VM
      proxmox_kvm:
        api_host: "{{ ansible_host }}"
        api_user: root@pam
        api_password: "{{ proxmox_password }}"
        name: "{{ vm_name }}"
        node: "{{ ansible_hostname }}"
        memory: "{{ vm_memory }}"
        cores: "{{ vm_cores }}"
        vga: std
        ostype: l26
        net:
          net0: 'virtio,bridge=vmbr0,tag={{ vm_vlan }}'
        ide:
          ide2: 'local:cloudinit'
        boot: 'cdn'
        bootdisk: scsi0
        scsihw: virtio-scsi-pci
        scsi:
          scsi0: 'local-lvm:{{ vm_disk_size }}'
        state: present
```

## 8. Sicherheits-Implementierung

### 8.1 pfSense Firewall-Konfiguration

**Schritt 1: pfSense VM erstellen**
- 2 CPU, 2GB RAM, 20GB Disk
- 2 Netzwerk-Interfaces (WAN + LAN)

**Schritt 2: VLAN-Interfaces konfigurieren**
```
Interfaces → Assignments → VLANs
- Add: em0.10 (Management)
- Add: em0.20 (Produktion)
- Add: em0.30 (Entwicklung)
- Add: em0.40 (Backup)
- Add: em0.50 (DMZ)
```

**Schritt 3: Firewall-Rules erstellen**
```
Management VLAN Rules:
- Allow: Management → Any (für Administration)
- Block: Any → Management (außer SSH/HTTPS)

Produktions VLAN Rules:
- Allow: Produktion → Internet (HTTP/HTTPS)
- Allow: Produktion → Management (für Monitoring)
- Block: Produktion → Entwicklung

Entwicklungs VLAN Rules:
- Allow: Entwicklung → Internet
- Block: Entwicklung → Produktion
- Block: Entwicklung → Management
```

### 8.2 Certificate Authority Setup

**Schritt 1: CA-VM erstellen**
```bash
# Ubuntu Server VM
apt install -y easy-rsa

# CA initialisieren
make-cadir ~/easy-rsa
cd ~/easy-rsa
./easyrsa init-pki
./easyrsa build-ca nopass
```

**Schritt 2: Server-Zertifikate erstellen**
```bash
# Server-Zertifikat für Proxmox
./easyrsa gen-req proxmox nopass
./easyrsa sign-req server proxmox

# Client-Zertifikat für VPN
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

## 9. Performance-Monitoring

### 9.1 System-Monitoring Setup

**Schritt 1: Node Exporter auf allen Systemen**
```bash
# Download und Installation
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.3.1.linux-amd64.tar.gz
sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/

# Systemd Service erstellen
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

# Service aktivieren
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### 9.2 Grafana Dashboard-Konfiguration

**Schritt 1: Prometheus als Datenquelle hinzufügen**
```
URL: http://prometheus:9090
Access: Server (Default)
```

**Schritt 2: Infrastructure Dashboard erstellen**
```json
{
  "dashboard": {
    "title": "Infrastructure Overview",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg by (instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100"
          }
        ]
      },
      {
        "title": "Disk Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - ((node_filesystem_avail_bytes{mountpoint=\"/\"} / node_filesystem_size_bytes{mountpoint=\"/\"}) * 100)"
          }
        ]
      }
    ]
  }
}
```

## 10. Disaster Recovery Procedures

### 10.1 Backup-Restore-Tests

**Schritt 1: VM-Restore-Test**
```bash
# Proxmox VM Restore
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2024_01_15-10_00_00.vma.zst 200

# Test der wiederhergestellten VM
qm start 200
qm status 200
```

**Schritt 2: Cluster-Recovery-Test**
```bash
# Simuliere Node-Ausfall
systemctl stop pve-cluster
systemctl stop corosync

# Recovery-Prozedur
pvecm expected 2  # Wenn nur 2 von 3 Nodes verfügbar
systemctl start corosync
systemctl start pve-cluster
```

### 10.2 Disaster Recovery Playbook

**Kompletter Cluster-Ausfall:**
```markdown
1. Hardware-Check durchführen
2. Proxmox VE auf neuer Hardware installieren
3. Cluster-Konfiguration aus Backup wiederherstellen
4. Storage-Konfiguration wiederherstellen
5. VMs aus Backup wiederherstellen
6. Netzwerk-Konfiguration prüfen
7. Funktionstest aller Services
8. Monitoring wieder aktivieren
```

## 11. Terraform Infrastructure as Code

### 11.1 Terraform Setup

**Schritt 1: Terraform installieren**
```bash
# Ubuntu/Debian
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```

**Schritt 2: Proxmox Provider konfigurieren**
```hcl
# main.tf
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "2.9.11"
    }
  }
}

provider "proxmox" {
  pm_api_url      = "https://192.168.10.10:8006/api2/json"
  pm_user         = "root@pam"
  pm_password     = var.proxmox_password
  pm_tls_insecure = true
}

# variables.tf
variable "proxmox_password" {
  description = "Proxmox root password"
  type        = string
  sensitive   = true
}

# VM-Template
resource "proxmox_vm_qemu" "web_server" {
  count       = 2
  name        = "web-${count.index + 1}"
  target_node = "pve01"
  clone       = "ubuntu-cloud-template"
  
  cores   = 2
  sockets = 1
  memory  = 2048
  
  disk {
    size     = "20G"
    type     = "scsi"
    storage  = "local-lvm"
  }
  
  network {
    model    = "virtio"
    bridge   = "vmbr0"
    tag      = 20
  }
  
  ciuser     = "ansible"
  cipassword = var.vm_password
  sshkeys    = file("~/.ssh/id_rsa.pub")
  
  ipconfig0 = "ip=192.168.20.${count.index + 10}/24,gw=192.168.20.1"
}
```

### 11.2 Terraform Deployment

**Schritt 1: Terraform initialisieren**
```bash
terraform init
terraform plan
terraform apply
```

**Schritt 2: Terraform State Management**
```bash
# Remote State Backend (optional)
terraform {
  backend "s3" {
    bucket = "terraform-state-bucket"
    key    = "infrastructure/terraform.tfstate"
    region = "eu-central-1"
  }
}
```

## 12. CI/CD Pipeline Setup

### 12.1 GitLab CI/CD

**Schritt 1: GitLab CE VM erstellen**
```bash
# Ubuntu VM: 4 CPU, 8GB RAM, 100GB Disk
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl

# GitLab CE Repository
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt-get install gitlab-ce

# GitLab konfigurieren
sudo gitlab-ctl reconfigure
```

**Schritt 2: .gitlab-ci.yml erstellen**
```yaml
stages:
  - validate
  - plan
  - deploy

variables:
  TF_ROOT: ${CI_PROJECT_DIR}
  TF_ADDRESS: ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/default

cache:
  key: "${TF_ROOT}"
  paths:
    - ${TF_ROOT}/.terraform

before_script:
  - cd ${TF_ROOT}
  - terraform --version
  - terraform init

validate:
  stage: validate
  script:
    - terraform validate
    - terraform fmt -check

plan:
  stage: plan
  script:
    - terraform plan -out="planfile"
  artifacts:
    paths:
      - planfile

deploy:
  stage: deploy
  script:
    - terraform apply -input=false planfile
  when: manual
  only:
    - main
```

### 12.2 Jenkins Pipeline

**Schritt 1: Jenkins VM Setup**
```bash
# Ubuntu VM: 2 CPU, 4GB RAM, 60GB Disk
sudo apt update
sudo apt install -y openjdk-11-jdk

# Jenkins installieren
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins

# Jenkins starten
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

**Schritt 2: Jenkinsfile erstellen**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Validate') {
            steps {
                sh 'terraform validate'
                sh 'ansible-playbook --syntax-check playbook.yml'
            }
        }
        
        stage('Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
                archiveArtifacts artifacts: 'tfplan'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh 'terraform apply tfplan'
                sh 'ansible-playbook -i inventory playbook.yml'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

## 13. Troubleshooting-Checklisten

### 13.1 Proxmox Cluster Troubleshooting

**Cluster-Probleme:**
```bash
# Cluster-Status prüfen
pvecm status
pvecm nodes
systemctl status pve-cluster

# Quorum-Probleme lösen
pvecm expected 1  # Nur bei Node-Ausfall
systemctl restart pve-cluster

# Logs prüfen
journalctl -u pve-cluster
tail -f /var/log/pve/cluster.log
```

**Storage-Probleme:**
```bash
# Ceph-Status prüfen
ceph -s
ceph osd tree
ceph health detail

# Storage-Konfiguration prüfen
cat /etc/pve/storage.cfg
pvesm status
```

### 13.2 Netzwerk-Troubleshooting

**VLAN-Probleme:**
```bash
# VLAN-Konfiguration prüfen
cat /etc/network/interfaces
ip link show
bridge vlan show

# Netzwerk-Konnektivität testen
ping -c 4 192.168.20.1
traceroute 192.168.20.10
```

**Firewall-Debugging:**
```bash
# pfSense Logs
/var/log/filter.log
/var/log/dhcpd.log

# Packet Capture
tcpdump -i em0.20 -n host 192.168.20.10
```

### 13.3 Performance-Troubleshooting

**System-Performance:**
```bash
# CPU-Auslastung
top
htop
iostat -x 1

# Memory-Analyse
free -h
vmstat 1
cat /proc/meminfo

# Disk-I/O
iotop
iostat -x 1
```

**Netzwerk-Performance:**
```bash
# Bandbreiten-Test
iperf3 -s  # Server
iperf3 -c <server-ip>  # Client

# Netzwerk-Statistiken
ss -tuln
netstat -i
```

## 14. Prüfungsvorbereitung Checklisten

### 14.1 Technische Kompetenz-Checkliste

**Virtualisierung:**
- [ ] Proxmox VE Installation und Konfiguration
- [ ] VM-Erstellung und -verwaltung
- [ ] Cluster-Setup und Hochverfügbarkeit
- [ ] Storage-Konfiguration (LVM, Ceph)
- [ ] Backup und Restore-Strategien

**Netzwerk:**
- [ ] VLAN-Konfiguration
- [ ] Firewall-Konfiguration (pfSense)
- [ ] VPN-Setup
- [ ] DNS und DHCP-Konfiguration
- [ ] Monitoring und Troubleshooting

**Windows:**
- [ ] Active Directory Installation
- [ ] Multi-Domain-Setup
- [ ] Gruppenrichtlinien
- [ ] Certificate Services
- [ ] PowerShell-Scripting

**Linux:**
- [ ] Ubuntu/Debian Administration
- [ ] Docker und Kubernetes
- [ ] Bash-Scripting
- [ ] System-Monitoring
- [ ] Log-Analyse

### 14.2 Soft Skills Checkliste

**Projektmanagement:**
- [ ] Projektplanung und -durchführung
- [ ] Risikomanagement
- [ ] Dokumentation
- [ ] Kostenkalkulation
- [ ] Zeitmanagement

**Kommunikation:**
- [ ] Technische Präsentationen
- [ ] Kundenkommunikation
- [ ] Teamarbeit
- [ ] Problemlösung
- [ ] Konfliktmanagement

### 14.3 Prüfungssimulation

**Zeitmanagement:**
- [ ] 35 Stunden Projektdurchführung
- [ ] 30 Minuten Präsentation
- [ ] 30 Minuten Fachgespräch

**Dokumentation:**
- [ ] Projektantrag
- [ ] Projektdokumentation
- [ ] Präsentationsunterlagen
- [ ] Reflexion des Projekts

**Typische Prüfungsfragen:**
1. Erklären Sie die Vorteile von Virtualisierung
2. Wie würden Sie eine Hochverfügbarkeits-Lösung implementieren?
3. Beschreiben Sie Ihre Backup-Strategie
4. Wie lösen Sie Netzwerk-Probleme systematisch?
5. Welche Sicherheitsaspekte müssen beachtet werden?

## 15. Wartung und Updates

### 15.1 Regelmäßige Wartungsaufgaben

**Wöchentlich:**
```bash
# System-Updates
apt update && apt upgrade -y
proxmox-ve update

# Backup-Verifikation
vzdump --dumpdir /backup --mode snapshot --compress gzip --vmid all

# Log-Rotation
logrotate -f /etc/logrotate.conf
```

**Monatlich:**
```bash
# Sicherheits-Updates
unattended-upgrades
rkhunter --check
chkrootkit

# Performance-Analyse
sar -A
iostat -x
```

### 15.2 Update-Strategien

**Proxmox VE Updates:**
```bash
# Repository-Updates
apt update
apt list --upgradable

# Proxmox-Updates
apt upgrade
pveam update
```

**Kubernetes Updates:**
```bash
# Cluster-Updates
kubeadm upgrade plan
kubeadm upgrade apply v1.25.0

# Node-Updates
kubectl drain node1 --ignore-daemonsets
kubeadm upgrade node
kubectl uncordon node1
```

Diese praktischen Übungen bieten eine detaillierte Anleitung für die Umsetzung aller erweiterten Aufgaben und bereiten umfassend auf die Fachinformatiker Systemintegration Prüfung vor.