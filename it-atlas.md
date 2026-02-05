# IT-ATLAS: Bank Infrastructure Reference

## Purpose

This document provides IR-ASSIST with contextual knowledge of typical large US bank infrastructure. Use this reference to:
- Understand where systems fit in the architecture
- Identify investigation paths and log sources
- Recognize high-value assets and critical systems
- Guide containment and remediation strategies

**Note:** This represents a generalized model of large bank infrastructure, not proprietary details.

---

## Infrastructure Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        BANK INFRASTRUCTURE LAYERS                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   INTERNET  │  │   CLIENTS   │  │  PARTNERS   │  │  REGULATORS │        │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘        │
│         │                │                │                │               │
│  ═══════╪════════════════╪════════════════╪════════════════╪═══════════    │
│         │         EXTERNAL PERIMETER (DMZ)                 │               │
│  ═══════╪════════════════════════════════════════════════════════════════  │
│         │                                                                   │
│  ┌──────┴──────────────────────────────────────────────────────────────┐   │
│  │                    CORPORATE NETWORK                                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │   │
│  │  │ Workstations│  │   Servers   │  │   Users     │                  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│         │                                                                   │
│  ═══════╪═══════════════════════════════════════════════════════════════   │
│         │         INTERNAL SEGMENTATION                                     │
│  ═══════╪═══════════════════════════════════════════════════════════════   │
│         │                                                                   │
│  ┌──────┴──────────────────────────────────────────────────────────────┐   │
│  │                    CORE BANKING ZONE                                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │   │
│  │  │Core Banking │  │  Databases  │  │  Mainframe  │                  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌────────────────────┐  ┌────────────────────┐                            │
│  │   BRANCH NETWORK   │  │    CLOUD (GCP)     │                            │
│  │  (500+ locations)  │  │  (Hybrid Connect)  │                            │
│  └────────────────────┘  └────────────────────┘                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Network Architecture

### Network Zones

| Zone | Purpose | Security Level | Key Systems |
|------|---------|----------------|-------------|
| **DMZ** | External-facing services | High scrutiny | Web servers, email gateway, VPN concentrators |
| **Corporate** | General business operations | Standard | Workstations, file servers, print servers |
| **Restricted** | Sensitive business functions | Elevated | HR systems, legal, executive |
| **Core Banking** | Transaction processing | Maximum | Core banking apps, mainframe, databases |
| **Branch** | Retail locations | Segmented | Teller workstations, ATMs, branch servers |
| **Cloud** | GCP workloads | Per-workload | Analytics, some apps, DR |

### Network Segmentation

