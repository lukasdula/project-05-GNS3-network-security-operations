

<br>

# **6 - Core Server Services**



<br>

## **6.1 Introduction**

This chapter introduces the core server services deployed on the **Windows Server 2022 located in VLAN 60** within the internal network (MGMT & SERVER ZONE). These services are implemented as a foundation for later security and firewall policy application on the OPNsense firewall.

Dynamic Host Configuration Protocol (DHCP) provides automatic IPv4 addressing for all user VLANs, allowing client devices to join the network without manual configuration.

Domain Name System (DNS) and Hypertext Transfer Protocol (HTTP) simulate a typical internal company environment. The organization operates its own internal domain, **infra.project**, and provides internal name resolution and a local web service.

Together, DHCP, DNS, and HTTP generate controlled service traffic between VLANs, which is later used to define and validate inter-VLAN firewall rules and port-based security policies.


<br>

## **6.2 Topology Diagram**

![](images/Pasted%20image%2020260126064507.png)

<br>

## **6.3 Objectives**

1. **Deploy DHCP service**
    
    - Install the DHCP Server role on the Windows Server.
        
    - Create DHCP scopes for client VLANs.
        
    - Define IP ranges, subnet masks, default gateways, and DNS settings.
        
    - Set lease duration according to lab requirements.
        
    - Activate all scopes and verify DHCP operation.
        
2. **Deploy internal HTTP service**
    
    - Configure a basic internal web service on the Windows Server.
        
    - Use the service to simulate an internal company web application.
        
    - Prepare HTTP traffic for later firewall and port-based policy control.
        
3. **Deploy internal DNS service**
    
    - Configure internal DNS for the domain **infra.project**.
        
    - Provide name resolution for internal services and hosts.
        
4. **Configure DHCP relay on OPNsense and allow firewal for DHCP, HTTP, DNS services**
        
5. **Verify connectivity and service reachability between VLAN**



<br>



## **6.4 Core Server Services Overview**

The following table summarizes the core server services deployed in the internal network. These services generate controlled application traffic that is later used for firewall rule definition, inter-VLAN access control, and security policy validation on the OPNsense firewall.

| **Service** | **Purpose**                                        | **VLAN Access**           | **Protocol / Ports** | **Address / Name**                |
| ----------- | -------------------------------------------------- | ------------------------- | -------------------- | --------------------------------- |
| DHCP        | Dynamic IPv4 address assignment for client devices | User VLANs<br>10,20,30,40 | UDP 67, UDP 68       | Windows Server IP                 |
| DNS         | Internal name resolution for the company network   | All internal VLANs        | UDP 53, TCP 53       | infra.project (Windows Server IP) |
| HTTP        | Internal company web service (test / intranet)     | Selected user VLANs       | TCP 80               | Windows Server IP                 |

*This service layout allows precise control of permitted traffic flows between VLANs and forms the basis for firewall and access control policies implemented in later chapters.*


<br>



## **6.5 DHCP Service**

This first configure section documents the deployment of the DHCP service as part of the core server services. The following subsections describe the individual steps required to install the DHCP role, define address scopes, configure supporting parameters, and verify correct DHCP operation across the internal network.


<br>


### **6.5.1 Install the DHCP Server Role (Windows Server)**

This subsection installs the DHCP Server role on the Windows Server.


#### **Graphical(GUI) installation:**

  
```

Open Server Manager → Manage → Add Roles and Features, select DHCP Server, confirm the required tools, and start the installation.

```
![](images/Pasted%20image%2020260125235453.png)

### **Results**

The DHCP Server role is installed and ready for post-installation configuration.

<br>

### **6.5.2 Create DHCP Scopes for Client VLANs**

This subsection defines DHCP scopes for all user VLANs. Each scope follows the same structure: a name, an IP pool, a subnet mask, and the default gateway for the given VLAN.

### Scope Overview for User VLANs

| **VLAN** | **Scope Name** | **Start IP** | **End IP**   | **Mask**      | **Gateway** | **DNS Server** |
| -------- | -------------- | ------------ | ------------ | ------------- | ----------- | -------------- |
| 10       | VLAN10-Office  | 10.70.10.100 | 10.70.10.199 | 255.255.255.0 | 10.70.10.1  | 10.70.60.10    |
| 20       | VLAN20-Finance | 10.70.20.100 | 10.70.20.199 | 255.255.255.0 | 10.70.20.1  | 10.70.60.10    |
| 30       | VLAN30-Sales   | 10.70.30.100 | 10.70.30.199 | 255.255.255.0 | 10.70.30.1  | 10.70.60.10    |
| 40       | VLAN40-Guest   | 10.70.40.100 | 10.70.40.199 | 255.255.255.0 | 10.70.40.1  | 10.70.60.10    |

