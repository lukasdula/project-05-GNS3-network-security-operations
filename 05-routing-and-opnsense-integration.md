
<br>

# **5 - Routing and OPNsense integration**

<br>

## **5.1 Introduction**

This chapter explains how OPNsense is integrated into the network as the main firewall and Layer 3 gateway. The goal is to provide clear routing between internal VLANs, controlled access to management networks, and a secure connection to the WAN. OPNsense is used because it combines routing, firewalling, and future VPN/NAT features in one central device.

First, internal VLANs are created and assigned to OPNsense interfaces. Each VLAN gets its own IPv4 gateway address, which allows traffic to move between internal networks in a controlled way. Firewall rules are added step by step, starting with management access (NET-ADMIN) and basic diagnostic access on the WAN interface.

VLAN 99 is used for switch management. Management IP addresses and routing are configured on switches so that monitoring and admin traffic can return correctly to OPNsense. Finally, simple static routing is added on WAN devices (EDGE, ISP, Internet-Cloud). This routing prepares the network for later NAT/PAT and VPN scenarios, while keeping the ISP isolated from internal networks and following real-world security principles.

<br>

## **5.2 Topology Diagram**

![](images/Pasted%20image%2020260124220915.png)

<br>

## **5.3 Objectives**



 1. OPNsense – Initial Access and Management**
    
    * Establish console access to OPNsense from the NET-ADMIN host
    
    * Verify basic management connectivity and GUI access
    

1.  OPNsense – VLAN Interfaces Assignment

    - Create VLAN interfaces tagged uplink to switch (802.1Q)
    
    * Assign VLAN interfaces and configure IPv4 gateway addresses
    
    
2.  VLAN 99 – Management Routing**
    
    
3.  Static Routing – WAN / ISP / Internet
    
4.  Verify – Routing Reachability
    

<br>


## **5.4 OPNsense Console Bootstrap Access**


This first section sets up basic management access to the OPNsense firewall before any VLAN interfaces are created. At this point, OPNsense has only the WAN interface configured and does not yet have Layer 3 access to the internal management network. A temporary IPv4 address is assigned to an internal physical interface to allow access to the Web GUI from the NET-ADMIN workstation.

This configuration is **temporary** and is used only to get initial access to the firewall. The final management gateway is later provided by a dedicated VLAN 50 interface, and the temporary IP address is then removed.

<br>


### **5.4.1 OPNsense Console – Temporary Management IP Configuration**

The following steps are performed directly in the OPNsense console menu to establish temporary Layer 3 connectivity between the firewall and the NET-ADMIN workstation.

<br>

**IPv4 parameters:**

- Interface: em1 
    
- IPv4 address: 10.70.50.1
    
- Subnet prefix length: /28
    

```plaintext
Interface: em1
IPv4 address: 10.70.50.1
Subnet prefix length: 28
DHCP: disabled
IPv6: disabled
```
![](images/Pasted%20image%2020260124145113.png)
![](images/Pasted%20image%2020260124145255.png)

**Results**

After this temporary configuration is applied, basic Layer 3 connectivity exists between the NET-ADMIN workstation and the OPNsense firewall. The Web GUI becomes reachable from the NET-ADMIN system using the firewall IPv4 address. This access is used only to continue configuration in the graphical interface.


<br>

### **5.4.2 Removal of IP Address from Physical Interface (em1)**

 
During VLAN routing configuration, the IPv4 address was removed from the physical interface `em1` in the OPNsense GUI.  

*The gateway addresses were moved to the individual VLAN interfaces instead.*

- The physical interface `em1` is used only as a trunk (802.1Q).
    
- Layer 3 addressing must exist on VLAN interfaces, not on the physical port.
    
- This prevents IP conflicts and ensures correct inter-VLAN routing.
    

<br>

#### OPNsense console – interface overview after IP removal from `em1`

![](images/Pasted%20image%2020260124182049.png)


<br>

## **5.5 VLAN Interfaces Devices**

  
This settings assigns VLAN interfaces to OPNsense and enables Layer 3 routing between internal networks.


- All VLAN interfaces are created on the physical trunk interface `em1`.
    
- Each VLAN is assigned as a dedicated interface in OPNsense.
    
- This design replaces classic router subinterfaces (Router-on-a-Stick concept).
    

![](images/Pasted%20image%2020260124192449.png)

OPNsense recognizes all internal VLANs 10-99 and can route traffic between them.

<br>

### **5.5.1 VLAN Interface Assignment**

This section assigns previously created VLAN devices to logical interfaces in OPNsense. This step makes each VLAN visible in the interface list and allows IP addressing and firewall policies to be applied.

### Interface Assignment


  * Each VLAN interface is assigned its own IPv4 gateway address.
    
   * This allows routing between VLANs through OPNsense.

![](images/Pasted%20image%2020260124193735.png)

**Result**

- All internal VLANs are now available as individual interfaces in OPNsense.
    
- Each VLAN can be configured with its own IPv4 address
    
- The firewall is prepared for Layer 3 routing between VLANs.




