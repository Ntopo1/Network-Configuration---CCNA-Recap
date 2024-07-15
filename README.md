
<h1> Two Office Network-Configuration---CCNA-Recap</h1>
<h2>Description</h2>
  In this home lab, I am going to configure a three-tier LAN containing two offices using Cisco Packet Tracer for the virtual configuration. I will complete network configuration including IPv4 and IPv6, static routes, VLANs, spanning tree, OSPF, EtherChannel, DHCP, DNS, NAT, SSH, security features, and wireless configurations. Most configurations will be completed in the Cisco CLI, and wireless LAN configuration will be completed in the Cisco GUI. This project is a capstone project to demonstrate knowledge of all CCNA topics. The project was created by Jeremy of JeremysITLab.
  
<br />
<h2>Utilities Used</h2>

- <b>Cisco Packet Tracer</b>


<h2>Program walk-through:</h2>


<h3>Network Topology</h3>
<p align="center">
<img src="https://i.imgur.com/ZlyHjLe.png" height="80%" width="80%" alt="Topology"/>
</p><br />
<align="left">This is the provided network topology. The devices are connected appropriately but not configured yet. The network connects to the internet via a dual-homed router at the top center of the image. Below the router, there are two Core Layer, multilayer switches, CSW1 and CSW2. The two core layer switches will be connected via a PaGP ether channel for redundancy and load balancing. Below the Core Switches, there are a total of 4 Distribution Layer, Layer 3 Switches: DSW-A1, DSW-A2, DSW-B1, and DSW-B2. The Access Layer consists of 6 Layer 2 switches: ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3. The Wireless LAN Controller (WLC1) is connected to ASW-A1. Lightweight Access Point (LWAP1) will be used for guest wireless access in Office A, and it is also connected to the access layer via ASW-A1. These Cisco C2960 switches have 24 Fast Ethernet ports, so each switch could have 24 end hosts connected. However, to demonstrate the knowledge of connecting an end host, each switch will only have one end host. In Office B, on the right, a second Lightweight Access Point (LWAP2) is used for guest Wi-Fi. LWAP2 is connected to the access layer via ASW-B1. There is one end host connected to the access layer via ASW-B2. SRV1, located in the bottom right of Office B, will be used as a Domain Name System (DNS) server, File Transfer Protocol (FTP) server, and a Syslog Log server for the network.
<br />
<p align="center">
<img src="https://i.imgur.com/3oSoI7g.png" height="80%" width="80%" alt="Diagram Layers"/>
</p><br />  
<br />
<h3>Part 1: Initial Setup</h3>
Step 1: Configure appropriate hostnames for each router and switch
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
  <br />
  <img src="https://i.imgur.com/SDTysiV.png" height="80%" width="80%" alt="hostname"/>
</p><br />
<align="left">In packet tracer, you can access a network device's CLI by double-clicking the device and then clicking the CLI tab. The hostname is configured from the Global config mode. The 'enable' command allows the user to access privileged Exec mode indicated by #. The 'configure terminal' command allows the user to access Global Exec mode, indicated by (config#) 
<br />
<br />
Step 2: Configure the enable secret jeremysitlab on each router and switch. Use type 9 hashing if available; otherwise, use type 5.
<br /> 

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
<p align="center">
<img src="https://i.imgur.com/yeapZHJ.png" height="80%" width="80%" alt="level 5 hashing"/>
</p><br />
<align="left">The 'enable secret' and 'enable algorithm-type script secret' commands each provide an added layer of security. These commands set a password for entering privileged EXEC mode, which is stored in a hashed format for security. The enable algorithm-type script secret command, or the type 9 hashing, provides the most secure hashing algorithm on current Cisco devices, but it's not supported on every device type. Type 5 hashing, used by the enable secret command, sets an MD5 hashed password on the device; it's considered less secure than level 9. Using hashing algorithms with passwords is always a good security practice. Even though MD5 hashing is no longer secure, it adds another layer of complexity for someone who may be attempting to tamper with the device. 
<br />
<br />
Step 3: Configure the user account cisco with secret ccna on each router and switch. Use type 9 hashing if available; otherwise, use type 5 <br/>

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
<p align="center">
<img src="https://i.imgur.com/BQp8nD8.png" height="80%" width="80%" alt="u/p level 5"/>
</p><br />
<align="left"> 
These commands create a local user account with a username 'cisco' and an encrypted password 'ccna'. Level 9 hashing of the password is preferred, but it is not supported on all devices. When the command 'username cisco algorithm-type script secret ccna' is not accepted on the device, the next preferred option for this lab is 'username cisco secret ccna'.
<br />
<br />
Step 4: Configure the console line to require login with a local user account. Set a 30-minute inactivity timeout. Enable synchronous logging.<br/>

  ```
line console 0
login local
exec-timeout 30
logging synchronous
```
<p align="center">
<img src="https://i.imgur.com/KpT2OCm.png" height="80%" width="80%" alt="console line config"/>
</p>
<align="left"> 
The 'line console 0' command is used to enter the console line configuration mode on a Cisco device. This mode allows you to configure settings for the console line, which is the physical console port on the device. The 'login local' command allows usernames and passwords configured locally on the device can be used to log in. The 'exec-timeout 30' command signs out the signed-in user after 30 minutes of inactivity. The 'logging synchronous' command ensures that system messages do not interrupt your command input on the consol, logging synchronous reprints your current command line after each log message. The step is repeated on each router and switch
<br /> 
<br /> 

