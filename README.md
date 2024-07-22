
<h1> Two Office Network-Configuration--CCNA-Recap</h1>
<h2>Description</h2>
 In this home lab project, I will configure a three-tier LAN encompassing two offices using Cisco Packet Tracer for virtual configurations. The project involves:

  - <b>Network Configuration: IPv4 and IPv6 addressing, static routes, VLANs, spanning tree, OSPF.</b>
  - <b>Advanced Features: EtherChannel, DHCP, DNS, NAT, SSH, security features.</b>
  - <b>Wireless Configurations: Completed via Cisco GUI.</b>

This capstone project is designed to showcase comprehensive knowledge of all CCNA topics. This project was developed by Jeremy of JeremysITLab.
  
<br/>
<h2>Utilities Used</h2>

- <b>Cisco Packet Tracer</b>


<h2>Program walk-through:</h2>


<h3>Network Topology</h3>
<p align="center">
<img src="https://i.imgur.com/ZlyHjLe.png" height="80%" width="80%" alt="Topology"/>
</p><br/>
<p align="left">Network Topology Description:

The network is structured as follows:

  Internet Connection:
> Dual-Homed Router: Located at the top center, connects the network to the internet.

  Core Layer:
> Core Switches (CSW1 and CSW2): Multilayer switches that will be interconnected using a PAgP EtherChannel for redundancy and load balancing.

  Distribution Layer:
> Layer 3 Switches (DSW-A1, DSW-A2, DSW-B1, DSW-B2): Four switches managing routing and distribution between core and access layers.

  Access Layer:
> Layer 2 Switches (ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, ASW-B3): Six switches providing connectivity to end hosts and other network devices.<br/>
>      Wireless LAN Controller (WLC1): Connected to ASW-A1.<br/>
>     Lightweight Access Points (LWAP1 and LWAP2):<br/>
>          LWAP1 provides guest wireless access in Office A and connects to ASW-A1.<br/>
>          LWAP2 provides guest Wi-Fi in Office B and connects to ASW-B1.<br/>
>      End Hosts: Each switch has one end host connected, with additional end hosts connected through ASW-B2 and the wireless access points.<br/>

  Servers:
> SRV1: Located in Office B, serving as a DNS server, FTP server, and Syslog server for the network.

Additional Notes:

>  Cisco C2960 switches are used, each with 24 Fast Ethernet ports, but only one end host is connected per switch for demonstration purposes</p>
<br/>
<p align="center">
<img src="https://i.imgur.com/3oSoI7g.png" height="80%" width="80%" alt="Diagram Layers"/>
</p><br/>  
<br/>
<h3>Part 1 - Initial Setup</h3>
<h4>Step 1: Configure appropriate hostnames for each router and switch</h4>
<br/>
  
  ```
enable
configuration terminal
```
  ```
hostname <name>
exit
```
  
<p align="center">
<img src="https://i.imgur.com/XFh94DR.png" height="80%" width="80%" alt="hostname"/>
</p>
<br/>
<p align="center">
<img src="https://i.imgur.com/SDTysiV.png" height="80%" width="80%" alt="hostname"/>
</p>
<br/>
<p align="left">In Packet Tracer, you can access a network device`s CLI(Command Line Interface) by double-clicking the device and then clicking the CLI tab. The hostname is configured from Global Configuration mode. The `enable` command allows the user to access Privileged EXEC mode, indicated by #. The `configure terminal` command allows the user to access Global Configuration mode, indicated by (config)#. </p>
<br/>
<br/>
<h4>Step 2: Configure the enable secret jeremysitlab on each router and switch.</h4>
Use type 9 hashing if available; otherwise, use type 5.
<br/>

  ```
enable algorithm-type script secret jeremysitlab
```
or
  ```
enable secret jeremysitlab
```


<p align="center">
<img src="https://i.imgur.com/h7tupSf.png" height="80%" width="80%" alt="level 9 hashing"/>
</p>
<br/>
<p align="center">
<img src="https://i.imgur.com/yeapZHJ.png" height="80%" width="80%" alt="level 5 hashing"/>
</p>
<br/>
<p align="left">The `enable secret` and `enable algorithm-type script secret` commands each provide an added layer of security. These commands set a password for entering Privileged EXEC mode, which is stored in a hashed format for security. The `enable algorithm-type script secret` command, using type 9 hashing, provides the most secure hashing algorithm on current Cisco devices, though it is not supported on every device type. Type 5 hashing, used by the enable secret command, employs MD5 hashing, which is considered less secure than type 9. Using hashing algorithms with passwords is always a good security practice. Although MD5 hashing is no longer considered secure, it adds another layer of complexity for anyone attempting to tamper with the device. 
<br/>
<br/>
<h4>Step 3: Configure the user account cisco with secret ccna on each router and switch.</h4>
Use type 9 hashing if available; otherwise, use type 5 
<br/>

   ```
username cisco algorithm-type script secret ccna

```
or
  ```
username cisco secret ccna
```

<p align="center">
<img src="https://i.imgur.com/q37zwUV.png" height="80%" width="80%" alt="u/p level 9"/>
</p>
<br/>
<p align="center">
<img src="https://i.imgur.com/BQp8nD8.png" height="80%" width="80%" alt="u/p level 5"/>
</p>
<br/>
<p align="left"> These commands create a local user account with a username `cisco` and an encrypted password `ccna`. Type 9 hashing of the password is preferred, but it is not supported on all devices. When the command `username cisco algorithm-type script secret ccna` is not accepted on the device, the next preferred option for this lab is `username cisco secret ccna`.
<br/>
<br/>
<h4>Step 4: Configure the console line to require login with a local user account. Set a 30-minute inactivity timeout. Enable synchronous logging.</h4>
<br/>
  
  ```
line console 0
  login local
  exec-timeout 30
  logging synchronous
```
<p align="center">
<img src="https://i.imgur.com/KpT2OCm.png" height="80%" width="80%" alt="console line config"/>
</p>
<br/>
<p align="left"> 
The `line console 0` command is used to enter the console line configuration mode on a Cisco device. This mode allows you to configure settings for the console line, which is the physical console port on the device. The `login local` command allows usernames and passwords configured locally on the device can be used to log in. The `exec-timeout 30` command signs out the signed-in user after 30 minutes of inactivity. The `logging synchronous` command ensures that system messages do not interrupt your command input on the console, logging synchronous reprints your current command line after each log message. The step is repeated on each router and switch.
<br/> 
<br/> 
  
<h3>Part 2 - VLANs, Layer-2 EtherChannel</h3>
<h4>Step 1: Configure a Layer-2 EtherChannel named PortChannel1 between DSW-A1 and DSW-A2 using a Cisco-proprietary protocol.</h4>  
Both switches should actively try to form an EtherChannel.
<br/>

  ```
interface range g1/0/4-5
  channel-protocol pagp
  channel-group 1 mode desirable
```
<p align="center">
<img src="https://i.imgur.com/P6VctL5.png" height="80%" width="80%" alt="PAgP EtherChannel"/>
</p>
<br/>
<p align="left"> 
This command configures a Layer 2 EtherChannel on a Cisco switch using the Port Aggregation Protocol (PAgP). PAgP is a Cisco proprietary protocol used to form an EtherChannel. EtherChannels bundle multiple physical links into a single logical link, providing redundancy, load balancing, and simplified network management, among other benefits. The `interface range` command allows for grouping interfaces to be configured simultaneously, rather than configuring each interface individually. The `channel-protocol pagp` line sets PAgP as the EtherChannel negotiation protocol for the group of interfaces; however, this line is not necessary. Finally, `channel-group 1 mode desirable` adds the selected interfaces to EtherChannel group 1 in desirable mode, which actively tries to form an EtherChannel using PAgP. These commands need to be entered on both DSW-A1 and DSW-A2 only.
<br/> 
<br/> 
<h4>Step 2: Configure a Layer-2 EtherChannel named PortChannel1 between DSW-B1 and DSW-B2 using an open standard protocol.</h4> 
Both switches should actively try to form an EtherChannel.
<br/>

  ```
interface range g1/0/4-5
  channel-protocol lagp
  channel-group 1 mode active
```
<p align="center">
<img src="https://i.imgur.com/DoKcwgb.png" height="80%" width="80%" alt="LACP EtherChannel"/>
</p>
<p align="left"> 
This command configures a Layer 2 EtherChannel on a switch using the Ling Aggregation Control Protocol (LACP). LACP is an open standard protocol used to form an EtherChannel, LACP is defined in IEEE 802.3ad.  The `channel-protocol lacp` line sets LACP as the EtherChannel negotiation protocol for the group of interfaces; however, it is not necessary. Finally, `channel-group 1 mode active` adds the selected interfaces to EtherChannel group 1 in active mode, which actively tries to form an EtherChannel using LACP. These commands need to be entered on both DSW-B1 and DSW-B2 only.
<br/> 
<br/> 
<h4>Step 3: Configure all links between Access and Distribution switches, including the EtherChannels, as trunk links.</h4> 
 Disable DTP on all ports. Set each trunk`s native VLAN to VLAN 1000(unused). <br/> 
 In Office A, allow VLANs 10, 20, 40, and 99 on all trunks.<br/> 
 In Office B, allow VLANs 10, 20, 30, and 99 on all trunks.<br/> 
<br/>
  
Office A Access Layer:
  ```
interface range g0/1-2
  switchport mode trunk
  switchport nonegotiate
  switchport trunk native vlan 1000
  switchport trunk allowed vlan 10,20,40,99
```
<p align="center">
<img src="https://i.imgur.com/qQ3y9dq.png" height="80%" width="80%" alt="LACP EtherChannel"/>
</p>

Office B Access Layer:
  ```
interface range g0/1-2
  switchport mode trunk
  switchport nonegotiate
  switchport trunk native vlan 1000
  switchport trunk allowed vlan 10,20,30,99
```
<p align="center">
<img src="https://i.imgur.com/O1otgd2.png" height="80%" width="80%" alt="LACP EtherChannel"/>
</p>
<p align="left"> This configuration establishes two trunk ports that will not negotiate trunking dynamically, defines a native VLAN (VLAN 1000), and restricts the allowed VLANs for the trunk. A Virtual Local Area Network (VLAN) is a logical grouping of devices within a larger physical network, enabling network segmentation. A trunk is used when you need to carry multiple VLANs on an interface. Using the command `switchport nonegotiate` ensures that the trunk will not attempt to negotiate with neighboring devices, providing more control and reducing potential misconfigurations. The command `switchport trunk native vlan 1000` sets the native VLAN to an unused VLAN, ensuring that all packets are tagged with the source VLAN, which prevents them from being inadvertently sent to an unintended destination. This reduces the risk of VLAN hopping and minimizes security risks. The command `switchport trunk allowed vlan 10,20,40,99` used in Office A and `switchport trunk allowed vlan 10,20,30,99` used in Office B specify which VLANs are allowed to be sent on the trunk. If a packet is not tagged with one of these VLANs for the respective offices, it is dropped. Office A commands are used on ASW-A1, ASW-A2, and ASW-A3. Office B commands are used on ASW-B1, ASW-B2, and ASW-B3.
  
Office A Distribution Layer:
  ```
interface range g1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
```
  ```
interface po1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
```

Office B Distribution Layer:
  ```
interface range g1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99
```
  ```
interface po1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99
```

