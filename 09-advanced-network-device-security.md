

<br>

# **9 - Advanced Network Device Security**


<br>


## **9.1 Introduction**

This is the final configuration chapter of the project and it focuses on advanced security protection for Cisco network devices. The main goal is to protect the management plane by reducing the risk of unauthorized access, repeated login attempts, and incorrect device access.

The chapter presents a practical device security setup commonly used in enterprise networks. Security controls are demonstrated on two representative devices: the EDGE-R3 router and the ACC-SW4 access switch, while following a consistent security policy across the network infrastructure.

Centralized authentication is implemented using a simple **RADIUS server** deployed on an **Xubuntu NET-ADMIN**. This approach allows real verification of AAA functionality without introducing additional enterprise identity services such as Active Directory.



<br>


## **9.2 Topology Diagram**

![](images/Pasted%20image%2020260130163926.png)

<br>

## 9.3 **Objectives**

1. Define a clear security overview for Cisco network devices used in the enterprise network and allow firewall Rulles (OPNsense).
    
    - Present the purpose and scope of device-level security.
    
    * Allow required management traffic between NET-ADMIN and EDGE-R3 through the OPNsense firewall.
    
2. Apply advanced management access security on the EDGE-R3 router.

    * Deploy a lightweight RADIUS server on NET-ADMIN to provide centralized authentication.
    
    - Configure local authentication, privileged access, and centralized AAA with RADIUS.
        
    - Protect console and remote access lines (backup) from unauthorized use.
        
    - Reduce attack surface by disabling unused router interfaces.
        
3. Implement access-layer security controls on the ACC-SW4 access switch.
    
    - Secure user-facing ports using port security with sticky MAC addresses.
        
    - Control violation action to limit network issues while keeping visibility.




<br>


## **9.4 Device Security Overview**

This section introduces device-level security used in the network and explains how management access to Cisco routers and switches is protected.

Before device security is configured, required management and authentication traffic is allowed through the OPNsense firewall. Firewall rules enable connectivity and RADIUS communication between the NET-ADMIN host and the EDGE-R3 router.

<br>

**The security configuration focuses on:**

- Management access protection and centralized authentication
    
- User authentication and privilege control
    
- Interface and port-level security
    
- Console and line access protection
    
<br>


Security is demonstrated on two devices: the EDGE-R3 router and the ACC-SW4 access switch. A lightweight RADIUS server running on the NET-ADMIN Xubuntu host is used as the primary authentication source, while local access is kept as an alternative.



<br>

## **9.4.1 Firewall Rules for Management and RADIUS Access (OPNsense)**


This section configures firewall rules on the OPNsense firewall to allow management connectivity and RADIUS authentication traffic between the NET-ADMIN host and the EDGE-R3 router. These rules are required for basic reachability testing and for correct AAA operation using a centralized RADIUS server.


<br>

### **OPNsense Firewall – ICMP (Ping) Rule**

This rule allows basic connectivity testing between the NET-ADMIN host and the EDGE-R3 router. ICMP is used only to verify network reachability and does not provide authentication functionality.

```prikaz
Action: Pass
Interface: VLAN-50 (NET-ADMIN)
Protocol: ICMP
Source: NET-ADMIN (10.70.50.10)
Destination: EDGE-R3 WAN IP (52.84.0.1)
Description: Allow ICMP from NET-ADMIN to EDGE-R3
```
![](images/Pasted%20image%2020260130100059.png)

<br>

### **Verify**

Ping to -> EDGE-R3

```prikaz
ping 52.84.0.1
```
![](images/Pasted%20image%2020260130100138.png)

<br>

### **Results**

- NET-ADMIN can successfully ping the EDGE-R3 router.
    
- Basic Layer 3 connectivity between the management host and the router is verified.

<br>

### **OPNsense Firewall – RADIUS (UDP) Rule**

This rule allows RADIUS authentication and accounting traffic required for AAA communication between the EDGE-R3 router and the RADIUS server running on the NET-ADMIN Xubuntu host.

```prikaz
Action: Pass
Interface: VLAN-50 (NET-ADMIN)
Protocol: UDP
Source: NET-ADMIN (10.70.50.10)
Destination: EDGE-R3 WAN IP (52.84.0.1)
Destination Port Range: 1812–1813
Description: Allow RADIUS authentication and accounting traffic
```
![](images/Pasted%20image%2020260130100511.png)

<br>

### **Results**

- RADIUS authentication requests can pass through the firewall.
    
- Centralized AAA authentication between EDGE-R3 and NET-ADMIN is possible.

<br>
### **WAN Firewall Rule – RADIUS Access to NET-ADMIN**


This rulle configures a required WAN-side firewall rule on the OPNsense firewall. The rule allows RADIUS authentication traffic initiated by the EDGE-R3 router to reach the NET-ADMIN RADIUS server. Without this rule, inbound authentication traffic is blocked by default on the WAN interface.