<br>


### **5.5.2 VLAN Gateway Configuration**

Each VLAN interface is configured with a static IPv4 address that acts as the default gateway.


- Static IPv4 configuration is used for all VLAN interfaces.
    
- The first usable address in each subnet is selected as the gateway.


#### **The example shows the SERVER VLAN configuration.**

![](images/Pasted%20image%2020260124182637.png)

**Result**  
All VLANs have a functional Layer 3 gateway on the OPNsense firewall.

<br>

### **5.5.3 Management Access (NET-ADMIN VLAN)**


This part restricts administrative access to OPNsense to the NET-ADMIN VLAN.


- Firewall rules are applied only on the NET-ADMIN interface.
    
- ICMP is allowed for connectivity testing.
    
- HTTPS is allowed for secure GUI access to the firewall.
    
- Access is limited to the firewall itself ("This Firewall").
    

![](images/Pasted%20image%2020260124182823.png)

**Result**  
Secure and controlled management access to OPNsense is established from the NET-ADMIN network.

<br>

## **5.6 WAN Firewall Rule (EDGE-R3 to OPNsense)**


This section configures a minimal firewall rule on the OPNsense WAN interface. The goal is to allow diagnostic ICMP connectivity from the upstream EDGE router to the firewall itself, without exposing any internal VLANs or services.

<br>

#### **WAN ICMP Rule Configuration**

This rule permits ICMP traffic arriving on the WAN interface and destined only for the firewall.

- Interface: WAN
    
- Direction: Inbound
    
- Protocol: ICMP (IPv4)
    
- Source: WAN net
    
- Destination: This Firewall
    
- Action: Pass
    

![](images/Pasted%20image%2020260124190110.png)

<br>

#### **Applied WAN Rules Overview**

After applying the configuration, the WAN ruleset contains a single explicit allow rule for ICMP, followed by the default implicit deny policy.

![](images/Pasted%20image%2020260124190010.png)


<br>


#### **Verify ping from EDGE-R3 to OPNsense Firewall**


 ```
 ping 52.84.0.2
 ```
![](images/Pasted%20image%2020260124185641.png)

<br>


### Result

The EDGE router can ping the OPNsense firewall on its WAN address for monitoring and basic testing. Access to internal VLAN networks is not allowed, and WAN security stays in place.



<br>


## **5.7 VLAN 99 Mgmt - Switch Routing and Default Route**



This next section configures VLAN 99 management on all switches. Each switch gets a static SVI address in VLAN 99, enables Layer 3 routing, and sets a default route to the OPNsense gateway. This allows management traffic (ping/monitoring) to return correctly to NET-ADMIN and servers.

<br>


## **Topology overview**

| Device    | Purpose                 | VLAN ID | Subnet (CIDR) | IP Address  | Default GW |
| --------- | ----------------------- | ------: | ------------- | ----------- | ---------- |
| DIST-SW1  | Switch management (SVI) |      99 | 10.70.99.0/26 | 10.70.99.11 | 10.70.99.1 |
| DIST-SW2  | Switch management (SVI) |      99 | 10.70.99.0/26 | 10.70.99.12 | 10.70.99.1 |
| ACC-SW3   | Switch management (SVI) |      99 | 10.70.99.0/26 | 10.70.99.13 | 10.70.99.1 |
| ACC-SW4   | Switch management (SVI) |      99 | 10.70.99.0/26 | 10.70.99.14 | 10.70.99.1 |
| AGG-SW5   | Switch management (SVI) |      99 | 10.70.99.0/26 | 10.70.99.15 | 10.70.99.1 |
| INFRA-SW6 | Switch management (SVI) |      99 | 10.70.99.0/26 | 10.70.99.16 | 10.70.99.1 |

<br>


#### **Configuration all Mgmt Switches** 

#### **DIST-SW1**

```plaintext
Enable
configure terminal
ip routing
interface vlan 99
description MGMT-VLAN99
ip address 10.70.99.11 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 10.70.99.1
end
write memory
```
![](images/Pasted%20image%2020260124221656.png)

<br>

#### **DIST-SW2**

```plaintext
Enable
configure terminal
ip routing
interface vlan 99
description MGMT-VLAN99
ip address 10.70.99.12 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 10.70.99.1
end
write memory
```
![](images/Pasted%20image%2020260124221728.png)

<br>

#### **ACC-SW3**

```plaintext
Enable
configure terminal
ip routing
interface vlan 99
description MGMT-VLAN99
ip address 10.70.99.13 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 10.70.99.1
end
write memory
```
![](images/Pasted%20image%2020260124221758.png)

<br>

#### **ACC-SW4**

```plaintext
Enable
configure terminal
ip routing
interface vlan 99
description MGMT-VLAN99
ip address 10.70.99.14 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 10.70.99.1
end
write memory
```
![](images/Pasted%20image%2020260124221821.png)

<br>

#### **AGG-SW5**

```plaintext
Enable
configure terminal
ip routing
interface vlan 99
description MGMT-VLAN99
ip address 10.70.99.15 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 10.70.99.1
end
write memory
```
![](images/Pasted%20image%2020260124221840.png)