<p align="left">The Office A commands are used on switches DSW-A1 and DSW-A2, while the Office B commands are used on switches DSW-B1 and DSW-B2. They serve the same purpose as described above but operate in the opposite direction, from the distribution layer switches to the access layer switches. The `interface po1 command is used to configure the EtherChannel, allowing traffic to be sent through it as well. During these configurations, you may receive several Syslog CDP warnings about native VLAN mismatches. This is normal while completing this step. Once all switches are configured appropriately, the Syslog warnings will stop.</p>
<br/> 
<br/> 

<h4>Step 4: Configure one of each office’s Distribution switches as a VTPv2 server. </h4>
 Use the domain name JeremysITLab. <br/> 
 Verify that other switches join the domain. <br/> 
 Configure all Access switches as VTP clients.<br/> 
 <br/> 

Distribution Layer Switches:

  ```
vtp domain JeremysITLab
vtp version 2
```
<p align="center">
<img src="https://i.imgur.com/NQj2kwv.png" height="80%" width="80%" alt="VTP Server"/>
</p><br/>
<p align="left"> The default VLAN Trunking Protocol (VTP) is a Cisco proprietary protocol used to manage VLAN configurations. VTP is enabled by default. Switches are in VTP server mode by default, which allows for the creation, modification, and deletion of VLANs within the VTP domain. The `vtp domain JeremysITLab` command creates or joins other switches on the network into a group or domain to share VTP information. The `vtp version 2` command ensures each switch utilizes the same version of VTP. These commands should be repeated on the distribution layer switches: DSW-A1, DSW-A2, DSW-B1, and DSW-B2.
<br/>
Access Layer Switches:

  ```
vtp mode client
vtp domain JeremysITLab
vtp version 2
```
<p align="center">
<img src="https://i.imgur.com/Ltl2m4s.png" height="80%" width="80%" alt="VTP Client"/>
</p><br/>
<p align="left"> The command `vtp mode client` changes the VTP permissions for the switch. VTP client switches receive VLAN information from VTP servers and apply it to their configuration. They cannot create, modify, or delete VLANs. The command `vtp domain JeremysITLab` joins the switch to a VTP domain named JeremysITLab, allowing it to share VTP information with other switches in the same domain. The command `vtp version 2` ensures that each switch utilizes the same version of VTP. These commands should be repeated on the access layer switches: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3.</p>
<br/>
<br/>
<h4>Step 5: In Office A, create and name the following VLANs on one of the Distribution switches.</h4>
 Ensure that VTP propagates the change. <br/>

 - <b>VLAN 10: PCs</b> 
 - <b>LAN 20: Phones</b> 
 - <b>VLAN 40: Wi-Fi</b>
 - <b>VLAN 99: Management</b>
<br/>
Office A:
  
  ```
vlan 10
name PCs
vlan 20
name Phones
vlan 40
name wi-fi
vlan 99
name Management
```
<p align="center">
<img src="https://i.imgur.com/Z9oGuoy.png" height="80%" width="80%" alt="name vlans"/>
</p><br/> 
<p align="left">The command `vlan 10` creates a VLAN with that number. The `name PCs` command assigns a name to that VLAN. These commands need to be entered on one of Office A`s VTP server switches, either DSW-A1 or DSW-A2. The switch will then send VTP advertisements to other members of the same VTP domain. VTP advertisements are only shared via trunk ports. Currently, the configurations between the distribution layer and core layers are set as access ports, so VTP information in Office A will not be shared with Office B and vice versa.
<br/>
<br/>
<h4>Step 6: In Office B, create and name the following VLANs on one of the Distribution switches.</h4>
 Ensure that VTP propagates the change. <br/>

 - <b>VLAN 10: PCs</b>
 - <b>VLAN 20: Phones</b>
 - <b>VLAN 30: Servers </b>
 - <b>VLAN 99: Management</b>
 <br/>
Office B:
  
  ```
vlan 10
name PCs
vlan 20
name Phones
vlan 30
name Servers
vlan 99
name Management
```

<p align="center">
<img src="https://i.imgur.com/JIikWct.png" height="80%" width="80%" alt="create/name vlans"/>
</p><br/> 
<p align="left">The above commands create and name each VLAN in Office B. The above command only needs to be entered on one of the VTP servers. VTP advertisements will be sent to each of the other switches within the VTP domain in Office B.
<br/>
<br/>
<h4>Step 7: Configure each Access switch’s access port. </h4>
 LWAPs will not use FlexConnect. <br/>
 PCs in VLAN 10, Phones in VLAN 20. SRV1 in VLAN 30. <br/>
 Manually configure access mode and explicitly disable DTP.<br/>
<br/>

 ASW-A1 and ASW-B1 
  ```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 99
```
<br/>
<p align="left"> Office B does not have a Wi-Fi VLAN because LWAPs only need access to the Management VLAN. They will tunnel information to the WLC in Office A, where the traffic will be assigned to VLAN 40. Therefore, the ports connecting to the LWAPs can be configured as access ports, not trunks.<br/>
<br/>
The `interface f0/1` command enters interface configuration mode. The `switchport mode access` command sets the port as an access port. The `switchport nonegotiate` command disables Dynamic Trunking Protocol (DTP), though this is somewhat redundant since DTP is not used on access ports.<br/>
<br/>
Enter these commands on both ASW-A1 in Office A and ASW-B1 in Office B.</p>
<br/>
 
ASW-A2, ASW-A3 and ASW-B2 
  ```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 10
switchport voice vlan 20
```

<p align="center">
<img src="https://i.imgur.com/mSqu8QO.png" height="80%" width="80%" alt="Access VLANs"/>
</p><br/>
<p align="left"> The above command makes int f0/1 an access port, joins VLAN 10 and VLAN 20, and disables DTP.</p>
<br/>
 
ASW-B3 
  ```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 30
```
<p align="center">
<img src="https://i.imgur.com/gm7U1fX.png" height="80%" width="80%" alt="Access VLAN"/>
</p><br/>
<p align="left"> This image shows the output of `show vlan brief` with interface fa0/1 in Server VLAN.</p>
<br/>
<br/>
<h4>Step 8: Configure ASW-A1’s connection to WLC1.</h4>  
 It must support the Wi-Fi and Management VLANs.<br/>
 The Management VLAN should be untagged.<br/>
 Disable DTP.<br/>
<br/>

 Office A ASW-A1
  ```
interface f0/2
switchport mode trunk
switchport trunk allowed vlan 40,99
switchport trunk native vlan 99
switchport nonegotiate
```
<p align="left"> These commands have already been explained in previous steps. This command is only entered on ASW-A1 in Office A, which connects to the WLC. Interface f0/2 is connected to WLC1 and needs to support multiple VLANs, so it is configured as a trunk. It must allow both the Wi-Fi VLAN and the Management VLAN over the trunk, which is done with the `switchport trunk allowed vlan 40,99 command`. To ensure that the Management VLAN traffic is untagged, VLAN 99 (the Management VLAN) is set as the native VLAN for the trunk. Additionally, to disable DTP, the `switchport nonegotiate` command is issued.</p>
<br/>
<br/>
<h4>Step 9: Administratively disable all unused ports on the Access and Distribution switches.</h4>
<br/>
Access and Distribution Layer Switches:

  ```
show interfaces status
```

  ```
interface range g1/0/6-24,g1/1/3-4
shutdown
```
<p align="center">
<img src="https://i.imgur.com/AsosK2s.png" height="80%" width="80%" alt="Shutdown interfaces"/>
</p><br/>
<p align="left"> Shutting down all unused ports is a good security practice. Switches will automatically allow new physical connections to connect to the network. Administratively shutting down the interface prevents access to the network. This step needs to be repeated on each switch in Office A and Office B: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, ASW-B3, DSW-A1, DSW-B1, and DSW-B2. The `show interfaces status` command should be used on each switch to determine which interfaces to shut down using the interface range command.</p>
<br/>
<br/>
<h3>Part 3 - IP Addresses, Layer-3 EtherChannel, HSRP </h3>
<h4>Step 1: Configure the following IP addresses on R1’s interfaces and enable them:</h4>
<br/>

- <b>G0/0/0: DHCP client</b>
- <b>G0/1/0: DHCP client</b>
- <b>G0/0: 10.0.0.33/30</b>
- <b>G0/1: 10.0.0.37/30</b>
- <b>Loopback0: 10.0.0.76/32</b>

 ```
interface g0/0/0
 ip address dhcp
 no shutdown

interface g0/1/0
 ip address dhcp
 no shutdown

interface g0/0
 ip address 10.0.0.33 255.255.255.252
 no shutdown

interface g0/1
 ip address 10.0.0.37 255.255.255.252
 no shutdown

interface l0
 ip address 10.0.0.76 255.255.255.255

```

