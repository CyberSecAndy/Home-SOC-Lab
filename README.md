# Home SOC Lab 🛡️

A comprehensive, production-grade Security Operations Center (SOC) lab for hands-on learning of threat detection, incident response, and security operations.

## 📋 Overview

This project provides a fully functional home lab environment for:

- **Threat Detection**: Create and test detection rules
- **Incident Response**: Practice IR techniques and methodologies
- **Threat Hunting**: Hunt for indicators of compromise (IOCs)
- **Log Analysis**: Analyze security logs at scale
- **Attack Simulation**: Run realistic attack scenarios
- **SIEM Operations**: Learn Splunk and/or Elastic Stack
- **Forensics**: Analyze artifacts and build evidence

## 🎯 Learning Objectives

After completing this lab, you will understand:

✅ Complete SOC workflow (detect → investigate → respond)  
✅ SIEM deployment and configuration  
✅ Log collection and parsing  
✅ Detection rule creation and tuning  
✅ Incident response procedures  
✅ Threat hunting methodologies  
✅ MITRE ATT&CK framework application  
✅ Windows security event analysis  
✅ PowerShell logging and analysis  
✅ Network traffic analysis  

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SIEM Server (Ubuntu)                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Elasticsearch  │  Kibana  │  Logstash/Filebeat    │   │
│  │  192.168.100.30 │          │  Port 5044 Listener   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │ Sysmon/PowerShell/Windows Logs
                            │ (Winlogbeat 5044)
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
   ┌────────────────┐               ┌──────────────────┐
   │ Windows Target │               │  Kali Attacker   │
   │ (192.168.100.10)              │ (192.168.100.20)  │
   │                               │                  │
   │ - Sysmon        │               │ - Metasploit    │
   │ - Winlogbeat    │               │ - Mimikatz      │
   │ - PowerShell    │               │ - Evil-WinRM    │
   │ - Logging       │               │ - Atomic RT      │
   └────────────────┘               └──────────────────┘

         Internal Network: 192.168.100.0/24
         (VirtualBox Internal Network "SOC-LAB")
```

## 📁 Repository Structure

```
Home-SOC-Lab/
├── README.md                          # This file
├── SETUP-GUIDE.md                     # Step-by-step deployment
├── ARCHITECTURE.md                    # Technical deep-dive
├── infrastructure/                    # Configuration files
│   ├── docker-compose.yml             # ELK Stack deployment
│   ├── logstash.conf                  # Log processing
│   ├── sysmon-config.xml              # Windows event logging
│   └── network-setup.md               # Network configuration
├── attack-scenarios/                  # Attack walkthroughs
│   ├── 001-credential-theft/
│   ├── 002-lateral-movement/
│   ├── 003-privilege-escalation/
│   ├── 004-data-exfiltration/
│   ├── 005-persistence/
│   └── 006-ransomware/
├── detections/                        # Detection rules
│   ├── sigma/                         # Sigma rules
│   ├── splunk/                        # Splunk SPL
│   └── elastic/                       # Elastic KQL
├── hunting/                           # Threat hunting queries
│   ├── playbooks/
│   └── queries/
├── investigations/                    # Case studies
│   └── case-templates/
├── scripts/                           # Automation scripts
│   ├── setup.sh                       # Lab initialization
│   └── cleanup.sh                     # Lab cleanup
└── resources/                         # Additional materials
    ├── mitre-mapping.md               # ATT&CK techniques
    ├── glossary.md                    # SOC terminology
    └── troubleshooting.md             # Common issues
