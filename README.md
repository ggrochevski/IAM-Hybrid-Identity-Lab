# Hybrid Identity & Access Management Lab

![Active Directory](https://img.shields.io/badge/Active%20Directory-0078D4?style=flat&logo=microsoft&logoColor=white)
![Microsoft Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Windows Server 2019](https://img.shields.io/badge/Windows%20Server%202019-0078D6?style=flat&logo=windows&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=flat&logo=powershell&logoColor=white)
![BloodHound](https://img.shields.io/badge/BloodHound-CC0000?style=flat&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=flat&logo=kalilinux&logoColor=white)
![Zero Trust](https://img.shields.io/badge/Zero%20Trust-222222?style=flat&logoColor=white)

---

## What This Project Demonstrates

This lab combines enterprise IAM administration with an identity security assessment — the same two-sided discipline that IAM teams practice in production environments.

| Skill Area | What Was Built |
|---|---|
| Active Directory Administration | AD DS, DNS, OUs, users, groups, GPOs, delegated admin |
| Microsoft Entra ID | Tenant setup, cloud identities, admin roles |
| Hybrid Identity | Entra Connect Sync, Password Hash Sync, AD as source of truth |
| RBAC & Access Governance | AGDLP model, least privilege, group-based permissions |
| Conditional Access & Zero Trust | MFA enforcement, device trust, risk-based policies |
| Identity Security Assessment | BloodHound ACL analysis, privilege escalation path discovery |
| Attack Validation | CrackMapExec (credential testing), Impacket secretsdump |
| Troubleshooting | DNS, NIC/APIPA, time sync, SCP, device registration, sync delays |

---

## Overview

This project simulates a modern enterprise hybrid identity environment — integrating on-premises Active Directory with Microsoft Entra ID and Microsoft 365 — then analyzes it for identity-based security risks using BloodHound, CrackMapExec, and Impacket.

The lab was built in two distinct phases: **administration** (building and managing the identity infrastructure) and **security assessment** (analyzing it for privilege escalation paths, validating findings, and demonstrating impact).

**Key outcomes:**
- Functional hybrid identity sync between `treasury.local` and `grochevski.onmicrosoft.com`
- Conditional Access enforcing MFA and device trust for all users
- BloodHound identified a valid privilege escalation path from a standard user to Domain Admin
- Attack path validated end-to-end: permission discovery → password reset → authentication → credential access

---

## Architecture

```mermaid
flowchart TB

subgraph On-Premises ["On-Premises (VirtualBox)"]
    DC01["DC01\nWindows Server 2019\nAD DS · DNS\n10.10.10.10"]
    Client["Windows 10 Client\nDomain Joined\n10.10.10.20"]
    Kali["Kali Linux\nBloodHound · Neo4j\nCrackMapExec · Impacket"]
end

subgraph Cloud ["Microsoft Cloud"]
    Entra["Microsoft Entra ID\ngrochevski.onmicrosoft.com"]
    M365["Microsoft 365\nCloud Applications"]
    CA["Conditional Access\nPolicies"]
end

DC01 -->|"Manages"| Client
DC01 -->|"Entra Connect Sync"| Entra
Kali -->|"BloodHound-Python\nLDAP/Kerberos Collection"| DC01
Entra --> M365
Entra --> CA
CA -->|"MFA + Device Trust"| Client
```

---

## Lab Environment

| Component | Details |
|---|---|
| Domain Controller | DC01 · Windows Server 2019 · 10.10.10.10 |
| Client | Windows 10 · Domain Joined · 10.10.10.20 |
| Cloud Tenant | Microsoft Entra ID · grochevski.onmicrosoft.com |
| Security Platform | Kali Linux · BloodHound CE · Neo4j |
| Domain | treasury.local |
| Users | 15+ user accounts across 9 departments |
| Groups | 10+ security groups (Global + Domain Local) |
| OUs | 9 departmental OUs + Corp-Groups, Corp-Computers, Corp-Admins |
| GPOs | Password policy, account lockout, security hardening |

---

## Organizational Structure

```
treasury.local
├── Corp-Users
│   ├── ABCC
│   ├── CashManagement
│   ├── CleanWaterTrust
│   ├── DebtManagement
│   ├── IT
│   ├── MSRB
│   ├── OEE
│   ├── UnclaimedProperty
│   └── VeteransBonus
├── Corp-Groups
│   ├── Global Groups       (GG_ prefix)
│   └── Domain Local Groups (DL_ prefix)
├── Corp-Computers
└── Corp-Admins
```

---

## Authorization Model (AGDLP)

Access to file share resources follows the AGDLP model — the enterprise-standard approach for scalable AD authorization.

```mermaid
flowchart LR
    User["User Account\ne.g. emily.davis"] --> GG
    GG["Global Group\nGG_OEE_Employees"] --> DL
    DL["Domain Local Group\nDL_OEE_Share_Modify"] --> Resource
    Resource["File Share\n\\\\DC01\\OEE"]
```

> Permissions are assigned **only** to Domain Local Groups. No user ever receives direct resource access.

---

## Hybrid Identity & Conditional Access

```mermaid
flowchart TD
    Login["User Login\n(Domain Credentials)"] --> AD
    AD["Active Directory\nIssues TGT via Kerberos"] -->|"Entra Connect Sync"| Entra
    Entra["Microsoft Entra ID"] --> CA
    CA{"Conditional Access\nPolicy Check"}
    CA -->|"All Users Policy"| MFA["MFA Challenge"]
    MFA --> DeviceCheck{"Hybrid Azure AD\nJoined?"}
    DeviceCheck -->|"Yes"| Allow["✅ Access Granted"]
    DeviceCheck -->|"No"| Block["❌ Access Blocked"]
```

**Policies configured:**
- Require MFA for all users across all cloud applications
- Sign-in risk → Require MFA (Identity Protection)
- User risk → Require password change
- Require Hybrid Azure AD Joined device for resource access

Hybrid join validated with:
```powershell
dsregcmd /status
# AzureAdJoined : YES
# DomainJoined  : YES
```

---

## Security Assessment — BloodHound Analysis

After building and validating the IAM environment, a security assessment was performed to determine whether misconfigured delegated permissions created privilege escalation paths. This phase reflects the defensive identity security work IAM teams perform in production environments.

### Methodology

```
1. Data Collection
   BloodHound-Python collected users, groups, computers,
   sessions, ACLs, and group memberships via LDAP/Kerberos

2. Attack Path Analysis
   BloodHound + Neo4j graphed AD object relationships
   and surfaced permission-based attack paths

3. Manual Validation
   Findings verified in ADUC by reviewing security
   descriptors and confirming delegated rights

4. Controlled Exploitation
   Attack path executed in isolated lab to confirm
   real-world impact of the misconfiguration

5. Findings Documentation
   Risk, impact, and severity documented;
   remediation recommendations produced
```

### Privilege Escalation Path Discovered

BloodHound identified the following attack path:

```mermaid
flowchart LR
    John["JOHN.SMITH\nStandard User"]
    Perms["GenericWrite\nForceChangePassword\n(Delegated Permissions)"]
    Sarah["SARAH.ADMIN\nTarget Account"]
    DA["Domain Admins\nGroup"]
    DC["DC01\nDomain Controller"]

    John -->|"Holds"| Perms
    Perms -->|"Applied To"| Sarah
    Sarah -->|"Member Of"| DA
    DA -->|"Controls"| DC
```

**Translation:** A standard user (`john.smith`) held delegated permissions that allowed resetting the password of `sarah.admin`, who was a member of Domain Admins — enabling a full path from standard user to domain compromise.

### Validation Steps

**Step 1 — BloodHound Discovery**

BloodHound-Python collected AD data and Neo4j graphed the relationship. `GenericWrite` and `ForceChangePassword` permissions were identified on `SARAH.ADMIN` with `JOHN.SMITH` as the principal.

**Step 2 — Manual Permission Verification (ADUC)**

Confirmed in Active Directory Users and Computers:
- Security descriptor on `sarah.admin` showed `JOHN.SMITH` with `GenericWrite` and `Reset Password` rights
- Rights confirmed as directly delegated, not inherited

**Step 3 — Password Reset**

Delegated `ForceChangePassword` right used to reset `sarah.admin`'s password in a controlled test.

**Step 4 — Authentication Validation (CrackMapExec)**

```bash
crackmapexec smb <DC_IP> -u sarah.admin -p <new_password>
# Result: [+] treasury.local\sarah.admin:<password> (Pwn3d!)
```

Authentication succeeded — account takeover confirmed via delegated permission.

**Step 5 — Impact Demonstration (Impacket)**

```bash
impacket-secretsdump treasury.local/sarah.admin:<password>@<DC_IP>
# Result: Credential material successfully retrieved
```

Domain Admin credential access demonstrated — showing potential for full domain compromise through identity misconfiguration.

---

## Security Findings

| # | Finding | Risk | Impact | Severity |
|---|---|---|---|---|
| 1 | Excessive delegated password reset rights on privileged account | Low-privileged user can reset DA account password | Unauthorized account takeover | 🔴 High |
| 2 | GenericWrite permissions on privileged user | Attacker can modify target user attributes | Privilege escalation | 🔴 High |
| 3 | Privileged group membership of compromised account | Sarah.Admin is a Domain Admin | Potential domain-wide compromise | 🔴 Critical |
| 4 | Attack path invisible without tooling | Path not discoverable through standard admin review | Undetected escalation risk | 🟠 Medium |
| 5 | Hybrid identity governance gap | Privileged on-prem accounts sync to cloud tenant | Cloud resource exposure | 🟠 Medium |

---

## MITRE ATT&CK Mapping

| Activity | Technique | ID |
|---|---|---|
| AD user and group enumeration | Account Discovery | T1087 |
| Group membership analysis | Permission Groups Discovery | T1069 |
| BloodHound relationship mapping | Domain Trust Discovery | T1482 |
| Delegated password reset | Account Manipulation | T1098 |
| Authentication with reset credentials | Valid Accounts | T1078 |
| Domain credential extraction | OS Credential Dumping | T1003 |

---

## Build Phases

| Phase | Focus | Key Activities |
|---|---|---|
| 0 | AD Foundation | Windows Server 2019, AD DS, DNS, domain promotion, `treasury.local` |
| 1 | Network & Hybrid Readiness | APIPA fix, NIC roles, DNS forwarders, internet connectivity |
| 2 | Directory Administration | 15+ user accounts, 10+ groups, 9 dept OUs, AGDLP model |
| 3 | Group Policy | Password complexity, account lockout, security hardening |
| 4 | Microsoft Entra ID | Tenant creation, cloud identities, admin roles, M365 integration |
| 5 | Entra Connect Sync | Express config, Password Hash Sync, identity synchronization verified |
| 6 | Conditional Access & MFA | MFA-for-all policy, risk-based policies, break-glass admin excluded |
| 7 | Hybrid Device Join | SCP config, time sync fix, `dsregcmd` validation, device-based CA policy |
| 8 | Security Assessment | BloodHound collection, attack path analysis, CME/Impacket validation |

---

## Tools & Technologies

**Identity & Directory**
`Active Directory DS` `Microsoft Entra ID` `Microsoft 365` `Entra Connect Sync` `Windows Server 2019` `Windows 10`

**Administration**
`ADUC` `GPMC` `DNS Manager` `PowerShell` `dsregcmd` `klist`

**Networking & Auth**
`DNS` `DHCP` `Kerberos` `LDAP` `Password Hash Sync`

**Security Assessment**
`BloodHound CE` `BloodHound-Python` `Neo4j` `CrackMapExec` `Impacket` `Kali Linux`

---

## Repository Structure

```
IAM-Hybrid-Identity-Lab/
├── docs/               # Phase documentation and lab notes
├── images/             # Architecture diagrams, BloodHound screenshots, validation evidence
└── README.md
```

---

## Key Takeaways

Building this lab as both an IAM administrator and a security assessor revealed how quickly standard delegation mistakes create high-severity attack paths — and why manual reviews alone are insufficient for detecting them. BloodHound surfaces relationships that no spreadsheet-based access review would catch, which is exactly why understanding identity attack paths is a core competency for IAM teams operating in AD environments.

---

*Built as a portfolio project demonstrating enterprise IAM administration, hybrid identity, and identity security assessment. All security testing was performed in an isolated, controlled lab environment.*
