
<br>

# **10 - Conclusion and Summary**


<br>

## **Final Evaluation**

This project presents a security-focused enterprise network built in GNS3. The design follows a firewall-centric model, where OPNsense acts as the main control and protection point. The project demonstrates how routing, switching, services, and security features are combined into one stable and protected network.

The topology uses a simple layered structure with edge routing, firewall processing, distribution and access switches, and separate management and server areas. Each part of the network has a defined role, which supports easier understanding, simpler troubleshooting, and controlled traffic flow between internal networks and external connectivity.

IP addressing and VLAN segmentation separate users, servers, management, and guest access. VLANs are created and routed through OPNsense, which works as the gateway between network segments. A dedicated management VLAN 99 is used for centralized device management and secure administration of switches, routers, and the firewall. At the switching layer, LACP improves link reliability, while PortFast and BPDU Guard protect access ports and reduce Layer 2 risks.

Core services run on a Windows Server 2022 placed in a protected server zone. DHCP provides automatic address assignment, DNS supports internal name resolution using the local domain infra.corp, and HTTP is used to demonstrate internal services. These services are used as part of the firewall and ACL policy on OPNsense, which defines how different VLANs can reach internal resources. User VLANs have limited access based on service needs, the guest network is isolated with Internet-only access, and the NET-ADMIN system is allowed administrative access across the network. External ISP and Internet networks have no direct access to internal systems.

Secure remote administration is demonstrated using a SOHO-ADMIN system connected through a WireGuard VPN on OPNsense. The administrator manages the network from home using encrypted access and SSH. Device-level security completes the design through centralized RADIUS authentication, protected management access, monitored login attempts, disabled unused interfaces, and port security on access switches. The result is a secure network design that reflects real practices used in small and medium-sized enterprise environments.

<br>

## **Topology Diagram**


![](images/Pasted%20image%2020260131065147.png)

<br>


## **Project Overview**

**1. Network Topology and Devices**  
The project starts with a clear definition of the network topology and used devices. The design introduces edge routing, firewall placement, switching layers, server systems, and administrative hosts. Each device has a defined role, which creates a structured base for the whole network.

**2. Addressing and VLAN Planning**  
This chapter focuses on IPv4 addressing and VLAN design. Network segments are separated based on function, including users, servers, management, and guest access. The addressing plan supports clear structure, scalability, and secure traffic separation.

**3. Basic Device Configuration**  
Basic configuration is applied to all routers and switches. This includes hostnames, interface addressing, port descriptions, and basic management settings. The goal is to create a consistent and readable configuration across all network devices.

**4. Switching and VLAN Configuration**  
This section covers VLAN creation, access and trunk port configuration, and Layer 2 protection features. LACP improves link reliability, while PortFast and BPDU Guard protect access ports and reduce the risk of switching issues.

**5. Routing and OPNsense Integration**  
Inter-VLAN communication is integrated with the OPNsense firewall. VLAN routing is centralized, and traffic between network segments is controlled through firewall rules. This chapter introduces the firewall-centric design used across the project.

**6. Core Server Services**  
Core infrastructure services are deployed on a Windows Server 2022. DHCP provides dynamic addressing, DNS supports internal name resolution using the infra.corp domain, and HTTP is used to demonstrate internal services within the protected server zone.

**7. OPNsense Firewall and NAT/PAT**  
This chapter focuses on firewall policy and traffic control. Access rules define how different VLANs can communicate, isolate the guest network, and protect internal systems from external access. NAT/PAT enables controlled Internet connectivity while keeping internal networks protected.

**8. Remote Access VPN – WireGuard**  
Secure remote administration is implemented using WireGuard on OPNsense. A SOHO administrator connects from an external network and manages internal systems through encrypted access without opening management services to the Internet.

**9. Advanced Network Device Security**  
The final chapter improves security at the device level. Centralized authentication using RADIUS controls administrative access, management interfaces are protected, unused ports are disabled, and port security limits access at the switch level. This completes the security model of the project.



<br>

---




<br>


**Back to project overview:** 



