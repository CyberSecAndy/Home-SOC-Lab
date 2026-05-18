# Home SOC Lab - Comprehensive Setup Guide

Complete step-by-step instructions for deploying a fully functional SOC lab environment.

## Prerequisites Checklist

- [ ] VirtualBox 6.1+ installed on host
- [ ] 16GB+ RAM available
- [ ] 150GB+ free disk space
- [ ] Windows 10/11 ISO downloaded
- [ ] Kali Linux ISO downloaded
- [ ] Stable internet connection
- [ ] Administrator access on host machine

## Hardware Recommendations

### Minimum (Tight but works)
- CPU: 4 cores
- RAM: 16GB
- Storage: 150GB SSD
- Network: 100Mbps

### Recommended
- CPU: 8+ cores
- RAM: 32GB
- Storage: 300GB SSD
- Network: 1Gbps

### Ideal
- CPU: 12+ cores
- RAM: 64GB
- Storage: 500GB+ SSD
- Network: 1Gbps

---

## Part 1: VirtualBox Network Setup (5 minutes)

### Create Internal Network

1. **Open VirtualBox**
   - Launch VirtualBox application

2. **Access Network Settings**
   - Click: File → Preferences (or VirtualBox → Preferences on Mac)
   - Select: Network from left sidebar
   - Click: Add (green + icon)

3. **Configure SOC-LAB Network**
   ```
   Network CIDR: 192.168.100.0/24
   Name: SOC-LAB
   IPv4 Address: 192.168.100.1
   IPv4 Netmask: 255.255.255.0
   DHCP Server: Disabled (we use static IPs)
   ```

4. **Click OK** and verify network appears in list

### Network Architecture

```
┌──────────────────────────────────┐
│   VirtualBox Internal Network    │
│        SOC-LAB 192.168.100.0/24  │
│                                  │
│  Gateway: 192.168.100.1          │
│  Windows: 192.168.100.10         │
│  Kali: 192.168.100.20            │
│  SIEM: 192.168.100.30            │
└──────────────────────────────────┘
```

---

## Part 2: Windows Target VM (1-2 hours)

### VM Creation

1. **Create New VM**
   - Name: `Windows-Target`
   - Type: Microsoft Windows
   - Version: Windows 10 (64-bit) or Windows 11
   - Memory: 4096 MB (4GB)
   - Disk: Create virtual hard disk
   - Disk Size: 50GB
   - Disk Type: VDI, Dynamically allocated

2. **VM Settings**
   - General → Advanced → Shared Clipboard: Bidirectional
   - Display → Video Memory: 128MB
   - Storage → Controller IDE: Mount Windows ISO
   - Network → Adapter 1:
     - Attached to: Internal Network
     - Name: SOC-LAB

3. **Boot VM** and install Windows

### Windows Installation

1. **Install Windows 10/11**
   - Accept license
   - Custom installation
   - Select entire disk
   - Follow setup wizard
   - Set username: `admin`
   - Set password: `SOCLab123!` (use strong password in production)

2. **Post-Installation**
   ```powershell
   # Update Windows
   Settings → Update & Security → Check for updates
   
   # Install VirtualBox Guest Additions
   Devices → Insert Guest Additions CD
   Run D:\VBoxWindowsAdditions.exe
   Reboot
   ```

3. **Disable Security Features** (for lab only!)
   ```powershell
   # Disable Windows Defender
   Settings → Virus & threat protection → Manage settings
   Toggle OFF: Real-time protection
   
   # Disable UAC
   Settings → Accounts → Change User Account Control settings
   Move slider to: Never notify
   Reboot
   
   # Disable Firewall (for lab)
   Settings → Update & Security → Windows Security → Firewall
   Toggle OFF: All networks
   ```

### Network Configuration (Static IP)

```powershell
# Open PowerShell as Administrator

# Set static IP
netsh int ip set addr "Ethernet" static 192.168.100.10 255.255.255.0 192.168.100.1

# Set DNS
netsh int ip set dnsservers "Ethernet" static 8.8.8.8

# Verify
ipconfig /all

# Should show:
# Ethernet adapter
# IPv4 Address: 192.168.100.10
# Subnet Mask: 255.255.255.0
# Default Gateway: 192.168.100.1
```

