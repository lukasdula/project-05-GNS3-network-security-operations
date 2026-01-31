
<br>

# **8 - Remote Access VPN (WireGuard)**


<br>

## **8.1 Introduction**

This eighth chapter describes secure remote administration using a **WireGuard VPN tunnel**. The SOHO-ADMIN host connects to the internal management network through the Internet, the ISP path, the EDGE-R3 router, and the OPNsense firewall. Management access is available only over the VPN tunnel, while direct access from the WAN to internal networks remains blocked.

The VPN design focuses on remote access to **the management VLAN (VLAN 99)** and network devices such as routers and switches. It allows secure connectivity to infrastructure components, including **remote access to the OPNsense firewall web GUI from the SOHO-ADMIN host**. User and server VLANs stay isolated and continue to follow the existing firewall policy. 

This design reflects common enterprise practice and prepares the network for secure remote operations.

<br>

## **8.2 Topology Diagram**

![](images/Pasted%20image%2020260128171433.png)


<br>


## **8.3 Objectives**


1. Verify basic connectivity between the SOHO-ADMIN host and the OPNsense firewall using ICMP.
    
2. Prepare the SOHO-ADMIN system for VPN access by installing the WireGuard client tools.
    
3. Configure the WireGuard VPN service on the OPNsense firewall.
    
4. Enable OPNsense management access through the management VLAN (VLAN 99).
    
5. Define firewall rules for WAN, WireGuard traffic and management access (ICMP, HTTPS, SSH).
    
6. Enable SSH access on a management network device (VLAN 99) as an example.
    
7. Verify secure remote administration from the SOHO-ADMIN host via the VPN tunnel.
    
8. Demonstrate remote access to the OPNsense web interface over the VPN connection.


<br>


## **8.4 Initial Connectivity Verification**

This first section verifies basic connectivity between the remote SOHO-ADMIN host and the OPNsense firewall before VPN setup and configuration. The goal is to confirm that routing and firewall rules already allow basic communication from the remote location to the firewall WAN interface.

### **OPNsense GUI – Firewall Rules (WAN)**

This step defines a temporary firewall rule on the OPNsense WAN interface to allow ICMP traffic from the SOHO-ADMIN network. The rule is used only for initial connectivity verification and troubleshooting before the VPN tunnel is configured.

```prikazy
Firewall Rules (WAN)
ALLOW ICMP
Source: SOHO-ADMIN network (18.192.0.0/30)
Destination: This Firewall (OPNsense WAN)
Description: Allow ICMP from SOHO-ADMIN to WAN
implicit deny all
```
![](images/Pasted%20image%2020260128091037.png)

<br>

### **SOHO-ADMIN Host**

This ping verifies connectivity from the SOHO-ADMIN host to the OPNsense WAN interface. 

- Ping to -> OPNsense WAN interface (via EDGE-R3)
    

```prikazy
ping 52.84.0.2
```
![](images/Pasted%20image%2020260128091130.png)

A successful ICMP reply confirms that routing through the Internet, ISP, and EDGE-R3 router works as expected.

<br>

## **8.5 SOHO-ADMIN – WireGuard Client Preparation**

This next section prepares the SOHO-ADMIN host for VPN access. The goal is to install the required WireGuard client tools and verify that the system is ready for VPN configuration. No VPN tunnel is activated at this stage. The configuration focuses only on client-side preparation.

### **Overview**

- Install WireGuard client tools on the SOHO-ADMIN system.
    
- Verify that WireGuard utilities are available on the system.
    
- Confirm that no active VPN tunnel is configured yet.
    

### **SOHO-ADMIN – WireGuard Installation**

The following commands install the WireGuard client tools on the SOHO-ADMIN host. These tools are required later for VPN interface configuration and tunnel activation.

```prikazy
sudo apt update
sudo apt install wireguard wireguard-tools -y
```
![](images/Pasted%20image%2020260128094137.png)

<br>

### **Verification – WireGuard Tools**

This step verifies that the WireGuard tools are installed correctly and available on the system. At this stage, no WireGuard interface is active.

```prikazy
wg
```
![](images/Pasted%20image%2020260128094306.png)

*The `wg` command runs successfully and returns no output, which confirms that the WireGuard tools are installed correctly and no active VPN interface is configured at this stage.*
### Results

The SOHO-ADMIN system has the WireGuard client tools installed successfully. The system is prepared for VPN configuration, while no VPN tunnel is active yet. Further configuration is performed in the next section.


<br>

## **8.6 WireGuard Configuration – OPNsense**

This section configures the WireGuard VPN service on the OPNsense firewall. The goal is to create a basic WireGuard instance that defines the local tunnel interface, cryptographic keys, and listening parameters as part of the overall VPN setup.

<br>

### Overview

