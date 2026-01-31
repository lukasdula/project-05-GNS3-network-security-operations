

<br>

# **2 - Addressing and VLAN Planning**


<br>

## **2.1 Introduction**


This chapter describes the IPv4 addressing design and VLAN structure used in the network. The internal addressing follows RFC1918 private IP ranges. The goal is to create a clear and easy-to-understand address layout where each network segment has its own subnet based on its role.

User VLANs use larger subnets (/24) to support more devices and future growth. Administrative, server, and management networks use smaller subnets (/28 and /26), because they include a limited number of devices with clearly defined roles.

VLANs are used to separate different types of traffic, while the firewall provides routing and default gateway services. This design prepares the network and its subnetting for the next chapters, prevents address conflicts, and reflects a realistic enterprise network layout.


<br>

## **2.2  Topology Diagram**

![](images/Pasted%20image%2020260124203627.png)

<br>


## **2.3 IP Addressing and Subnet Overview**

| **Device / Interface** | **Description**                                  | **IP Address**            | **Subnet Mask** | **Default Gateway** | **Assigned Network** |
| ---------------------- | ------------------------------------------------ | ------------------------- | --------------- | ------------------- | -------------------- |
| SOHO-ADMIN Gi0/0       | Remote admin host (VPN endpoint)                 | 18.192.0.2                | 255.255.255.252 | 18.192.0.1          | 18.192.0.0/30        |
| INTERNET Gi0/5         | Internet to SOHO-ADMIN point-to-point link       | 18.192.0.1                | 255.255.255.252 | N/A                 | 18.192.0.0/30        |
| INTERNET Gi0/0         | Internet to ISP point-to-point link              | 93.184.216.1              | 255.255.255.252 | N/A                 | 93.184.216.0/30      |
| ISP Gi0/0              | ISP to Internet point-to-point link              | 93.184.216.2              | 255.255.255.252 | N/A                 | 93.184.216.0/30      |
| ISP Gi0/5              | ISP to EDGE-R3 point-to-point link               | 31.13.64.1                | 255.255.255.252 | N/A                 | 31.13.64.0/30        |
| EDGE-R3 Gi0/5          | EDGE-R3 to ISP point-to-point link               | 31.13.64.2                | 255.255.255.252 | N/A                 | 31.13.64.0/30        |
| EDGE-R3 Loopback0      | Edge router loopback (router-id / mgmt identity) | 3.3.3.3                   | 255.255.255.255 | N/A                 | 3.3.3.3/32           |
| EDGE-R3 Gi0/0          | EDGE-R3 to FW-OPNs WAN transit                   | 52.84.0.1                 | 255.255.255.252 | N/A                 | 52.84.0.0/30         |
| FW-OPNs e0             | OPNsense WAN interface (to EDGE-R3)              | 52.84.0.2                 | 255.255.255.252 | 52.84.0.1           | 52.84.0.0/30         |
| OFFICE PC              | Office client (DHCP)                             | 10.70.10.100-10.70.10.199 | 255.255.255.0   | 10.70.10.1          | 10.70.10.0/24        |
| FINANCE PC             | Finance client (DHCP)                            | 10.70.20.100-10.70.20.199 | 255.255.255.0   | 10.70.20.1          | 10.70.20.0/24        |
| SALES PC               | Sales client (DHCP)                              | 10.70.30.100-10.70.30.199 | 255.255.255.0   | 10.70.30.1          | 10.70.30.0/24        |
| GUEST (Xubuntu)        | Guest client (DHCP)                              | 10.70.40.100-10.70.40.199 | 255.255.255.0   | 10.70.40.1          | 10.70.40.0/24        |
| NET-ADMIN (Xubuntu)    | Administrative workstation (static)              | 10.70.50.10               | 255.255.255.240 | 10.70.50.1          | 10.70.50.0/28        |
| SERVER (Windows)       | Infrastructure server (static)                   | 10.70.60.10               | 255.255.255.240 | 10.70.60.1          | 10.70.60.0/28        |
| DIST-SW1 VLAN 99 SVI   | Switch management (static)                       | 10.70.99.11               | 255.255.255.192 | 10.70.99.1          | 10.70.99.0/26        |
| DIST-SW2 VLAN 99 SVI   | Switch management (static)                       | 10.70.99.12               | 255.255.255.192 | 10.70.99.1          | 10.70.99.0/26        |
| ACC-SW3 VLAN 99 SVI    | Switch management (static)                       | 10.70.99.13               | 255.255.255.192 | 10.70.99.1          | 10.70.99.0/26        |
| ACC-SW4 VLAN 99 SVI    | Switch management (static)                       | 10.70.99.14               | 255.255.255.192 | 10.70.99.1          | 10.70.99.0/26        |
| AGG-SW5 VLAN 99 SVI    | Switch management (static)                       | 10.70.99.15               | 255.255.255.192 | 10.70.99.1          | 10.70.99.0/26        |
| INFRA-SW6 VLAN 99 SVI  | Switch management (static)                       | 10.70.99.16               | 255.255.255.192 | 10.70.99.1          | 10.70.99.0/26        |


<br>


## **2.4 VLAN Planning**

| **VLAN ID** | **VLAN Name** | **Description / Purpose**               | **Assigned Network** | **Default Gateway** | **Assigned Devices**                           | **Switch Assoc**                                         |
| ----------- | ------------- | --------------------------------------- | -------------------- | ------------------- | ---------------------------------------------- | -------------------------------------------------------- |
| 10          | office        | Office user workstations                | 10.70.10.0/24        | 10.70.10.1          | Office PCs                                     | ACC-SW3                                                  |
| 20          | finance       | Finance user workstations               | 10.70.20.0/24        | 10.70.20.1          | Finance PCs                                    | ACC-SW3                                                  |
| 30          | sales         | Sales user workstations                 | 10.70.30.0/24        | 10.70.30.1          | Sales PCs                                      | ACC-SW4                                                  |
| 40          | guest         | Guest user with restricted access       | 10.70.40.0/24        | 10.70.40.1          | Guest (Xubuntu)                                | ACC-SW4                                                  |
| 50          | net-admin     | Administrative access                   | 10.70.50.0/28        | 10.70.50.1          | NET-ADMIN (Xubuntu)                            | INFRA-SW6                                                |
| 60          | server        | Infrastructure services (DNS/DHCP/HTTP) | 10.70.60.0/28        | 10.70.60.1          | SERVER (Windows)                               | INFRA-SW6                                                |
| 99          | mgmt          | Network device management               | 10.70.99.0/26        | 10.70.99.1          | SW1, SW2, ACC-SW3, ACC-SW4, AGG-SW5, INFRA-SW6 | DIST-SW1, DIST-SW2, ACC-SW3, ACC-SW4, AGG-SW5, INFRA-SW6 |

> Notes: Default gateway IP addresses are provided by OPNsense VLAN interfaces. VLAN 99 is dedicated to switch management and is not used by end-user devices.


<br>

## **2.5 Conclusion**

This second main part in this project defines the IPv4 addressing structure and VLAN layout used in the network. Subnets are assigned based on network roles to keep the design clear, scalable, and easy to manage. User networks use larger address spaces, while administrative, server, and management segments use smaller and controlled subnets.

The addressing and VLAN design creates a stable foundation for the rest of the project. In the next chapter, the focus moves to basic device configuration, interface setup, and clear interface descriptions to prepare the network for Layer 2 switching and Layer 3 routing.


<br>

---


<br>



**Next part:**