

<br>

# 3 - Basic Device Configuration


<br>

## **3.1 Introduction**

This chapter establishes a clean and stable baseline configuration for the network topology before any Layer 2, routing, or security features are introduced. The purpose is to ensure that all core devices are reachable, easy to recognize, and ready for further configuration.

Static IPv4 addressing is applied to selected infrastructure hosts and WAN interfaces that require fixed and predictable connectivity. Hostnames are configured on all routers and switches, WAN and point-to-point links receive static IPv4 addresses, and loopback interfaces are used to provide stable logical identifiers.

Interface descriptions are added to document link purpose and connected devices, and basic connectivity verification confirms correct interface state and addressing. This baseline prepares the topology for VLAN configuration and Layer 2 design in the next chapters.


<br>

## **3.2 Topology Diagram**

![](images/Pasted%20image%2020260124155912.png)


<br>

## **3.3 Objectives**

1. Configure static IPv4 addressing on the NET-ADMIN workstation and the Windows server.
    
2. Configure consistent hostnames on all routers and switches.
    
3. Assign static IPv4 addresses to WAN and point-to-point interfaces on edge, ISP, and Internet-facing devices.
    
4. Configure static IPv4 addressing on OPNsense WAN interface.
    
5. Apply interface descriptions on routers and switches to clearly document link purpose and connected devices.
    

<br>

## **3.4 Static IPv4 Addressing - NET-ADMIN and Windows Server**

This first section defines static IPv4 configuration for the NET-ADMIN workstation and the Windows server. Both devices require fixed and predictable IP addresses to ensure reliable access for management and core services before routing, VLAN configuration, or DHCP are introduced.

<br>

### **NET-ADMIN (Xubuntu) - Static IPv4 Configuration**

The NET-ADMIN workstation uses a static IPv4 address to provide stable administrative access during all configuration and verification stages.

**Open configuration file:**

```plaintext
sudo nano /etc/netplan/01-network-manager-all.yaml
```
![](images/Pasted%20image%2020260124014423.png)


**Configuration:**

```plaintext
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens3:
      addresses: [10.70.50.10/28]
      nameservers:
        addresses: [10.70.60.10]
      routes:
        - to: 0.0.0.0/0
          via: 10.70.50.1
```
![](images/Pasted%20image%2020260126042815.png)


**Apply configuration:**

```plaintext
sudo netplan apply
```



**Verification:**

```plaintext
ip a
```
![](images/Pasted%20image%2020260124014738.png)


**Results**

- NET-ADMIN is configured with the static IPv4 address 10.70.50.10/28.
    

<br>

### **Windows Server - Static IPv4 Configuration**

Static IPv4 addressing is applied to the Windows server because it provides infrastructure services and must remain reachable at a consistent address.

**IPv4 settings:**

- IP address: 10.70.60.10
    
- Subnet mask: 255.255.255.240
    
- Default gateway: 10.70.60.1
    

**Configuration (PowerShell):**

```plaintext
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.70.60.10 -PrefixLength 28 -DefaultGateway 10.70.60.1
```
![](images/Pasted%20image%2020260124015347.png)


**Verification:**

```plaintext
ipconfig
ping 10.70.60.10
```
![](images/Pasted%20image%2020260124015424.png)


**Results**

- Windows Server is configured with the static IPv4 address 10.70.60.10/28.
    
- Local verification confirms correct IPv4 settings and basic ICMP response.


<br>

## **3.5 Device Hostname Configuration**

This next section describes hostname configuration for Cisco network devices only. Hostnames are used to clearly identify routers and switches during configuration, verification, and troubleshooting.

**The following example demonstrates hostname configuration on the edge router. The same configuration method is applied to all other Cisco routers and switches in the topology.**


> Notes:
> 
> - VPCS end devices use predefined device names that are automatically derived from their labels and do not require manual hostname configuration.
>     
> - Xubuntu systems use system-level hostnames configured outside the scope of this documentation.
>     
> - The OPNsense firewall uses a system hostname configured through the web interface and is documented separately in the chapter dedicated to OPNsense configuration.
>     


<br>

### **Hostname Configuration Example - EDGE-R3**

Configure the hostname on the edge router using Cisco IOS CLI:

```plaintext
enable
configure terminal
hostname EDGE-R3
end
write memory
```
![](images/Pasted%20image%2020260123033530.png)

<br>

### **Hostname Overview**

| **Device Role**           | **Device Name** | **Hostname**  |
| --------------------- | ----------- | --------- |
| Edge Router           | EDGE-R3     | EDGE-R3   |
| ISP Router            | ISP         | ISP       |
| Internet Router       | Internet    | Internet  |
| Distribution Switch   | DIST-SW1    | DIST-SW1  |
| Distribution Switch   | DIST-SW2    | DIST-SW2  |
| Aggregation Switch    | AGG-SW5     | AGG-SW5   |
| Infrastructure Switch | INFRA-SW6   | INFRA-SW6 |
| Access Switch         | ACC-SW3     | ACC-SW3   |
| Access Switch         | ACC-SW4     | ACC-SW4   |


