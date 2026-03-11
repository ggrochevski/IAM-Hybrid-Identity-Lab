# Lab Architecture

## Environment Overview

This lab was built using Oracle VirtualBox to simulate a small enterprise identity infrastructure using Microsoft Active Directory.

The environment consists of a Domain Controller running Active Directory Domain Services and a domain-joined Windows client used to simulate a workstation authenticating against the directory.

---

## Virtualization Platform

- Platform: Oracle VirtualBox
- Network Type: Host-Only Network

The Host-Only network allows the virtual machines to communicate with each other while remaining isolated from the external network.

---

## Systems

| System | Role | Operating System | IP Address |
|------|------|------|------|
| DC01 | Domain Controller | Windows Server 2019 | 10.10.10.10 |
| CLIENT01 | Domain Joined Workstation | Windows 10 | 10.10.10.20 |

---

## Domain Configuration

Domain Name:

```
treasury.local
```

The domain acts as the central identity directory where user accounts, groups, and access permissions are managed.

---

## Server Roles

The Domain Controller (DC01) hosts the following roles:

- Active Directory Domain Services (AD DS)
- DNS Server

Active Directory provides identity management while DNS enables service discovery required for authentication.

---

## DNS Configuration

The Windows client is configured to use the Domain Controller as its DNS server.

```
Preferred DNS Server: 10.10.10.10
```

This configuration allows the client to locate domain services such as:

- LDAP
- Kerberos
- Domain Controllers

DNS configuration was validated implicitly through:

- successful domain join
- successful domain login
- Kerberos ticket issuance
- successful access to domain resources

---

## Domain Join Validation

The Windows 10 client was successfully joined to the domain.

```
treasury.local
```

Validation included:

- logging in using domain credentials
- accessing domain resources
- Kerberos ticket generation
- group-based authorization behavior

These tests confirmed that the domain infrastructure was functioning correctly.
