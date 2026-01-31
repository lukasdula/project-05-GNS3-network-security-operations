
<br>

# **1 - Network Topology and Devices**

<br>

## **1.1 Introduction**

This first chapter introduces the network topology designed for this Project 05. It explains the design approach and the logical structure of the topology shown in the diagram, and describes how the main components of the network are organized and connected.

The topology is built around a firewall-centric design, where a dedicated firewall represents the central security and control point of the network. Under the firewall, distribution and access switches form the internal switching structure that connects user networks, management systems, and server infrastructure. A separate management and server area is connected through the distribution layer, and a remote administrator accesses the network from a SOHO environment via the Internet.

This chapter explains the logic of the network design and how the topology is designed to operate across the project. It creates a clear base for the next chapters, where the design is further expanded with addressing, segmentation, and security features.


>**Note.:** Due to limitations of Cisco virtual switch images in GNS3, the lab uses a reduced number of switch interfaces. This configuration logic is applied across the network.

<br>

## **1.2  Topology Diagram**

![](images/Pasted%20image%2020260124154834.png)


<br>

## **1.3 Network Zones**

This section describes the main network zones in the topology and their roles.

<br>


**Internet and ISP**

- Simulated external connectivity providing access to public networks.
    

**Firewall Zone (OPNsense)**

- Central security zone controlling traffic between internal networks and external connectivity.
    

**Distribution Zone**

- Connects the firewall to the access zone and management/server networks
    

**Access Zone**

- Provides Layer 2 access for end devices in Office, Finance, Sales, and Guest VLANs.
    

**Management and Server Zone**

- NET-ADMIN system provides controlled administrative access to network devices.
    
- Windows Server hosts core services such as DHCP, DNS, and HTTP for internal use.
    

**SOHO Administration**

- Demonstrates secure remote access from a home environment to the enterprise network using VPN.


<br>

## **1.4 Used Devices**

| **Device**         | **Image / Type**    | **Interfaces**     | **Access**       | **Purpose**                                                                                                                           |
| ------------------ | ------------------- | ------------------ | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| EDGE-R3            | Cisco IOSv          | 6x GigabitEthernet | Console          | Edge router providing WAN connectivity between the enterprise network and the ISP.                                                    |
| ISP                | Cisco IOSv          | 6x GigabitEthernet | Console          | ISP router providing upstream Internet connectivity.                                                                                  |
| Internet           | Cisco IOSv          | 6x GigabitEthernet | Console          | Simulated Internet cloud representing external networks and transit.                                                                  |
| FW-OPNS            | OPNsense Firewall   | 3x Ethernet        | Console / Web UI | Central firewall enforcing security policies, traffic filtering, and NAT; managed via Web UI from NET-ADMIN.                          |
| DIST-SW1, DIST-SW2 | Cisco IOSv-L2       | 4x GigabitEthernet | Console          | Distribution switches aggregating internal traffic. Interconnected for redundancy, with a single uplink to the firewall via DIST-SW1. |
| AGG-SW5            | Cisco IOSv-L2       | 4x GigabitEthernet | Console          | Aggregation switch connecting the distribution zone with the management and server infrastructure.                                    |
| INFRA-SW6          | Cisco IOSv-L2       | 4x GigabitEthernet | Console          | Infrastructure switch providing Layer 2 connectivity for management and server systems.                                               |
| ACC-SW3, ACC-SW4   | Cisco IOSv-L2       | 4x GigabitEthernet | Console          | Access switches providing Layer 2 connectivity for end devices in user VLANs.                                                         |
| NET-ADMIN          | QEMU Xubuntu VM     | 1x GigabitEthernet | VNC              | Administrative workstation used for network and firewall management.                                                                  |
| SERVER             | Windows Server 2022 | 1x Ethernet        | VNC              | Infrastructure server providing services such as DHCP, DNS, and HTTP.                                                                 |
| OFFICE             | VPCS                | 1x Ethernet        | Console          | Office client device connected to the access network.                                                                                 |
| FINANCE            | VPCS                | 1x Ethernet        | Console          | Finance client device connected to the access network.                                                                                |
| SALES              | VPCS                | 1x Ethernet        | Console          | Sales client device connected to the access network.                                                                                  |
| GUEST              | QEMU Xubuntu VM     | 1x GigabitEthernet | VNC              | Guest client system connected to the access network with restricted access.                                                           |
| SOHO-ADMIN         | QEMU Xubuntu VM     | 1x GigabitEthernet | VNC              | Remote administrator system simulating secure access from a home environment via VPN.                                                 |