- Create a WireGuard instance on the OPNsense firewall.
    
- Define tunnel address and listening port.
    
- Generate and assign cryptographic keys.
    
- Prepare the VPN interface for peer configuration.
    
<br>

### **8.6.1 OPNsense GUI – WireGuard Instance**

This first subsection creates the initial WireGuard instance on the OPNsense firewall. The instance defines the local VPN endpoint and tunnel parameters. At this stage, no peers are configured and no VPN traffic is allowed yet.

**Navigation path:**  
WireGuard -> Instances -> Add / Edit

#### **OPNsense GUI – WireGuard Instance Configuration**

```prikazy
Instance name: WG-REMOTE-ACCESS
Enabled: yes
Instance ID: 0
Public key: auto-generated
Private key: auto-generated
Listen port: 51820
Tunnel address: 10.200.200.1/24
Peers: none
Disable routes: unchecked
```
![](images/Pasted%20image%2020260128101334.png)

#### **WireGuard Instance – Verification**

![](images/Pasted%20image%2020260128101414.png)

<br>

### **8.6.2 WireGuard Peer Key Generation – SOHO-ADMIN**

This second subsection generates WireGuard keys and basic client parameters using the OPNsense Peer Generator. The goal is to create the client identity for the SOHO-ADMIN host and prepare the required information for later configuration steps. The generated keys are stored temporarily and used in subsequent subsections.

#### **Purpose**

- Generate public and private keys for the SOHO-ADMIN WireGuard client.
    
- Assign a dedicated VPN address to the remote client.
    
- Prepare client parameters for peer configuration on OPNsense.
    
- Store generated keys for later use.
    

#### **OPNsense GUI – Peer Generator**

The Peer Generator is used to create a client-side WireGuard configuration. At this stage, the configuration is not applied directly. The generated values are recorded and used in later steps.

Navigation path:  
WireGuard -> Peer generator

```prikazy
Instance: WG-REMOTE-ACCESS
Name: SOHO-ADMIN
Address: 10.200.200.2/32
Allowed IPs: 10.200.200.2/32
Public key: generated
Private key: generated
```
![](images/Pasted%20image%2020260128110916.png)


>**Notes**: 
>- The public key is used in the next subsection when configuring the WireGuard peer on OPNsense. 
>- The private key is used later on the SOHO-ADMIN host during client configuration.
   

<br>

### **8.6.3 WireGuard Peer Configuration - OPNsense**

This section configures the WireGuard peer for the remote SOHO administrator. The peer defines the client identity and connects it to the WireGuard instance created in the previous step.

### **Edit peer**

This configuration links the SOHO-ADMIN client to the WireGuard server instance. The public key is copied from the Peer Generator. Endpoint parameters stay empty because this is a remote-access VPN setup.

```prikazy
Enabled: yes
Name: SOHO-ADMIN
Public key: copied from Peer Generator (client public key)
Allowed IPs: 10.200.200.3/32
Endpoint address: empty
Endpoint port: empty
Instance: WG-REMOTE-ACCESS
```
![](images/Pasted%20image%2020260128110814.png)

#### **WireGuard Peers – Verification**

![](images/Pasted%20image%2020260128111136.png)


**The peer is now bound to the WireGuard instance and ready for secure remote access. The tunnel is initiated by the client while server-side settings stay controlled.**

<br>

### **8.6.4 WireGuard Interface Assignment**

This subsection assigns the WireGuard instance as a logical interface in OPNsense. Creating an interface is required to apply firewall rules, control access, and allow routed traffic between the VPN tunnel and internal VLANs.

### Interface assignment

```
Device: wg0 (WireGuard - WG-REMOTE-ACCESS)
Description: WG_REMOTE_ACCESS
```
![](images/Pasted%20image%2020260128111954.png)

- The WireGuard tunnel is available as a standard OPNsense interface.

<br>


## **8.7 OPNsense Administration and Firewall Rules**

This section configures firewall rules for secure remote administration over the WireGuard VPN. It focuses on controlling management traffic through the WAN, WireGuard, and management VLAN interfaces. Only required protocols are allowed, while all other access remains restricted by firewall policy.

<br>

### **Overview**

- A WAN firewall rule allows WireGuard VPN connections to the firewall
    
- WireGuard interface rules control VPN management traffic
    
- Management access is limited to the management VLAN (VLAN 99)
    
- Only ICMP, HTTPS, and SSH traffic is permitted for remote administration
    

<br>


### **8.7.1 Firewall Rules - WireGuard Interface (ICMP)**

This rule allows basic ICMP traffic from the WireGuard tunnel. It is used to verify VPN connectivity before enabling management access.

