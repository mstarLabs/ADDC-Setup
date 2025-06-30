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
![DC01_VM_Configuration](https://github.com/user-attachments/assets/d42e1ba6-36cf-4783-afa5-f31c794c609c)

### 2. Add AD DS Feature
 - Once isntalled first need to set the static IP to `192.168.10.5` and DNS to `127.0.0.1`
 - Opened Server Manager navigated to "Add Roles and Features"
 - Checked Active Directory Domain Services
 - Kept default settings
 - After install promoted server to domain controllor

Ref 2: ADDS Promote to Domain Controllor
![DC01_PromoteToDC](https://github.com/user-attachments/assets/56001562-a967-47c3-9311-c44bbf0f625c)

 - Added a new forest `lab.local`
 - Kept DNS making DC01 the DNS server as well
 - Once installed switched to LAB\Administrator user
 - Opened DNS Manager to ensure Host(A) was created and DNS was set to `8.8.8.8` and `1.1.1.1`

Ref 3: DNS Manager
![DC01_DNSManager](https://github.com/user-attachments/assets/49ff9ed2-f458-47bb-985b-62e23048adad)

 - Opened `Active Director Users and Computers` to create some new OU folders for department users
 - Created a Sales and HR test user

Ref 3: User Listing
![DC01_Users](https://github.com/user-attachments/assets/99be2487-f96b-47f1-aad6-dab0f0c68278)

### 3. Domain Join Sales Client (Win10)
 - Navigated to `https://www.microsoft.com/en-us/software-download/windows10` and downloaded Windows 10 Installation Media
 - Created a Windows 10 ISO using Microsofts creation media tool
 - Open VirtualBox, create new VM selecting the downloaded ISO of Windows 10
 - Provide VM with 2 CPU, 4G RAM, and 50G Storage
 - Configure 1 adapter to simulate VLAN configuration (LabNet_VLAN20)
 - During instalation selected Windows 10 Pro

Ref 4: Windows 10 VM Configuration
![SalesClient_VM_Configuration](https://github.com/user-attachments/assets/cb9e3cff-6162-431b-8233-fbd0b0cd15f9)

- Check communication from VLAN20_SALES to VLAN10_INFRA ensure they can see eachother in the simulated VLANs with pfSense
- Connect to lab.local domain using the Sales Test user as login

Ref 5: Sales Client Domain Joined
![SalesClient_DomainJoin](https://github.com/user-attachments/assets/a9361b8c-d495-4f32-b953-20e53c124846)

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

Ref 6: Tested port connection to DC01
![SalesClient_PortTest](https://github.com/user-attachments/assets/5fc18da7-e9b1-40fe-ae12-85ee914d9128)
 
 - Discovered an article that provides a list of ports that need to be added to pfSense to allow AD Domain join
 - Made changes to VLAN20
 - I was missing port `135` for RPC endpoint mapper, port `139` for NetBIOS Sessions Service, ports  `49152 - 65535` for dynamic RPC ports port `137` and `138` were optional but added them anyway
 - To get internet I added TCP/UPD port `443` and `80` and got rid of the allow all to any rule
 - On VLAN10 I set a DC to any on port TCP/UDP 53 allow to get DNS working.
 - These changed allowed the Sales client on VLAN20 to resolve DNS and join the domain lab.local

---

##  Skills Practiced

- Active Directory Domain Service setup
- Creating new OU containers and users within the domain
- Ensuring DNS Manger is configured
- 