<br>

#### **INFRA-SW6**

```plaintext
Enable
configure terminal
ip routing
interface vlan 99
description MGMT-VLAN99
ip address 10.70.99.16 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 10.70.99.1
end
write memory
```
![](images/Pasted%20image%2020260124221902.png)

<br>

### **Diagnostics**

#### **Example ACC-SW4**

* **ping to DIST-SW1 (10.70.99.11)**

```plaintext
show ip interface brief
show ip route
ping 10.70.99.11
```
![](images/Pasted%20image%2020260124222120.png)

Routing and addressing on the switch are working correctly. Interface status, routing table, and reachability to internal networks are verified.
pin
>Notes: Ping to the VLAN 99 gateway is blocked because OPNsense firewall rules will be configured in a later chapter.


**Result**

All switches have a reachable management SVI in VLAN 99 and can return traffic via the OPNsense gateway, enabling stable monitoring and remote management from NET-ADMIN.


<br>


## **5.8 Static Routing - WAN and Internet-Cloud**

This part defines simple and secure static routing between the enterprise edge router, the ISP router, and the Internet transit device. The goal is to allow outbound connectivity from the internal network through OPNsense while keeping the ISP isolated from internal enterprise networks.


**The routing design follows real-world security principles:**

- Internal networks are hidden behind the edge firewall (OPNsense).
    
- The ISP does not have routes to internal VLANs.
    
- The Internet device acts only as a transit network.
    

<br>

### **Routing Logic Overview**

- **EDGE-R3** sends all unknown traffic toward the ISP.
    
- **ISP** forwards all unknown traffic toward the Internet transit.
    
- **Internet** forwards traffic toward the ISP for external reachability (SOHO / VPN in later chapters).
    
- **OPNsense** does not require static routes for this stage because it is directly connected to EDGE-R3 on the WAN link.

<br>

#### **EDGE-R3 (Edge Router)**

All traffic leaving the internal network is sent to the ISP router. The edge router does not send internal routes upstream.

```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 31.13.64.1
end
write memory
```
![](images/Pasted%20image%2020260124222346.png)

<br>

#### **ISP Router**

The ISP forwards all unknown traffic toward the Internet-Cloud. No static routes to internal networks are configured, which keeps isolation and reflects real ISP behavior.

```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 93.184.216.1
end
write memory
```
![](images/Pasted%20image%2020260124222322.png)

<br>

#### **Internet Transit Device**

The Internet device acts as a simple transit router and forwards traffic toward the ISP.

```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 93.184.216.2
end
write memory
```
![](images/Pasted%20image%2020260124222256.png)


### ISP Router - Return Route to OPNsense WAN

This step adds a minimal return route on the ISP router for the OPNsense WAN transit network. This is required for later SOHO / VPN testing, so external hosts can reach the firewall WAN interface (52.84.0.2) through the ISP path.

```
enable
configure terminal
ip route 52.84.0.0 255.255.255.252 31.13.64.2
end
write memory
```
![](images/Pasted%20image%2020260128063658.png)

*External reachability toward the OPNsense WAN network is restored. SOHO-ADMIN can reach the EDGE-R3 WAN-side interface, and the ISP has a specific return path for the 52.84.0.0/30 transit network (without learning internal VLAN routes).*

<br>

**Result**

Outbound traffic from the enterprise network can reach external networks through EDGE-R3, ISP, and the Internet transit path. The routing design is secure, predictable, and ready for NAT/PAT and VPN configuration in later chapters.

<br>

## **5.9 Verify - Internet/ISP/SOHO Reachability**

This verification confirms that the Internet-Cloud and ISP can forward traffic to the SOHO-ADMIN site. It also confirms that the WAN path is ready for later VPN testing.


#### ISP -> SOHO-ADMIN connectivity (for later VPN)

1. On ISP Gi0/0 ping to INTERNET Gi/05 
    
```
ping 18.192.0.1
```
![](images/Pasted%20image%2020260124222457.png)


<br>

#### SOHO-ADMIN -> ISP connectivity (for later VPN)



2. On SOHO-ADMIN Gi0/0 ping to INTERNET Gi0/0 -> ISP (network)
    

```
ping 93.184.216.1
```  
![](images/Pasted%20image%2020260128063748.png)


**Results**

The successful ping tests confirm that traffic can traverse across different networks and devices in the Internet–ISP–SOHO path.  
This verifies correct static routing and basic reachability across WAN segments.











<br><br>


## **5.10 Conclusion**

This fifth chapter integrated OPNsense as the central firewall and routing device. Internal VLANs were assigned gateway addresses, firewall rules were applied for secure management access, and VLAN 99 routing was configured for switch administration.

Static routing on WAN devices established a clear traffic path between the internal network, ISP, and Internet transit. The network is now ready for the next chapter, where DHCP, HTTP, and DNS services will be configured.

<br>


---


<br>

**Next Chapter:** [Core Server Services](06-core-server-services.md)
