
<br>

# **7 - OPNsense firewall a NAT PAT**


<br>

## **7.1 Introduction**

This chapter focuses on firewall rules and outbound NAT/PAT configuration on OPNsense. The main goal is to control traffic between VLANs and define clear access rules for internal networks and the Internet.

Firewall policies are set to allow only required communication. User VLANs can access selected internal services and the Internet, while management and server access is strictly limited and separated. **Unauthorized access between VLANs is blocked by default.**

Outbound NAT/PAT is configured using automatic rules to translate internal addresses to the firewall WAN address. The setup follows a standard edge firewall model and prepares the network for real Internet use, even though the lab runs in an offline environment.


>Notes: OPNsense provides a stateful firewall with clear per-VLAN rules, traffic logging, and flexible NAT control. Compared to basic filtering on Cisco routers, it allows better visibility, easier rule management in GUI environment. 

<br>

## **7.2 Topology Diagram**

![](images/Pasted%20image%2020260127052529.png)

<br>

## **7.3 Objectives**

1. Introduces an overview of the firewall ACL policy and access rules.
    
2. Define and apply firewall rules for the Net-Admin VLAN.
    
3. Restrict and protect the Management VLAN 99.
    
4. Configure controlled Internet and service access for the Office VLAN.
    
5. Apply security rules for the Finance VLAN with limited internal access.
    
6. Configure the Sales VLAN with isolated user-level access.
    
7. Implement Guest VLAN rules with Internet-only connectivity.
    
8. Define outbound-only rules for the Server VLAN.
    
9. Verify the firewall (ACL) policy across all VLANs.
    
10. Configure outbound NAT/PAT on the OPNsense firewall.


<br>

## **7.4 Firewall Access Control (ACL) <-> OPNsense**

This first section describes the firewall access control rules implemented on the OPNsense firewall. The rules are based on a default deny approach, where only required traffic is explicitly allowed. Each VLAN represents a separate security zone, and traffic between zones is permitted only when it is required for normal network operation.

The access control policy follows common enterprise practice. User VLANs have controlled access to internal services and the Internet, management traffic is limited to administrative networks, and the guest network is isolated from internal resources. The firewall operates as a stateful device, which means return traffic is automatically allowed for established connections. This section focuses on rule logic and security purpose, while verification and NAT/PAT configuration are described later in the chapter.


<br>

## **ACL policy Overview**

Policy uses a default deny approach. DNS is centralized on the internal Windows DNS server for user VLANs. Guest is isolated from internal services.