<p align="left">At this step, we are beginning to assign static IPv4 addresses. Generally, in your network, it`s good practice to set static IP addresses on network interfaces so that you can have set addresses to diagnose network errors. The external-facing interfaces, G0/0/0 and G0/1/0, use Dynamic Host Configuration Protocol (DHCP) to have their interfaces assigned an address from the Internet Service Provider that they are connected to. Later, Network Address Translation (NAT) will be applied so that internal devices will be able to reach the Internet.<br/>
<br/>
From Global Configuration mode, enter the command `interface [interface #]` to enter interface configuration mode (config-if)#, followed by the command `ip address [ip address] [netmask]` to assign a static IP address. The command `no shutdown` needs to be used on each interface because router interfaces are down by default, unlike switches. The `no shutdown` command is not needed on the loopback interface l0 because the command `int l0` creates a virtual loopback interface, which is active by default. Now, the output of the command `do show ip interface brief` from Global configuration mode should look like this:</p>
<p align="center">
<img src="https://i.imgur.com/SAq9sCL.png" height="80%" width="80%" alt="R1 ipv4 interfaces"/>
</p>
<br/>
<br/>
<h4>Step 2: Enable IPv4 routing on all Core and Distribution switches</h4>
<br/>

 ```
ip routing
```

<p align="center">
<img src="https://i.imgur.com/jdaNlgB.png" height="80%" width="80%" alt="ip routing"/>
</p>
<p align="left">This command needs to be entered on all core and distribution switches: CSW1, CSW2, DSW-A1, DSW-A2, DSW-B1, and DSW-B2. Switches, by default, utilize Layer 2 addressing (MAC addresses). They do not use or need IP addresses nor have an IP routing table. These core and distribution switches are all Layer 3 or multilayer switches. The command `ip routing` allows each of these switches to utilize Layer 3 addressing (IP protocols) while maintaining their Layer 2 functions. Now they can build IP routing tables.</p>
<br/>
<br/>
<h4>Step 3: Create a Layer-3 EtherChannel between CSW1 and CSW2 using a Cisco-proprietary protocol.</h4>
 Both switches should actively try to form an EtherChannel. <br/>
 Configure the following IP addresses:

- <b>CSW1 PortChannel1: 10.0.0.41/30</b>
- <b>CSW2 PortChannel1: 10.0.0.42/30</b>

CSW1:

   ```
interface range g1/0/2-3
no switchport
channel-group 1 mode desirable
interface po1
ip address 10.0.0.41 255.255.255.252
```
CSW2:

  ```
interface range g1/0/2-3
no switchport
channel-group 1 mode desirable
interface po1
ip address 10.0.0.42 255.255.255.252
```
<p align="center">
<img src="https://i.imgur.com/44CGceb.png" height="80%" width="80%" alt="Layer 3 EtherChannel"/>
</p><br/>
<p align="left">The above commands create a Layer 3 EtherChannel using PAgP. The first command groups the interfaces in interface configuration mode. The `no switchport` command disables Layer 2 addressing on the ports, converting them into Layer 3 ports. The `channel-group 1 mode desirable` command configures the EtherChannel using PAgP in desirable mode, which actively tries to form the EtherChannel. The final two lines, starting with `interface po1`, enters configuration mode for the PortChannel 1 interface and assign an IP address so that the EtherChannel can communicate with other Layer 3 network devices.</p>
<br/>
<br/>
</h4>Step 4: Configure the following IP addresses on CSW1:</h4>
  
- <b>G1/0/1: 10.0.0.34/30</b>  
- <b>G1/1/1: 10.0.0.45/30</b>
- <b>G1/1/2: 10.0.0.49/30</b>
- <b>G1/1/3: 10.0.0.53/30</b>  
- <b>G1/1/4: 10.0.0.57/30 </b>  
- <b>Loopback0: 10.0.0.77/32</b>
  
Disable all unused interfaces.

  ```
interface g1/0/1
ip address 10.0.0.34 255.255.255.252
interface g1/1/1
ip address 10.0.0.45 255.255.255.252
interface g1/1/2 
ip address 10.0.0.49 255.255.255.252
interface g1/1/3
ip address 10.0.0.53 255.255.255.252
interface g1/1/4
ip address 10.0.0.57 255.255.255.252
interface L0
ip address 10.0.0.77 255.255.255.255
interface range g1/0/4-24
shutdown
```


<p align="left">These commands assign IP addresses to individual interfaces. The command `show ip int br | include unassigned` was used to identify which interfaces are not in use before entering the interface range. Interfaces `g1/0/2-3` were excluded from the shutdown command as they are used for the EtherChannel. It is good security practice to administratively shut down switch interfaces that are not in use. Additionally, moving these switches to an unused VLAN, enabling port security, and disabling CDP or LLDP are also good security measures for unused ports. The IP Interface brief should now look like this:</p>
<p align="center">
<img src="https://i.imgur.com/KKH5Ieq.png" height="80%" width="80%" alt="ip add"/>
</p><br/>
<br/>
<h4>Step 5: Configure the following IP addresses on CSW2:</h4>
  
- <b>G1/0/1: 10.0.0.38/30</b>  
- <b>G1/1/1: 10.0.0.61/30</b>
- <b>G1/1/2: 10.0.0.65/30</b>
- <b>G1/1/3: 10.0.0.69/30</b>  
- <b>G1/1/4: 10.0.0.73/30 </b>  
- <b>Loopback0: 10.0.0.78/32</b>
  
Disable all unused interfaces.

  ```
interface g1/0/1
ip address 10.0.0.38 255.255.255.252
interface g1/1/1
ip address 10.0.0.61 255.255.255.252
interface g1/1/2 
ip address 10.0.0.65 255.255.255.252
interface g1/1/3
ip address 10.0.0.69 255.255.255.252
int g1/1/4
ip address 10.0.0.73 255.255.252
interface L0
ip address 10.0.0.78 255.255.255.255
interface range g1/0/4-24
shutdown
```


<p align="left">The same commands are applied to CSW1, but with updated IP addresses. It’s good security practice to administratively shut down switch interfaces that are not in use. Additionally, moving these switches to an unused VLAN, enabling port security, and disabling CDP or LLDP are also effective security measures for unused ports. The IP Interface brief should now look like this:</p>
<p align="center">
<img src="https://i.imgur.com/kucSUUK.png" height="80%" width="80%" alt="ip addresses"/>
</p><br/>
<br/>
<h4>Step 6: Configure the following IP addresses on DSW-A1:</h4>
  
- <b>G1/1/1: 10.0.0.46/30</b>
- <b>G1/1/2: 10.0.0.62/30/<b> 
- <b>Loopback0: 10.0.0.7/32</b>

 ```
interface g1/1/1
ip address 10.0.0.46 255.255.255.252
int g1/1/2
ip address 10.0.0.62 255.255.255.252
interface L0
ip address 10.0.0.79 255.255.255.255
```


<p align="left">These are the IP interface configurations for DSW-A1. The IP Interface brief should now look like this:</p>
<p align="center">
<img src="https://i.imgur.com/a7fl291.png" height="80%" width="80%" alt="dsw-A1"/>
</p><br/>
br/>
<h4>Step 7: Configure the following IP addresses on DSW-A2: </h4>
  
- <b>G1/1/1: 10.0.0.50/30</b>
- <b>G1/1/2: 10.0.0.66/30</b>
- <b>Loopback0: 10.0.0.80/32</b>
  

```
interface g1/1/1
ip address 10.0.0.50 255.255.255.252
int g1/1/2
ip address 10.0.0.66 255.255.255.252
interface L0
ip address 10.0.0.80 255.255.255.255
```


<p align="left">Entering these IP addresses is very tedious, but they must be all entered correctly along with correct netmasks. The IP Interface brief should now look like this:</p>
<p align="center">
<img src="https://i.imgur.com/xdU65i8.png" height="80%" width="80%" alt="dsw-a2"/>
</p><br/>
<br/>
<h4>Step 8: Configure the following IP addresses on DSW-B1:</h4>
  
- <b>G1/1/1: 10.0.0.54/30</b>
- <b>G1/1/2: 10.0.0.70/30</b>
- <b>Loopback0: 10.0.0.81/32</b>
  

```
interface g1/1/1
ip address 10.0.0.54 255.255.255.252
int g1/1/2
ip address 10.0.0.70 255.255.255.252
interface L0
ip address 10.0.0.81 255.255.255.255
```

<p align="left"> The IP Interface brief on DSW-B1 should now look like this:</p>
<p align="center">
<img src="https://i.imgur.com/nJ4M81r.png" height="80%" width="80%" alt="dsw-b1"/>
</p>
<br/>
<br/>
<h4>Step 9: Configure the following IP addresses on DSW-B2: </h4>
<br/>
  
- <b>G1/1/1: 10.0.0.58/30</b>
- <b>G1/1/2: 10.0.0.74/30</b>
- <b>Loopback0: 10.0.0.82/32</b>
  

```
interface g1/1/1
ip address 10.0.0.58 255.255.255.252
int g1/1/2
ip address 10.0.0.74 255.255.255.252
interface L0
ip address 10.0.0.82 255.255.255.255
```


<p align="left">I made typos while entering some of these interface configurations, so it's good to check your work by using the command `show ip interface brief` from privileged exec mode or `do show ip interface brief` from global config mode. If any typos were made, use the `interface [# of interface with error]` command, then `no ip address [incorrect address] [netmask]` to delete the incorrect entry. Afterward, reenter the correct IP address. It is also advisable to use the `write` command from privileged exec mode or `do write` from global config mode to save your work. These commands write the running configuration to the startup configuration, ensuring your work is saved.</p>
<br/> 
<p align="left">The IP Interface brief on DSW-B2 should now look like this:</p>
<p align="center">
<img src="https://i.imgur.com/7ut7p1V.png" height="80%" width="80%" alt="dsw-b2"/>
</p><br/>
<br/>
<h4>Step 10: Manually configure SRV1’s IP settings:</h4> 
  
- <b>Default Gateway: 10.5.0.1</b>
- <b>IPv4 Address: 10.5.0.4</b>
- <b>Subnet Mask: 255.255.255.0</b>
  
<p align="left">To enter the configuration setting on end hosts, in packet tracer, you must use the GUI to manually set the configuration.</p>
<br/>
<p align="center">
<img src="https://i.imgur.com/RII6kW3.png" height="80%" width="80%" alt="SRV1"/>
<img src="https://i.imgur.com/CDEaAJB.png" height="80%" width="80%" alt="SRV1"/>
</p><br/>
<br/>
<h4>Step 11: Configure the following management IP addresses on the Access switches (interface VLAN 99):</h4>

- <b>ASW-A1: 10.0.0.4/28</b>
- <b>ASW-A2: 10.0.0.5/28</b>
- <b>ASW-A3: 10.0.0.6/28</b>
- <b>ASW-B1: 10.0.0.20/28</b>
- <b>ASW-B2: 10.0.0.21/28</b>
- <b>ASW-B3: 10.0.0.22/28</b>

And configure the appropriate subnet’s first usable address as the default gateway.<br/>

```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.4 255.255.255.240
```
ASW-A2:

  ```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.5 255.255.255.240
```
ASW-A3:

  ```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.6 255.255.255.240
```
<p align="center">
<img src="https://i.imgur.com/V1VATBg.png" height="80%" width="80%" alt="asw-a2"/>
</p><br/>
ASW-B1:

  ```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.20 255.255.255.240
```
ASW-B2:

  ```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.21 255.255.255.240
```

ASW-B3:

   ```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.22 255.255.255.240

```
<p align="center">
<img src="https://i.imgur.com/n4MbmL8.png" height="80%" width="80%" alt="asw-b2"/>
</p><br/>
<p align="left">These interface configurations create virtual interfaces that allow the management VLAN to communicate with the access layer, Layer 2 switches. The `default-gateway` command is added to the configuration because Layer 2 switches do not have routing tables and thus do not know where to send IP traffic. The `default-gateway` command specifies where the switch should forward IP traffic. Switches in Office A and Office B have different default gateway addresses because each office has a /28 subnet, which provides 16 total addresses. For Office A, 10.0.0.0 is the network address, and 10.0.0.15 is the broadcast address, making 10.0.0.1 the first usable address. For Office B, 10.0.0.16 is the network address, and 10.0.0.31 is the broadcast address, making 10.0.0.17 the first usable address.</p>
<br/>
<br/>
<h4>Step 12: Configure HSRPv2 group 1 for Office A’s Management subnet (VLAN 99). Make DSW-A1 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A1.</h4>
  
  - <b>Subnet: 10.0.0.0/28</b> 
  - <b>VIP: 10.0.0.1</b> 
  - <b>DSW-A1: 10.0.0.2</b> 
  - <b>DSW-A2: 10.0.0.3</b>
  
 DSW-A1:
   ```
interface vlan 99
ip address 10.0.0.2 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1
standby 1 priority 105
standby 1 preempt
```
<p align="center">
<img src="https://i.imgur.com/ghXifzc.png" height="80%" width="80%" alt="asw-b2"/>
</p><br/>

DSW-A2:
   ```
interface vlan 99
ip address 10.0.0.3 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1
```
<p align="center">
<img src="https://i.imgur.com/IpQEeju.png" height="80%" width="80%" alt="asw-b2"/>
</p><br/>
<p align="left">These configurations access VLAN 99, assign an IP address, set Hot Standby Redundancy Protocol version 2 (HSRPv2), and assign the virtual IP address that the switches will share. HSRP versions 1 and 2 are not compatible, so ensure both switches use standby version 2 with the command `standby version 2`. A virtual IP (VIP) address differs from a standard IP address because both switches share the virtual address. In this case, the VIP is set with the `standby 1 ip 10.0.0.1` command in HSRP group 1. Each switch is initially set with a priority of 100. The instructions specify making DSW-A1 the active router by increasing the priority by 5 above the default. The command `standby 1 priority 105` accomplishes this. First-hop redundancy protocols (FHRPs) provide load balancing (if configured) and redundancy in the network by designating a primary and secondary router. If the primary router fails, the secondary router assumes the role of the primary to ensure uninterrupted traffic flow. HSRP, a Cisco proprietary FHRP, elects the active router based on the highest priority, in this case, DSW-A1 with a priority of 105, or the highest IP address. DSW-A2 is the standby router and will take over traffic forwarding if DSW-A1 fails. The command `standby preempt` ensures an active election if DSW-A1 comes back online. Without this command, DSW-A2 would remain the active router if it had taken over during DSW-A1's failure until it failed or another router with preemption enabled joined the standby group.<br/>
<br/>
These switches can function as routers because they are multilayer switches, capable of performing both switching and routing tasks.</p>
<br/>
<br/>
<h4>Step 13. Configure HSRPv2 group 2 for Office A’s PCs subnet (VLAN 10). </h4>
 Make DSW-A1 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A1.<br/> 

- <b>Subnet 10.1.0.0/24</b>
- <b>VIP: 10.1.0.1</b>
- <b>DSW-A1: 10.1.0.2</b>
- <b>DSW-A2: 10.1.0.3</b>

DSW-A1:
   ```
interface vlan 10
ip address 10.1.0.2 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1
standby 2 priority 105
standby 2 preempt
```
<p align="center">
<img src="https://i.imgur.com/WLTTP9R.png" height="80%" width="80%" alt="v10a1"/>
</p><br/>

DSW-A2:
   ```
interface vlan 10
ip address 10.1.0.3 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1
```
<p align="center">
<img src="https://i.imgur.com/hlxsoWA.png" height="80%" width="80%" alt="v10a2"/>
</p><br/>
<p align="left">These commands perform the same functions as step 12 but for the PCs' subnet. The standby group is changed to standby group 2 to enable VLAN separation, redundancy, and failover. This setup also allows for a separate VIP to be assigned specifically for this VLAN. Separating each VLAN into its own standby group ensures that network traffic can continue to flow even if there is a misconfiguration or failure in one VLAN, as each VLAN’s traffic is handled independently of the others.</p>
<br/>
<br/>
<h4>Step 14. Configure HSRPv2 group 3 for Office A’s Phones subnet (VLAN 20). </h4>
 Make DSW-A2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A2.<br/>

- <b>Subnet: 10.2.0.0/24</b> 
- <b>VIP: 10.2.0.1</b>
- <b>DSW-A1: 10.2.0.2</b>
- <b>DSW-A2: 10.2.0.3</b>

DSW-A1:
   ```
interface vlan 20
ip address 10.2.0.2 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1

```
<p align="center">
<img src="https://i.imgur.com/ufOBlry.png" height="80%" width="80%" alt="v20a1"/>
</p>
<br/>

DSW-A2:
   ```
interface vlan 20
ip address 10.2.0.3 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1
standby 3 priority 105
standby 3 preempt
```
<p align="center">
<img src="https://i.imgur.com/ImSobvT.png" height="80%" width="80%" alt="v20a2"/>
</p>
<br/>
<p align="left">These configurations perform the same function but with updated IP addresses, making DSW-A2 the active router and DSW-A1 the standby router. This is an example of how HSRPv2 can provide load balancing. The phones subnet will prioritize sending traffic via DSW-A2 instead of DSW-A1, unlike the management and PCs subnets.</p>
<br/>
<br/>
<h4>Step 15. Configure HSRPv2 group 4 for Office A’s Wi-Fi subnet (VLAN 40). </h4>
 Make DSW-A2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-A2.<br/>

- <b>Subnet: 10.6.0.0/24</b>
- <b>VIP: 10.6.0.1  </b>
- <b>DSW-A1: 10.6.0.2</b>
- <b>DSW-A2: 10.6.0.3</b>

DSW-A1:
   ```
interface vlan 40
ip address 10.6.0.2 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1

```
<p align="center">
<img src="https://i.imgur.com/G6O3f7F.png" height="80%" width="80%" alt="v40a1"/>
</p><br/>

DSW-A2:
   ```
interface vlan 40
ip address 10.6.0.3 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1
standby 4 priority 105
standby 4 preempt
```
<p align="center">
<img src="https://i.imgur.com/iCK2dFA.png" height="80%" width="80%" alt="v40a2"/>
</p><br/>
<p align="left">Wi-Fi traffic (VLAN 40) will utilize DSW-A2 as the primary router and DSW-A1 as the standby router. We can also see that the Wi-Fi subnet will support 254 hosts due to the /24 subnet mask. In contrast, the previous configurations with a /28 subnet mask only allow for 14 hosts per subnet. This demonstrates how HSRP provides redundancy, failover, and load balancing—essential features for a reliable network. Networks are expected to run 24/7, and incorporating redundancy is crucial to achieve this. We have completed the HSRP configuration for Office A.</p>
<br/>
<br/>
<h4>Step 16: Configure HSRPv2 group 1 for Office B’s Management subnet (VLAN 99). </h4>
 Make DSW-B1 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-B1.<br/>

- <b>Subnet: 10.0.0.16/28</b>
- <b>VIP: 10.0.0.17</b>
- <b>DSW-B1: 10.0.0.18</b>
- <b>DSW-B2: 10.0.0.19</b>

DSW-B1:

   ```
interface vlan 99
ip address 10.0.0.18 255.255.255.240
standby version 2
standby 1  ip 10.0.0.17
standby 1 priority 105
standby 1 preempt
```
<p align="center">
<img src="https://i.imgur.com/B28PcDG.png" height="80%" width="80%" alt="v99a1"/>
</p><br/>

DSW-B2:
  ```
interface vlan 99
ip address 10.0.0.19 255.255.255.240
standby version 2
standby 1 ip 10.0.0.17
```
<p align="center">
<img src="https://i.imgur.com/xtX7W5x.png" height="80%" width="80%" alt="v99a2"/>
</p><br/>
<p align="left">These configurations are similar to step 11 but in the Office B network. DSW-B1 Active, DSW-B2 standby for the Management VLAN</p>
<br/>
<br/>
<h4>Step 17. Configure HSRPv2 group 2 for Office B’s PCs subnet (VLAN 10). </h4>
 Make DSW-B1 the active router by increasing its priority to 5 above the default, and enable preemption on DSW-B1.<br/>

- <b>Subnet: 10.3.0.0/24</b>
- <b>VIP: 10.3.0.1</b>
- <b>DSW-B1: 10.3.0.2</b>
- <b>DSW-B2: 10.3.0.3</b>

DSW-B1:

   ```
interface vlan 10
ip address 10.3.0.2 255.255.255.0
standby version 2
standby 2  ip 10.3.0.1
standby 2 priority 105
standby 2 preempt
```
<p align="center">
<img src="https://i.imgur.com/spcqYvd.png" height="80%" width="80%" alt="v10b1"/>
</p><br/>

DSW-B2:
  ```
interface vlan 10
ip address 10.3.0.3 255.255.255.0
standby version 2
standby 2 ip 10.3.0.1
```
<p align="left">These are the same configurations as step 13, but in the 10.3.0.0 subnet, allowing 244 hosts in Office B. DSW-B1 is the active router and DSW-B2 is the standby router in the PCs subnet.</p>
<br/>
<br/>
<h4>Step 18. Configure HSRPv2 group 3 for Office B’s Phones subnet (VLAN 20).</h4>
 Make DSW-B2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-B2.<br/>

- <b>Subnet: 10.4.0.0/24</b>
- <b>VIP: 10.4.0.1</b>
- <b>DSW-B1: 10.4.0.2</b>
- <b>DSW-B2: 10.4.0.3</b>

DSW-B1:

   ```
interface vlan 20
ip address 10.4.0.2 255.255.255.0
standby version 2
standby 3  ip 10.4.0.1
```
<p align="center">
<img src="https://i.imgur.com/vAbl9pt.png" height="80%" width="80%" alt="v20b1"/>
</p><br/>

DSW-B2:
   ```
interface vlan 20
ip address 10.4.0.3 255.255.255.0
standby version 2
standby 3 ip 10.4.0.1
standby 3 priority 105
standby 3 preempt
```
<p align="center">
<img src="https://i.imgur.com/ubUXDkr.png" height="80%" width="80%" alt="v20b2"/>
</p><br/>
<p align="left">Another example of load balancing. The Phones subnet will use DSW-B2 as the active router, and DSW-B1 as the standby router.</p>
<br/>
<br/>
<h4>Step 19. Configure HSRPv2 group 4 for Office B’s Servers subnet (VLAN 30). </h4>
 Make DSW-B2 the Active router by increasing its priority to 5 above the default, and enable preemption on DSW-B2.<br/>

- <b>Subnet: 10.5.0.0/24</b>
- <b>VIP: 10.5.0.1</b>
- <b>DSW-B1 10.5.0.2</b>
- <b>DSW-B2: 10.5.0.3</b>

DSW-B1:

   ```
interface vlan 30
ip address 10.5.0.2 255.255.255.0
standby version 2
standby 4  ip 10.5.0.1
```
<p align="center">
<img src="https://i.imgur.com/e57pk7R.png" height="80%" width="80%" alt="v30b1"/>
</p><br/>

DSW-B2:
  ```
interface vlan 30
ip address 10.5.0.3 255.255.255.0
standby version 2
standby 4 ip 10.5.0.1
standby 4 priority 105
standby 4 preempt
```
<p align="center">
<img src="https://i.imgur.com/DTmnJui.png" height="80%" width="80%" alt="v30b1"/>
</p><br/>
<p align="left">That’s it for configuring HSRPv2 in Office B. The `logging synchronous` command entered earlier is very helpful at this stage. If you mix up the VLAN IP address and the VIP, you will receive numerous Syslog messages indicating misconfigurations. If you encounter these Syslog errors, use the up arrow to repeat the last command, but add `no` at the beginning of the line to undo the last command. Typos can create Syslog messages during HSRP configuration. Configure the HSRP standby router for each VLAN with an STP priority one increment above the lowest priority.</p>
<br/>
<br/>
<h3>Part 4 - Rapid Spanning Tree Protocol</h3>
<h4>Step 1: Configure Rapid PVST+ on all Access and Distribution switches. </h4>
 Configure Rapid PVST+ on all Access and Distribution switches. <br/>
 Configure Rapid PVST+ on all Access and Distribution switches. <br/>
 Ensure that the Root Bridge for each VLAN aligns with the HSRP Active router by configuring the lowest possible STP priority.<br/>
 
   ```
spanning-tree mode rapid-pvst
```
<p align="center">
<img src="https://i.imgur.com/ETI9pf2.png" height="80%" width="80%" alt="v20a1"/>
</p><br/>
<p align="left">This command needs to be entered on each Access and Distribution Switch: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, ASW-B3, DSW-A1, DSW-A2, DSW-B1 and DSW-B2. This enables Rapid Per VLAN Spanning Treet (RPVST+). Rapid PVST+ is a Cisco improvement over Rapid Spanning Tree. Spanning tree protocols intend to prevent Layer 2 Broadcast storms by blocking redundancy ports. Spanning Tree reduces network traffic, to keep networks up and running by eliminating unneeded network traffic. Rapid PVST+ improves on RSTP, by creating individual spanning trees for each VLAN, allowing for more granular control and allows for traffic load balancing by utilizing different paths for different VLANs.</p>

DSW-A1
   ```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,40 priority 4096
```
<p align="center">
<img src="https://i.imgur.com/3eVnSSu.png" height="80%" width="80%" alt="v20a1"/>
</p><br/>

DSW-A2
   ```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,40 priority 4096
```


DSW-B1
   ```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,30 priority 4096
```

DSW-B2
  ```
spanning-tree vlan 10,99 priority 4096
spanning-tree vlan 20,30 priority 0
```
The root bridges(aka switches) for each VLAN receive a priority of 0. In Spanning Tree, the root bridge is elected by the bridge that has the lowest priority, if two or more bridges have the same priority, then the root bridge is determined by the bridge with the lowest Bridge ID(mac address). Since DSW-A1 and DSW-B1 are the active routers in HSRP for VLANs 10,99 they both receive a priority of 0(lowest priority level), with the command `spanning-tree vlan 10,99 priority 0`. Bridge priorities are adjusted in increments of 4096 (2^12) so one increment above the lowest priority is 4096, hence the command `spanning-tree vlan 20,40 priority 4096`. In Office B VLAN 30 is substituted for vlan 40 in the commands.
<br/>
<br/>
<h4>Step 2: Enable PortFast and BPDU Guard on all ports connected to end hosts (including WLC1). Perform the configurations in interface config mode.</h4>

  ```
interface f0/1
spanning-tree portfast
spanning-tree bpduguard enable
```
<p align="center">
<img src="https://i.imgur.com/E32PaFH.png" height="80%" width="80%" alt="v20a1"/>
</p><br/>
This command is repeated on each Access Layer Switch: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3
<br/>
<br/>
<h3>Part 5 – Static and Dynamic Routing</h3>
<h4>Step 1: Configure OSPF on R1 (LAN-facing interfaces) and all Core and Distribution switches (all Layer-3 interfaces)</h4>
a. Use process ID 1 and Area 0 
b. Manually configure each device’s RID to match the loopback interface IP.
c. On switches, use the network command to match the exact IP address of each interface.
d. On R1, enable OSPF in interface config mode.
e. Make sure OSPF is enabled on all loopback interfaces, too. Loopback interfaces should be passive.
f. Each Distribution switch’s SVIs (except the Management VLAN SVI) should be passive, too.
g. Configure all physical connections between OSPF neighbors to use a network type that doesn’t elect a DR/BDR. NOTE: This doesn’t work on the Layer-3 PortChannel interfaces between CSW1/CSW2. Leave them as the default network type.

R1:

  ```
router ospf 1
router-id 10.0.0.76
interface l0
ip ospf 1 area 0
passive-interface l0
interface range g0/0-2
ip ospf 1 area 0
ip ospf network point-to-point
```
<p align="center">
<img src="https://i.imgur.com/rgEGqjF.png" height="80%" width="80%" alt="v20a1"/>
</p><br/>
<p align="left">I used the command `do show ip interface brief` to figure out the loopback interface`s IP address. In the screenshot, you can see I forgot to add the loopback interface when originally completing this lab, so I added the interface here. It`s not necessary to do this over again, In the R1 configurations, Part 3 Step 1, the loopback interface was already added.
<br/>
OSPF(Open Shortest Path First) is a Link-State dynamic IP routing protocol, where all routers in an area (autonomous system) share a map until all routers have the same map of IP routes. To enable OSPF on a router, use the command `router osfp (process id)`, it will also put you in router configuration mode. The command `router-id (IP address)` is used to manually set the router-id. To enable OSPF on a single interface, enter interface configuration with the command `interface (interface #)`, then enter the command `ip ospf (same process id) area (area #)` The command `passive-interface (interface #)` to conserve router resources. This will prevent the interface from sending OSPF Hello messages. It`s used on this loopback interface because we know it cannot create an OSPF neighbor relation, since it`s a virtual interface. Lastly, we use the command `ip ospf network point-to-point`, since there are only 2 routers in the network between R1 and CSW1, and R1 and CSW2. The point-to-point network typer creates full adjacencies and eliminates the DR/ BDR election process. 
<br/>

CSW1:

   ```
router osfp 1
router-id 10.0.0.77
network 10.0.0.77 0.0.0.0 area 0
passive-interface l0
network 10.0.0.41 0.0.0.0 area 0
network 10.0.0.34 0.0.0.0 area 0
network 10.0.0.45 0.0.0.0 area 0
network 10.0.0.49 0.0.0.0 area 0
network 10.0.0.53 0.0.0.0 area 0
network 10.0.0.57 0.0.0.0 area 0
interface range g1/0/1, g1/1/1-4
ip ospf network point-to-point
```
<p align="center">
<img src="https://i.imgur.com/RWOgqUd.png" height="80%" width="80%" alt="ospf core"/>
</p><br/>
CSW2:

   ```
router osfp 1
router-id 10.0.0.78
network 10.0.0.78 0.0.0.0 area 0
passive-interface l0
network 10.0.0.61 0.0.0.0 area 0
network 10.0.0.65 0.0.0.0 area 0
network 10.0.0.69 0.0.0.0 area 0
network 10.0.0.73 0.0.0.0 area 0
network 10.0.0.78 0.0.0.0 area 0
network 10.0.0.38 0.0.0.0 area 0
interface range g1/0/1, g1/1/1-4
ip ospf network point-to-point
```

These commands do the same as above, but instead of enabling OSPF on the individual interface, the `network` command allows you to enable OSFP on an entire network, wild card masks are used instead of netmasks.

DSW-A1:

  ```
router osfp 1
router-id 10.0.0.79
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 40
passive-interface vlan 99
network 10.0.0.46 0.0.0.0 area 0
network 10.0.0.62 0.0.0.0 area 0
network 10.0.0.79 0.0.0.0 area 0
network 10.1.0.2 0.0.0.0 area 0
network 10.2.0.2 0.0.0.0 area 0
network 10.6.0.2 0.0.0.0 area 0
network 10.0.0.2 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```
DSW-A2:

  ```
router osfp 1
router-id 10.0.0.80
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 40
passive-interface vlan 99
network 10.0.0.50 0.0.0.0 area 0
network 10.0.0.66 0.0.0.0 area 0
network 10.0.0.80 0.0.0.0 area 0
network 10.1.0.3 0.0.0.0 area 0
network 10.2.0.3 0.0.0.0 area 0
network 10.6.0.3 0.0.0.0 area 0
network 10.0.0.3 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```

DSW-B1:

  ```
router osfp 1
router-id 10.0.0.81
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30
passive-interface vlan 99
network 10.0.0.54 0.0.0.0 area 0
network 10.0.0.70 0.0.0.0 area 0
network 10.0.0.81 0.0.0.0 area 0
network 10.3.0.2 0.0.0.0 area 0
network 10.4.0.2 0.0.0.0 area 0
network 10.5.0.2 0.0.0.0 area 0
network 10.0.0.18 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```
DSW-B2:

  ```
router osfp 1
router-id 10.0.0.82
passive-interface l0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30
passive-interface vlan 99
network 10.0.0.58 0.0.0.0 area 0
network 10.0.0.74 0.0.0.0 area 0
network 10.0.0.81 0.0.0.0 area 0
network 10.3.0.3 0.0.0.0 area 0
network 10.4.0.3 0.0.0.0 area 0
network 10.5.0.3 0.0.0.0 area 0
network 10.0.0.18 0.0.0.0 area 0
int range g1/1/1-2
ip ospf network point-point
```
<p align="center">
<img src="https://i.imgur.com/wj9D1dT.png" height="80%" width="80%" alt="ospf core"/>
</p><br/>
<p align=`left`> These commands enable OSPF on the router, manually set the router ID, set passive-interfaces, activate OSPF on the network, and make the connection between routers point-to-point networks. OSPF is a layer 3 protocol, so it is not used on the Access Layer Switches.
<br/>
<br/>
<h4>2. Configure one static default route for each of R1’s Internet connections. They should be recursive routes.</h4>
a. Make the route via G0/1/0 a floating static route by configuring an AD value 1 greater than the default.
b. R1 should function as an OSPF ASBR, advertising its default route to other routers in the OSPF domain.

  ```
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2
default-information originate
```
<p align="center">
<img src="https://i.imgur.com/C8vaicz.png" height="80%" width="80%" alt="ospf default"/>
</p><br/>
<p align=`left`>I started by using the `do show ip interface g0/1/0` command to get information about the interface. I can see that the interfaces are part of the /30 subnet, meaning 4 total addresses in the subnet. The network address is 203.0.113.4, broadcast address is 203.0.113.7. int g0/1/0 is 203.0.113.6 so I assumed the ISP`s address is 203.0.113.5, I confirmed by pinging the address. A standard static route has an Administrative Distance (AD) of 1, to create a floating static route, it must have an AD higher than 1, so the 2 is added to the end of the ip route command to make it 1 AD higher than the default static route ad. The same methodology was used to determine the IP address for the other ISP. The command `default-information originate` advertises the default route out of the network to other routers. It also makes R1 an Autonomous System Boundary Router (ASBR) or a router connected to an external network, in this case, the internet.
<br/>
<br/>
<h3>Part 6 – Network Services: DHCP, DNS, NTP, SNMP, Syslog, FTP, SSH, NAT</h3>
<h4>Step 1: Configure the following DHCP pools on R1 to make it serve as the DHCP server for hosts in Offices A and B. Exclude the first ten usable host addresses of each pool; they must not be leased to DHCP clients.</h4>
a. Pool: A-Mgmt
i. Subnet: 10.0.0.0/28
ii. Default gateway: 10.0.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
v. WLC: 10.0.0.7
b. Pool: A-PC
i. Subnet: 10.1.0.0/24
ii. Default gateway: 10.1.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
c. Pool: A-Phone
i. Subnet: 10.2.0.0/24
ii. Default gateway: 10.2.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
d. Pool: B-Mgmt
i. Subnet: 10.0.0.16/28
ii. Default gateway: 10.0.0.17
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
v. WLC: 10.0.0.7
e. Pool: B-PC
i. Subnet: 10.3.0.0/24
ii. Default gateway: 10.3.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
f. Pool: B-Phone
i. Subnet: 10.4.0.0/24
ii. Default gateway: 10.4.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)
g. Pool: Wi-Fi
i. Subnet: 10.6.0.0/24
ii. Default gateway: 10.6.0.1
iii. Domain name: jeremysitlab.com
iv. DNS server: 10.5.0.4 (SRV1)

R1:
   ```
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 10.1.0.1 10.1.0.10
ip dhcp excluded-address 10.2.0.1 10.2.0.10
ip dhcp excluded-address 10.0.0.17 10.0.0.26
ip dhcp excluded-address 10.3.0.1 10.3.0.10
ip dhcp excluded-address 10.4.0.1 10.4.0.10
ip dhcp excluded-address 10.6.0.1 10.6.0.10
```

<p align=`left`>These are the first 10 addresses from each of the subnets. The DHCP-excluded addresses are excluded independent of a DHCP pool, so this is entered directly in Global Config mode. The `ip dhcp excluded address` command is followed by the first ip address in the range, followed by the last address in the range so R1 will not assign the address to clients.</p>

R1:
   ```
ip dhcp pool A-Mgmt
network 10.0.0.0 255.255.255.240
default-router 10.0.0.1
dns-server 10.5.0.4
domain-name jeremysitlab.com
option 43 ip 10.0.0.7

ip dhcp pool A-PC
network 10.1.0.0 255.255.255.0
default-router 10.1.0.1
dns-server 10.5.0.4
domain-name jeremysitlab.com

ip dhcp pool A-Phone
network 10.2.0.0 255.255.255.0
default-router 10.2.0.1
dns-server 10.5.0.4
domain-name jeremysitlab.com
```

<p align="center">
<img src="https://i.imgur.com/cviR7Vv.png" height="80%" width="80%" alt="dhcp office a"/>
</p><br/>
<p align=`left`> The command `ip DHCP pool (pool name)` creates a DHCP pool. The network command tells the pool which addresses are eligible for that specific DHCP pool. The `default-router (ip address)` command sets the default-router, or the router to send traffic to. The `DNS-server (ip address)` command sets the Domain Name System (DNS) server`s IP address for the DHCP pool. For this network, SRV1 is the DNS server for the entire network. So whenever human-readable addresses need to be translated to ipv4 addresses the end hosts will reach out to SRV1 to get the correct translated address. The `domain-name (domain-name)` command sets the domain name for each subnet. The `option 43 ip 10.0.0.7` command sets the IP address for the WLC.</p>

R1:
   ```
ip dhcp pool B-Mgmt
network 10.0.0.16 255.255.255.240
default-router 10.0.0.17
dns-server 10.5.0.4
domain-name jeremysitlab.com
option 43 ip 10.0.0.7

ip dhcp pool B-PC
network 10.3.0.0 255.255.255.0
default-router 10.3.0.1
dns-server 10.5.0.4
domain-name jeremysitlab.com

ip dhcp pool B-Phone
network 10.4.0.0 255.255.255.0
default-router 10.4.0.1
dns-server 10.5.0.4
domain-name jeremysitlab.com

ip dhcp pool WiFi
network 10.6.0.0 255.255.255.0
default-router 10.6.0.1
dns-server 10.5.0.4
domain-name jeremysitlab.com
```
<p align="center">
<img src="https://i.imgur.com/nAPLY0c.png" height="80%" width="80%" alt="dhcp office b"/>
</p><br/>
<br/>
<br/>
<h4>Step 2: Configure the Distribution switches to relay wired DHCP clients’ broadcast messages to R1’s Loopback0 IP address.</h4>

DSW-A1 and DSW-A2
  ```
interface vlan 10
ip helper-address 10.0.0.76
interface vlan 20
ip helper-address 10.0.0.76
interface vlan 40
ip helper-address 10.0.0.76
interface vlan 99
ip helper-address 10.0.0.76
```
<p align="center">
<img src="https://i.imgur.com/Ov6Mw2I.png" height="80%" width="80%" alt="dhcp office b"/>
</p><br/>

DSW-B1 and DSW-B2
   ```
interface vlan 10
ip helper-address 10.0.0.76
interface vlan 20
ip helper-address 10.0.0.76
interface vlan 30
ip helper-address 10.0.0.76
interface vlan 99
ip helper-address 10.0.0.76
```
<p align="center">
<img src="https://i.imgur.com/beG2Sdq.png" height="80%" width="80%" alt="dhcp office b"/>
</p><br/>
<p align=`left`> These commands set a DHCP helper relay to the loopback interface on R1. So Whenever the Switch Virtual Interface (SVI) in each VLAN receives a DHCP message, it will forward the message to R1. This step is essential, or hosts outside of R1`s immediate network would not be able to lease IP addresses via DHCP.
<br/>
<br/>
<h4>Step 3: Configure the following DNS entries on SRV1 </h4>
a. google.com = 172.253.62.100<br/>
b. youtube.com = 152.250.31.93<br/>
c. jeremysitlab.com = 66.235.200.145<br/>
d. www.jeremysitlab.com = jeremysitlab.com<br/>
<p align="center">
<img src="https://i.imgur.com/KafG75a.png" height="80%" width="80%" alt="dns setup"/>
</p><br/>
<p align=`left`> In this step you must use the GUI of SRV1 to set DNS(Domain Name Systems) records. The first 3 records are A Records, which map IPv4 addresses(172.253.62.100) to a human-readable domain name(googgle.com). So whenever the network pings or tries to reach google.com, the DNS server will translate the message destination address to 172.253.62.100. Canonical Record or CRecord maps a domain name to another domain name allowing either address search to have the same results. Even-though in our mind, google.com and www.google.com, are the same web pages, a computer does not think that way. It only knows exactly what it`s told. so on this network www.google.com is not reachable, because there is no C record mapping the two domain names. The below image shows PC 1 pinging certain addresses, confirming that the DNS server is working.<br/>
<p align="center">
<img src="https://i.imgur.com/XAH8HTp.png" height="80%" width="80%" alt="dns confirmation"/>
</p><br/>
<br/>
<h4>Step 4: Configure all routers and switches to use the domain name jeremysitlab.com and use SRV1 as their DNS server.</h4>

   ```
ip domain name jeremysitlab.com
ip name-server 10.5.0.4
```
<p align="center">
<img src="https://i.imgur.com/5Zi1yyW.png" height="80%" width="80%" alt="domain name and dns server"/>
</p><br/>
These commands need to be completed on each network device.
<br/>
<br/>
<h4>Step 5: Configure NTP on R1:</h4>
a. Make R1 a stratum 5 NTP server.<br/>
b. R1 should learn the time from NTP server 216.239.35.0.<br/>

   ```
ntp server 216.239.35.0
ntp master 5
```
<p align="center">
<img src="https://i.imgur.com/Umw9P6Q.png" height="80%" width="80%" alt="ntp setup"/>
</p><br/>
<p align=`left`> NTP (Network Time Protocol) is used for clock synchronization. Manual configuration of clocks on the device leaves room for human error and time rounding errors. NTP is a way to synchronize clocks on a network to each other and to an external NTP server, which usually is a very accurate clock. This eliminates time drift and helps with the accuracy of Syslog messages. The command `ntp server (ip address)` tells the device where the NTP server is located. The command `ntp master (5)` allows this device to be referenced when configuring NTP on the rest of the network, so each device does not need to send traffic to the NTP server. The number 5 sets the stratum, or reference distance to the NTP server, a lower number is considered more accurate.
<br/>
<br/>
<h4>Step 6: All Core, Distribution, and Access switches should use R1’s loopback interface as their NTP server.</h4> 
a. Clients should authenticate R1 using key number 1 and the password ccna

R1:
   ```
ntp authentication-key 1 md5 ccna
ntp trusted-key 1
```

<p align="center">
<img src="https://i.imgur.com/YRNaNJM.png" height="80%" width="80%" alt="ntp authentication-key"/>
</p><br/>
<p align=`left`> R1 is the NTP reference server for the network, so other networks are going to request NTP information from this device. The command `ntp authentication-key 1 md5 ccna` sets an NTP key number (1), using the md5 hashing algorithm, and the password is ccna. The command `ntp trusted-key 1` tells the router to use the key generated in the last step to authenticate NTP clients.</p>


All other network devices:
   ```
ntp authentication-key 1 md5 ccna
ntp trusted-key 1
ntp server 10.0.0.76 key 1
```
Other Network Devices:
<p align="center">
<img src="https://i.imgur.com/Ocjcn86.png" height="80%" width="80%" alt="ntp client"/>
</p><br/>
<p align=`left`> The command `ntp authentication-key 1 md5 ccna` defines an authentication key for NTP. The command `ntp trusted-key 1` specifies which keys are trusted for authenticating NTP servers. The command `ntp server 10.0.0.76 key 1` configures an NTP server for the device to synchronize with, using a specified authentication key. These commands should be repeated on all other network devices, other than R1, to establish an NTP connection.
<br/>
<br/>
<h4>Step 7: Configure the SNMP community string SNMPSTRING on all routers and switches. The string should allow GET messages, but not SET messages.</h4>

   ```
snmp-server community SNMPSTRING ro
```
<p align="center">
<img src="https://i.imgur.com/7cepIH8.png" height="80%" width="80%" alt="ntp client"/>
</p><br/>
<p align=`left`> SNMP (Simple Network Management Protocol) allows for remote monitoring of the device by a SNMP Manager. The command `snmp-server community SNMPSTRING ro` with read-only permissions, meaning the SNMP get messages will be permitted, but SET messages will not.
<br/>
<br/>
<h4>Step 8: Configure Syslog on all routers and switches:</h4>
a. Send Syslog messages to SRV1. Messages of all severity levels should be logged.<br/>
b. Enable logging to the buffer. Reserve 8192 bytes of memory for the buffer.<br/>

  ```
logging 10.5.0.4
logging trap debugging
logging buffered 8192
```
<p align="center">
<img src="https://i.imgur.com/7cepIH8.png" height="80%" width="80%" alt="ntp client"/>
</p><br/>
<p align=`left`> The command `logging 10.5.0.4` sets the Syslog server location to SRV1`s IP address. `logging trap debugging` will now send Syslog `debugging (level 7)` messages via UDP to SRV, this sends all Syslog messages. The command `logging buffered 8192` configures the device to keep log messages in an internal buffer of 8192 bytes. So Syslog messages will be stored locally and on the Syslog server (SRV1). This a good security practice, because if Syslog messages are cleared locally, either by an admin after fixing an error or by someone trying to cover their tracks, a central repository will keep all messages.
<br/>
<br/>
<h4>Step 9: Use FTP on R1 to download a new IOS version from SRV1:</h4>
a. Configure R1’s default FTP credentials: username cisco, password cisco.
b. Use FTP to copy the file c2900-universalk9-mz.SPA.155-3.M4a.bin from SRV1 to R1’s flash drive.
c. Reboot R1 using the new IOS file, and then delete the old one from Flash.

   ```
ip ftp username cisco
ip ftp password cisco
do copy ftp flash
```
>10.5.0.4
>c2900-universalk9-mz.SPA.155-3.M4a.bin
<p align="center">
<img src="https://i.imgur.com/GHo6ur1.png" height="80%" width="80%" alt="ftp file transfer"/>
</p><br/>
<p align=`left`> FTP(File Transfer Protocol) server requires authentication to access the server, unlike TFTP(Trivial File Transfer Protocol). `ip ftp username cisco` and `ip ftp password cisco` set the credentials to authenticate on the FTP server. The command `do copy ftp flash` tells the router to copy(command) a file via ftp(protocol) to flash memory(destination). 10.5.0.4 is the IP address of the FTP server, and then you are asked for a source file name.</p>

  ```
boot system flash c2900-universalk9-mz.SPA.155-3.M4a.bin
do write
```
<p align="center">
<img src="https://i.imgur.com/arpExGN.png" height="80%" width="80%" alt="boot source"/>
</p><br/>
<p align=`left`> The command `boot system flash c2900-universalk9-mz.SPA.155-3.M4a.bin` tells the router to on the next boot use the system file located in flash memory name:c2900-universalk9-mz.SPA.155-3.M4a.bin. Writing this command to the running configuration will not automatically update the software. A system reboot is still required.</p>

  ```
do reload
```
<p align="center">
<img src="https://i.imgur.com/ZBq2D39.png" height="80%" width="80%" alt="software update"/>
</p><br/>
<p align=`left`> System software has now been updated

   ```
delete flash:c2900-universalk9-mz.SPA.151-4.M4a.bin
```
<p align="center">
<img src="https://i.imgur.com/pPG38eb.png" height="80%" width="80%" alt="delete flash"/>
</p><br/>
<p align=`left`> The old IOS version has now been deleted, freeing up system resources.
<br/>
<br/>
<h4>Step 10: Configure SSH for secure remote access on all routers and switches.</h4>
a. Use the largest modulus size for the RSA keys.<br/>
b. Allow SSHv2 connections only.<br/>
c. Create standard ACL 1, only allowing packets sourced from Office A’s PCs subnet. Apply the ACL to all VTY lines to restrict SSH access.<br/>
d. Allow only SSH connections to the VTY lines.<br/>
e. Require users to log in with a local user account when connecting via SSH.<br/>
f. Configure synchronous logging on the VTY lines.<br/>

   ```
crypto key generates rsa
```
>4096
  ```
ip ssh version 2
access-list 1 permit 10.1.0.0 0.0.0.255
line vty 0 15
access-class 1 in
transport input ssh
login local
logging synchronous
```
<p align="center">
<img src="https://i.imgur.com/7aLU41v.png" height="80%" width="80%" alt="ssh config"/>
</p><br/>
<p align=`left`> The command `crypto key generates rsa` creates a pair of RSA (Rivest-Shamir-Adleman) keys, which are used for encryption and decryption. This command enables secure communications over the network, SSH (Secure Shell). 4096 generates the largest key possible. The larger the key, the stronger the encryption. The command `ip ssh version 2`  sets the SSH version to 2. SSH version 2 is more secure and has more features compared to SSH version 1. The command, `access-list 1 permit 10.1.0.0 0.0.0.255` creates an access control list (ACL) numbered 1 that permits traffic from the IP range 10.1.0.0 to 10.1.0.255, all hosts in Office A`s PC subnet. The following command, `line vty 0 15`,  enters line configuration mode for all 16 VTY (Virtual Teletype) lines (0 through 15). VTY lines are used for remote access to the router via protocols such as Telnet or SSH. The command `access-class 1 in` allows hosts in the PC subnet to access the VTY lines. ACL`s have an implicit deny function, so all other sources are denied from accessing the VTY lines. `transport input ssh` allows, only ssh connection, Telnet connections are no longer permitted. The command `login local` requires a local username and password to ssh to R1. This is for Authentication and Accounting purposes. Lastly, the command `logging synchronous` ensures log messages will not disrupt command input, making it easier to manage the device.<br/>
<br/>
The below image confirms that ssh from PC1 to R1 works:
<p align="center">
<img src="https://i.imgur.com/sFC2Lml.png" height="80%" width="80%" alt="ssh config"/>
</p><br/>
<br/>
<h4>Step 11: Configure static NAT on R1 to enable hosts on the Internet to access SRV1 via the IP address 203.0.113.113</h4>

   ```
ip nat source static 10.5.0.4 203.0.113.113
interface range g0/0/0, g0/1/0
ip nat outside
int range g0/0-1
ip nat inside
```
<p align="center">
<img src="https://i.imgur.com/abGUivw.png" height="80%" width="80%" alt="static nat srv1"/>
</p><br/>
<p align=`left`> Network Address Translation (NAT) is used to modify the source and/ or destination IP address. Static nat configures one-to-one mapping of Private IP addresses (all IP addresses statically set so far) to public IP addresses (leased addresses on R1`s G0/0/0 and G0/1/0 interfaces). the command `ip nat source static 10.5.0.4 203.0.113.113` means any traffic originating from 10.5.0.4 will appear to come from 203.0.113.113 when it exits the network, and any traffic destined for 203.0.113.113 will be directed to 10.5.0.4 internally. The next 4 commands set the internal and external sides of the router for NAT translations.</p>
<br/>
<br/>
<h4>Step 12: Configure pool-based dynamic PAT on R1 to enable hosts in the Office A PCs, Office A Phones, Office B PCs, Office B Phones, and Wi-Fi subnets to access the Internet.</h4>
a. Use standard ACL 2 to define the appropriate inside local address ranges in the following order:<br/>
i. Office A PCs: 10.1.0.0/24<br/>
ii. Office A Phones: 10.2.0.0/24<br/>
iii. Office B PCs: 10.3.0.0/24<br/>
iv. Office B Phones: 10.4.0.0/24<br/>
v. Wi-Fi: 10.6.0.0/24<br/>
b. Define a range of inside global addresses called POOL1, specifying the range 203.0.113.200 to 203.0.113.207 with a /29 netmask.<br/>
c. Map ACL 2 to POOL1 and enable PAT.<br/>

   ```
access-list 2 permit 10.1.0.0 0.0.0.255
access-list 2 permit 10.2.0.0 0.0.0.255
access-list 2 permit 10.3.0.0 0.0.0.255
access-list 2 permit 10.4.0.0 0.0.0.255
access-list 2 permit 10.6.0.0 0.0.0.255
ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248
ip nat inside source list 2 pool POOL1 overload
```
<p align="center">
<img src="https://i.imgur.com/r14xE0J.png" height="80%" width="80%" alt="static nat srv1"/>
</p><br/>
<p align=`left`> The first 5 lines create a second ACL that will permit traffic from the required subnets. The command `ip nat pool POOL1 203.0.113.200 203.0.113.207 netmask 255.255.255.248` defines a NAT pool named POOL1 which specifies a range of public IP addresses available for NAT. The command `ip nat inside source list 2 pool POOL1 overload` configures dynamic NAT with PAT (Port Address Translation) using the specified NAT pool, POOL1. This allows internal devices to share a range of public IP addresses for internet access while using port numbers to distinguish between different internal connections. The configuration ensures efficient use of public IP addresses and provides scalable network access.
<br/>
<br/>
<h4>Step 13: Disable CDP on all devices and enable LLDP instead. </h4>
a. Disable LLDP Tx on each Access switch’s access port (F0/1)<br/>