Only **VLAN 10** is shown in detail in this documentation. The remaining scopes are created using the same steps and the values listed above.

<br>

### **6.5.3 Configuration VLAN 10 (Office) - Example**


```
Scope Name: VLAN10-Office
Description: DHCP scope for VLAN 10
```
![](images/Pasted%20image%2020260126000741.png)


<br>


 ```
 IP Range:
  Start IP: 10.70.10.100
  End IP:   10.70.10.199
  Subnet Mask: 255.255.255.0
 ```
![](images/Pasted%20image%2020260126000854.png)


<br>

```
Lease Duration: 8 Days
```
![](images/Pasted%20image%2020260126001009.png)


>**Notes:** The lease duration is set to **8 days**, which works well for a stable office network with mostly fixed workstations. This setting reduces frequent DHCP renewals while still allowing IP addresses to be reused when devices leave the network.



<br>


```
Router (Default Gateway): 10.70.10.1
```
![](images/Pasted%20image%2020260126001152.png)



<br>


```
Parent domain: infra.project
DNS Server: 10.70.60.10
```
![](images/Pasted%20image%2020260126044412.png)

<br>

After completing the wizard, the scope is activated and the address pool becomes available for clients in VLAN 10. This configuration serves as the reference example for the remaining VLAN scopes.

<br>

### **6.5.4 Active DHCP Scopes Summary**

![](images/Pasted%20image%2020260126003619.png)

*The DHCP scope for VLAN 10 is successfully created and activated. All remaining user VLAN scopes are configured in the same way using the scope overview table.*

<br>

### **6.5.5 Verification**

This last subsection DHCP verifies that the DHCP service is running and that scopes are active on the Windows Server.

```prikaz
Get-Service DHCPServer
Get-DhcpServerv4Scope
```
![](images/Pasted%20image%2020260126003745.png)
### Results

The DHCP service is running and all user VLAN scopes are present and active.


<br>


## 6.6 HTTP **Configuration** 


This next section describes the deployment of a basic HTTP service on the Windows Server. The goal is to provide a simple internal web service accessible from user VLANs. The HTTP service is used later as a reference application for firewall and ACL policy testing.

**Step by Step:**

1. Install the Web Server (IIS) role on the Windows Server.
    
2. Verify that the Default Web Site is created and running.
    
3. Allow inbound HTTP traffic on port 80.
    
4. Verify HTTP connectivity on the Windows Server
    

<br>

### **6.6.1 Install IIS (Web Server Role)**

The Web Server (IIS) role is installed using Server Manager.


```
IIS Service: Installed and Running  
```
![](images/Pasted%20image%2020260126005605.png)

*The IIS service is installed and running.*  
*The Default Web Site is created automatically and remains active, ready to serve HTTP requests from clients.*


<br>

### **6.6.2 IIS Service Verification**

After installation, IIS services start automatically and the Default Web Site is created.

```prikaz
Get-Service W3SVC
Get-Website
```
![](images/Pasted%20image%2020260126005928.png)

### **Results**

- IIS service (W3SVC) is running
    
- Default Web Site is started and bound to HTTP port 80
    

<br>

### **6.6.3 HTTP Port Verification (Server Side)**


```prikaz
netstat -ano | findstr ":80"
```
![](images/Pasted%20image%2020260126010835.png)

This check confirms that the IIS service is actively listening on TCP port 80 on the Windows Server.


<br>



## **6.7 DNS Configuration on Windows Server**

This section covers the basic DNS configuration on **Windows Server 2022** deployed in **VLAN 60 (server)**. The DNS service is used for internal name resolution within the lab network, allowing clients to reach internal services by hostname instead of IP address.

<br>

### **Step by Step**

1. Install the **DNS Server** role via **Server Manager → Add Roles and Features** on Windows Server 2022.
    
2. Create a **Forward Lookup Zone** named **infra.project
    
3. Add a new **Host (A) record** with hostname **server** and IP address **10.70.60.10**.
    
4. Configure **DHCP Option 006 (DNS Servers)** to distribute **10.70.60.10** to clients.
    
5. Verify that the record **server.infra.project → 10.70.60.10** exists in DNS Manager.
    
6. Confirm that the DNS service is running and able to resolve the server name.
    


<br>


### **6.7.1 DNS Role Installation**