| Source (OPNsense interface) | Destination                            | Allowed services (protocol/ports)                                                                                | Purpose / notes                                                                    |
| --------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| vlan0.50 [netadmin]         | All internal VLANs (10/20/30/40/60/99) | ICMP, SSH (TCP 22), HTTP (TCP 80), HTTPS (TCP 443)                                                               | Administration and troubleshooting.                                                |
| vlan0.50 [netadmin]         | mgmt devices in VLAN99                 | SNMP (UDP 161), SNMP traps (UDP 162), Syslog (UDP 514)                                                           | Monitoring and logging (policy-ready).                                             |
| vlan0.50 [netadmin]         | WAN net / ISP next-hop                 | ICMP, DNS (UDP/TCP 53), HTTP (TCP 80), HTTPS (TCP 443)                                                           | Diagnostics and NAT/PAT verification.                                              |
| vlan0.99 [mgmt]             | mgmt net (VLAN99)                      | ICMP, SSH (TCP 22)                                                                                               | Switch-to-switch / device management reachability inside MGMT.                     |
| vlan0.99 [mgmt]             | vlan0.60 [server]                      | Syslog (UDP 514), SNMP traps (UDP 162)                                                                           | Logs and monitoring events to the server.                                          |
| vlan0.99 [mgmt]             | Internet (WAN)                         | NTP (UDP 123), ICMP                                                                                              | Time sync and basic diagnostics (NAT/PAT ready).                                   |
| vlan0.10 [office]           | vlan0.60 [server]                      | DHCP relay (UDP 67-68), DNS (UDP/TCP 53), HTTP (TCP 80), HTTPS (TCP 443)                                         | Internal services (infra.project).<br>                                             |
| vlan0.20 [finance]          | vlan0.60 [server]                      | DHCP relay (UDP 67-68), DNS (UDP/TCP 53), HTTP (TCP 80), HTTPS (TCP 443)                                         | Internal services (infra.project).<br>                                             |
| vlan0.30 [sales]            | vlan0.60 [server]                      | DHCP relay (UDP 67-68), DNS (UDP/TCP 53), HTTP (TCP 80), HTTPS (TCP 443)                                         | Internal services (infra.project).<br>                                             |
| vlan0.10 [office]           | Internet (WAN)                         | HTTP (TCP 80), HTTPS (TCP 443), NTP (UDP 123), ICMP, Mail submission (TCP 587, 465), Mail receive (TCP 993, 995) | User internet access (NAT/PAT ready). DNS is not allowed directly to Internet.<br> |
| vlan0.20 [finance]          | Internet (WAN)                         | HTTP (TCP 80), HTTPS (TCP 443), NTP (UDP 123), ICMP, Mail submission (TCP 587, 465), Mail receive (TCP 993, 995) | User internet access (NAT/PAT ready). DNS is not allowed directly to Internet.     |
| vlan0.30 [sales]            | Internet (WAN)                         | HTTP (TCP 80), HTTPS (TCP 443), NTP (UDP 123), ICMP, Mail submission (TCP 587, 465), Mail receive (TCP 993, 995) | User internet access (NAT/PAT ready). DNS is not allowed directly to Internet.     |
| vlan0.10/20/30 [users]      | Other user VLANs (10/20/30)            | -                                                                                                                | Block traffic between user VLANs                                                   |
| vlan0.10/20/30 [users]      | vlan0.99 [mgmt]                        | -                                                                                                                | Block access to management plane.                                                  |
| vlan0.40 [guest]            | Internet (WAN)                         | DNS (UDP/TCP 53) to OPNsense resolver, HTTP (TCP 80), HTTPS (TCP 443), NTP (UDP 123), ICMP                       | Guest internet-only (NAT/PAT ready). Guest does not use internal DNS.              |
| vlan0.40 [guest]            | Internal VLANs (10/20/30/50/60/99)     | -                                                                                                                | Full isolation from internal networks. Acces to Server (DCHP)                      |
| vlan0.60 [server]           | Internet (WAN)                         | DNS (UDP/TCP 53), HTTP (TCP 80), HTTPS (TCP 443), NTP (UDP 123), ICMP                                            | External DNS / updates / time sync (NAT/PAT ready).                                |

<br>


>**Notes**:
>
>- User VLANs (10/20/30) use the internal Windows DNS server in VLAN60 for name resolution (including external domains).
 >   
>- Guest VLAN uses OPNsense DNS resolver and has no access to internal services (no infra.project access).
>    
>- SMTP port 25 is not allowed by design from user VLANs.
 >   
>- FTP (TCP 21) is not allowed by design. SFTP is covered by SSH (TCP 22) for admin use.


<br>


## 7.5 Firewall Rules – VLAN 50 (netadmin)

The netadmin VLAN is used for network administration and troubleshooting. It has controlled access to internal networks, management devices, and selected external services. The rules allow full administrative access with clear and explicit control.

#### **Inbound rules on interface vlan0.50**

 ```
ALLOW  ICMP                -> All internal VLANs (10/20/30/40/60/99)
ALLOW  SSH (TCP 22)        -> All internal VLANs
ALLOW  HTTP (TCP 80)       -> All internal VLANs
ALLOW  HTTPS (TCP 443)     -> All internal VLANs
ALLOW  SNMP (UDP 161)      -> VLAN 99 (mgmt)
ALLOW  SNMP traps (UDP 162)-> VLAN 99 (mgmt)
ALLOW  Syslog (UDP 514)    -> VLAN 60 (server)
ALLOW DNS (UDP/TCP 53)     -> VLAN 60 (server)
ALLOW  DNS (UDP/TCP 53)    -> Internet (WAN)
ALLOW  HTTP (TCP 80)       -> Internet (WAN)
ALLOW  HTTPS (TCP 443)     -> Internet (WAN)
ALLOW  ICMP                -> Internet (WAN)
implicit deny all
 ```
