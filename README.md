# ADDC-Setup

This project documents the installation and base configuration of Active Directory Domain Controller to simulate a business-grade on-prem setup for endpoints to join a domain and enforce group policy controls.

---

## Overview
**Platform:** VirtualBox  
**ISO Used:** `pfSense-CE-2.8.0`  
**Base Setup Includes:**
- Multiple NICs to simulate VLAN-separated networks
- Static IP assignments
- DHCP services per segment
- Foundational firewall rules

---

## Virtual Machine Configuration  

|  Adapter  | VirtualBox Network  | Purpose           |
|-----------|---------------------|-------------------|
| Adapter 1 | NAT             | Internet access (WAN) |
| Adapter 2 | LabNet_Trunk    | Managenment/WebUI     |
| Adapter 3 | LabNet_VLAN10   | Infrastructure (DC01) |
| Adapter 4 | LabNet_VLAN20   | Sales Clients  |

> **Note:** Each VLAN is mapped to a separate VirtualBox Internal Network. VLAN tagging (802.1Q) is not supported in VirtualBox, so each virtual NIC represents a "VLAN" segment.

---

## Initial Setup

### 1. Download and Create VM
 - Navigated to `https://www.pfsense.org/download/` and downloaded ISO for virtual machines
 - Open VirtualBox, create new VM selecting the downloaded ISO of pfSense
 - Provide VM with 1 CPU, 2G RAM, and 15G Storage
 - Configure 4 adapters to simulate VLAN configuration

Ref 1: pfSense VM Configuration
![pfSense_VM_Configuration](https://github.com/user-attachments/assets/156e5807-00e6-44d6-a6dc-a0040d144a96)

### 2. Configure pfSense Interfaces
 - Connect second VM to LabNet_Trunk interface to get a connection to pfSense
 - Navigate to `192.168.1.1` to access WebUI of pfSense
 - After login change default credentials for security
 - Navigate to `Interfaces>Assignments` and ensure all 4 interfaced are showing and connected `em0` = WAN, `em1` = Management (LabNet_Trunk), `em2` = VLAN10_INFRA (LabNet_VLAN10), `em3` = VLAN20_SALES (LabNet_VLAN20)
 - Edit `em2` and `em3` changing their name to simulate VLANs, enable interface, set IPv4 to static, set VLAN IP
> VLAN10_INFRA IP: 192.168.10.1/24, VLAN20_SALES IP: 192.168.20.1/24

Ref 2: Interface
![pfSense_Interfaces](https://github.com/user-attachments/assets/0bd3cb82-8197-439a-81f2-bb0ad15e4586)

 - Navigate to `Services>DHCP Server`, select the simulated VLAN interface
 - Turn on DHCP, setting the range `192.168.10.100 - 192.168.10.200` DNS `8.8.8.8` and `1.1.1.1`
> Do this for each VLAN changing the 3rd octect to the correct VLAN IP

Ref 3: DHCP Settings
![pfSense_DHCP_Settings](https://github.com/user-attachments/assets/61a727a4-0ae7-417a-9459-c381c105d27b)

### 3. Configure Firewall Rules
 - Navigate to `Firewall>Rules` and select `VLAN10` to begin setting rules.
 - To allow the DC01 to access pfSense WebUI create rule `Action: Pass, Interface: 192.168.10.5 Protocol: TCP, Source: 192.168.10.5, Destination: 192.168.10.1, Port: 443, Description: Allow access to pfSense WebUI`
 - Temporarily add an allow all outbound rule for internet `Action: Pass, Interface: 192.168.10.5 Protocol: TCP/UPD, Source: 192.168.10.5, Destination: Any, Port: Any, Description: Allow all outbound internet access`
 - Create rules for domain join ports; LDAP, Kerberos, SMB, NTP, DNS. 

Ref 4: VLAN10_INFRA Rules
![VLAN10_Firewall_Rules](https://github.com/user-attachments/assets/9fee0708-fdea-4e20-b7f9-78a4211f25f0)

- Create VLAN20_SALES rule for domain joining. Allowing Kerberos, LDAP, SMB, NTP, DNS.
- Create Temporary allow all outbout rule for internet `Action: Pass, Interface: VLAN20_SALES Protocol: TCP/UPD, Source: VLAN20_SALES, Destination: Any, Port: Any, Description: Allow all outbound internet access`

Ref 5: VLAN20_SALES Rules
![VLAN20_Firewall_Rules](https://github.com/user-attachments/assets/7fe51df3-1cf0-47a2-868f-3efa058281cd)

> VLAN30_HR and VLAN40_HR_FS01 rules will be craeted later as there is not enough network interfaced in VirtualBox to have all simulated VLANs at the same time. 
---

##  Skills Practiced

- Virtual Firewall Deployment (pfSense CE)
- Multi-NIC Configureation in VirtualBox
- DHCP and Static IP Addressing per Network Segment
- Inter-VLAN Routing and Access Control
- Firewall Rule Creation and Port Based Filtering
- Documentation of Technical Configuration