### Install Sysmon (CRITICAL for logging)

Sysmon provides detailed Windows event logging needed for SOC lab.

```powershell
# Run as Administrator

# Download Sysmon
$url = "https://download.sysinternals.com/files/Sysmon.zip"
$dest = "C:\Sysmon"
New-Item -ItemType Directory -Path $dest -Force
Invoke-WebRequest -Uri $url -OutFile "$dest\Sysmon.zip"
Expand-Archive "$dest\Sysmon.zip" -DestinationPath $dest

# Download SwiftOnSecurity Sysmon config
$config_url = "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml"
Invoke-WebRequest -Uri $config_url -OutFile "$dest\sysmonconfig.xml"

# Install Sysmon with config
cd $dest
.\Sysmon64.exe -accepteula -i sysmonconfig.xml

# Verify installation
Get-Service Sysmon64
# Should show: Running

# Check Sysmon events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | Select-Object -First 5
```

### Install Winlogbeat (Log Forwarding)

Winlogbeat collects Windows logs and forwards to SIEM.

```powershell
# Run as Administrator

# Download Winlogbeat
$wb_url = "https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-8.13.0-windows-x86_64.zip"
$wb_dest = "C:\Winlogbeat"
New-Item -ItemType Directory -Path $wb_dest -Force
Invoke-WebRequest -Uri $wb_url -OutFile "$wb_dest\winlogbeat.zip"
Expand-Archive "$wb_dest\winlogbeat.zip" -DestinationPath $wb_dest

# Note the extracted folder name
cd "C:\Winlogbeat\winlogbeat-8.13.0-windows-x86_64"
```

**Create winlogbeat.yml** (replace existing file):

```yaml
winlogbeat.event_logs:
  - name: Application
  - name: System
  - name: Security
  - name: Microsoft-Windows-Sysmon/Operational
  - name: Microsoft-Windows-PowerShell/Operational
  - name: Windows PowerShell

output.elasticsearch:
  hosts: ["192.168.100.30:9200"]
  protocol: "http"
  index: "winlogbeat-%{+yyyy.MM.dd}"

logging.level: info
logging.to_files: true
logging.files:
  path: C:\ProgramData\winlogbeat\logs
  name: winlogbeat
  keepfiles: 7
  permissions: 0600

processors:
  - add_host_metadata: ~
  - add_process_metadata:
      when.not.regexp.message: '(?i)powershell'
```

**Install Winlogbeat Service**:

```powershell
cd "C:\Winlogbeat\winlogbeat-8.13.0-windows-x86_64"

# Install service
powershell -ExecutionPolicy UnRestricted -File .\install-service-winlogbeat.ps1

# Start service
Start-Service winlogbeat

# Verify
Get-Service winlogbeat
# Should show: Running

# Check logs
Get-Content "C:\ProgramData\winlogbeat\logs\winlogbeat"
```

### Enable PowerShell Logging

Enable comprehensive PowerShell event logging for threat hunting.

```powershell
# Run as Administrator

# Enable Module Logging
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -Force
Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging" -Name "EnableModuleLogging" -Value 1
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames" -Force
Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging\ModuleNames" -Name "*" -Value "*"

# Enable Script Block Logging
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force
Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name "EnableScriptBlockLogging" -Value 1

# Enable Protected Event Logging (if Windows 11)
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription" -Force
Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription" -Name "EnableTranscripting" -Value 1

# Restart EventLog service
Restart-Service EventLog -Force

# Verify
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" | Select-Object -First 5
```

---

## Part 3: Kali Attacker VM (30-45 minutes)

### VM Creation

1. **Create New VM**
   - Name: `Kali-Attacker`
   - Type: Linux
   - Version: Debian (64-bit) or Linux (64-bit)
   - Memory: 4096 MB (4GB)
   - Disk: Create virtual hard disk
   - Disk Size: 30GB
   - Disk Type: VDI, Dynamically allocated

2. **VM Settings**
   - Display → Video Memory: 64MB
   - Storage → Controller IDE: Mount Kali ISO
   - Network → Adapter 1:
     - Attached to: Internal Network
     - Name: SOC-LAB

3. **Boot and Install Kali**

### Kali Installation