```prikazy
Action: Pass
Interface: WireGuard (Group)
Direction: In
TCP/IP Version: IPv4
Protocol: ICMP
Source: WireGuard net
Destination: any
Description: Allow ICMP from WireGuard
```
![](images/Pasted%20image%2020260128172339.png)

<br>


### **8.7.2 Firewall Rules - WireGuard Interface (HTTPS)**

This rule allows secure access to the OPNsense web interface over the VPN connection.

```prikazy
Action: Pass
Interface: WireGuard (Group)
Direction: In
TCP/IP Version: IPv4
Protocol: TCP
Source: WireGuard net
Destination: This Firewall
Destination Port: 443 (HTTPS)
Description: Allow HTTPS to OPNsense from WireGuard
```
![](images/Pasted%20image%2020260128172523.png)

<br>


### **8.7.3 Firewall Rules - WireGuard Interface (SSH)**

This rule allows SSH access from the VPN to network devices located in the management VLAN.

```prikazy
Action: Pass
Interface: WireGuard (Group)
Direction: In
TCP/IP Version: IPv4
Protocol: TCP
Source: WireGuard net
Destination: mgmt net (VLAN 99)
Destination Port: 22 (SSH)
Description: Allow SSH from WireGuard to mgmt VLAN
```
![](images/Pasted%20image%2020260128172446.png)

<br>



### **8.7.4 Firewall Rules - Management VLAN (VLAN 99)**

Additional firewall rules on the management VLAN allow SSH access from the WireGuard network. Existing internal management rules stay the same.

```prikazy
Action: Pass
Interface: mgmt (VLAN 99)
Direction: In
TCP/IP Version: IPv4
Protocol: TCP
Source: WireGuard net
Destination: mgmt net
Destination Port: 22 (SSH)
Description: Allow SSH from WireGuard to management devices
```
![](images/Pasted%20image%2020260128172611.png)





### **8.4.5 Firewall Rules - WAN (WireGuard VPN)**

This section allows incoming WireGuard VPN connections on the WAN interface. The rule permits UDP traffic to the firewall on the WireGuard service port and enables the VPN tunnel to establish.

```
Action: Pass
Interface: WAN
Direction: In
TCP/IP Version: IPv4
Protocol: UDP
Source: 18.192.0.2/32 (SOHO-ADMIN)
Destination: This Firewall
Destination Port: 51820
Description: Allow WireGuard VPN from SOHO-ADMIN
```
![](images/Pasted%20image%2020260128172648.png)

The rule opens only the required UDP port on WAN. All VPN access is then controlled by firewall rules on the WireGuard and management VLAN interfaces.


<br>


### **Result**

Remote administration access is controlled by firewall rules on the WireGuard,WAN and management VLAN interfaces. WireGuard provides the secure transport layer, while ICMP is used for connectivity testing, HTTPS for OPNsense access, and SSH for managing devices in VLAN 99.


<br>

## **8.8 SSH Access Configuration - Management Switch**

This section configures secure SSH access on a management switch located in VLAN 99. The goal is to prepare the device for controlled remote administration over the WireGuard VPN.
The configuration focuses on local authentication, restricted access methods, session protection, and basic hardening of both remote (VTY) and local (console) access.

<br>

### **Overview**

- Enable SSH version 2 for secure remote management
    
- Configure local user authentication with privileged access
    
- Define a management domain for cryptographic key generation
    
- Harden VTY and console access using timeouts and login control
    
- Apply basic security measures such as password encryption and banners
    

<br>

### **SSH Configuration on DIST-SW2 (VLAN 99)**

The following configuration enables SSH access on the management switch using local authentication. Access is restricted to secure methods only and prepared for use through the WireGuard VPN.

```prikazy
enable
configure terminal
service password-encryption
username admin privilege 15 secret HsGKsghS8ysL56s
ip domain-name project.mgmt
crypto key generate rsa 
2048
ip ssh version 2
line console 0
login local
exec-timeout 10 0
logging synchronous
exit
line vty 0 15
login local
transport input ssh
exec-timeout 10 0
exit
banner motd ! WARNING: Authorized management access only !
exit
write memory
```
![](images/Pasted%20image%2020260128141531.png)

<br>

### **Result**

SSH access to the management switch is enabled and limited to VLAN 99. The device is prepared for secure remote administration via the WireGuard VPN, with hardened access controls applied to both remote and local management sessions.


<br>


## **8.9 WireGuard Client Configuration - SOHO-ADMIN**

This section configures the WireGuard VPN client on the SOHO-ADMIN host. The goal is to establish a secure point-to-point tunnel to OPNsense, verify connectivity, and prepare the environment for remote administration access (SSH and Web GUI) in the next section.

<br>


### **SOHO-ADMIN - WireGuard Client Configuration**

The SOHO-ADMIN host is configured as a WireGuard peer. The configuration uses a static tunnel address, the public WAN endpoint of OPNsense, and explicit routing for the management network.

