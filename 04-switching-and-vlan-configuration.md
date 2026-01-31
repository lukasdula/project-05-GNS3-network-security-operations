
<br>

# **4 - Switching and VLAN Configuration**


<br>

## **4.1 Introduction**

This chapter defines the Layer 2 switching design used across the network. It describes how VLANs are created and carried between access, distribution, aggregation, and infrastructure switches to allow logical separation of users, management traffic, and server services.

Trunk links are configured to transport all required VLANs through the switching topology toward the firewall, which provides the default gateway for each VLAN. Rapid Spanning Tree Protocol (RSTP) is used to prevent loops and allow fast recovery. Access ports are protected with PortFast and BPDU Guard to avoid topology issues caused by end devices or incorrect connections.

Link Aggregation Control Protocol (LACP) is first applied between the distribution switches to provide redundancy at the distribution layer. It is then applied between the aggregation and infrastructure switches, which carry critical management and server traffic. LACP combines multiple physical links into a single logical connection, ensuring redundancy and preparing the Layer 2 environment for the subsequent Layer 3 configuration.


<br>


## **4.2 Topology Diagram**

![](images/Pasted%20image%2020260124161300.png)


<br>

## **4.2 VLAN Overview**

| **VLAN ID** | **VLAN Name** | **Purpose / Description** | **Assigned Devices** | **Switch Association**                                   |
| ----------: | ------------- | ------------------------- | -------------------- | -------------------------------------------------------- |
|          10 | fffice        | Office user workstations  | OFFICE PC            | ACC-SW3                                                  |
|          20 | finance       | Finance department users  | FINANCE PC           | ACC-SW3                                                  |
|          30 | sales         | Sales department users    | SALES PC             | ACC-SW4                                                  |
|          40 | guest         | Guest network access      | GUEST (Xubuntu)      | ACC-SW4                                                  |
|          50 | net-admin     | Network administration    | NET-ADMIN (Xubuntu)  | INFRA-SW6                                                |
|          60 | server        | Infrastructure services   | SERVER (Windows)     | INFRA-SW6                                                |
|          99 | mgmt          | Switch management (SVI)   | All switches (SVI)   | DIST-SW1, DIST-SW2, AGG-SW5, ACC-SW3, ACC-SW4, INFRA-SW6 |


<br>

## **4.3 Objectives**

1. Create all required VLANs on every switch according to the defined network design.
    
2. Assign access ports to the correct VLANs based on connected end devices and apply PortFast and BPDU Guard on all access ports to protect the Layer 2 topology.
    
3. Configure trunk ports between switches to carry all required VLANs across the switching infrastructure.
    
4. Configure LACP between the distribution switches for redundancy, then between the aggregation and infrastructure switches for higher bandwidth and redundancy of management and server traffic.
    
5. Configure Rapid Spanning Tree Protocol (RSTP) to control the Layer 2 topology and define root bridge roles.
    
6. Verify VLAN, access ports, trunk, LACP, and spanning tree configuration using appropriate diagnostic commands.



<br>



## **4.4 VLAN Creation**

VLANs are created on every switch for consistent operation and simpler verification across the switching layer. This approach supports clean troubleshooting and future changes (for example, moving an access port to a different switch) without requiring additional VLAN database updates. This is a management and consistency decision.

#### **DIST-SW1 - vlan creation**

```prikaz
enable
conf t
vlan 10
name office
vlan 20
name finance
vlan 30
name sales
vlan 40
name guest
vlan 50
name net-admin
vlan 60
name server
vlan 99
name mgmt
end
write memory
```
![](images/Pasted%20image%2020260124123346.png)

<br>

#### **DIST-SW2 - vlan creation**

```prikaz
enable
conf t
vlan 10
name office
vlan 20
name finance
vlan 30
name sales
vlan 40
name guest
vlan 50
name net-admin
vlan 60
name server
vlan 99
name mgmt
end
write memory
```
![](images/Pasted%20image%2020260124123440.png)

<br>

#### **ACC-SW3 - vlan creation**

```prikaz
en
conf t
vlan 10
name office
vlan 20
name finance
vlan 30
name sales
vlan 40
name guest
vlan 50
name net-admin
vlan 60
name server
vlan 99
name mgmt
end
write memory
```
![](images/Pasted%20image%2020260124124126.png)