```bash
# Install Kali with graphical installer
# Use default partitioning
# Set root password: SOCLab123!
# Install GRUB bootloader
# Reboot
```

### Network Configuration (Static IP)

```bash
# Edit network configuration
sudo nano /etc/network/interfaces

# Add:
auto eth0
iface eth0 inet static
  address 192.168.100.20
  netmask 255.255.255.0
  gateway 192.168.100.1
  dns-nameservers 8.8.8.8 8.8.4.4

# Save (Ctrl+O, Enter, Ctrl+X)

# Apply changes
sudo systemctl restart networking

# Verify
ip addr show
# Should show: inet 192.168.100.20/24
```

### Update and Install Tools

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Metasploit Framework
sudo apt install -y metasploit-framework

# Install PowerShell (for Windows script execution)
sudo apt install -y powershell

# Install Mimikatz (pre-compiled on Kali)
which mimikatz

# Install additional tools
sudo apt install -y \
  curl \
  wget \
  git \
  nmap \
  tcpdump \
  wireshark \
  hashcat \
  john

# Test connectivity to Windows Target
ping 192.168.100.10
# Should respond
```

---

## Part 4: SIEM Server VM (1-2 hours)

### VM Creation

1. **Create New VM**
   - Name: `SIEM-Server`
   - Type: Linux
   - Version: Ubuntu (64-bit)
   - Memory: 8192 MB (8GB) minimum, 16GB recommended
   - Disk: Create virtual hard disk
   - Disk Size: 50GB
   - Disk Type: VDI, Dynamically allocated

2. **VM Settings**
   - Display → Video Memory: 64MB
   - Storage → Controller IDE: Mount Ubuntu ISO
   - Network → Adapter 1:
     - Attached to: Internal Network
     - Name: SOC-LAB

3. **Boot and Install Ubuntu 22.04 LTS**

### Ubuntu Installation

```bash
# During installation:
# - Language: English
# - Keyboard: Choose your layout
# - Network: DHCP (will configure static later)
# - Storage: Use entire disk
# - User: Create user "admin" with password "SOCLab123!"
# - Software: Default (OpenSSH Server recommended)
```

### Network Configuration (Static IP)

```bash
# Edit netplan configuration
sudo nano /etc/netplan/00-installer-config.yaml

# Replace with:
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses: [192.168.100.30/24]
      gateway4: 192.168.100.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# Save and apply
sudo netplan apply

# Verify
ip addr show
# Should show: inet 192.168.100.30/24
```

### Install Docker & Docker Compose

Docker makes deploying ELK Stack trivial.

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify Docker
docker --version
# Should show: Docker version 20.x+

# Install Docker Compose (often included with recent Docker)
docker-compose --version

# If not installed:
sudo apt install -y docker-compose
```

### Deploy ELK Stack

**Create directory for ELK stack**:

```bash
mkdir -p ~/soc-lab/elastic
cd ~/soc-lab/elastic
```

**Create docker-compose.yml**:

```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.13.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - xpack.security.enrollment.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - soc-network
    restart: unless-stopped

  kibana:
    image: docker.elastic.co/kibana/kibana:8.13.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - soc-network
    restart: unless-stopped

  logstash:
    image: docker.elastic.co/logstash/logstash:8.13.0
    container_name: logstash
    ports:
      - "5044:5044"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
      - logstash_data:/usr/share/logstash/data
    environment:
      - "LS_JAVA_OPTS=-Xmx256m -Xms256m"
    depends_on:
      - elasticsearch
    networks:
      - soc-network
    restart: unless-stopped

volumes:
  elasticsearch_data:
    driver: local
  logstash_data:
    driver: local

networks:
  soc-network:
    driver: bridge
```

**Create logstash.conf**:

```
input {
  beats {
    port => 5044
  }
}

filter {
  # Parse Sysmon events
  if [agent][type] == "winlogbeat" {
    mutate {
      add_field => { "[@metadata][index_name]" => "windows-events-%{+YYYY.MM.dd}" }
    }
  }

  # Extract important fields
  if [event][id] {
    mutate {
      add_field => { "event_type" => "%{[event][id]}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{[@metadata][index_name]}-%{+YYYY.MM.dd}"
  }
}
```

**Deploy**:

