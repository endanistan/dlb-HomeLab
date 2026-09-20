![PowerShell Banner](images/banner.svg)
# dlb-HomeLab
My Homelab overview

# Infrastructure Homelab & Enterprise Architecture

A production-aligned Windows Server and networking lab built with a focus on High Availability (HA), automated configuration management, and zero-trust security principles.

---

## Physical & Network Infrastructure

### Physical Compute Nodes
* **2x Mini-Desktops (Singleton Hyper-V Hosts)**
  * **CPU:** AMD Ryzen 7 7745HX
  * **RAM:** 32 GB DDR5
  * **Storage:** 2x 1 TB M.2 NVMe Gen4 SSD
  * **NICs:** 4x GbE Wired NICs

### Network & Segmentation
* **Ubiquiti UniFi Ecosystem**
  * **Gateway:** Inter-VLAN routing, firewall policies, and security segmentation.
  * **Switching & AP:** 2x Managed Switches, 1x Access Point.

![Network Overview](images/NetworkSchema.jpg)

---

## Virtual Machines & Workloads

| Hostname | OS | Role / Services | Key Architectural Details |
| :--- | :--- | :--- | :--- |
| `DC1` | ws25 Server Core | AD DS, DNS | FSMO Roles: Schema Master, Domain Naming Master |
| `DC2` | ws25 Server Core | AD DS, DNS | FSMO Roles: PDC Emulator, RID Master, Infrastructure Master |
| `DHCP1` | ws25 Server Core | DHCP Server | Primary DHCP server (Active/Standby relationship) |
| `DHCP2` | ws25 Server Core | DHCP Server | Hot Standby failover partner to `DHCP1` |
| `FS1` | ws25 Server Core | File Services | DFS-N (`\\Contoso\Data Files`), DFS-R, Data Deduplication, FSRM |
| `FS2` | ws25 Server Core | File Services | DFS-R replication partner to `FS1` |
| `CA1` | ws25 Server Core | AD CS (PKI) | Enterprise Certificate Authority |
| `CA-CRL1` | ws2025 Server Core | IIS Web Server | Standalone HTTP endpoint publishing CA1 Certificate Revocation List (CRL) |
| `DSC1` | ws25 Server Core | IaC / Automation | PowerShell DSC Pull Server using DFS repository & GPO targeting |
| `VPN1` | ws25 Server Core | RRAS / Remote Access | P2S IKEv2 Tunnel, Let's Encrypt TLS, RADIUS auth via `NPS1` |
| `NPS1` | ws25 Desktop Experience | Network Policy Server | RADIUS Server enforcing EAP-TLS (User & Device certs) for VPN & Wi-Fi |
| `WACGW1` | ws25 Server Core | Management Gateway | Windows Admin Center v2 gateway over WinRM HTTPS (Kerberos) |
| `AUTO1` | ws25 Server Core | Script Automator Platform | Uses PowerShell scripts and Just Enough Administration endpoints to automate active directory state |

---

## Management, Hardening & Tools

### Administration & Operations
* **Remote Management:** Restricted to WinRM over HTTPS (TCP 5986), SSH, and Windows Admin Center (WACv2).
* **Configuration Management:** PowerShell Desired State Configuration (DSC) Pull Server backed by Group Policy Preferences.

### Security Hardening & Identity
* **Least Privilege:** Just Enough Administration (JEA) endpoints configured for delegated service management.
* **Identity & Service Accounts:** Group Managed Service Accounts (gMSA) utilized for automated services.
* **Public Key Infrastructure:** Full AD CS environment enforcing certificate-based and Kerberos authentication across networks.