All routers and switches:
   ```
no cdp run
lldp run
```
<p align="center">
<img src="https://i.imgur.com/JLXfrjk.png" height="80%" width="80%" alt="cdp/lldp"/>
</p><br/>
<p align=`left`> The command `no cdp run` disables Cisco Discovery Protocol (CDP) on the device, a Cisco proprietary protocol used by devices to share information about themselves and their directly connected neighbors. The command `lldp run` enables the Link Layer Discovery Protocol (LLDP) on the device. LLDP is a vendor-neutral protocol and can be used across devices from different manufacturers, it functions similarly to CDP.<br/>
<br/>
  
Access Layer Switches:
   ```
interface f0/1
no lldp transmit
```

<p align="center">
<img src="https://i.imgur.com/EclsYBc.png" height="80%" width="80%" alt="no lldp tx"/>
</p><br/>
<p align=`left`> This disables the transmission of LLDP packets on the f0/1 interfaces. This is used to reduce network traffic, enhance security by not disclosing device information, or manage specific interface behavior.
<br/>
<br/>
<h3>Part 7 – Security: ACLs and Layer-2 Security Features</h3>
<h4>Step 1: Configure extended ACL OfficeA_to_OfficeB where appropriate:</h4>
a. Allow ICMP messages from the Office A PCs subnet to the Office B PCs subnet.<br/>
b. Block all other traffic from the Office A PCs subnet to the Office B PCs subnet.<br/>
c. Allow all other traffic.<br/>
d. Apply the ACL according to general best practice for extended ACLs.<br/>