```

## 🚀 Quick Start (5 Minutes)

### Prerequisites
- VirtualBox or Proxmox
- 16GB+ RAM
- 150GB+ disk space
- Windows ISO
- Kali Linux ISO

### Three-Step Deployment

**Step 1**: Create VirtualBox internal network
```bash
# VirtualBox UI: Preferences → Network → Add
# Name: SOC-LAB
# IPv4: 192.168.100.1/24
```

**Step 2**: Deploy VMs
- Windows Target: 192.168.100.10 (4GB RAM, 50GB disk)
- Kali Attacker: 192.168.100.20 (4GB RAM, 30GB disk)
- SIEM Server: 192.168.100.30 (8GB RAM, 50GB disk)

**Step 3**: Follow SETUP-GUIDE.md for detailed configuration

For complete instructions, see **[SETUP-GUIDE.md](SETUP-GUIDE.md)**

## 🎓 Attack Scenarios

### Scenario 1: Credential Theft
**MITRE ATT&CK**: T1003 (Credential Dumping)  
**Duration**: 30-45 minutes  
**Techniques**: Mimikatz, LSASS dump, Password hash extraction  
**Learning**: Detect credential access, analyze process memory access

### Scenario 2: Lateral Movement
**MITRE ATT&CK**: T1021 (Remote Services)  
**Duration**: 45-60 minutes  
**Techniques**: PsExec, WMI, RDP, Pass-the-Hash  
**Learning**: Detect lateral movement, track network authentication

### Scenario 3: Privilege Escalation
**MITRE ATT&CK**: T1548 (Privilege Escalation)  
**Duration**: 30-45 minutes  
**Techniques**: UAC bypass, DLL injection, Token impersonation  
**Learning**: Identify privilege escalation attempts, detect suspicious processes

### Scenario 4: Data Exfiltration
**MITRE ATT&CK**: T1041 (Exfiltration Over C2)  
**Duration**: 45-60 minutes  
**Techniques**: DNS tunneling, HTTPS tunneling, SMB exfil  
**Learning**: Detect data exfiltration, hunt for suspicious network patterns

### Scenario 5: Persistence
**MITRE ATT&CK**: T1547 (Boot or Logon Autostart)  
**Duration**: 45-60 minutes  
**Techniques**: Registry keys, Scheduled tasks, WMI subscriptions  
**Learning**: Identify persistence mechanisms, create detection rules

### Scenario 6: Ransomware
**MITRE ATT&CK**: T1561 (Disk Content Wipe)  
**Duration**: 60-90 minutes  
**Techniques**: File encryption, MBR wipe, Shadow copy deletion  
**Learning**: Detect ransomware, containment procedures, recovery

## 🔍 Detection & Hunting

### Detection Rules

Each scenario includes:
- Sigma rules (universal format)
- Splunk SPL queries
- Elastic KQL queries
- Example detections with screenshots

### Threat Hunting

Hunting playbooks for:
- Encoded PowerShell commands
- Suspicious process execution
- Unusual network connections
- Registry modification patterns
- Service installation anomalies

## 📊 SIEM Platforms

### Elastic Stack (Recommended)
- Free tier available
- Excellent visualization
- Simple to deploy (Docker)
- Great for learning

### Splunk
- Industry standard
- 500MB/day free tier
- More expensive
- Enterprise features

**This lab uses Elastic Stack by default**

## 🛠️ Key Technologies

| Component | Purpose | Version |
|-----------|---------|----------|
| **VirtualBox** | Hypervisor | 6.1+ |
| **Sysmon** | Event logging | 14.0+ |
| **Winlogbeat** | Log forwarding | 8.0+ |
| **Elasticsearch** | Log storage | 8.0+ |
| **Kibana** | Visualization | 8.0+ |
| **Logstash** | Log processing | 8.0+ |
| **Metasploit** | Exploitation | Latest |
| **Mimikatz** | Credential access | Latest |
| **Atomic Red Team** | Attack simulation | Latest |

## 📈 Learning Path

```
1. Setup (2-3 hours)
   ↓
2. Deploy SIEM (1-2 hours)
   ↓
3. Run Attack Scenarios 1-3 (3-4 hours)
   ↓
4. Create Detection Rules (2-3 hours)
   ↓
5. Perform Threat Hunting (2-3 hours)
   ↓
6. Write Incident Reports (2-3 hours)
   ↓
7. Advanced Scenarios 4-6 (4-5 hours)
   ↓