### **OPNsense Firewall – WAN RADIUS Rule**

```prikaz
Action: Pass
Interface: WAN
Protocol: UDP
Source: EDGE-R3 (52.84.0.1)
Destination: NET-ADMIN (10.70.50.10)
Destination Port Range: 1812–1813
Description: Allow RADIUS traffic from EDGE-R3 to NET-ADMIN
```
![](images/Pasted%20image%2020260130110128.png)

<br>

### **Results**

- RADIUS authentication requests from EDGE-R3 can reach the NET-ADMIN server.
    
- Centralized AAA authentication functions correctly through the firewall.




<br>

## **9.5 Router Security – EDGE-R3 -> RADIUS Server**

This section applies advanced management access security to the EDGE-R3 router. The configuration focuses on secure administrator authentication, protection of management access lines, monitoring of login attempts, and reduction of the router attack surface. Each security function is configured separately and demonstrated using a complete configuration sequence.

<br>

### **9.5.1 RADIUS Server Deployment on Xubuntu (NET-ADMIN)**


This  first subsection introduces a lightweight RADIUS server deployed on the Xubuntu NET-ADMIN host. The server provides centralized authentication for network devices and is used by the EDGE-R3 router for AAA login and authorization. The goal is to demonstrate real and functional RADIUS-based authentication in a simple and controlled lab environment.

<br>

### **Overview**

- RADIUS service is deployed on the NET-ADMIN Xubuntu host.
    
- The server listens on the management network and is reachable by network devices.
    
- A shared secret is used to secure communication between the router and the RADIUS server and must be identical on both sides.
    
- Local authentication remains available on devices as a fallback option.
    
<br>


### **Device Information**

- Host: NET-ADMIN (Xubuntu)
    
- IP address: 10.70.50.10
    
- Default gateway: 10.70.50.1
    
- Service: FreeRADIUS
    

<br>

### Configuration

```prikaz
sudo apt update
sudo apt install freeradius -y

sudo nano /etc/freeradius/3.0/clients.conf
```
![](images/Pasted%20image%2020260130101350.png)

```prikaz
client EDGE-R3 {
 ipaddr = 52.84.0.1
 secret = ksdlKSmI45sIy#s5
}
```
![](images/Pasted%20image%2020260130101703.png)
![](images/Pasted%20image%2020260130101943.png)

<br>

```prikaz
sudo nano /etc/freeradius/3.0/users
```
![](images/Pasted%20image%2020260130102038.png)

<br>

```prikaz
netadmin ClearText-Password := "sdfGs24S-e85seGS"
```
![](images/Pasted%20image%2020260130102527.png)

<br>

### **Verify**

```prikaz
sudo systemctl restart freeradius
sudo systemctl status freeradius
```
![](images/Pasted%20image%2020260130102852.png)

<br>

### **Results**

- The RADIUS service is installed and running on the Xubuntu host.
    
- The EDGE-R3 router is registered as a valid RADIUS client.
    
- A test administrative user is available for centralized authentication. User passwords on the RADIUS server are independent from local router credentials.
    



<br>



### **9.5.2 Local Authentication, Privileged Access, and AAA with RADIUS**

This part configures secure local authentication and privileged access on the router and enables centralized authentication using AAA with a RADIUS server.

**Key security functions:**

- Protect privileged EXEC mode using an encrypted enable secret.
    
- Create a local administrator account with full privileges.
    
- Enable AAA and use RADIUS for centralized authentication with local fallback.
    


<br>

```prikaz
enable
configure terminal
service password-encryption
security passwords min-length 12
enable secret R!Sh43r-priv#20
username netadmin privilege 15 secret N3tAdm!n-3#Secsfe1
aaa new-model
radius server RADIUS-SRV
address ipv4 10.70.50.10 auth-port 1812 acct-port 1813
key ksdlKSmI45sIy#s5
aaa authentication login default group radius local
aaa authorization exec default group radius local
end
write memory
```
![](images/Pasted%20image%2020260130110753.png)

<br>

### **Results**

- The configuration is applied successfully and becomes active immediately.
    


<br>

### **Verify**

```prikaz
test aaa group radius netadmin sdfGs24S-e85seGS legacy
show radius statistics
```
![](images/Pasted%20image%2020260130152824.png)

<br>

### **Results**

The router now uses the RADIUS server for user login authentication.  
The user is verified by the RADIUS server before access is granted.  
After successful login, an enable password is still required to enter privileged mode. This setup adds an extra security layer and works as expected.


<br>

### **9.5.3 Console and Remote Access Line Protection**

This section secures console and remote (VTY) access on the router. Authentication and session timeout settings are used to reduce the risk of unauthorized access and to close inactive management sessions automatically. The configuration uses a clear and practical approach suitable for enterprise environments.