```bash
# Start containers
docker-compose up -d

# Check status
docker-compose ps
# All should show "Up"

# View logs
docker-compose logs -f elasticsearch
# Wait for "started"

# Test Elasticsearch
curl http://localhost:9200/
# Should return cluster info
```

### Configure Firewall (if needed)

```bash
# Allow ports
sudo ufw allow 5601  # Kibana
sudo ufw allow 5044  # Logstash (Winlogbeat)
sudo ufw allow 9200  # Elasticsearch (localhost only in production)

# Enable firewall
sudo ufw enable
```

### Access Kibana

```
URL: http://192.168.100.30:5601
Username: elastic
Password: (none configured, skip login)
```

---

## Part 5: Network Connectivity Verification (10 minutes)

### From Windows Target

```powershell
# Test SIEM connectivity
Test-NetConnection 192.168.100.30 -Port 9200
Test-NetConnection 192.168.100.30 -Port 5044

# Both should show: TcpTestSucceeded: True

# Test Kali connectivity
Test-NetConnection 192.168.100.20 -Port 22
# Should succeed
```

### From SIEM Server

```bash
# Check if receiving logs
curl -X GET "localhost:9200/_cat/indices?v"

# Should show winlogbeat indices appearing
# Example: winlogbeat-2024.05.18

# If no indices yet, Winlogbeat may not be running
# Check Winlogbeat service on Windows target
```

### In Kibana

1. Navigate to http://192.168.100.30:5601
2. Menu → Management → Dev Tools
3. Run:
```
GET _cat/indices
```

You should see indices like:
```
winlogbeat-2024.05.18
```

If not appearing after 5 minutes:
1. Verify Windows Winlogbeat service is running
2. Check network connectivity between VMs
3. Review Winlogbeat configuration

---

## Part 6: Troubleshooting

### Logs not appearing in SIEM

**Step 1**: Verify Winlogbeat is running
```powershell
Get-Service winlogbeat
# Should show: Running
```

**Step 2**: Check Winlogbeat logs
```powershell
Get-Content "C:\ProgramData\winlogbeat\logs\winlogbeat"
# Look for connection errors
```

**Step 3**: Verify network connectivity
```powershell
Test-NetConnection 192.168.100.30 -Port 5044 -InformationLevel Detailed
# Should show: TcpTestSucceeded: True
```

**Step 4**: Restart Winlogbeat
```powershell
Restart-Service winlogbeat
Start-Sleep -Seconds 10
Get-Service winlogbeat
```

### VMs can't communicate

1. **Verify VirtualBox network**:
   - VirtualBox → Preferences → Network
   - Confirm SOC-LAB network exists

2. **Check VM network settings**:
   - VM → Settings → Network
   - Adapter 1: Internal Network, Name: SOC-LAB

3. **Test from each VM**:
   ```
   Windows: ping 192.168.100.30
   Linux: ping 192.168.100.10
   ```

### Elasticsearch memory issues

If Elasticsearch won't start:

```bash
# Check logs
docker-compose logs elasticsearch

# If memory error, reduce allocation
# Edit docker-compose.yml:
environment:
  - "ES_JAVA_OPTS=-Xms256m -Xmx256m"

# Rebuild
docker-compose down
docker-compose up -d
```

---

## Part 7: Verification Checklist

Before starting attack scenarios:

- [ ] All 3 VMs running
- [ ] Windows target has Sysmon running
- [ ] Winlogbeat service running on Windows
- [ ] SIEM containers running (`docker ps`)
- [ ] Kibana accessible at http://192.168.100.30:5601
- [ ] Logs appearing in Elasticsearch indices
- [ ] Network connectivity between all VMs verified
- [ ] PowerShell logging enabled on Windows
- [ ] Attack tools installed on Kali (Metasploit, Mimikatz)

---

## Next Steps

Once deployment is complete:

1. ✅ Run [Attack Scenario 1: Credential Theft](attack-scenarios/001-credential-theft/README.md)
2. ✅ Observe logs in Kibana
3. ✅ Create detection rules
4. ✅ Hunt for indicators
5. ✅ Complete remaining scenarios

**Total deployment time: 3-4 hours**

---

**Status**: Production Ready  
**Version**: 1.0.0  
**Last Updated**: 2026-05-18