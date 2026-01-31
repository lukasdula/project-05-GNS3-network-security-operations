
<br>

# **Network Security Operations**


<br>

## **Introduction and Objectives**

This project presents a Cisco-based, security-focused enterprise network built in GNS3 as part of a study and portfolio series. The design is based on a firewall-centric model, where OPNsense acts as the main control point for traffic filtering, access control, and network protection. The project focuses on practical security operations applied across routing, switching, services, and management access.

The network is designed with clear separation between user, server, management, and guest networks using VLAN segmentation and controlled routing. Security policies are applied at the firewall level and supported by device-level protections on routers and switches. Centralized services, secure remote access, and controlled administration are used to demonstrate how a network can stay protected while still working reliably and easy to manage.

The main objective of this project is to show how security principles are applied across the entire network. This includes firewall rules, access control policies, secure management, VPN-based remote administration, and device authentication. The project aims to reflect realistic security practices used in small and medium-sized enterprise networks, with a focus on clear design, control, and operational reliability.



<br>


## **Topology Diagram**

![](images/Pasted%20image%2020260131083348.png)


<br>

## **Network Zones**

- **Internet / ISP**  
    Simulated external Internet that represents networks outside the organization.
    
- **WAN / Edge Zone**  
    Connection between the internal network and the external Internet, using the edge router and firewall.
    
- **Firewall Zone (OPNsense)**  
    Central security and control zone where traffic filtering, access rules, and zone separation are applied.
    
- **Management Zone (VLAN 99)**  
    Dedicated network for secure administration of routers, switches, and the firewall.
    
- **Server Zone** (Windows server 2022)
    Protected internal zone hosting core services such as DHCP, DNS, and HTTP.
    
- **User Zone**  
    Internal network for company users with controlled access to services and other zones.
    
- **Guest Zone**  
    Separated network segment with Internet-only access and no connectivity to internal resources.
    
- **Remote Administration (SOHO)**  
    External administrator access provided using a secure VPN connection for remote management.


<br>

## **Project Structure**

1. [Network Topology and Devices](01-network-topology-and-devices.md)
    
2. [Addressing and VLAN Planning](02-addressing-and-vlan-planning.md)
    
3. [Basic Device Configuration](03-basic-device-configuration.md)
    
4. [Switching and VLAN Configuration](04-switching-and-vlan-configuration.md)
    
5. [Routing and OPNsense Integration](05-routing-and-opnsense-integration.md)
    
6. [Core Server Services](06-core-server-services.md)
    
7. [OPNsense Firewall and NAT/PAT](07-opnsense-firewall-and-nat-pat.md)
    
8. [Remote Access VPN – WireGuard](08-remote-access-vpn-wireguard.md)
    
9. [Advanced Network Device Security](09-advanced-network-device-security.md)
    
10. [Conclusion and Summary](10-conclusion-and-summary.md)



<br>


## **Key Project Features**

- Firewall-centric network design with OPNsense
    
- VLAN-based network segmentation (users, servers, management)
    
- Centralized VLAN routing and traffic control
    
- Management VLAN 99 for secure device administration
    
- Layer 2 switching with access and trunk ports
    
- Link aggregation using LACP
    
- Port protection with PortFast and BPDU Guard
    
- Windows Server 2022 providing DHCP, DNS (infra.corp), and HTTP services
    
- Firewall rule policy controlling access between network zones
    
- Secure remote administration using WireGuard VPN
    
- Centralized authentication using RADIUS
    
- Device security configuration on routers and switches


<br>

## **Tools and Environment**

- **GNS3 version 2.2.54**
    
- **OPNsense Firewall**  
    OPNsense 25.7 (amd64, DVD installer image)
    
- **Xubuntu VM**  
    Kernel-based QEMU virtual machine running inside GNS3
    
- **Cisco IOSv Router**  
    VIOS-ADVENTERPRISEK9-M, Version 15.9(3)M6
    
- **Cisco IOSv-L2 Switch**  
    vios_l2-ADVENTERPRISEK9-M, Version 15.2(20170321)
    
- **Visual Studio Code**  
    Documentation editing
    
- **Obsidian**  
    Notes, summaries, and screenshots



<br>




## **Author’s Note**

This project closes the first series of five GNS3 projects focused mostly on Cisco-based networks and core networking fundamentals. It connects routing, switching, VLANs, redundancy, and services, and adds a stronger focus on network security, providing important experience for future projects.

A key learning point was using OPNsense for the first time as both a router and firewall rather than a Cisco router. Working with VLAN routing and firewall rules using the OPNsense GUI was clear and intuitive, and defining access policies was simpler than using ACLs in Cisco CLI.

I also got practical experience with WireGuard VPN and remote administration from a SOHO environment, as well as with RADIUS authentication for better device security. This network feels like the most secure design I have built so far, and working on the security parts was interesting and motivating while I was learning to work with OPNsense.


<br>

---

<br>


© 2026 – Lukáš Dula | Home Network Project & Portfolio