![](images/Pasted%20image%2020260127054024.png)


<br>


## **7.6 Firewall Rules – VLAN 99 (mgmt)**

The **management VLAN** is used for infrastructure and device management.  
Traffic is limited to essential management, monitoring, and diagnostics only.

#### **Inbound rules on interface vlan0.99**

 ```
VLAN 99 (mgmt) – inbound rules on interface vlan0.99
ALLOW  ICMP                -> mgmt net (VLAN 99)
ALLOW  SSH (TCP 22)        -> mgmt net (VLAN 99)
ALLOW  Syslog (UDP 514)    -> VLAN 60 (server)
ALLOW  SNMP traps (UDP 162)-> VLAN 60 (server)
ALLOW  NTP (UDP 123)       -> Internet (WAN)
ALLOW  ICMP                -> Internet (WAN)
implicit deny all

 ```
![](images/Pasted%20image%2020260127010744.png)



<br>


## **7.7 Firewall Rules – VLAN 10 (office)**

The **office VLAN** is used for standard user workstations.  
This VLAN has access to internal infrastructure services provided by the server and limited access to the Internet.  
Direct access to other user VLANs and the management network is not permitted.



#### **Inbound rules on interface vlan0.10**

 ```
ALLOW  DHCP relay (UDP 67-68)   -> VLAN 60 (server)
ALLOW  DNS (UDP/TCP 53)         -> VLAN 60 (server)
ALLOW  HTTP (TCP 80)            -> VLAN 60 (server)
ALLOW  HTTPS (TCP 443)          -> VLAN 60 (server)
ALLOW  HTTP (TCP 80)            -> Internet (WAN)
ALLOW  HTTPS (TCP 443)          -> Internet (WAN)
ALLOW  NTP (UDP 123)            -> Internet (WAN)
ALLOW  ICMP                     -> Internet (WAN)
implicit deny all

 ```
![](images/Pasted%20image%2020260127014244.png)

<br>

## **7.8 Firewall Rules – VLAN 20 (finance)**

The **finance VLAN** is used for user workstations with access to internal services and the Internet. Traffic is limited to required services only. Access to other user VLANs and the management network is not permitted.


#### **Inbound rules on interface vlan0.20**

 ```
ALLOW  DHCP relay (UDP 67-68)   -> VLAN 60 (server)
ALLOW  DNS (UDP/TCP 53)         -> VLAN 60 (server)
ALLOW  HTTP (TCP 80)            -> VLAN 60 (server)
ALLOW  HTTPS (TCP 443)          -> VLAN 60 (server)
ALLOW  HTTP (TCP 80)            -> Internet (WAN)
ALLOW  HTTPS (TCP 443)          -> Internet (WAN)
ALLOW  NTP (UDP 123)            -> Internet (WAN)
ALLOW  ICMP                     -> Internet (WAN)
implicit deny all
 ```
![](images/Pasted%20image%2020260127020845.png)

<br>

## **7.9 Firewall Rules – VLAN 30 (sales)**

The **sales VLAN** is used for standard user workstations.  
This network has access to internal application services hosted in the server VLAN and controlled access to the Internet. Direct access to other user VLANs and the management network is not allowed. DNS resolution is provided only by the internal DNS server.

##### **Inbound rules on interface vlan0.30**

 ```
ALLOW  DHCP relay (UDP 67-68)   -> VLAN 60 (server)
ALLOW  DNS (UDP/TCP 53)         -> VLAN 60 (server)
ALLOW  HTTP (TCP 80)            -> VLAN 60 (server)
ALLOW  HTTPS (TCP 443)          -> VLAN 60 (server)
ALLOW  HTTP (TCP 80)            -> Internet (WAN)
ALLOW  HTTPS (TCP 443)          -> Internet (WAN)
ALLOW  NTP (UDP 123)            -> Internet (WAN)
ALLOW  ICMP                     -> Internet (WAN)
implicit deny all
 ```