<br>

## **3.6 WAN and Transit Interface Configuration**

This section documents the static IPv4 configuration of WAN and transit interfaces between external network segments. These interfaces form point-to-point links between the Internet cloud, ISP router, and the enterprise edge router. The goal of this section is to establish basic Layer 3 connectivity before routing, NAT, or firewall policies are introduced.

<br>

> Notes:  
> In real-world environments, IP addressing on ISP and Internet-facing infrastructure is typically managed by the service provider. In this project, these addresses are configured manually for simulation and learning purposes only.


<br>

### **Internet Router – Interface Configuration**

**Links:**

- Gi0/5 -> SOHO-ADMIN Gi0/0 (18.192.0.0/30)
    
- Gi0/0 -> ISP Gi0/0 (93.184.216.0/30)
    

**Configuration:**

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
ip address 18.192.0.1 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/0
ip address 93.184.216.1 255.255.255.252
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124021026.png)

<br>

### **ISP Router – Interface and Loopback Configuration**

**Links:**

- Gi0/0 -> Internet Router Gi0/0 (93.184.216.0/30)
    
- Gi0/5 -> EDGE-R3 Gi0/5 (31.13.64.0/30)
    

**Configuration:**

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
ip address 93.184.216.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/5
ip address 31.13.64.1 255.255.255.252
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124021051.png)

<br>

### **Edge Router – WAN Interface and Loopback Configuration (EDGE-R3)**

**Links:**

- Gi0/5 -> ISP Gi0/5 (31.13.64.0/30)
    
- Gi0/0 -> FW-OPNs WAN interface (52.84.0.0/30)
    

**Configuration:**

```plaintext
enable
configure terminal
interface GigabitEthernet0/5
ip address 31.13.64.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/0
ip address 52.84.0.1 255.255.255.252
no shutdown
exit
interface Loopback0
ip address 3.3.3.3 255.255.255.255
exit
end
write memory
```
![](images/Pasted%20image%2020260123033558.png)

<br>

### **Verification**

**Example verification on EDGE-R3:**

```plaintext
show ip interface brief
```
![](images/Pasted%20image%2020260123033617.png)


**Results:**

The output confirms that WAN and transit interfaces on EDGE-R3 are correctly addressed and operational (up/up). The loopback interface is active, and basic Layer 3 connectivity to directly connected devices is established.

<br>


## **3.7 OPNsense Console Interface Assignment**

This section documents the initial console-based configuration of the OPNsense firewall. In this stage, physical interface is assigned to logical roles and the WAN interface connected to the EDGE router is prepared for IP addressing. The goal is to establish basic WAN connectivity before any routing, NAT, firewall rules, or GUI-based configuration is introduced.

<br>

### **OPNsense Boot Screen :))**

![](images/Pasted%20image%2020260123030652.png)


### **OPNsense Console Menu Overview**


![](images/Pasted%20image%2020260123031132.png)


<br>

### **WAN Interface IP Configuration (Console)**

This step configures a static IPv4 address on the OPNsense WAN interface connected to the EDGE router. The WAN interface represents the untrusted upstream network and must be reachable before any further firewall, NAT, or routing configuration is applied.


- From the OPNsense console menu, select **2 – Set interface IP address**.
    
- Choose **WAN** as the interface to configure.
    
- Disable DHCP for the WAN interface.
    
- Assign a static IPv4 address according to the WAN transit network.
    
- Configure the default gateway pointing to the EDGE router.
    
- Leave IPv6 and DHCP services disabled on the WAN interface.


<br>

**Configuration parameters:**


```
Interface: WAN (em0)
IPv4 address: 52.84.0.2
Subnet prefix length: 30
Default gateway: 52.84.0.1
```
![](images/Pasted%20image%2020260123033135.png)


<br>

### **Verification**

After configuring the WAN interface, basic connectivity to the EDGE router is verified using an ICMP test from the OPNsense console.


- From the console menu, select **7 – Ping host**.
    
- Enter the IPv4 address of the EDGE router WAN-facing interface.
    

```
Ping target: 52.84.0.1
```
![](images/Pasted%20image%2020260123033808.png)

**Results:**

A successful ping confirms that the WAN interface on OPNsense is correctly configured and that Layer 3 connectivity between OPNsense and the EDGE router is established.

>**Notes:** At this stage, ICMP traffic from the edge router to the OPNsense WAN interface is blocked by the default OPNsense security policy. This behavior is expected and confirms that the firewall is active and enforcing access control on the WAN interface. Firewall rules and security policies for OPNsense will be configured and documented later in the dedicated OPNsense configuration chapter.



<br>

## **3.8 Interface Description Configuration**

This last section set configuration of interface descriptions on selected Cisco network devices. Interface descriptions are used to clearly identify link purpose, connected devices, and interface roles, which improves routing and switching devices to make link roles easy to recognize during configuration and troubleshooting. 