### Configuration
```
line console 0
 login authentication default
 exec-timeout 5 0
 exit
line vty 0 4
 login authentication default
 transport input ssh
 exec-timeout 5 0
 exit
```
![](images/Pasted%20image%2020260130153759.png)



### **Results**
    
- Remote (VTY) access uses AAA authentication with local fallback.
    
- Idle console and SSH sessions close automatically after 5 minutes.
    
- Management access is simple and secure.
    


<br>


> Notes: The RADIUS server is the primary login method. If the server is offline, the router uses **local fallback**. This means the device **stays** accessible because it checks the local username and secret in its memory. This is a safety net for troubleshooting.


<br>


### **9.5.4 Login Protection and Access Monitoring**

This part protects the router against repeated and unauthorized login attempts. Login blocking and logging features are enabled to improve security visibility and reduce security risks.

**Key security functions:**

- Temporarily block login access after multiple failed attempts.
    
- Log successful and failed login events.
    
- Improve monitoring of management access activity.
<br>


```prikaz
enable
configure terminal
login block-for 120 attempts 3 within 60
login on-failure log
login on-success log
end
write memory
```
![](images/Pasted%20image%2020260130154000.png)


<br>

### **9.5.5 Interface Security and Attack Surface Reduction**

This part reduces the router attack surface by disabling not used physical interfaces. Only interfaces required for network operation stay active.

**Key security functions:**

- Disable not used router interfaces.
    
- Clearly label not used ports using interface descriptions.
    
- Reduce the risk of unauthorized physical connections.
    

<br>


```prikaz
configure terminal
interface GigabitEthernet0/1
description UNUSED
shutdown
exit
interface GigabitEthernet0/2
description UNUSED
shutdown
exit
interface GigabitEthernet0/3
description UNUSED
shutdown
exit
interface GigabitEthernet0/4
description UNUSED
shutdown
exit
end
write memory
```
![](images/Pasted%20image%2020260130154123.png)

<br>

### **Results**

- The configuration is applied successfully and becomes active immediately.
    


>Notes: Unused interfaces are off by default, but we still turn them off manually and mark them as "UNUSED". This makes the config clear, prevents mistakes, and stops people from connecting to the network without permission.



<br>

### **Verify**

```prikaz
show ip interface brief
```
![](images/Pasted%20image%2020260130154143.png)



### Section Summary -> Router Security (EDGE-R3)

This section demonstrates advanced security configuration applied to the EDGE-R3 router to protect management access. Centralized authentication using RADIUS, secured local user accounts, and protected access lines are configured to control administrator access.

Additional security controls are applied to monitor login attempts and reduce the router attack surface by disabling interfaces that are not used. Together, these configurations establish a clear and practical security baseline for secure enterprise router management.

<br>


## **9.6 Access Layer Port Security (ACC-SW4)**


This final section secures user-facing access ports on the ACC-SW4 switch. Port Security is used to limit which devices can connect to the network and to reduce the risk of unauthorized access. Sticky MAC addresses are enabled so the switch can automatically learn and remember allowed devices.


<br>

### **Configuration Goals**

- Protect user ports with Port Security
    
- Automatically learn device MAC addresses (sticky MAC)
    
- Limit the number of devices per port
    
- Keep visibility when a violation happens
    


<br>


### **Configuration Commands (ACC-SW4)**

```plaintext
configure terminal
interface range GigabitEthernet0/1 - 2
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation restrict
exit
end
write memory
```
![](images/Pasted%20image%2020260130160908.png)

<br>

### **Result**

Port Security is now active on user access ports. Each port allows only one learned device. If another device is connected, traffic is blocked but the port remains active.


<br>


### **Verification Commands**

```plaintext
show port-security
show port-security interface GigabitEthernet0/1
show running-config interface GigabitEthernet0/1
```
![](images/Pasted%20image%2020260130161749.png)

<br>

### **Result**

The output confirms that Port Security is enabled, a sticky MAC address is learned, and the violation mode is set to restrict.

<br>

### **Summary**

This section adds simple but good security. Only known devices can use the ports. This lowers the risk of outside connections. We also use logging to see what is happening on the network without turning off the ports.


<br>

## **9.7 Conclusion**

This chapter focused on advanced security for network devices. First, firewall rules were prepared to allow safe management and RADIUS traffic. Then centralized authentication was implemented using a RADIUS server on the NET-ADMIN host. The EDGE-R3 router was secured with AAA, local fallback access, protected console and VTY lines, login monitoring, and reduced attack surface by disabling unused interfaces.

In the final part, access-layer security was applied on the ACC-SW4 switch using Port Security with sticky MAC addresses. All configurations were tested and verified. The network is fully configured, tested, and works as expected **based on** the defined goals, security policy, and chosen design. The next chapter is the final summary of the whole project.


<br>


---


<br>

**Final Chapter:** [Conclusion and Summary](10-conclusion-and-summary.md)
