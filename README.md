# ADDS-Setup

This project documents the installation and base configuration of Active Directory Domain Controller to simulate a business-grade on-prem setup for endpoints to join a domain and enforce group policy controls.

---

## Overview
**Platform:** VirtualBox  
**ISO Used:** `Windows Server 2019`  
**Base Setup Includes:**
- Internal NIC set to simulate VLAN-separated networks
- Static IP assignment
- Dirctory Services Restore Mode (DSRM) Password: **vmlab2019!**
- DNS server
- Foundational group policies

---

## Virtual Machine Configuration  

### DC01 (Windows Server 2019)
|  Adapter  | VirtualBox Network  | Purpose           |
|-----------|---------------------|-------------------|
| Adapter 1 | LabNet_VLAN10   | Infrastructure (DC01) |

### Sales_Client (Win 10)
|  Adapter  | VirtualBox Network  | Purpose           |
|-----------|---------------------|-------------------|
| Adapter 1 | LabNet_VLAN20   | Sales Clients (Act as VLAN) |

> **Note:** Each VLAN is mapped to a separate VirtualBox Internal Network. VLAN tagging (802.1Q) is not supported in VirtualBox, so each virtual NIC represents a "VLAN" segment.

---

## Initial Setup

### 1. Download and Create VM (DC01)
 - Navigated to `https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019` and downloaded ISO
 - Open VirtualBox, create new VM selecting the downloaded ISO of Windows Server 2019
 - Provide VM with 2 CPU, 4G RAM, and 30G Storage
 - Configure 1 adapter to simulate VLAN configuration (LabNet_VLAN10)

Ref 1: Windows Server 2019 VM Configuration
![pfSense_VM_Configuration](https://github.com/user-attachments/assets/156e5807-00e6-44d6-a6dc-a0040d144a96)

### 2. Add AD DS Feature
 - Once isntalled first need to set the static IP to `192.168.10.5` and DNS to `127.0.0.1`
 - Opened Server Manager navigated to "Add Roles and Features"
 - Checked Active Directory Domain Services
 - Kept default settings
 - After install promoted server to domain controllor

Ref 2: ADDS Promote to Domain Controllor
![pfSense_Interfaces](https://github.com/user-attachments/assets/0bd3cb82-8197-439a-81f2-bb0ad15e4586)

 - Added a new forest `lab.local`
 - Kept DNS making DC01 the DNS server as well
 - Once installed switched to LAB\Administrator user
 - Opened DNS Manager to ensure Host(A) was created and DNS was set to `8.8.8.8` and `1.1.1.1`

Ref 3: DNS Manager
![pfSense_DHCP_Settings](https://github.com/user-attachments/assets/61a727a4-0ae7-417a-9459-c381c105d27b)

 - Opened `Active Director Users and Computers` to create some new OU folders for department users
 - Created a Sales and HR test user

Ref 3: User Listing

### 3. Domain Join Sales Client (Win10)
 - Navigated to `https://www.microsoft.com/en-us/software-download/windows10` and downloaded Windows 10 Installation Media
 - Created a Windows 10 ISO using Microsofts creation media tool
 - Open VirtualBox, create new VM selecting the downloaded ISO of Windows 10
 - Provide VM with 2 CPU, 4G RAM, and 50G Storage
 - Configure 1 adapter to simulate VLAN configuration (LabNet_VLAN20)
 - During instalation selected Windows 10 Pro

Ref 4: Windows 10 VM Configuration
![VLAN10_Firewall_Rules](https://github.com/user-attachments/assets/9fee0708-fdea-4e20-b7f9-78a4211f25f0)

- Check communication from VLAN20_SALES to VLAN10_INFRA ensure they can see eachother in the simulated VLANs with pfSense
- Connect to lab.local domain using the Sales Test user as login

Ref 5: Sales Client Domain Joined
![VLAN20_Firewall_Rules](https://github.com/user-attachments/assets/7fe51df3-1cf0-47a2-868f-3efa058281cd)

> VLAN30_HR and VLAN40_HR_FS01 rules will be craeted later as there is not enough network interfaced in VirtualBox to have all simulated VLANs at the same time.

---

## Troubleshooting Issues

### Domain Joining
 - Domin join worked right off the bat, but figured this was do to the catch allow all rule at the end
 - When rule is disabled domain join does not work nor did nslookup to ensure DNS was working
 - Discovered pfSense source or destiniaton labled with "interface name" address did not mean all IPs on that interface
 - Changed all firewall setting to "interface name" subnet which did mean all IPs on that interface
 - DNS was still not working after that change so looked at VLAN10 rules to ensure I made the change to subnet as well
 - Even with the change still could not get DNS to work; After a change of destination to any on port 53 with the source teh DC01, DNS started resolving
 - I am still not sure why setting he destination to Any works only thing I can think of is has to do with routing in pfSense
 - Ran testing in powershell tihe a `Test-NetConnection lab.local -Port 389` as well as port `135`, and port `445` These all came back good but domain join still did not work
 - Discovered an article that provides a list of ports that need to be added to pfSense to allow AD Domain join

---

##  Skills Practiced

- Virtual Firewall Deployment (pfSense CE)
- Multi-NIC Configureation in VirtualBox
- DHCP and Static IP Addressing per Network Segment
- Inter-VLAN Routing and Access Control
- Firewall Rule Creation and Port Based Filtering
- Documentation of Technical Configuration