![](images/Pasted%20image%2020260127022102.png)


<br>

## **7.10 Firewall Rules – VLAN 40 (guest)**

The **Guest VLAN** is designed for internet access only.  
Access to internal networks is fully blocked. The only exception is **DHCP relay to the internal server**, which allows guest clients to obtain IP configuration. DNS traffic is allowed **only to the Internet**, not to the internal DNS server.


#### **Inbound rules on interface vlan.40**


 ```
ALLOW DHCP relay (UDP 67-68)  -> VLAN 60 (server)
ALLOW DNS (UDP/TCP 53)        -> Internet (WAN)
ALLOW HTTP (TCP 80)           -> Internet (WAN)
ALLOW HTTPS (TCP 443)         -> Internet (WAN)
ALLOW NTP (UDP 123)           -> Internet (WAN)
ALLOW ICMP                    -> Internet (WAN)
implicit deny all
 ```
![](images/Pasted%20image%2020260127024821.png)


<br>

## **7.11 Firewall Rules – VLAN 60 (server)**

The **Server VLAN** hosts core internal services such as DNS, DHCP relay target, and web services.  
The server is allowed to access the Internet only for updates, external DNS resolution, and time synchronization. Access to user VLANs is controlled and limited by their own rules.


##### **Inbound rules on interface vlan0.60**


 ```
ALLOW DNS (UDP/TCP 53)   -> Internet (WAN)
ALLOW HTTP (TCP 80)      -> Internet (WAN)
ALLOW HTTPS (TCP 443)    -> Internet (WAN)
ALLOW NTP (UDP 123)      -> Internet (WAN)
ALLOW ICMP               -> Internet (WAN)
implicit deny all

 ```
![](images/Pasted%20image%2020260127030114.png)



<br>


### **Summary**

This firewall policy controls traffic between VLANs and allows only needed connections.  
Each network has clear rules based on its role.

- **Netadmin** – Access to internal networks and basic Internet testing.
    
- **User VLANs** – No access between users, only allowed services and Internet traffic.
    
- **Guest VLAN** – Internet access only, no access to internal networks.
    
- **Server VLAN** – Limited Internet access for DNS, updates, and time sync.
    
- **Management VLAN** – Separate network for managing devices.
    

**This setup limits unnecessary access, protects internal networks, and keeps the network clear and easy to manage.**


<br>

## **7.12 Verification ACL Policy**

This section verifies that the firewall policy behaves as designed. Each VLAN is tested using basic connectivity checks to confirm allowed traffic works and restricted traffic is blocked.

<br>

### NET-ADMIN (VLAN 50)

- Ping management switch (DIST-SW2, VLAN 99) - allowed
    
- Ping user VLAN (Office, VLAN 10) - allowed
    
- Verify DNS resolution via server
    
- Verify HTTP access to internal server
    

```prikazy
ping 10.70.99.12
ping 10.70.10.100
nslookup server.infra.project
wget http://10.70.60.10
```
![](images/Pasted%20image%2020260127042531.png)

**Result:** Allowed access to management, user VLANs, DNS and internal HTTP as expected.

<br>

### OFFICE (VLAN 10)

* Verify DHCP address - OK
    
- Ping to -> internal server (VLAN 60) - fail
    
- Ping to -> other user VLAN (Finance) – fail
    
* Ping to -> Management ACC-SW3 (VLAN 99) – fail

```prikazy
ping 10.70.60.10
ping 10.70.20.100
ping 10.70.99.13
```
![](images/Pasted%20image%2020260127042645.png)

Office VLAN is correctly isolated from other internal networks.  
Access to the internal server (VLAN 60), Finance (VLAN 20), and Management ACC-SW3 (VLAN 99) is blocked as expected.  
**This confirms that traffic between user VLANs is not allowed.**

<br>

### FINANCE (VLAN 20)

* Verify DHCP address - OK
    
- Ping to -> internal server (VLAN 60) - fail
    
- Ping to -> Sales (VLAN 30) – fail
    
* Ping to -> Guest (VLAN 40) – fail

```prikazy
ping 10.70.60.10
ping 10.70.30.100
ping 10.70.40.100
```
![](images/Pasted%20image%2020260127042824.png)