<br>

#### **ACC-SW4 - vlan creation**

```prikaz
en
conf t
vlan 10
name office
vlan 20
name finance
vlan 30
name sales
vlan 40
name guest
vlan 50
name net-admin
vlan 60
name server
vlan 99
name mgmt
end
write memory
```
![](images/Pasted%20image%2020260124123544.png)

<br>

#### **AGG-SW5 - vlan creation**

```prikaz
en
conf t
vlan 10
name office
vlan 20
name finance
vlan 30
name sales
vlan 40
name guest
vlan 50
name net-admin
vlan 60
name server
vlan 99
name mgmt
end
write memory
```
![](images/Pasted%20image%2020260124123706.png)

<br>

#### **INFRA-SW6 - vlan creation**

```prikaz
en
conf t
vlan 10
name office
vlan 20
name finance
vlan 30
name sales
vlan 40
name guest
vlan 50
name net-admin
vlan 60
name server
vlan 99
name mgmt
end
write memory
```
![](images/Pasted%20image%2020260124123748.png)

<br>

### **Result**

All required VLANs are successfully created on every switch. The VLAN structure is consistent across the entire switching layer, providing a clean and predictable foundation for access port assignment, trunk configuration, and further Layer 2 features in the following sections.


<br>


## **4.5 Access Ports Configuration**

Access ports are configured only on switch interfaces connected to end devices. These ports carry traffic for a single VLAN and do not participate in trunking. PortFast and BPDU Guard are applied exclusively on access ports to protect the Layer 2 topology and allow fast port activation for end devices.

PortFast allows access ports to transition immediately to the forwarding state, which reduces connection delay for user devices. BPDU Guard disables a port if a BPDU is received, protecting the network from accidental loops or incorrect device connections.

<br>

#### **ACC-SW3 - Access Ports**

```prikaz
enable
configure terminal
interface GigabitEthernet0/1
switchport mode access
switchport access vlan 10
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
interface GigabitEthernet0/2
switchport mode access
switchport access vlan 20
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124124217.png)

<br>

#### **ACC-SW4 - Access Ports**

```prikaz
enable
configure terminal
interface GigabitEthernet0/1
switchport mode access
switchport access vlan 40
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
interface GigabitEthernet0/2
switchport mode access
switchport access vlan 30
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124123903.png)

<br>

#### **INFRA-SW6 - Access Ports**

```prikaz
enable
configure terminal
interface GigabitEthernet0/1
switchport mode access
switchport access vlan 60
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
interface GigabitEthernet0/2
switchport mode access
switchport access vlan 50
spanning-tree portfast
spanning-tree bpduguard enable
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124123935.png)
<br>

### **Result**

Access ports are correctly assigned to their respective VLANs and protected with PortFast and BPDU Guard. End devices gain immediate connectivity while the switching topology remains protected from loops and bad configuration.


<br>


## **4.6 Trunk Ports Configuration**

Trunk ports are configured on all inter-switch links to carry multiple VLANs across the switching layer. The switching devices in this project use an IOSv-L2 image, which requires explicit configuration of 802.1Q encapsulation on trunk ports. For this reason, trunk encapsulation is defined manually on all trunk interfaces. DTP negotiation is also disabled on all trunk ports using the `switchport nonegotiate` command to keep trunk operation fully controlled and explicit.


>Notes: `switchport nonegotiate` is not applied on trunk links that are part of an LACP EtherChannel. On these links, trunk parameters are defined only on the Port-Channel interface to avoid configuration mismatch. Standard trunk links use `nonegotiate`, while LACP-based trunks rely on Port-Channel configuration.




<br>

#### **DIST-SW1 - Trunk Ports**

Trunk ports on DIST-SW1:

- Gi0/1 (to FW-OPNsense)
    
- Gi0/0 (to DIST-SW2)
    
- Gi0/3 (to ACC-SW3)
    
* Gi0/2 (to DIST-SW2)


```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/1, GigabitEthernet0/2, GigabitEthernet0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124161925.png)

<br>

#### **DIST-SW2 - Trunk Ports**

Trunk ports on DIST-SW2:

- Gi0/2 (to DIST-SW1)
    
- Gi0/0 (to DIST-SW1)
    
- Gi0/3 (to ACC-SW4)
    
