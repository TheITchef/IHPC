# Hybrid Homelab Project — Master Document (2026 Edition)
## The single source of truth for understanding, navigating, and practicing the project.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-07-13 · **Owner:** Ioannis Mintzivyris

---

# 1. What This Project Is
This project is a realistic hybrid IT environment designed to teach enterprise‑grade identity, networking, compute, security, and automation.

It includes:
- Active Directory Domain Services
- PKI (two‑tier CA)
- ADFS federation
- Azure AD Connect
- Hyper‑V + ESXi compute
- Cisco L3 routing, VLANs, ACLs
- Monitoring, backup, disaster recovery
- Full documentation + Git workflow

This is not a toy homelab — it is a **training simulator for real enterprise architecture**.

---

# 2. Repository Structure

theitchef-hybrid-project/
│
├── README.md
├── CHANGELOG.md
├── ADR/
│   ├── adr-hypervisor-choice.md
│   ├── adr-adfs-vs-modern-auth.md
│   └── adr-network-segmentation.md
│
├── docs/
│   ├── governance/
│   ├── architecture/
│   ├── design/
│   ├── procedures/
│   ├── diagrams/
│   └── learning/
│
├── config/
├── src/
├── assets/
└── threads/
Code


Each folder has a clear purpose:
- **governance/** → rules, standards, SOPs  
- **architecture/** → high‑level system design  
- **design/** → low‑level technical plans  
- **procedures/** → deployment, rollback, validation  
- **diagrams/** → visuals  
- **learning/** → your personal learning log  

### Governance Documents
The project's rulebook lives in `docs/`:
- **Project Charter** — why the project exists, success criteria, constraints
- **Documentation Standard** (`docs/documentation-standard.md`) — how every document is written
- **Change-Management SOP** (`docs/governance/change-management-sop.md`) — how changes are made, tracked, and reversed
- **Kanban board** (`docs/kanban-board.md`) — live work tracking

---

# 3. Kanban Workflow (How Work Is Organized)

Backlog → Ready → Priority → WIP → Review → Done
Code


Meaning:
- **Backlog** = ideas  
- **Ready** = prepared tasks  
- **Priority** = tasks to do next  
- **WIP** = tasks in progress  
- **Review** = checking your work  
- **Done** = finished  

This keeps your work structured and stress‑free.

---

# 4. Git Workflow (How Work Is Executed)

### 1. One task = one branch  
Example:

feature/define-security-zones
Code


### 2. Commit messages follow the standard

Task: <task>
Status: <In Progress | Completed>
Summary: <short description>
Code


### 3. Merge → delete branch  
Clean history, clean repo.

---

# 5. Documentation Standard (How Docs Are Written)

Every document in this project follows the **Documentation Standard** (`docs/documentation-standard.md`) — its eight-part format, metadata block, and writing rules. The Standard is the single home for how documents are written; this Master Document does not restate it.

Documentation is your **memory written down**.

---

# 6. Architecture Overview

Your architecture has five domains:

### Identity  
AD DS, DNS, DHCP, PKI, ADFS, Azure AD Connect

### Compute  
Hyper‑V, ESXi

### Network  
Cisco WS‑3850, C3560CG, 891F  
VLANs, routing, ACLs

### Security  
Zones, trust boundaries, segmentation, flow matrix

### Monitoring  
Health checks, logs, alerts, certificate expiry, replication

---

# 7. **Active Directory Architecture (Senior Engineer / Architect Level)**

Active Directory is the **identity backbone** of your hybrid environment.

This section explains AD at a senior engineer / architect level — but in a way that remains clear and beginner‑friendly.

---

## 7.1 What Active Directory Actually Is
AD is a **distributed identity and policy system** built on:
- NTDS.dit (directory database)
- KCC + DRS (replication engine)
- Kerberos + NTLM (authentication)
- ACLs + SIDs (authorization)
- Group Policy (policy engine)
- DNS SRV records (service location)

AD is a **multi‑master, replicated, transactional identity fabric**.

---

## 7.2 Logical Structure (Forest → Domain → OU → Objects)

### Forest (security boundary)
Architect concerns:
- Schema  
- Global Catalog  
- Trusts  
- Forest functional level  

### Domain (administrative boundary)
Architect concerns:
- FSMO roles  
- Password policies  
- Replication scope  

### OU Structure (delegation boundary)
Architect concerns:
- OU hierarchy  
- Delegation model  
- GPO linking strategy  

### Objects (identity layer)
Users, groups, computers, service accounts.

---

## 7.3 Physical Structure (Sites → DCs → Replication)

### Sites
Architect concerns:
- Site links  
- Replication schedules  
- Bridgehead servers  

### Domain Controllers
Architect concerns:
- Placement  
- GC placement  
- RODC usage  
- Backup strategy  

### Replication Topology
Architect concerns:
- Intra‑site vs inter‑site  
- Latency  
- Lingering objects  
- Tombstone lifetime  
- DFSR vs FRS SYSVOL  

---

## 7.4 Authentication Architecture (Kerberos Deep Dive)

Key concepts:
- KDC  
- TGT  
- Service Ticket  
- SPN  
- PAC  
- Delegation (constrained/unconstrained)

Architect responsibilities:
- SPN hygiene  
- Delegation design  
- Service account strategy  
- Tiering model (Tier 0/1/2)  
- PAW enforcement  

---

## 7.5 Authorization Architecture (Groups + ACLs)

Use **AGDLP** or **AGUDLP**:
- Accounts  
- Global groups  
- Domain Local groups  
- Permissions  

This is how enterprises manage access cleanly.

---

## 7.6 Replication (Senior Level)

Key components:
- KCC  
- ISTG  
- DRS  
- USN  
- High‑watermark vector  
- Up‑to‑date vector  

Architect concerns:
- Latency  
- Lingering objects  
- USN rollback  
- SYSVOL consistency  

---

## 7.7 FSMO Roles (Architect View)

Forest‑wide:
- Schema Master  
- Domain Naming Master  

Domain‑wide:
- RID Master  
- PDC Emulator  
- Infrastructure Master  

Architect responsibilities:
- Placement  
- Transfer vs seize  
- Monitoring  
- Recovery  

---

## 7.8 Group Policy (Architect Level)

Architect responsibilities:
- GPO design  
- Linking strategy  
- Loopback processing  
- WMI filtering  
- Versioning  
- Replication  

Best practice:
- Keep GPOs small  
- Avoid mega‑GPOs  
- Document everything  

---

## 7.9 AD Security Architecture

Tiering model:
- Tier 0 — DCs, PKI, ADFS  
- Tier 1 — Servers  
- Tier 2 — Clients  

Hardening:
- Disable NTLM  
- Enforce LDAP signing  
- Protect KRBTGT  
- Protect service accounts  
- PAWs  

---

## 7.10 AD Disaster Recovery

Critical components:
- NTDS.dit  
- SYSVOL  
- DNS  
- PKI integration  

Architect responsibilities:
- Authoritative restore  
- Non‑authoritative restore  
- DC rebuild  
- KRBTGT reset  
- Forest recovery plan  

---

## 7.11 AD Monitoring

Monitor:
- Replication  
- DNS  
- SYSVOL  
- Kerberos failures  
- NTLM usage  
- DC performance  
- Certificate expiry  

Tools:
- dcdiag  
- repadmin  
- nltest  
- Event Viewer  
- Azure AD Connect Health  
- Sysmon  
- SIEM  

---

## 7.12 AD Interview Topics (Senior Level)

Expected topics:
- Kerberos deep dive  
- SPN troubleshooting  
- Delegation models  
- Replication troubleshooting  
- FSMO recovery  
- SYSVOL migration  
- AD site topology  
- GPO architecture  
- Tiering model  
- Service account strategy  
- Forest recovery  

---

# 8. Security Model Overview

Security is built around:
- Zones  
- Trust boundaries  
- Inter‑zone flows  
- ACLs  
- Segmentation model  
- PKI trust chain  

---

# 9. Rollback & Recovery

Every deployment has:
- Deployment procedure  
- Rollback procedure  
- Validation checklist  

This teaches senior‑level operational discipline.

---

# 10. Disaster Recovery

DR plans exist for:
- AD  
- PKI  
- ESXi  
- Hyper‑V  
- Cisco configs  

---

# 11. Monitoring & Alerting

Monitoring is part of architecture:
- AD health  
- PKI expiry  
- ESXi host health  
- Hyper‑V host health  
- Cisco syslog/SNMP  
- Windows Event Logs  

---

# 12. Architectural Decision Records (ADR)

ADRs explain **why** you made certain choices:
- Hypervisor choice  
- ADFS vs modern auth  
- Segmentation model  
- Zone definitions  

---

# 13. Learning Log

After each task, write:
- What you built  
- What surprised you  
- What you'd do differently  

This becomes interview gold.

---

# 14. Daily Workflow (How You Practice)

1. Pick a task  
2. Create a branch  
3. Open relevant docs  
4. Do the work  
5. Commit  
6. Merge  
7. Update Kanban  

This is how real IT teams work.

---

# 15. The Only Rules You Must Remember

1. One task = one branch  
2. One branch = one deliverable  
3. Every commit follows the standard  
4. Every document follows the standard  
5. Every task follows the lifecycle SOP  

Follow these and you will always be doing it right.

---