DSW-A1 and DSW-A2
   ```
ip access-list extended OfficeA_to_OfficeB
permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255
deny ip 10.1.0.0 0.0.0.0.255 10.3.0.0 0.0.0.255
permit any any
interface vlan 10
ip access-group OfficeA_to_OfficeB in
```
<p align="center">
<img src="https://i.imgur.com/GCrHEBm.png" height="80%" width="80%" alt="extended acl"/>
</p><br/>
<p align=`left`> This command starts by creating a named extended ACL. Named ACLs, can be named rather than numbered, making them easier to identify and modify. Extended ACLS allows more granular control over network traffic by filtering packets based on various criteria, including source and destination IP addresses, protocol types, and port numbers. The command `permit icmp 10.1.0.0 0.0.0.255 10.3.0.0 0.0.0.255" ensures that only ICMP traffic is allowed between the specified networks (10.1.0.0/24 and 10.3.0.0/24). `deny ip 10.1.0.0 0.0.0.0.255 10.3.0.0 0.0.0.255` denies all other traffic between the two networks. ACLs have an implicit deny, the command `permit any any` allows all other traffic from OfficeA to OfficeB. The `ip access-group OfficeA_to_OfficeB in` sets the ACL to filter traffic when it enters the router on DSW-A1 and DSW-A2. Extended ACLS should be set as close to the source as possible, because of how big of an impact they can have on traffic flows, and the processing power required.
<br/>
<br/>
<h4>Step 2. Configure Port Security on each Access switch`s F0/1 port:</h4>
a. Allow the minimum necessary number of MAC addresses on each port.<br/>
i. SRV1 does not use virtualization, so it uses a single MAC address.<br/>
b. Configure a violation mode that blocks invalid traffic without affecting valid traffic. The switches should send notifications when invalid traffic is detected.<br/>
c. Switches should automatically save the secure MAC addresses they learn to the running-config<br/>