- Gi0/1 (to AGG-SW5)
    

```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/1, GigabitEthernet0/2, GigabitEthernet0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124162048.png)

<br>

#### **ACC-SW3 - Trunk Ports**

Trunk ports on ACC-SW3:

- Gi0/3 (to DIST-SW1)
    
- Gi0/0 (to ACC-SW4)
    

```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,99
switchport nonegotiate
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124124452.png)
<br>


#### **ACC-SW4 - Trunk Ports**

Trunk ports on ACC-SW4:

- Gi0/3 (to DIST-SW2)
    
- Gi0/0 (to ACC-SW3)
    

```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,99
switchport nonegotiate
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124124520.png)

<br>

#### **AGG-SW5 - Trunk Ports**

Trunk ports on AGG-SW5:

- Gi0/1 (to DIST-SW2)
    
- Gi0/0 (to INFRA-SW6)
    
- Gi0/3 (to INFRA-SW6)
    

```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/1, GigabitEthernet0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124130729.png)


<br>



#### **INFRA-SW6 - Trunk Ports**

Trunk ports on INFRA-SW6:

- Gi0/0 (to AGG-SW5)
    
- Gi0/3 (to AGG-SW5)
    

```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,60,99
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124130750.png)

<br>

### Result

All inter-switch links are configured as static 802.1Q trunk ports with explicit VLAN lists. DTP negotiation is disabled to keep trunk links consistent and controlled, and trunk connectivity is set across the entire switching topology.


<br>

## **4.7 LACP EtherChannel Configuration**

Link Aggregation Control Protocol (LACP) groups multiple physical links into a single logical connection (Port-Channel). This provides higher total bandwidth and link redundancy, while Spanning Tree treats the Port-Channel as a single logical link.

In this topology, LACP is applied **between the distribution switches to provide redundancy at the distribution layer**, and **between AGG-SW5 and INFRA-SW6**, as this interconnection carries VLAN 99 management traffic and VLAN 50 / VLAN 60 server traffic. If one physical link fails, the Port-Channel remains operational and traffic continues over the remaining links.

#### **LACP between the distribution switches**

![](images/Pasted%20image%2020260124162219.png)

<br>

#### **LACP between between AGG-SW5 and INFRA-SW6**

![](images/Pasted%20image%2020260124162302.png)


<br>

#### **EtherChannel Overview**

* Two physical links between **DIST-SW1 and DIST-SW2** are grouped into one logical Port-Channel 
    
- Two physical links between AGG-SW5 and INFRA-SW6 are grouped into one logical Port-Channel
    
- All member interfaces operate as trunk ports
    
- Spanning Tree treats the Port-Channel as a single link
    



<br>

#### **DIST-SW1 - Configuration**

The following interfaces are added to EtherChannel using LACP in active mode:

- Gi0/0 (to DIST-SW2)
    
- Gi0/2 (to DIST-SW2)
    

```plaintext
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/2
channel-group 2 mode active
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124163121.png)


<br>


DIST-SW2 - Configuration

The following interfaces are added to EtherChannel using LACP in active mode:

- Gi0/0 (to DIST-SW1)
    
- Gi0/2 (to DIST-SW1)
    

```plaintext
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/2
channel-group 2 mode active
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124163156.png)


<br>

#### **AGG-SW5 - Configuration**

The following interfaces are added to EtherChannel using LACP in active mode:

- Gi0/0 (to INFRA-SW6)
    
- Gi0/3 (to INFRA-SW6)
    

```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/3
channel-group 1 mode active
no shutdown
exit
end
write memory

```
![](images/Pasted%20image%2020260124130004.png)

<br>

#### **INFRA-SW6 - Configuration**

The following interfaces are added to EtherChannel using LACP in active mode:

- Gi0/0 (to AGG-SW5)
    
- Gi0/3 (to AGG-SW5)
    

```prikaz
enable
configure terminal
interface range GigabitEthernet0/0, GigabitEthernet0/3
channel-group 1 mode active
no shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260124130028.png)

<br>


### **Result**

LACP EtherChannels are formed **between the distribution switches** and **between AGG-SW5 and INFRA-SW6**. In both cases, multiple physical links operate as a single Port-Channel, improving link redundancy and increasing total available bandwidth for management and server VLAN traffic.


<br>