Finance VLAN is correctly isolated from other internal networks.  
Access to the internal server (VLAN 60), Sales (VLAN 30), and Guest (VLAN 40) is blocked as expected.  
**This confirms that traffic between user VLANs is not allowed.**


<br>

### SALES (VLAN 30)

* Verify DHCP address - OK
    
- Ping to -> internal server (VLAN 60) - fail
    
- Ping to -> Management DIST-SW2 (VLAN 99) – fail
    
* Ping to -> Finance (VLAN 20) - fail

```prikazy
ping 10.70.60.10
ping 10.70.99.12
ping 10.70.20.100
```
![](images/Pasted%20image%2020260127043103.png)

Sales VLAN is properly restricted by the firewall policy.  
Access to the internal server (VLAN 60), Management DIST-SW2 (VLAN 99), and Finance (VLAN 20) is blocked.  



<br>

### GUEST (VLAN 40)

- Verify DHCP address - OK (10.70.40.100)
    
- Open browser and Verify DNS server.infra.project - fail
    
- Ping to -> internal server – fail
    
* Ping to -> Finance (VLAN 20) - fail

```prikazy
ip a
nslookup server.infra.project
ping 10.70.60.10
ping 10.70.20.100
```
![](images/Pasted%20image%2020260127043409.png)

**Result:** Guest VLAN gets a valid DHCP address and has basic Internet access. Internal DNS and all internal networks are blocked as intended. Guest network is isolated and follows the security policy.

<br>

### SERVER (VLAN 60)

- Verify DNS resolution local web - ok
    
- Verify HTTP address local web  - ok
    
- Ping to -> NET-ADMIN – fail
    
* Ping to -> Finance (VLAN 20) - fail

```prikazy
open browser -> server.infra.project
open browser -> http://10.70.60.10
ping 10.70.50.10 
ping 10.70.20.100
```
![](images/Pasted%20image%2020260127044156.png)

**Result:** Server VLAN can reach own services as DNS and HTTP. Access to user and admin VLANs is restricted.

<br>

### **Verification Summary**

All firewall rules work as defined by the policy. Required traffic is allowed, traffic between user VLANs is blocked, and Internet access is limited based on each VLAN role. The network security setup is verified and consistent across all segments.


<br>


## **7.13 – Firewall: NAT / PAT (Outbound)**

Outbound NAT is configured using automatic rule generation.  
All internal VLANs are translated to the WAN address of the firewall.  
The design follows a typical edge-firewall model and is prepared for real Internet connectivity.  

*External reachability is not tested as the lab runs in an offline environment.*

<br>

### **Outbound NAT – OPNsense Automatic Rules**

![](images/Pasted%20image%2020260127042143.png)



<br>

---

<br>


### **Firewall Logging and Live Traffic Monitoring**


![](images/Pasted%20image%2020260127060529.png)

>**Notes**: This screen shows live firewall logs in OPNsense.  
 The Live View is used to monitor allowed and blocked traffic in real time and to check that firewall rules work correctly.  
 Blocked packets (Default deny) and allowed connections can be clearly identified per interface and VLAN.


<br>


### **OPNsense – Operational Dashboard**

![](images/Pasted%20image%2020260127053536.png)


>**Notes**: The dashboard shows the operational state of the firewall after applying all rules and NAT policies.  
 Active services, gateways, and live traffic confirm correct configuration.



<br>

## **7.14 Conclusion**

This chapter implemented a firewall policy using OPNsense based on the **Default Deny** principle. All traffic is blocked by default, and only required communication is allowed per VLAN with clear role separation.

User VLANs can access only approved internal services and the Internet, while management and server networks are protected from user traffic. This limits attack paths and helps stop threats from spreading inside the network. All firewall rules were verified through practical testing.

Automatic outbound NAT/PAT hides internal networks behind the firewall WAN address and prepares the design for real-world use. The firewall policy follows common enterprise security practices and provides strong and structured network protection.

The next chapter focuses on **advanced security configuration of Cisco switches and routers**.


<br>

---


<br>

**Next Chapter:**





















































