ASW-A1, ASW-B1 and ASW-B3
  ```
interface f0/1
switchport port-security
switchport port-security violation restrict
switchport port-security mac-address sticky
```
<p align="center">
<img src="https://i.imgur.com/v2LssyJ.png" height="80%" width="80%" alt="port security"/>
</p><br/>

ASW-A2, ASW-A3 and ASW-B2
  ```
interface f0/1
switchport port-security
switchport port-security violation restrict
switchport port-security mac-address sticky
switchport port-security maximum 2
```
<p align="center">
<img src="https://i.imgur.com/S0ghFY2.png" height="80%" width="80%" alt="port security phones"/>
</p><br/>
<p align=`left`> Port Security is a Cisco security protocol that allows MAC address limiting, sticky MAC addresses, and different security responses to these violations (shutdown, restrict, and protect). Port security is applied per interface. to enable port security, enter interface config mode with `interface (interface #)` followed by `switchport port-security`. The port security violation mode restricts, drops packets with unknown source addresses, and logs the violation. Violation mode shutdown, makes the interface enter an err-disable state when a packet is sent with an unknown MAC address and generates a Syslog message. Violation mode protect just drops a packet when a packet enters from an unknown MAC address. The command `switchport port-security mac-address sticky` dynamically learns and saves the MAC address of the first packet sent into the port. The default configuration for Sticky MAc addresses only saves one address. The command `switchport port-security maximum 2` is used on ASW-A2, ASW-A3, and ASW-B2 because they also have the Phone vlan, so 2 addresses per connection to interface f0/1 and phone traffic needs to be able to travel across the network.
<br/>
<br/>
<h4>3. Configure DHCP Snooping on all Access switches.</h4>
a. Enable it for all active VLANs in each LAN.<br/>
b. Trust the appropriate ports.<br/>
c. Disable insertion of DHCP Option 82.<br/>
d. Set a DHCP rate limit of 15 pps on active untrusted ports.<br/>
e. Set a higher limit (100 pps) on ASW-A1’s connection to WLC1.<br/>