## **4.8 RSTP Configuration**

Rapid Spanning Tree Protocol (RSTP) is enabled to prevent Layer 2 loops and allow fast recovery after a link change. Root bridge roles are defined on the distribution layer to keep the spanning tree control point in the center of the topology.

DIST-SW2 is selected as the primary root bridge because it connects directly to the firewall, the aggregation switch, and the access layer. DIST-SW1 is configured as the secondary root bridge to provide a stable fallback if the primary root is unavailable.


<br>

#### **DIST-SW2 - RSTP Root Primary**

```prikaz
enable
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,50,60,99 priority 4096
end
write memory
```
![](images/Pasted%20image%2020260124131525.png)

<br>

#### **DIST-SW1 - RSTP Root Secondary**

```prikaz
enable
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,50,60,99 priority 8192
end
write memory
```
![](images/Pasted%20image%2020260124131552.png)

<br>


#### **ACC-SW3 - RSTP Enable**

```prikaz
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```
![](images/Pasted%20image%2020260124131621.png)

<br>

#### **ACC-SW4 - RSTP Enable**

```prikaz
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```
![](images/Pasted%20image%2020260124131644.png)
<br>


#### **AGG-SW5 - RSTP Enable**

```prikaz
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```
![](images/Pasted%20image%2020260124131701.png)

<br>


#### **INFRA-SW6 - RSTP Enable**

```prikaz
enable
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```
![](images/Pasted%20image%2020260124131718.png)

<br>


### **Result**

RSTP is enabled across the switching layer. DIST-SW2 acts as the primary root bridge and DIST-SW1 acts as the secondary root bridge, which keeps spanning tree control on the distribution layer and supports stable loop-free switching.


<br>

## **4.9 Verification**

This last section verifies that all Layer 2 features configured in this chapter are working correctly. Each example focuses on a different switch and validates a specific function such as VLAN creation, access ports, trunking, EtherChannel, and RSTP.


<br>


#### **Example 1 - VLAN Verification (DIST-SW1)**

```prikaz
enable
show vlan brief
```
![](images/Pasted%20image%2020260124131742.png)


<br>
This command confirms that all required VLANs are present and active on the switch.


<br>


#### **Example 2 - Access Ports Verification (ACC-SW3)**

```prikaz
enable
show interfaces status
show running-config interface GigabitEthernet0/1
```
![](images/Pasted%20image%2020260124131820.png)

<br>

These commands verify that access ports are assigned to the correct VLANs and are operational.


<br>


#### **Example 3 - Trunk Ports Verification (ACC-SW4)**

```prikaz
enable
show interfaces trunk
```
![](images/Pasted%20image%2020260124131850.png)

<br>

This command confirms that trunk ports are up and carry the expected VLANs.


<br>


#### **Example 4 - EtherChannel Verification (AGG-SW5)**

```prikaz
enable
show etherchannel summary
```
![](images/Pasted%20image%2020260124131921.png)

<br>

This command verifies that the LACP EtherChannel is formed correctly and that all member interfaces are active.

<br>


#### **Example 5 - RSTP Verification (DIST-SW2)**

```prikaz
enable
show spanning-tree summary
show spanning-tree root
```
![](images/Pasted%20image%2020260124163413.png)

Rapid PVST+ is active and operates correctly across all VLANs.  
DIST-SW2 is elected as the primary root bridge, confirming the intended Layer 2 topology.


<br>

### Result

All verification checks confirm that VLANs, access ports, trunk links, EtherChannel, and RSTP operate as designed across the switching topology.


<br>


## **4.10 Conclusion** 

Throughout this chapter, the switching layer is prepared using key Layer 2 features that improve traffic stability and control. VLANs are configured across all switches, access ports are correctly assigned and protected with PortFast and BPDU Guard, and trunk links carry all required VLANs. Critical inter-switch links are **supported by LACP**, which is applied between the distribution switches for redundancy and between the aggregation and infrastructure switches for management and server traffic. RSTP ensures a loop-free topology with clearly defined root roles.

As a result, the Layer 2 environment is stable and ready for operation, and the network is prepared for the next chapter focused on Layer 3 routing and inter-VLAN communication.



<br>

---



<br>


**Next Chapter:** [Routing and OPNsense Integration](05-routing-and-opnsense-integration.md)