The DNS Server role is installed using the Add Roles and Features Wizard. After completing the installation, the role appears in Server Manager and can be managed through DNS Manager.



```
DNS Service: Installed and Running  
```
![](images/Pasted%20image%2020260126020048.png)

<br>

### **6.7.2 Forward Lookup Zone Creation**

A Forward Lookup Zone named **infra.project** is created to provide hostname-to-IP resolution within the internal network.

![](images/Pasted%20image%2020260126044853.png)





<br>

### **6.7.3 Adding Host Record**

A new Host (A) record named **infra.project**  is created and mapped to IP address **10.70.60.10** 


![](images/Pasted%20image%2020260126044947.png)


<br>

### **6.7.4 Verification**


#### Windows-Server

```
Get-Service DNS
```
![](images/Pasted%20image%2020260126022448.png)

<br>

DNS resolution is verified by accessing the internal web service using the domain name instead of the IP address.

```plaintext
http://server.infra.project
```
![](images/Pasted%20image%2020260126045431.png)

### **Results**

DNS resolution was successfully verified using both command-line tools and a web browser confirming correct integration of DNS, DHCP, and IIS services.


<br>

## **6.8 DHCP Relay Configuration on OPNsense**


In this part, DHCP relay is configured on OPNsense to enable DHCP communication between clients located in different VLANs and the centralized DHCP server. Because DHCP uses broadcast messages, which do not cross Layer 3 limits, a relay mechanism is required to forward these requests to the DHCP server. At the same time, OPNsense acts as a firewall between VLANs, so the necessary services must be clearly allowed to make sure correct network functionality. Firewall rules will be addressed in detail in a dedicated subsection later in this chapter.

#### **DHCP Relay – Configuration Steps**

1. Open the OPNsense web interface and navigate to **Services → DHCPv4 → Relay**.
    
2. Enable the DHCP Relay service.
    
3. Select all VLAN interfaces that require DHCP service from the centralized server.
    
4. Set the DHCP Server address to the IP address of the Windows DHCP server.
    
5. Apply and save the configuration.
    



<br>


![](images/Pasted%20image%2020260126062554.png)

At this point, OPNsense forwards DHCP requests from the selected VLANs to the DHCP server, allowing clients to obtain IP configuration dynamically across routed networks.


<br>

## 6.8.1 Firewall Rules for Service Testing


This section defines firewall rules used to test DHCP, DNS, HTTP, and basic connectivity between VLANs. Some rules are intentionally open and used only for testing services. All user VLANs use the same rule set, so only one VLAN (VLAN10 – office) is shown as an example.

> Note: These firewall rules are temporary and used only for service testing in this chapter. They will be limited and secured in later sections.


<br>

### VLAN10 – Office (Example for User VLANs)

The following rules apply equally to all user VLANs (office, finance, sales, guest). Only VLAN10 is shown as a representative example.

```plaintext
Allow ICMP (IPv4)
Source: any
Destination: any
Purpose: Connectivity testing (ping)

Allow UDP 67–68
Source: any
Destination: Windows Server
Purpose: DHCP relay traffic

Allow UDP 53
Source: any
Destination: Windows Server
Purpose: DNS resolution

Allow TCP 80
Source: any
Destination: Windows Server
Purpose: HTTP (IIS) service testing
```
![](images/Pasted%20image%2020260126034516.png)

<br>

### VLAN50 – NET-ADMIN

Administrative access rules allow management and verification from the NET-ADMIN VLAN.

```plaintext
Allow ICMP (IPv4)
Source: netadmin
Destination: any
Purpose: Network reachability testing

Allow UDP 53
Source: any
Destination: Windows Server
Purpose: DNS resolution

Allow TCP 80
Source: any
Destination: Windows Server
Purpose: HTTP (IIS) verification
```
![](images/Pasted%20image%2020260126054430.png)

<br>


### VLAN60 – Server (Windows Server)

These rules ensure that the Windows Server can respond correctly to client and administrative requests.

```plaintext
Allow ICMP (IPv4)
Source: any
Destination: Windows Server
Purpose: Server reachability

Allow UDP 53
Source: any
Destination: any
Purpose: DNS service availability

Allow TCP 80
Source: any
Destination: any
Purpose: IIS web service access
```
![](images/Pasted%20image%2020260126035554.png)


<br>


### **Results**

With these firewall rules in place, clients across VLANs can obtain IP configuration via DHCP relay, resolve DNS records, access the IIS web service, and verify connectivity using ICMP. This confirms correct inter-VLAN routing and service reachability for testing purposes.