ASW-A1:
   ```
ip dhcp snooping
ip dhcp snooping vlan 10,20,40,99
no ip dhcp snooping information option
interface range g0/1-2
ip dhcp snooping trust
interface f0/1
ip dhcp snooping limit rate 15
interface f0/2
ip dhcp snooping limit 100
```
<p align="center">
<img src="https://i.imgur.com/Sp8dXav.png" height="80%" width="80%" alt="dhcp snooping"/>
</p><br/>

ASW-A2 and ASW-3:
   ```
ip dhcp snooping
ip dhcp snooping vlan 10,20,40,99
no ip dhcp snooping information option
interface range g0/1-2
ip dhcp snooping trust
int f0/1
ip dhcp limit rate 15
```

Office B:
   ```
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,99
no ip dhcp snooping information option
interface range g0/1-2
ip dhcp snooping trust
int f0/1
ip dhcp limit rate 15
```

<p align=`left`> DHCP snooping is a security feature on switches that filters DHCP messages on untrusted ports. DHCP snooping protects against DHCP poisoning attacks, such as man-in-the-middle attacks and DHCP exhaustion attacks. By default, all ports are untrusted after the command `ip dhcp snooping` has been enabled. Upstream ports should be trusted ports, configured with the command `ip dhcp snooping trust` from interface configuration mode. The switch will no longer monitor DHCP messages entering from these ports, allowing DHCP server response messages to be unfiltered. DHCP client messages will still be filtered.
<br/>
The command `ip dhcp snooping` enables DHCP snooping on a per-VLAN basis. If a DHCP message is received on an untrusted port, the switch inspects the message. If it is a DHCP server message, it discards it, since DHCP server ports should be trusted. If it is a DHCP client Discover/Request message, it checks if the frame`s source MAC address and the DHCP message`s CHADDR (client hardware address) fields match. If there is a match, the message is forwarded; if there is a mismatch, it is discarded. For DHCP Release/Decline messages, the switch checks if the packet`s source IP address and the receiving interface match an entry in the DHCP Snooping Binding Table. If they match, the message is forwarded; if not, it is discarded. When a client successfully leases an IP address from a server, a new entry is created in the DHCP Snooping Binding Table.
<br/>
The command `ip dhcp limit rate` limits the amount of DHCP messages allowed through an interface. This is to stop DHCP exhaustion attacks. The purpose of the `no ip dhcp snooping information option` command is to disable the insertion of the DHCP Option 82 (also known as the DHCP relay agent information option) in DHCP packets that are forwarded by the switch. Some DHCP servers or clients might not support or correctly process DHCP Option 82.</p>
<br/>
<br/>
<h4> Step 4: Configure DAI on all Access switches</h4>
a. Enable it for all active VLANs in each LAN.<br/>
b. Trust the appropriate ports.<br/>
c. Enable all optional validation checks.<br/>

Office A:
   ```
