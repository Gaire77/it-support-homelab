# 03 - Active Directory Setup

This section documents the deployment of the Domain Controller for the lab, including Windows Server 2022 installation, server configuration, Active Directory Domain Services (AD DS) installation, and forest/domain creation.

## Overview

The Domain Controller is the heart of the lab environment. It hosts Active Directory, DNS, and DHCP, providing centralized identity, name resolution, and IP address management for all domain-joined client workstations. The decisions made in this section determine how every subsequent component (GPOs, user provisioning, ticket scenarios) will operate.

## VM Specifications

A new virtual machine was created in Oracle VM VirtualBox with specifications appropriate for a Domain Controller running AD DS, DNS, and DHCP roles.

| Setting | Value | Reasoning |
|---------|-------|-----------|
| VM Name | WindowsServer10 (Computer renamed DC01 post-install) | Matches the lab naming convention |
| OS Type | Windows Server 2022 (64-bit) | Matches the ISO being installed |
| RAM | 12288 MB (12 GB) | Generous allocation to support AD DS, DNS, DHCP, and future role additions |
| CPUs | 2 cores | Sufficient for a single-DC lab environment |
| Disk | 50 GB dynamically allocated VDI | Enough for OS plus AD database growth |
| Network Adapter 1 | Host-only (192.168.56.x) | Isolates the lab network from production traffic |
| Network Adapter 2 | NAT | Provides internet access for Windows Updates |

![VirtualBox Server VM Settings](../08-Screenshots/03-Active-Directory-Setup/01-virtualbox-server-vm-settings.png)

## Windows Server 2022 Installation

The Windows Server 2022 evaluation ISO was attached to the VM as a virtual optical drive. The VM was booted from the ISO to begin Windows Server installation.

![Server Setup - Language Selection](../08-Screenshots/03-Active-Directory-Setup/02-server-language-selection.png)

The Standard with Desktop Experience edition was selected to provide a graphical interface for management. The license terms were reviewed and accepted.

![License Terms](../08-Screenshots/03-Active-Directory-Setup/03-server-license-terms.png)

The installation proceeded through file copying, feature installation, and system preparation phases.

![Installation in Progress](../08-Screenshots/03-Active-Directory-Setup/04-server-installation-progress.png)

After installation, the built-in Administrator account was configured with a strong password during initial setup.

![Administrator Password Setup](../08-Screenshots/03-Active-Directory-Setup/05-server-administrator-password-setup.png)

On first boot, Server Manager opened automatically, presenting the standard configuration wizard.

![First Boot - Server Manager](../08-Screenshots/03-Active-Directory-Setup/06-server-first-boot-server-manager.png)## Server Configuration

### Computer Rename

The default auto-generated computer name was changed to DC01 to align with the lab's naming convention. This identifies the machine as the primary Domain Controller and follows standard enterprise naming practices.

![Rename Computer to DC01](../08-Screenshots/03-Active-Directory-Setup/07-computer-rename-to-dc01.png)

### Static IP Configuration

A static IP address was assigned to the Host-only network adapter to ensure the Domain Controller is always reachable at a known address - a requirement for AD DS and DNS to function reliably.

| Adapter | Purpose | IP Address | Subnet Mask |
|---------|---------|------------|-------------|
| Ethernet (Host-only) | Lab internal network | 192.168.56.10 | 255.255.255.0 |
| Ethernet 2 (NAT) | Internet access for updates | 10.0.3.15 (DHCP) | 255.255.255.0 |

The dual-adapter configuration isolates lab traffic on the Host-only network while still allowing the server to reach Microsoft for updates through NAT. This mirrors a real production pattern of separating management and external traffic.

![ipconfig - Static IP Verification](../08-Screenshots/03-Active-Directory-Setup/08-ipconfig-static-ip-verification.png)

## Active Directory Domain Services Installation

With the server renamed and addressed, the AD DS role was added through the Add Roles and Features Wizard. Group Policy Management and Remote Server Administration Tools (RSAT) were installed alongside as part of the AD DS role group.

![AD DS Role Installation Complete](../08-Screenshots/03-Active-Directory-Setup/09-ad-ds-role-installation-complete.png)

### Domain Controller Promotion

After role installation, the server was promoted to a Domain Controller using the post-deployment configuration wizard. A new forest was created with the root domain `lab.local`.

Configuration choices made during promotion:

- Deployment Operation: Add a new forest
- Root Domain Name: lab.local
- Forest Functional Level: Windows Server 2016 (compatibility-friendly)
- Domain Functional Level: Windows Server 2016
- DNS Server: Installed as part of promotion (recommended)
- Global Catalog: Yes (required for first DC in a forest)
- Read Only Domain Controller: No
- DSRM Password: Set during promotion (stored securely outside the lab documentation)
- NetBIOS Name: LAB (auto-generated from lab.local)

The server rebooted to complete the promotion.

## Verification

After promotion, Server Manager confirmed that AD DS and DNS roles were installed and healthy.

![Server Manager - AD DS and DNS Roles](../08-Screenshots/03-Active-Directory-Setup/10-server-manager-ad-ds-dns-roles.png)

The Tools menu in Server Manager exposed the full Active Directory toolkit including Active Directory Users and Computers (ADUC), Active Directory Administrative Center, Active Directory Sites and Services, Group Policy Management, and DNS Manager.

![Server Tools Menu - AD Utilities](../08-Screenshots/03-Active-Directory-Setup/11-server-tools-menu-ad-utilities.png)

Active Directory Users and Computers was opened to verify the new domain. The lab.local domain appeared in the navigation tree.

![ADUC - Lab.local Domain Visible](../08-Screenshots/03-Active-Directory-Setup/12-aduc-lab-local-domain.png)

Expanding lab.local revealed the default Organizational Unit structure including Builtin, Computers, Domain Controllers, ForeignSecurityPrincipals, Managed Service Accounts, and Users. The Domain Controllers OU correctly contained DC01 as the only Domain Controller.

![ADUC - Default OUs with DC01](../08-Screenshots/03-Active-Directory-Setup/13-aduc-default-ous-with-dc01.png)

## Final State

After this section, the lab has a fully operational Active Directory environment with the lab.local forest and domain created, DC01 functioning as the Domain Controller running AD DS and DNS, the Domain Controller accessible at the static IP 192.168.56.10 on the lab network, and the standard Active Directory utilities installed and ready for further configuration.

## Lessons Learned

Setting a static IP for the Domain Controller before installing AD DS is essential. AD DS depends on stable DNS, and DNS depends on a stable IP address. If the DC's IP were to change after promotion, the entire domain would have stability problems.

The dual network adapter approach (Host-only for lab traffic, NAT for internet) reflects real production patterns where management and external networks are separated. It also keeps the lab cleanly isolated from any other devices on the host machine's network.

Choosing the Standard with Desktop Experience edition over Server Core during installation made administrative tasks significantly easier for a learning environment. Server Core is more efficient in production, but the GUI is invaluable for learning the AD toolset.

## Next Section

04 - DNS and DHCP Configuration: Configuring DNS forwarders, reverse lookup zones, and DHCP scope for client workstation IP assignment.