Total: ~20-25 hours to complete
```

## ✅ Verification Checklist

Before starting attack scenarios:

- [ ] Windows VM deployed with Sysmon
- [ ] Kali VM deployed with tools
- [ ] SIEM server with Elasticsearch running
- [ ] Winlogbeat forwarding logs to SIEM
- [ ] Kibana accessible at http://192.168.100.30:5601
- [ ] Sample logs appearing in Kibana
- [ ] Network connectivity between all VMs
- [ ] Attack tools (Metasploit, Mimikatz) installed on Kali

## 🔗 Cross-References

This lab integrates with other portfolio projects:

- **[SIEM Detection & Threat Hunting](https://github.com/CyberSecAndy/siem-detection-threat-hunting)** - Uses logs from this lab
- **[Detection Rule Engineering](https://github.com/CyberSecAndy/detection-rule-engineering)** - Creates rules tested here
- **[Incident Response Case Studies](https://github.com/CyberSecAndy/incident-response-case-studies)** - Documents incidents from this lab
- **[Malware Traffic Analysis](https://github.com/CyberSecAndy/malware-traffic-analysis)** - Analyzes network traffic from scenarios
- **[SOC Automation Scripts](https://github.com/CyberSecAndy/soc-automation-scripts)** - Automates analysis tasks

## 📚 Additional Resources

### Documentation
- **[SETUP-GUIDE.md](SETUP-GUIDE.md)** - Complete deployment steps
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Technical details
- **[MITRE ATT&CK Mapping](resources/mitre-mapping.md)** - Technique reference
- **[Troubleshooting Guide](resources/troubleshooting.md)** - Common issues

### External Resources
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Elastic Documentation](https://www.elastic.co/guide/)
- [Sysmon Cheat Sheet](https://github.com/SwiftOnSecurity/sysmon-config)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [Sigma Rules](https://github.com/SigmaHQ/sigma)

## 🎓 Tips for Success

1. **Start Simple**: Begin with Scenario 1, progress sequentially
2. **Document Everything**: Screenshots, queries, findings
3. **Understand Logs**: Know what each log field means
4. **Verify Detection**: Ensure rules actually trigger on attacks
5. **Test Hunting**: Validate hunting queries with known detections
6. **Iterate**: Refine rules based on testing
7. **Review Timeline**: Understand complete attack chain
8. **Map to MITRE**: Relate detections to ATT&CK techniques

## ❓ FAQ

**Q: How much disk space do I need?**  
A: Minimum 150GB total. 50GB per VM (Windows target, Kali, SIEM server) plus buffer.

**Q: Can I use this on AWS/Azure instead of VirtualBox?**  
A: Yes, but costs will be higher. VirtualBox is recommended for home lab.

**Q: How long does setup take?**  
A: 3-4 hours for complete deployment including OS installation.

**Q: What if I only have 8GB RAM?**  
A: You can run Windows + Kali simultaneously (2x 2GB), run SIEM on host OS.

**Q: Can I use Proxmox instead of VirtualBox?**  
A: Yes, Proxmox is excellent and provides better performance.

## 🐛 Troubleshooting

### VMs can't communicate
- Verify VirtualBox internal network created
- Check VM network adapter settings
- Test with `ping` from each VM

### Logs not appearing in SIEM
- Check Winlogbeat service status
- Verify network connectivity to SIEM port 5044
- Review Winlogbeat logs for errors

### SIEM running out of disk space
- Configure log retention policies
- Use index lifecycle management
- Monitor with `df -h` on SIEM server

For more issues, see **[Troubleshooting Guide](resources/troubleshooting.md)**

## 🤝 Contributing

Contributions welcome! Please:

1. Test scenarios thoroughly
2. Document new techniques
3. Include MITRE mapping
4. Add detection rules
5. Update README accordingly

## 📝 License

MIT License - See LICENSE file

## 📧 Support

For issues or questions:
1. Check [Troubleshooting Guide](resources/troubleshooting.md)
2. Review attack scenario walkthroughs
3. Create a GitHub issue

---

**Status**: ✅ Production Ready  
**Version**: 1.0.0  
**Last Updated**: 2026-05-18  
**Difficulty**: Intermediate → Advanced  
**Time to Complete**: 20-25 hours  

**Happy Hunting! 🕵️**