<br>


## **6.9 - Service Verification**

This last section verifies basic network service functionality across VLANs. The verification focuses on connectivity and correct operation of DHCP, DNS, and HTTP services to ensure that addressing, name resolution, and basic service reachability work as expected before proceeding to more advanced testing.


<br>

### **6.9.1 - DHCP Verification ICMP Connectivity Verification**

<br>

##### **OFFICE (VLAN10)**

- ping -> GUEST (VLAN 40)
    
- ping -> WINDOWS-SERVER (VLAN 60)
    
- ping -> NET-ADMIN (VLAN 50)

 ```
ping 10.70.40.100 
ping 10.70.60.10 
ping 10.70.50.10 
 ```
![](images/Pasted%20image%2020260126053920.png)


<br>

##### **GUEST (VLAN40)**

- ping -> SALES (VLAN 30)
    
- ping -> NET-ADMIN (VLAN 50)
    
- ping -> WINDOWS-SERVER (VLAN 60)
    

 ```
ping 10.70.30.100 
ping 10.70.50.10 
ping 10.70.60.10 
 ```
![](images/Pasted%20image%2020260126054036.png)

<br>

##### **NET-ADMIN (VLAN50)**

- ping -> FINANCE (VLAN 20)
    
- ping -> OFFICE (Vlan 10)
    
- ping -> WINDOWS-SERVER (VLAN 60)
    

 ```
ping 10.70.20.100 
ping 10.70.10.100 
ping 10.70.60.10 
 ```
![](images/Pasted%20image%2020260126054602.png)

<br>

### **Results**

ICMP connectivity between user VLANs, the NET-ADMIN network, and the Windows Server is successfully verified. All tested devices respond to ping requests, confirming that:

- DHCP relay is functioning correctly across VLANs
    
- Clients received valid IP addresses from the expected DHCP scopes
    
- Inter-VLAN routing is operational
    
- Firewall rules allow required testing traffic at this stage




<br>

---



### **6.9.2 - HTTP verification**


This subsection verifies HTTP connectivity to the internal web server. The goal is to confirm that clients from different VLANs can reach the IIS web service using IPv4 and that firewall rules allow HTTP traffic as expected. 

<br>

#### **NET-ADMIN (VLAN 50)**

From the Net-Admin workstation, the web server is accessed directly via its IPv4 address using a web browser.

```
http://10.70.60.10
wget http://10.70.60.10
```
![](images/Pasted%20image%2020260126060426.png)

Successful page load confirms that HTTP access from the Net-Admin VLAN to the Windows Server is working correctly.

<br>

#### **GUEST (VLAN 40)**

From the Guest device, HTTP connectivity is verified both via a web browser and a simple command-line check.

```
http://10.70.60.10
wget http://10.70.60.10
```
![](images/Pasted%20image%2020260126060334.png)

The successful response confirms that HTTP traffic from the Guest VLAN to the Windows Server is allowed and functional.


<br>

### Results

HTTP connectivity to the Windows Server has been successfully verified from multiple VLANs. The IIS web service is reachable, and the firewall configuration allows HTTP traffic as intended.

<br>

---

<br>

### **6.9.3 - DNS Verification**


This subsection verifies that DNS name resolution works correctly for internal services. The test confirms that clients can resolve the server hostname using the configured DNS server and domain settings.

<br>

#### **Guest Device**

DNS verification is performed from the **GUEST** device, as the same DNS configuration and rules apply to all user VLANs.

**Terminal verification:**

```
nslookup server.infra.project
http://server.infra.project
```
![](images/Pasted%20image%2020260126070528.png)

**DNS resolution is working correctly across VLANs**.

### Results

DNS name resolution works as expected. Clients are able to resolve internal hostnames using the DNS server, which confirms correct DNS configuration, DHCP options, and firewall rules for DNS traffic.


<br>

## **6.10 Conclusion**

In this chapter, core server services were configured and tested step by step. First, DHCP was set up and verified so that all VLANs receive correct IP addresses. After that, HTTP access to the Windows Server was tested from multiple networks, which confirmed that the web service is reachable and working as expected. Finally, DNS resolution was verified, and hostnames were correctly resolved to IP addresses from client devices.

At this point, all main services are working together and network connectivity across VLANs is confirmed. DHCP, HTTP, and DNS operate correctly, which shows that the server and network setup is stable. The next chapter will focus on OPNsense firewall configuration, including access control rules and NAT/PAT, to improve network security and traffic control.

<br>

---



<br>


**Next Chapter:**



































































































































