ip arp inspection vlan 10,20,40,99
ip arp inspection validate ip dst-mac src mac
interface range g0/1-2
ip arp inspection trust
```
<p align="center">
<img src="https://i.imgur.com/RBTpYRm.png" height="80%" width="80%" alt="dai office a"/>
</p><br/>

Office B:
   ```
ip arp inspection vlan 10,20,30,99
ip arp inspection validate ip dst-mac src mac
interface range g0/1-2
ip arp inspection trust
```
<p align="center">
<img src="https://i.imgur.com/OznQuR2.png" height="80%" width="80%" alt="dai office b"/>
</p><br/>
<p align=`left`> Dynamic ARP Inspection (DAI) A security feature of switches that is used to filter ARP messages received on untrusted ports, to prevent ARP poisoning. It functions similarly to DHCP snooping and uses some of the features like the HDCP snooping binding table, so DHCP snooping needs to be enabled for DAI to function. DAI inspects the sender MAC and sender IP fields of the ARP messages received on untrusted ports and checks that there is a matching entry in the DHCP snooping binding table. If the ARP message matches the DHCP snooping binding table, the message is forwarded, if not, the message is dropped.
<br/>
The commands are similar, enable ARP inspection on the VLAN (it only needs to be enabled on the VLAN). The command `ip arp inspection validate ip dst-mac src mac` tells the switch to compare all 3 validation types, the IP address, Destination MAc Address, and the source address for the messsage to be forwarded. Then `ip arp inspection trust` is issued on the uplink interfaces to create trusted ports that are no longer inspected.
<br/>
<br/>
<h3>Part 8 - IPv6</h3>
<h4>Step 1: To prepare for a migration to IPv6, enable IPv6 routing and configure IPv6 addresses on R1, CSW1, and CSW2: </h4>
a. R1 G0/0/0: 2001:db8:a::2/64<br/>
b. R1 G0/1/0: 2001:db8:b::2/64<br/>
c. R1 G0/0 and CSW1 G1/0/1: Use prefix 2001:db8:a1::/64 and EUI-64 to generate an interface ID for each interface.<br/>
d. R1 G0/1 and CSW2 G1/0/1: Use prefix 2001:db8:a2::/64 and EUI-64 to generate an interface ID for each interface.<br/>
e. CSW1 Po1 and CSW2 Po1: Enable IPv6 without using the ‘ipv6 address’ command.<br/>

R1:
   ```
ipv6 unicast-routing
interface g0/0/0
ipv6 address 2001:db8:a::2/64
interface g0/1/0
ipv6 address 2001:db8:b::2/64
interface g0/0
ipv6 address 2001:db8:a1::/64 EUI-64
interface g0/1
ipv6 address 2001:db8:a2::/64 EUI-64
```
<p align="center">
<img src="https://i.imgur.com/njD8a90.png" height="80%" width="80%" alt="ipv6 r1"/>
</p><br/>

CSW1:
   ```
ipv6 unicast-routing
interface g1/0/1
ipv6 address 2001:db8:a1::/64 EUI-64
interface po1
ipv6 enable
```
<p align="center">
<img src="https://i.imgur.com/V7V0wrU.png" height="80%" width="80%" alt="ipv6 csw1"/>
</p><br/>

CSW2:
   ```
ipv6 unicast-routing
interface g1/0/1
ipv6 address 2001:db8:a2::/64 EUI-64
interface po1
ipv6 enable
```

<p align=`left`> IPv6 is the predocessor to IPv4. IPv4 does not provide enough unique IP addresses for the world, IPv6 was created to fix this problem. The total number of possible IPv6 addresses is 340,282,366,920,938,463,463,374,607,431,768,211,456 addresses. The primary advantage of IPv6 over IPv4, allowing for a virtually unlimited number of unique IP addresses. IPv6 is still not widely adopted, but preparing a network for the eventual change, it is good to prepare your network for future changes and growth. The command `ipv6 unicast-routing` enables ipv6 on the router. The commands ` ipv6 address 2001:db8:a::2/64` and `ipv6 address 2001:db8:b::2/64` assign a specific IPv6 address with a prefix length of /64. The commands `ipv6 address 2001:db8:a1::/64 EUI-64` and `ipv6 address 2001:db8:a2::/64 EUI-64` generate an IPv6 address. They use the first 64 bits of the prefix length, then use EUI-64, which uses the interface MAC address, adds the hexadecimal FFFE into the middle of the MAC, and inverts the 7th bit of the number. It generates the last 64 bits from the MAC to create a unique 128-bit IPv6 address. The command `ipv6 enable` enables ipv6 on the interface and generates a link-local IPv6 address</p>
<br/>
<br/>
<h4>Step 2: Configure two default static routes on R1:</h4>
a. A recursive route via next hop 2001:db8:a::1.<br/>
b. A fully-specified route via next hop 2001:db8:b::1. Make it a floating route by configuring the AD 1 higher than the default.<br/>

   ```
ipv6 route ::/0 2001:db8:a::1
ipv6 route ::/0 g0/1/0 2001:db8:b::1 2
```
<p align="center">
<img src="https://i.imgur.com/HJmUYrG.png" height="80%" width="80%" alt="ipv6 route"/>
</p><br/>
<p align=`left`> IPv6 default routes use the address ::/0. The first route `ipv6 route ::/0 2001:db8:a::1` will be the only route entered in the routing table. Floating IPv6 routes function the same, they must have an AD higher than the static route, to be used as a back-up and they will not be shown on the routing table. A fully specified route indicates that the interface and next-hip IP address are used.</p>
<br/>
<br/>
<h3>Part 9 – Wireless </h3>
<h4>Step 1:  Access the GUI of WLC1 (https://10.0.0.7) from one of the PCs.</h4>
a. Username: admin<br/>
b. Password: adminPW12<br/>
<p align="center">
<img src="https://i.imgur.com/dBClI7O.png" height="80%" width="80%" alt="Desktop access WLC"/>
</p><br/>
<p align=`left`>PCs have a Web browser program that can be accessed from the desktop. The web browser is not connected to the real internet, but it can access IP addresses within the network. In the URL bar enter the IP address of the WLC (https://10.0.0.7) for a secure connection via HTTPS. Then enter the login credentials to access the GUI(graphical user interface) of WLC1. </p>
<br/>
<br/>
<h4>Step 2: Configure a dynamic interface for the Wi-Fi WLAN (10.6.0.0/24) </h34>
a. Name: Wi-Fi<br/>
b. VLAN: 40<br/>
c. Port number: 1<br/>
d. IP address: .2 of its subnet<br/>
e. Gateway: .1 of its subnet<br/>
f. DHCP server: 10.0.0.76<br/>
<p align="center">
<img src="https://i.imgur.com/YIeSqjJ.png" height="80%" width="80%" alt="interfaces tab"/>
</p><br/>
<p align="center">
<img src="https://i.imgur.com/p5Bdvfi.png" height="80%" width="80%" alt="create interface"/>
</p><br/>
<p align="center">
<img src="https://i.imgur.com/L34RQXw.png" height="80%" width="80%" alt="configure the interface"/>
</p><br/>
<p align=`left`> After logging into the WLC, you click the Controller tab at the top center of the page. Select interfaces on the right menu, then select "New..." to create a new interface. Start by creating an Interface name and entering the VLAN ID used in the network configuration (VLAN40) then click apply in the top right corner. You can now configure the interface, enter the port #, IP address, netmask gateway and Primary DHCP server, to dynamically assign IP addresses on the Wi-Fi subnet. Then click Apply to save changes.</p>
<br/>
<br/>
<h4>Step 3: Configure and enable the following WLAN:</h4>
a. Profile name: Wi-Fi<br/>
b. SSID: Wi-Fi<br/>
c. ID: 1<br/>
d. Status: Enabled<br/>
e. Security: WPA2 Policy with AES encryption, PSK of cisco123<br/>

<p align="center">
<img src="https://i.imgur.com/lEsTlFW.png" height="80%" width="80%" alt="wlan creation"/>
</p><br/>
<p align="center">
<img src="https://i.imgur.com/qINFgYU.png" height="80%" width="80%" alt="wlan assign interface"/>
</p><br/>
<p align="center">
<img src="https://i.imgur.com/udEL4kZ.png" height="80%" width="80%" alt="wlan security 1"/>
</p><br/>
<p align="center">
<img src="https://i.imgur.com/Y5uC3dv.png" height="80%" width="80%" alt="wlan security2"/>
</p><br/>
<p align=`left`> We proceed to create the new wireless LAN by selecting WLANS New in the top menu then New. You can then name the interface profile name, and the SSID(Service Set Identifier) is a unique identifier used in wireless networks to distinguish one wireless network from another. The drop-down will allow you to select the ID #, 1 is fine. then click "Apply" in the top right corner. In the General tab, select enabled, to enable the WLAN, change the interface group to the interface we just created. Select the "Security` Tab.. Select WPA+WPA2 from the Layer 2 security  drop-down. Tick the boxes, WPA2 Policy to enable WPA2, AES to enable AES encryption of all WLAN traffic, and PSK(pre-shared key). Then scroll down, select ASCII from the PSK shared format drop-down, and type the PSK password to access the network (cisco123). Then scroll back up to apply changes by clicking "Apply" in the top right corner.</p>
<br/
<br/