<h3>Part 2: VLANs, Layer-2 EtherChannel</h3>
Step 1: Configure a Layer-2 EtherChannel named PortChannel1 between DSW-A1 and DSW-A2 using a Cisco-proprietary protocol. Both switches should actively try to form an EtherChannel. <br/>

<p align="center">
<img src="https://i.imgur.com/1BG71U6.png" height="80%" width="80%" alt="Dependencies"/>
</p>
<br />
<align="left">TheHive requires Java, Elasticsearch, and Cassandra to run.
</p>
<br />
Step 6: Add an agent to Wazuh<br />
<p align="center">
<img src="https://i.imgur.com/Z7s3173.png" height="80%" width="80%" alt="Agent add"/>
</p>
<br /><br />
Step 7: Configure TheHive<br />
<p align="center">
<img src="https://i.imgur.com/DT1NbRi.png" height="80%" width="80%" alt="Cassandra.yaml"/>
</p>
<br />
<align="left">I started by using nano to configure Cassandra, by editing the file located at /etc/cassandra/cassandra.yaml. I set the listen address, the rpc address, and the seed address to reflect the public IP of TheHive. Then restart the service after the configuration.
<br />
<br /> 
<p align="center">
<img src="https://i.imgur.com/gKyFbAF.png" height="80%" width="80%" alt="Elasticsearch"/>
</p>
<br />
<align="left">I started by using nano to configure Elasticsearch, by editing the file located at /etc/elasticsearch/elasticsearch.yml. I set the cluster name, removed the comment to enable 'node.name', set the network host address to the public IP address of TheHive, and enabled 'cluster.initial_master_node'. Then restarted elasticsearch
<br />
<br />
<p align="center">
<img src="https://i.imgur.com/G7i7W5b.png" height="80%" width="80%" alt="Elasticsearch"/>
</p>
<br />
<align="left">I used nano to configure TheHive, by editing the file located at /etc/thehive/application.conf. I set the hostnames' IP address, the cluster-name, and  the application.baseUrl. Then restarted TheHive
<br />
<br />
Step 8: Configure Wazuh
<br/>
<p align="center">
<img src="https://i.imgur.com/Zt5xbSE.png" height="80%" width="80%" alt="Config Wazuh"/>
</p><br />
I edited the ossec.conf file to ingest Sysmon logs. I then ran Mimkatz on the VM to generate security event logs
<br/>
<br />
<p align="center">
<img src="https://i.imgur.com/1LyLUHR.png" height="80%" width="80%" alt="ossec file"/>
</p><br />
I edited the /var/ossec/etc/ossec.conf file on the server to archive and display .json logs. Then restarted Wazuh.
<br/>
<br />
<p align="center">
<img src="https://i.imgur.com/vTUd5D7.png" height="80%" width="80%" alt="filebeat"/>
</p><br />
I edited the /etc/filebeat/filebeat.yml file on the server to archive logs. Then restarted the filebeats service.
<br/>
<br />
<p align="center">
<img src="https://i.imgur.com/fMDuLPK.png" height="80%" width="80%" alt="archiving"/>
</p><br />
I added an index in Wazuh to display all archived logs even if an alert was not generated.
</p>
<br />
<br />
Step 9: Create a custom rule in Wazuh to generate an alert on Mimikatz usage<br/>
<p align="center">
<img src="https://i.imgur.com/kb5Wy58.png" height="80%" width="80%" alt="Rule"/>
</p><br /> I set the alert to generate when the program's original filename was run, so even if an attacker renamed Mimikatz an alert will still be generated.
<br />
<br />
<h3>Part 3:Connect Shuffle(our SOAR)</h3>
Step 1: Connect Wazuh Alerts
<br /> 
<p align="center">
<img src="https://i.imgur.com/1LyLUHR.png" height="80%" width="80%" alt="Webhook"/>
</p>
<br /> First I connect a webhook to my Wazuh named Wazuh-alerts by editing my Wazuh-manager's ossec file.
<br /> 
<br />
Step 2: Extract File Hash from Alert 
<p align="center">
<img src="https://i.imgur.com/m7yA65E.png" height="80%" width="80%" alt="Regex"/>
</p>
<br /> I connected the Webhook to a Shuffle tool to parse the Wazuh alert for the SHA256 hash of the alert.
<br /> 
<br />
Step 3: Check Reputation Score w/ VirusTotal 
<p align="center">
<img src="https://i.imgur.com/0FLswQz.png" height="80%" width="80%" alt="VT App"/>
</p>
<br /> I connected the Suffle tool to a VirusTotal app to check the reputation score of the parsed SHA256 from the alert
<br /> 
<br />
Step 4: Send Details to TheHive to Create an Alert
<p align="center">
<img src="https://i.imgur.com/CMlfKXN.png" height="80%" width="80%" alt="TheHive App"/>
</p>
<br /> I then connected TheHive to my VirusTotal app inside of Shuffle so I could have cases created inside of TheHive once Wazuh generates an alert.
<br /> 
<br />
Step 5: Send an E-mail to the SOC Analyst to Begin the Investigation 
<p align="center">
<img src="https://i.imgur.com/So95PGA.png" height="80%" width="80%" alt="e-mail app"/>
</p>
<br /> I then connected an E-mail app so Shuffle will email me/ the analyst every time an alert is generated to take action.
<br /> 
<br />
<p align="center">
<img src="https://i.imgur.com/vtKH3Kl.png" height="80%" width="80%" alt="Done"/>
</p>
<br /> The last thing I do is connect the Wazuh manager to workflow to have Wazuh actively respond and block the source IP when a Mimikatz alert is generated. Now we have fully functioning SIEM and SOAR!
