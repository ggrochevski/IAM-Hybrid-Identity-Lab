# Lab Architecture

## Environment Summary
This lab was built in Oracle VirtualBox using a Host-Only network to simulate an isolated enterprise identity environment.

## Systems

### Domain Controller
- **Hostname:** DC01
- **OS:** Windows Server 2019
- **Roles Installed:**
  - Active Directory Domain Services (AD DS)
  - DNS Server
- **IP Address:** 10.10.10.10

### Client Machine
- **OS:** Windows 10
- **Status:** Domain Joined
- **IP Address:** 10.10.10.x

## Network Configuration
The Windows 10 client was configured to use the Domain Controller as its DNS server.

- **Preferred DNS Server:** 10.10.10.10

This allowed:
- successful domain join
- domain logon
- Kerberos ticket issuance
- Active Directory service discovery

## Domain
- **Domain Name:** `treasury.local`

## Validation Performed
The architecture was validated through:
- successful promotion of the Domain Controller
- successful domain join
- successful login using domain users
- Kerberos ticket validation
- access to domain resources