<br>


## **1.5 Connection Overview**

| **Device** | **Interface** | **Connected to** | **Peer Interface** | **Purpose**                                                                               |
| ---------- | ------------- | ---------------- | ------------------ | ----------------------------------------------------------------------------------------- |
| Internet   | Gi0/0         | ISP              | Gi0/0              | WAN link simulating public Internet connectivity.                                         |
| Internet   | Gi0/5         | SOHO-ADMIN       | Gi0/0              | External host link used for remote access testing.                                        |
| ISP        | Gi0/5         | EDGE-R3          | Gi0/5              | WAN uplink between ISP and the enterprise edge router.                                    |
| EDGE-R3    | Gi0/0         | FW-OPNS          | e0                 | Upstream link from edge router to the firewall WAN interface.                             |
| FW-OPNS    | e1            | DIST-SW1         | Gi0/1              | Firewall internal link to distribution switch DIST-SW1.                                   |
| DIST-SW1   | Gi0/3         | ACC-SW3          | Gi0/3              | Downlink from distribution to access switch ACC-SW3.                                      |
| DIST-SW2   | Gi0/3         | ACC-SW4          | Gi0/3              | Downlink from distribution to access switch ACC-SW4.                                      |
| DIST-SW1   | Gi0/0         | DIST-SW2         | Gi0/0              | LACP EtherChannel providing redundant Layer 2 connectivity between distribution switches. |
| DIST-SW1   | Gi0/2         | DIST-SW2         | Gi0/2              | LACP EtherChannel providing redundant Layer 2 connectivity between distribution switches. |
| ACC-SW3    | Gi0/0         | ACC-SW4          | Gi0/0              | Inter-access link providing Layer 2 connectivity between access switches.                 |
| DIST-SW2   | Gi0/1         | AGG-SW5          | Gi0/1              | Uplink from distribution layer to aggregation switch for infrastructure access.           |
| AGG-SW5    | Gi0/0         | INFRA-SW6        | Gi0/0              | Infrastructure uplink providing connectivity into the MGMT and Server Zone. (LACP)        |
| AGG-SW5    | Gi0/3         | INFRA-SW6        | Gi0/3              | Additional infrastructure link for redundancy and capacity. (LACP)                        |
| INFRA-SW6  | Gi0/2         | NET-ADMIN        | Gi0/0              | Access link for the NET-ADMIN management workstation (VLAN-50).                           |
| INFRA-SW6  | Gi0/1         | SERVER           | Ethernet0          | Access link for the Windows Server providing DHCP, DNS, and HTTP (VLAN-60).               |
| ACC-SW3    | Gi0/1         | OFFICE           | e0                 | Access link for the Office VLAN client (VLAN-10).                                         |
| ACC-SW3    | Gi0/2         | FINANCE          | e0                 | Access link for the Finance VLAN client (VLAN-20).                                        |
| ACC-SW4    | Gi0/2         | SALES            | e0                 | Access link for the Sales VLAN client (VLAN-30).                                          |
| ACC-SW4    | Gi0/1         | GUEST            | Gi0/0              | Access link for the Guest VLAN client (VLAN-40).                                          |

<br>

## **1.6 Conclusion**

This chapter defines the network topology and describes the main parts of the design. The topology follows a firewall-centric, layered approach that separates external connectivity, internal switching, and management and server infrastructure.

The topology provides a clear foundation for further configuration. The next chapter focuses on IP addressing, subnetting, and VLAN planning, where the addressing structure and VLAN roles are defined.

<br>

---

<br>

**Next Chapter:**