```
┌─────────────────────────────────────────────────────────────────┐
│                    NETWORK SEGMENTATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   INTERNET                                                      │
│       │                                                         │
│       ▼                                                         │
│   ┌───────────────┐                                             │
│   │  Edge FW      │  Palo Alto (PA-5260)                        │
│   │  (Perimeter)  │  - IPS/IDS enabled                          │
│   └───────┬───────┘  - Threat Prevention                        │
│           │          - URL Filtering                            │
│           ▼                                                     │
│   ┌───────────────┐                                             │
│   │     DMZ       │  VLAN 10-19                                 │
│   └───────┬───────┘                                             │
│           │                                                     │
│           ▼                                                     │
│   ┌───────────────┐                                             │
│   │  Internal FW  │  Palo Alto (PA-5250)                        │
│   │  (Core)       │  - Microsegmentation                        │
│   └───────┬───────┘  - User-ID integration                      │
│           │                                                     │
│     ┌─────┴─────┬─────────────┬─────────────┐                   │
│     ▼           ▼             ▼             ▼                   │
│ ┌────────┐ ┌────────┐   ┌──────────┐  ┌──────────┐              │
│ │Corporate│ │Restricted│  │Core Bank │  │ Branch   │             │
│ │VLAN     │ │VLAN      │  │VLAN      │  │ WAN      │             │
│ │100-199  │ │200-299   │  │300-399   │  │ MPLS     │             │
│ └────────┘ └────────┘   └──────────┘  └──────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Network Details

| Component | Technology | Notes |
|-----------|------------|-------|
| Edge Firewalls | Palo Alto PA-5260 | Perimeter defense, IPS, URL filtering |
| Internal Firewalls | Palo Alto PA-5250 | Microsegmentation, User-ID |
| Core Switches | Cisco Nexus 9000 | Datacenter fabric |
| Branch WAN | MPLS + SD-WAN | Encrypted tunnels to branches |
| Cloud Connect | GCP Dedicated Interconnect | Hybrid connectivity |
| DNS | Internal: Windows DNS, External: Infoblox | Split-horizon |
| Proxy | Zscaler ZIA | Cloud proxy for web traffic |

### Traffic Flow Patterns (Normal)

| Source | Destination | Protocol | Purpose |
|--------|-------------|----------|---------|
| Workstation | Proxy (Zscaler) | HTTPS/443 | Web browsing |
| Workstation | Domain Controller | Kerberos/88, LDAP/389 | Authentication |
| Workstation | File Server | SMB/445 | File access |
| Server | Core Banking | Proprietary/varies | Transactions |
| Branch | Datacenter | MPLS tunnel | Branch operations |
| Any | SIEM | Syslog/514, 6514 | Log forwarding |

---

## Identity & Access Management

### Directory Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    ACTIVE DIRECTORY FOREST                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Forest Root: bank.corp                                        │
│       │                                                         │
│       ├── bank.corp (accounts, resources)                       │
│       │       │                                                 │
│       │       ├── OU=Corporate                                  │
│       │       │       ├── OU=Users                              │
│       │       │       ├── OU=Workstations                       │
│       │       │       └── OU=Servers                            │
│       │       │                                                 │
│       │       ├── OU=Branches                                   │
│       │       │       ├── OU=Region-East                        │
│       │       │       ├── OU=Region-Central                     │
│       │       │       └── OU=Region-West                        │
│       │       │                                                 │
│       │       ├── OU=Privileged                                 │
│       │       │       ├── OU=Tier0-Admins                       │
│       │       │       ├── OU=Tier1-Admins                       │
│       │       │       └── OU=Tier2-Admins                       │
│       │       │                                                 │
│       │       └── OU=Service-Accounts                           │
│       │                                                         │
│       └── dmz.bank.corp (DMZ resources)                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Privileged Access Tiers

| Tier | Access Level | Examples | Controls |
|------|--------------|----------|----------|
| **Tier 0** | Domain/Forest | Domain Admins, Enterprise Admins, DC access | PAW required, MFA, 4-hour max session |
| **Tier 1** | Server | Server admins, application admins | PAW recommended, MFA, logged |
| **Tier 2** | Workstation | Helpdesk, local admin | MFA, logged |
| **Service** | Application | svc_* accounts | Managed passwords, no interactive logon |

### Authentication Flow

```
User → Workstation → Domain Controller → Kerberos TGT
                           │
                           ▼
                     Service Ticket → Target Resource
                           │
                           ▼
                     CyberArk (for privileged)
                           │
                           ▼
                     MFA (Duo) → Approved
