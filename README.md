# Hybrid Identity & Access Management Lab

## Overview

This project began as a traditional on-premises Active Directory lab and was later upgraded into a Hybrid Identity and Access Management environment using Microsoft Entra ID and Microsoft Entra Connect Sync.

The lab simulates how a real organization manages:

- on-premises identity
- cloud identity
- authentication
- authorization
- device identity
- Conditional Access
- Zero Trust access controls

In addition to the hybrid upgrade, the Active Directory environment was later redesigned into a cleaner enterprise-style OU structure with role-based access control using the AGDLP model and PowerShell-based user provisioning.

---

## Project Goals

The goal of this lab was to simulate a real-world enterprise IAM environment by demonstrating:

- on-prem Active Directory as the identity source
- Microsoft Entra ID as the cloud identity platform
- synchronized hybrid identities using Entra Connect Sync
- Kerberos-based authentication
- group-based authorization using AGDLP
- MFA enforcement through Conditional Access
- hybrid Microsoft Entra joined device trust
- Zero Trust style access enforcement
- enterprise OU restructuring and identity lifecycle organization

---

## Core Technologies Used

- Windows Server 2019
- Active Directory Domain Services (AD DS)
- DNS
- Group Policy
- Kerberos
- SMB / NTFS permissions
- Microsoft Entra ID
- Microsoft Entra Connect Sync
- Conditional Access
- Microsoft Authenticator
- PowerShell
- Oracle VirtualBox

---

## Phase 0 - On-Prem Active Directory Foundation

A Windows Server 2019 machine named **DC01** was deployed and promoted to a Domain Controller for the domain:

`treasury.local`

Core roles installed:

- Active Directory Domain Services
- DNS Server

This established the on-prem identity provider for the environment.

### DNS Configuration

DNS was configured as part of the AD DS deployment and used to support:

- domain resource resolution
- Kerberos authentication
- domain join
- Active Directory service discovery

The Domain Controller was configured as the primary DNS server for the internal domain.

### Authentication Validation

Kerberos authentication was validated using:

```powershell
klist
