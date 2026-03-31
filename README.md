# Hybrid Identity and Access Management (IAM) Lab

## Project Overview

This project demonstrates the design, implementation, and validation of a Hybrid Identity and Access Management (IAM) environment integrating on-premises Active Directory with Microsoft Entra ID.

The lab began as a traditional on-prem AD environment and was later upgraded to a hybrid architecture using Microsoft Entra Connect Sync.

The objective was to simulate how real organizations manage:

* On-prem identity
* Cloud identity
* Device identity
* Authentication and authorization
* Conditional Access
* Zero Trust access control

---

## Architecture Overview

### On-Premises Environment

* Domain Controller: Windows Server 2019 (DC01)
* Domain: treasury.local
* Roles:

  * Active Directory Domain Services (AD DS)
  * DNS Server

### Network Design

Dual-NIC architecture:

**Internal Network (AD Communication)**

* Subnet: 10.10.10.0/24
* DC01: 10.10.10.10
* Windows 10 Client: 10.10.10.20

**External Network (Internet / Cloud Access)**

* NAT Network: 10.0.2.0/24

### DNS Configuration

* DC01 configured as primary DNS server
* DNS Forwarders:

  * 8.8.8.8
  * 1.1.1.1

---

## Phase 0 — Active Directory Foundation

### Domain Deployment

* Promoted Windows Server to Domain Controller
* Configured domain: treasury.local
* Verified authentication and domain join functionality

### Organizational Unit Structure

Enterprise-style OU structure implemented:

* Corp-Users (department-based)
* Corp-Groups (Global and Domain Local)
* Corp-Computers
* Corp-Admins

Departments included:

* ABCC
* CashManagement
* CleanWaterTrust
* DebtManagement
* DefinedContributionPlans
* IT
* MSRB
* OEE
* ServiceAccounts
* UnclaimedProperty
* VeteransBonus

### Identity and Access Model (AGDLP)

Implemented Role-Based Access Control using AGDLP:

Accounts → Global Groups → Domain Local Groups → Permissions

* Users assigned to Global Groups
* Global Groups nested into Domain Local Groups
* Permissions assigned only to Domain Local Groups

No direct user permissions were used.

---

## File Share and Authorization

### Resource

* Path: C:\TreasuryShares\OEE
* Share: \DC01\OEE

### Permissions

* DL_OEE_Share_Read → Read
* DL_OEE_Share_Modify → Modify

Validated:

* NTFS permissions
* Share permissions
* Group-based authorization

---

## Authentication Validation

Kerberos authentication validated using:

klist

Confirmed:

* Ticket Granting Ticket (TGT)
* Service tickets for resource access

---

## Group Policy (Security Baseline)

Configured:

* Password complexity
* Minimum password length
* Account lockout policy

---

## Phase 1 — Network and Hybrid Readiness

### Issues Resolved

* APIPA address on NAT adapter
* DNS misconfiguration
* No internet connectivity from domain clients

### Fixes Applied

* Correct NIC roles (Internal vs NAT)
* Set DNS to Domain Controller
* Added DNS forwarders
* Validated connectivity using:

  * ipconfig /all
  * ping
  * nslookup

---

## Phase 2 — Microsoft Entra ID Setup

### Tenant

* Domain: grochevski.onmicrosoft.com
* Admin account created

### Capabilities Used

* User management
* Conditional Access
* Device management
* Identity Protection features

---

## Phase 3 — Entra Connect Sync

### Configuration

* Installed Microsoft Entra Connect on DC01
* Used Express Settings
* Connected:

  * On-prem AD
  * Entra ID tenant

### Result

Users synchronized from AD to Entra ID:

* On-prem AD became source of truth
* Cloud identities created successfully

---

## Phase 4 — Multi-Factor Authentication (MFA)

### Conditional Access Policy

* Policy: Require MFA for All Users
* Scope: All users (admin excluded)
* Target: All cloud apps

### Result

* MFA registration enforced
* Microsoft Authenticator required

---

## Phase 5 — Risk-Based Conditional Access

Configured adaptive security policies:

* Sign-in Risk → Require MFA
* User Risk → Require Password Change

This introduced:

* Risk-based authentication
* Automated identity protection

---

## Phase 6 — Hybrid Device Join

### Initial Issue

Device showed:

* AzureAdJoined: NO
* DomainJoined: YES

### Root Causes

* Incorrect join method (workplace join only)
* OU placement issues
* Time synchronization failure

### Fixes

* Configured NTP time sync
* Resolved token timing errors
* Configured SCP using Entra Connect
* Corrected AD object placement

### Validation

Command:

dsregcmd /status

Result:

* AzureAdJoined: YES
* DomainJoined: YES

---

## Phase 7 — Device-Based Conditional Access

### Policy

Require Hybrid Joined Device

### Configuration

* Applied to test user
* Targeted all cloud apps
* Required hybrid-joined device

### Outcome

* Access allowed from trusted device
* Access blocked from unmanaged devices

---

## Phase 8 — Access Validation

### Trusted Device

* Access granted

### Untrusted Device

* Access blocked with Conditional Access policy

This validated Zero Trust enforcement.

---

## Writeback and Identity Model

* Synchronization: AD → Entra ID
* No full cloud-to-AD writeback

On-prem AD remains source of truth.

---

## Key Concepts Demonstrated

### Identity Infrastructure

* Active Directory Domain Services
* DNS for identity resolution

### Authentication

* Kerberos
* MFA

### Authorization

* AGDLP model
* NTFS and Share permissions

### Hybrid Identity

* Entra ID
* Entra Connect Sync
* Hybrid Azure AD Join

### Zero Trust

* Conditional Access
* Device trust enforcement

### Troubleshooting

* DNS resolution issues
* Network configuration
* Time synchronization errors
* Hybrid join failures

---

## Final Outcome

This lab demonstrates a fully functional Hybrid IAM environment where:

* Identities originate in Active Directory
* Users synchronize to Microsoft Entra ID
* Devices are hybrid joined
* MFA is enforced
* Conditional Access controls access
* Untrusted devices are blocked

---

## Portfolio Summary

Built a Hybrid Identity and Access Management lab integrating on-prem Active Directory with Microsoft Entra ID using Entra Connect Sync. Implemented Kerberos authentication, AGDLP authorization, hybrid identity synchronization, MFA, Conditional Access, and device-based Zero Trust enforcement. Validated secure access by allowing only trusted hybrid-joined devices and blocking unmanaged endpoints.

---