```

### Identity Systems

| System | Purpose | Integration Points |
|--------|---------|-------------------|
| Active Directory | Primary identity | All Windows systems |
| CyberArk | Privileged Access Management | Tier 0/1 accounts, service accounts |
| Duo | Multi-Factor Authentication | VPN, privileged access, sensitive apps |
| SailPoint | Identity Governance | Provisioning, access reviews |
| Azure AD | Cloud identity (hybrid) | O365, some cloud apps |

---

## Security Operations Stack

### Detection & Monitoring

| Tool | Function | Coverage | Log Source |
|------|----------|----------|------------|
| **Splunk** | SIEM | Enterprise-wide | Central log aggregation |
| **CrowdStrike Falcon** | EDR | All endpoints | Endpoint telemetry |
| **Palo Alto Cortex XDR** | Network + Endpoint | Network, servers | Network + endpoint correlation |
| **Darktrace** | Network anomaly | Internal network | Network traffic analysis |
| **Proofpoint** | Email security | Email gateway | Email logs, threats |

### Log Sources & Retention

| Source | Log Type | Retention | Notes |
|--------|----------|-----------|-------|
| Windows Event Logs | Security, System, Application | 90 days hot, 1 year cold | Forwarded to Splunk |
| Firewall | Traffic, Threat | 90 days hot, 1 year cold | Palo Alto to Splunk |
| Proxy | Web access | 90 days | Zscaler logs |
| DNS | Query logs | 30 days | Internal DNS servers |
| VPN | Connection logs | 90 days | Cisco AnyConnect |
| EDR | Endpoint telemetry | 30 days hot, 90 days searchable | CrowdStrike |
| Authentication | AD, Duo, CyberArk | 1 year | Critical for investigation |

### Security Tools Quick Reference

| Tool | Primary Use | Key Capabilities |
|------|-------------|------------------|
| **Splunk** | Log analysis, alerting | Search, correlation, dashboards |
| **CrowdStrike** | Endpoint investigation | Process tree, file analysis, containment |
| **CyberArk** | Credential investigation | Session recordings, credential access logs |
| **Palo Alto** | Network investigation | Traffic logs, threat logs, URL logs |
| **Proofpoint** | Email investigation | Email trace, attachment analysis |

---

## Critical Assets (High Value Targets)

### Tier 1 Critical (Immediate escalation if compromised)

| Asset Category | Examples | Impact |
|----------------|----------|--------|
| Domain Controllers | DC01, DC02, etc. | Complete domain compromise |
| Core Banking | FIS, Fiserv, Jack Henry systems | Transaction integrity |
| SWIFT Infrastructure | SWIFT servers, HSMs | Wire fraud, regulatory |
| Certificate Authority | Internal PKI | Trust compromise |
| Backup Systems | Veeam, Commvault servers | Recovery capability |
| Security Tools | SIEM, EDR consoles | Visibility loss |

### Tier 2 Critical (Escalate within 1 hour)

| Asset Category | Examples | Impact |
|----------------|----------|--------|
| Database Servers | SQL clusters, Oracle RAC | Data breach |
| File Servers | Sensitive data repositories | Data breach |
| Email Servers | Exchange, O365 admin | BEC, data leak |
| HR Systems | Workday, PeopleSoft | PII exposure |
| Executive Workstations | C-suite devices | High-value target |

### Data Classification

| Classification | Examples | Handling |
|----------------|----------|----------|
| **Restricted** | Customer PII, account data, credentials | Encrypted at rest/transit, strict access |
| **Confidential** | Internal financials, strategy docs | Need-to-know access |
| **Internal** | General business documents | Employee access |
| **Public** | Marketing materials | No restrictions |

---

## Branch Infrastructure

### Typical Branch Layout

```
┌─────────────────────────────────────────────────────────────────┐
│                    BRANCH NETWORK                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   WAN Link (MPLS)                                               │
│       │                                                         │
│       ▼                                                         │
│   ┌───────────────┐                                             │
│   │ Branch Router │  Cisco ISR                                  │
│   │ + Firewall    │  SD-WAN enabled                             │
│   └───────┬───────┘                                             │
│           │                                                     │
│     ┌─────┴─────┬─────────────┐                                 │
│     ▼           ▼             ▼                                 │
│ ┌────────┐ ┌────────┐   ┌──────────┐                            │
│ │ Teller │ │ ATM    │   │ Branch   │                            │
│ │ VLAN   │ │ VLAN   │   │ Server   │                            │
│ └────────┘ └────────┘   └──────────┘                            │
│                                                                 │
│ Teller Workstations: 3-8 per branch                             │
│ ATMs: 1-3 per branch                                            │
│ Branch Server: Local AD, print, cache                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Branch-Specific Concerns