A consistent description format is applied across devices. Only internal network devices are included. External provider equipment is excluded, as interface descriptions on provider-managed devices are outside the administrative scope of this project.

<br>

### **Port Description Mapping**

The table below summarizes interface descriptions used in this section.

| **Device** | **Interface** | **Description**                            |
| ---------- | ------------- | ------------------------------------------ |
| EDGE-R3    | Gi0/0         | To FW-OPNs e0 (WAN transit)                |
| EDGE-R3    | Gi0/5         | To ISP Gi0/5 (WAN)                         |
| DIST-SW1   | Gi0/1         | To FW-OPNs e1 (uplink)                     |
| DIST-SW1   | Gi0/0         | To DIST-SW2 Gi0/0 (L2 lacp)                |
| DIST-SW1   | Gi0/3         | To ACC-SW3 Gi0/3 (access uplink)           |
| DIST-SW1   | Gi0/2         | To DIST-SW2 Gi0/2 (L2 lacp)                |
| DIST-SW2   | Gi0/2         | To DIST-SW1 Gi0/2 (L2 lacp)                |
| DIST-SW2   | Gi0/0         | To DIST-SW1 Gi0/0 (L2 lacp)                |
| DIST-SW2   | Gi0/3         | To ACC-SW4 Gi0/3 (access uplink)           |
| DIST-SW2   | Gi0/1         | To AGG-SW5 Gi0/1 (aggregation uplink)      |
| ACC-SW3    | Gi0/3         | To DIST-SW1 Gi0/3 (distribution uplink)    |
| ACC-SW3    | Gi0/0         | To ACC-SW4 Gi0/0 (L2 redundancy)           |
| ACC-SW3    | Gi0/2         | To FINANCE-PC e0 (access port)             |
| ACC-SW3    | Gi0/1         | To OFFICE-PC e0 (access port)              |
| ACC-SW4    | Gi0/3         | To DIST-SW2 Gi0/3 (distribution uplink)    |
| ACC-SW4    | Gi0/0         | To ACC-SW3 Gi0/0 (L2 redundancy)           |
| ACC-SW4    | Gi0/2         | To SALES-PC e0 (access port)               |
| ACC-SW4    | Gi0/1         | To Guest-PC Gi0/0 (access port)            |
| AGG-SW5    | Gi0/1         | To DIST-SW2 Gi0/1 (distribution uplink)    |
| AGG-SW5    | Gi0/0         | To INFRA-SW6 Gi0/0 (infrastructure uplink) |
| AGG-SW5    | Gi0/3         | To INFRA-SW6 Gi0/3 (infrastructure uplink) |
| INFRA-SW6  | Gi0/0         | To AGG-SW5 Gi0/0 (aggregation uplink)      |
| INFRA-SW6  | Gi0/3         | To AGG-SW5 Gi0/3 (aggregation uplink)      |
| INFRA-SW6  | Gi0/2         | To NET-ADMIN Gi0/0 (access port)           |
| INFRA-SW6  | Gi0/1         | To SERVER e0 (access port)                 |

<br>

### **Configuration Example - EDGE-R3**

Links:

- Gi0/0 -> FW-OPNs e0 (WAN transit)
    
- Gi0/5 -> ISP Gi0/5 (WAN)
    

Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/0
description To FW-OPNs e0 
exit
interface GigabitEthernet0/5
description To ISP Gi0/5 
exit
end
write memory
```
![](images/Pasted%20image%2020260124021129.png)

<br>

### **Configuration Example - DIST-SW1**

Links:

- Gi0/1 -> FW-OPNs e1 (uplink)
    
- Gi0/0 -> DIST-SW2 Gi0/0 (L2 lacp)
    
* Gi0/3 -> ACC-SW3 Gi0/3 (To ACC-SW3 Gi0/3 (access uplink)
    
* Gi0/2 -> DIST-SW2  (L2 lacp)


Configuration:

```plaintext
enable
configure terminal
interface GigabitEthernet0/1
description To FW-OPNs e1 
exit
interface GigabitEthernet0/0
description To DIST-SW2 Gi0/0 
exit
interface GigabitEthernet0/3
description To ACC-SW3 Gi0/3
exit
interface GigabitEthernet0/2
description To DIST-SW2 Gi0/2 
end
write memory
```
![](images/Pasted%20image%2020260124160759.png)

<br>

### **Verification**


#### **EDGE-R3**

Verify configured interface descriptions using the following command:

```plaintext
show interfaces description
```
![](images/Pasted%20image%2020260124021223.png)

Results:  
Configured interfaces display clear and consistent descriptions that match the physical and logical topology.


<br>

## **3.9 Conclusion**


This Chapter 3 established the basic network setup, including static IPv4 addressing, consistent device hostnames, and WAN and point-to-point interface configuration on routing and Internet-facing devices. The OPNsense firewall was added at the network edge as the main security point.

With interface descriptions applied across routers and switches, the topology is now clearly documented and ready. The network is prepared for the next chapter focused on Layer 2 switching configuration.

<br>


---

<br>

**Next Chapter:**

