
<h1> Two Office Network-Configuration---CCNA-Recap</h1>
<h2>Description</h2>
  In this home lab, I am going to configure a three-tier LAN containing two offices using Cisco Packet Tracer for the virtual configuration. I will complete network configuration including IPv4 and IPv6, static routes, VLANs, spanning tree, OSPF, EtherChannel, DHCP, DNS, NAT, SSH, security features, and wireless configurations. Most configurations will be completed in the Cisco CLI, and wireless LAN configuration will be completed in the Cisco GUI. This project is a capstone project to demonstrate knowledge of all CCNA topics. The project was created by Jeremy of JeremysITLab.
  
<br />
<h2>Utilities Used</h2>

- <b>Cisco Packet Tracer</b>


<h2>Program walk-through:</h2>


<h3>Network Topology</h3>
<p align="center">
<img src="https://i.imgur.com/ZlyHjLe.png" height="80%" width="80%" alt="Diagram"/>
</p><br />
<align="left">This is the network topology provided. The devices were connected appropriately but not configured yet. The network is connected to the internet via a dual-homed router at the top center of the image. Below the router, there are two Core Layer, multilayer switches, CSW1 and CSW2. The two core layer switches will connected via a PaGP ether channel for redundancy and load balancing. Below the Core Switches, there are a total of 4 Distribution Layer, Layer 3 Switches, DSW-A1, DSW-A2, DSW-B1, and DSW-B2. The Access Layer consists of 6 layer 2 switches, ASW-A1, ASW-A2, ASW-A3, ASW-B1, ASW-B2, and ASW-B3. The Wireless Lan Controller(WLC1) is connected to ASW-A1. Lightweight Access Point (LWAP1) will be used for guest wireless access in Office A, it's also connected to the access layer via ASW-A1. These Cisco C2960 switches have 24 Feast Ethernet switches, so each switch could have 24 end hosts connected, but do demonstrate the knowledge of connecting an end host, each switch will only have one end host. In Office B, on the right, a second Lightweight Access Point (LWAP2) is used for guest Wi-Fi. LWAP2 is connected to the access layer via ASW-B1. Tehre is one end hsot connected to the access layer via ASW-B2. SRV1, located in the bottom right of Office B, will be used as a Domain Name System(DNS) server, File Transfer Protocol(FTP) server, and a Syslog Log server for the network.
<br />
<p align="center">
<img src="https://i.imgur.com/3oSoI7g.png" height="80%" width="80%" alt="Diagram"/>
</p><br />  
<br />
<h3>Part 1: Initial setup</h3>
Step 1: Configure appropriate hostnames for each router/ switch
<br/>
  
  ```
enable
  configuration terminal
```
  ```
hostname <name>
```
  
<p align="center">
<img src="https://i.imgur.com/XFh94DR.png" height="80%" width="80%" alt="Sysmon logging"/>
  <br />
  <img src="https://i.imgur.com/SDTysiV.png" height="80%" width="80%" alt="Sysmon logging"/>
</p><br />
In packet tracer, you can access a network device's cli by double-clicking the device and then clicking the CLI tab. The hostname is configured from the Global config mode.

#########
<br />
<br />
Step 2: Create a VM in Azure to host my Wazuh SIEM.
<br /> 
<p align="center">
<img src="https://i.imgur.com/IPf1t81.png" height="80%" width="80%" alt="Wazuh Server"/>
</p>
<br />
Step 3: Install Wazuh <br/>
<p align="center">
<img src="https://i.imgur.com/NCrq1Ns.png" height="80%" width="80%" alt="Install Server"/>
</p>
<br />
Step 4: Create a VM in Azure to host TheHive<br/>
<p align="center">
<img src="https://i.imgur.com/PBpxsVa.png" height="80%" width="80%" alt="TheHive Server"/>
</p><br />
<br /> 
Step 5: Install TheHive and dependencies <br/>
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