| System | Risk | Investigation Notes |
|--------|------|---------------------|
| ATM | Skimming, jackpotting, malware | Isolated VLAN, limited connectivity |
| Teller Workstations | Customer data access | High PII exposure |
| Branch Server | Local cache of credentials | Lateral movement target |
| Physical Access | Tailgating, social engineering | Badge logs, cameras |

---

## Cloud Infrastructure (GCP)

### Hybrid Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    GCP INTEGRATION                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   On-Premises                      GCP                          │
│   ───────────                      ───                          │
│                                                                 │
│   ┌─────────────┐    Dedicated    ┌─────────────┐              │
│   │ Datacenter  │◄──Interconnect──►│ VPC Network │              │
│   └─────────────┘    (encrypted)  └──────┬──────┘              │
│         │                                │                      │
│         │                          ┌─────┴─────┐                │
│         │                          ▼           ▼                │
│         │                    ┌─────────┐ ┌─────────┐            │
│         │                    │ GKE     │ │ BigQuery│            │
│         │                    │ Clusters│ │ (Analyt)│            │
│         │                    └─────────┘ └─────────┘            │
│         │                                                       │
│   ┌─────┴─────┐                                                 │
│   │ IAM Sync  │──── Cloud Identity Federation                   │
│   └───────────┘                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### GCP Security Controls

| Control | Implementation | Notes |
|---------|----------------|-------|
| Identity | Cloud Identity federated with AD | SSO via Azure AD |
| Network | Private Google Access, VPC Service Controls | No public IPs on workloads |
| Logging | Cloud Audit Logs → Splunk | 400-day retention |
| Secrets | Secret Manager | Application credentials |
| Encryption | Customer-managed keys (CMEK) | HSM-backed |

---

## Investigation Quick Reference

### By Incident Type

| Incident Type | Primary Log Sources | Key Systems | First Actions |
|---------------|---------------------|-------------|---------------|
| **Malware/Ransomware** | CrowdStrike, Windows Events | Affected endpoint, file servers | Isolate endpoint, check lateral movement |
| **Phishing** | Proofpoint, O365, CrowdStrike | Email, user workstation | Block sender, check who clicked, credential reset |
| **Unauthorized Access** | AD logs, CyberArk, Duo | Domain controllers, target system | Disable account, review access history |
| **Data Exfiltration** | Proxy, Firewall, DLP | Source system, egress points | Block destination, preserve evidence |
| **Lateral Movement** | AD logs, CrowdStrike, Firewall | Source, targets, DC | Map scope, contain affected systems |
| **Credential Theft** | CyberArk, AD, CrowdStrike | DC, PAM, source endpoint | Force password reset, check usage |

### Key Log Queries (Splunk)

```spl
# Failed logons for a user
index=wineventlog EventCode=4625 user=<username>

# Successful logons from unusual location
index=wineventlog EventCode=4624 user=<username> | stats count by src_ip

# Process execution on endpoint
index=crowdstrike host=<hostname> | table _time, ImageFileName, CommandLine

# Firewall blocks to suspicious destination
index=pan_traffic action=denied dest_ip=<ip>

# DNS queries for domain
index=dns query=*<domain>*
```

### Containment Options by System Type

| System | Containment Method | Tool | Reversibility |
|--------|-------------------|------|---------------|
| Workstation | Network isolation | CrowdStrike | Easy |
| Server | Firewall block | Palo Alto | Easy |
| User Account | Disable in AD | AD console | Easy |
| Service Account | Disable + rotate | CyberArk | Medium |
| Network Segment | VLAN isolation | Switch/Firewall | Medium |
| Cloud Workload | VPC rules | GCP Console | Easy |

---

## Contact Quick Reference

> **[PLACEHOLDER: Populate with actual contact information]**

| Role | Name | Phone | Email |
|------|------|-------|-------|
| SOC Manager | | | |
| Incident Commander (Primary) | | | |
| Incident Commander (Backup) | | | |
| Network Team Lead | | | |
| Windows/AD Team Lead | | | |
| Database Team Lead | | | |
| Cloud Team Lead | | | |
| Legal | | | |
| Communications | | | |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | February 2026 | [Author] | Initial creation |