```prikazy
sudo nano /etc/wireguard/wg0.conf
```
![](images/Pasted%20image%2020260128154855.png)

```
[Interface]
PrivateKey = oHahRfm0EC2g3S+6P+RG8DcaFg07INLQ9SZNgdvPU8=
Address = 10.200.200.2/32
DNS = 10.70.60.10

[Peer]
PublicKey = neufrsdAyVPambTR9jhQN6QLML0LwOvAJ8a4Rj2vgSo=
Endpoint = 52.84.0.2:51820
AllowedIPs = 10.200.200.0/24, 10.70.99.0/26
PersistentKeepalive = 25
```
![](images/Pasted%20image%2020260128155109.png)


<br>

### **VPN Tunnel Activation and Verification**

The following commands activate the WireGuard tunnel, display the tunnel status, and verify connectivity to the OPNsense WireGuard interface.

```prikazy
sudo wg-quick up wg0
sudo wg
ip a show wg0
ping 10.200.200.1
```
![](images/Pasted%20image%2020260128155153.png)


<br>

### **Result**

**The WireGuard tunnel on the SOHO-ADMIN host is successfully established. The interface is up, a valid handshake is confirmed, and ICMP connectivity to the OPNsense WireGuard address is verified.** The environment is now fully prepared for the final demonstration of remote administration access over VPN, including SSH access to management devices and HTTPS access to the OPNsense web interface in the next section.


<br>

## **8.10 Remote Administration Demonstration (SOHO-ADMIN)**

This final section demonstrates successful remote administration over the WireGuard VPN from the SOHO-ADMIN host. The demonstration covers secure access to the OPNsense web interface and SSH access to management network devices in VLAN 99.

<br>

### **Overview**

- Remote access is performed from the SOHO-ADMIN host over the WireGuard tunnel
    
- OPNsense web administration is accessed via HTTPS
    
- Management switch access is verified using SSH
    
- Internal connectivity inside the management VLAN is validated
    

<br>

### **8.10.1 Remote Access to OPNsense Web GUI**

The first step verifies remote firewall administration. The SOHO-ADMIN host accesses the OPNsense web interface over the VPN using the management gateway address.

```prikazy
https://10.70.99.1
```
![](images/Pasted%20image%2020260128162616.png)

### **Result**

The OPNsense web interface is reachable from the SOHO-ADMIN host over the WireGuard VPN. Remote firewall administration is successfully performed from an external location.

<br>

### **8.10.2 Remote SSH Access to Management Switch (DIST-SW2)**

This step demonstrates secure SSH access from the SOHO-ADMIN host to a management switch in VLAN 99. Connectivity inside the management network is also verified.

**Process overview:**

- Verify reachability of the management switch
    
- Establish an SSH session to DIST-SW2
    
- Test internal management VLAN connectivity
    
- Verify switch status
    

```prikazy
ping 10.70.99.12
ssh dist-sw2
ping 10.70.99.13
show ip interface brief
```
![](images/Pasted%20image%2020260128161744.png)

### **Result**

The SOHO-ADMIN host successfully establishes an SSH session to the management switch DIST-SW2 over the WireGuard VPN. Internal connectivity within VLAN 99 is confirmed, and the switch is fully accessible for remote administration.

<br>

> **Notes**:
> 
> The SSH connection initially failed due to a key exchange and host key algorithm mismatch between the modern OpenSSH client and the Cisco IOS device.
> 
> To resolve this, a local SSH client configuration was created to explicitly allow legacy algorithms for this specific switch.
> 
> After updating the SSH config file, the connection was successfully established using a simplified SSH command.


<br>

### **Summary**

Remote administration over WireGuard is fully functional. The SOHO-ADMIN host can securely access the OPNsense firewall via HTTPS and manage network devices in the management VLAN using SSH. **The environment is ready for secure real-world remote administration scenarios.**

<br>

## **8.11 Conclusion**

This chapter shows the full setup of secure remote administration using a WireGuard VPN. It starts with configuring the WireGuard instance and peer on OPNsense, assigning tunnel IP addresses, enabling the WireGuard interface, and creating firewall rules on the WAN, WireGuard, and management VLAN interfaces to allow only required management traffic.

Next, SSH access is configured on the management switch in VLAN 99, including local user authentication, SSH key generation, and basic session security settings. The SOHO-ADMIN host is then configured as a WireGuard client, the VPN tunnel is activated, and connectivity is verified using handshake status, routing, and ICMP tests. Remote administration is confirmed by accessing the OPNsense web interface and connecting to the management switch over SSH.

The network is now ready for secure remote administration. The next chapter focuses on security hardening on Cisco network devices and access protection.

<br>

---


<br>

**Next Chapter:**