# Public SSH Honeypot Analysis

There are no raw logs, files, or malware samples included in this repository, only the writeup and supporting documents for analysis.

## Project Overview
I deployed a public-facing SSH honeypot on a residential network using Cowrie and Docker, my goal for this project was to observe and analyze real activity on a normal network to better understand how attackers and automated systems behave in the real world.

During the 72-hour collection period, the honeypot recorded 36,433 total Cowrie events and 5,672 SSH sessions from sources around the world. These sessions included authentication attempts, command execution, and attempted payload downloads. I then processed the collected data into spreadsheets with tables and visualized the results with charts to identify patterns in attacker behavior.

### Wazuh SIEM Integration

I later expanded this Cowrie SSH honeypot into a small SOC-style monitoring setup using Wazuh. This added log ingestion, custom Cowrie detection rules, MITRE ATT&CK mappings, and a filtered dashboard view for reviewing honeypot alerts.

Writeup here: [Wazuh SIEM Integration](wazuh.md)


## Organization
I ran the honeypot on an old HP Pavilion laptop with a dedicated Ubuntu Server installation, and remotely managed it over SSH from a separate desktop. To simplify deployment and isolate the honeypot from the host, I used Docker Compose to define scripts to start up the containerized services. The main service used is <a href="https://github.com/cowrie/cowrie">Cowrie</a>, an open-source SSH/Telnet honeypot that emulates a vulnerable Linux server. Cowrie shows attackers a fake shell environment which records any interactions they attempt to make. 


<p align="center">
  <img width="1390" height="576" alt="image" src="https://github.com/user-attachments/assets/f47a253f-0e99-4cbd-859d-7b57641f3501" />
  Starting Cowrie and an open SSH session "authenticated" in the fake shell
</p>


Cowrie was configured to expose the fake SSH service on port 2222 and Telnet on port 2223, while the home router was configured to port forward traffic from ports 22 and 23 to the honeypot container, ensuring malicious traffic could only access the emulated environment and not the actual host. 

Cowrie generates structured JSON logs containing data such as usernames, passwords, source IP addresses, commands, and files downloaded. It also stores terminal activity when attackers authenticate and any payloads that are downloaded, allowing deeper analysis on attacker behavior. Simultaneously, tcpdump was being run in the background inside a tmux session to gather raw network data as a PCAP which could be used later to connect network activity with the Cowrie logs.

<p align="center">
  <img width="1000" height="645" alt="image" src="https://github.com/user-attachments/assets/32af24f9-cc93-49ed-93bf-4152037d1a82" />
  Initial Cowrie Events
</p>


Finally, the collected data was put into a spreadsheet to generate pivot tables and charts to visualize the data. This helped me to analyze trends like most common login combinations or to perform geographic mapping on the source IPs.

### Tools Used

Ubuntu Server - Operating system for hosting

SSH - Remote management of honeypot and host

Docker - Easy containerized deployment

Cowrie - Open-source SSH/Telnet honeypot

tcpdump - Packet capture tool to record raw network traffic

Wireshark - Packet capture analysis tool

Google Sheets - Spreadsheets, data processing and visualization

### Limitations

When interpreting the results of this project, it is important to keep in mind its limitations. The honeypot was deployed for 72 hours, a relatively short time. This means traffic collected likely does not represent every type of internet activity a network might see, and deploying for a longer time could produce different patterns and more varied behavior.

Cowrie is also an emulated Linux environment, not an actual vulnerable host. This makes the project safer for me, but can cause attacks to fail because of missing files or inconsistent system behavior. The goal of the project is to observe what attackers are trying to do without allowing them access to the host system.

### Conpot

I was originally going to include <a href="https://github.com/mushorg/conpot">Conpot,</a> another open-source honeypot that emulates industrial control systems, but after a 24-hour test period I got no results. I was able to verify a connection from an external network, but got no hits other than my own. I believe this is because attackers would likely not target residential IP ranges with industrial protocols and systems, so I decided not to include it for the final test.


## Results

### Cowrie Logs

During the 72-hour period, the honeypot collected thousands of unsolicited SSH connection attempts from around the world. Cowrie generated JSON-formatted logs containing data about these attempts such as credentials, terminal activity, and files downloaded. Using a Python script I consolidated this data into a CSV spreadsheet to further analyze it using pivot tables and charts. 

Data shows the activity was not evenly distributed across the collection period, actually appearing largely in bursts, suggesting waves of automated scans.
<p align="center">
  <img width="1134" height="715" alt="image" src="https://github.com/user-attachments/assets/8158eb28-d113-46af-99de-7f76602530be" />
  Connections Per Hour
</p>



Analysis of the authentication methods used  revealed the automated behavior I was expecting, most frequently seeing credentials like root, admin, and 123456. Large numbers of sessions from different sources all doing the same thing points to botnets or other malware campaigns automatically scanning with dictionaries containing common credentials.

<img width="1290" height="388" alt="image" src="https://github.com/user-attachments/assets/b2cefdd5-df89-4da8-8280-3ac9acfdded4" />

<p align="center">
  <img width="1315" height="600" alt="image" src="https://github.com/user-attachments/assets/712009a4-0225-4980-a225-a9ecb3e6fc35" />
  Common Credentials
</p>


Additionally, source IP analysis from each connection attempt showed many attempts from a fairly small number of systems. Each one had a source IP, and using geolocation mapping I was able to show the global infrastructure. It is important to keep in mind this geolocation is not entirely accurate, and very likely reflects the locations of compromised systems, virtual private servers, or a distributed botnet, not the true origin of the attackers.

The top 20 source IP addresses were in the same network range, registered to White Label Services LLC in Istanbul. This appears to be likely hosting infrastructure, so the traffic is very likely not from the company itself. These systems were likely compromised and being used as part of a botnet for automated scanning activity.

<p align="center">
  <img width="952" height="798" alt="image" src="https://github.com/user-attachments/assets/0781cb6c-bd24-4692-b9ff-df240b18c460" />
  <br/>
  The top source IP I found was reported over 1000 times on <a href="https://www.abuseipdb.com/check/31.40.204.166">AbuseIPDB</a> and has a 100% confidence of abuse.
</p>

<img width="1237" height="753" alt="image" src="https://github.com/user-attachments/assets/d1adc16b-a25f-461e-83f4-0e6b2a468f26" />
<p align="center">
  <img width="1066" height="873" alt="image" src="https://github.com/user-attachments/assets/faca0436-8456-45bd-b198-3b6cf9b2bd0b" />
  Top 50 Source IP addresses and their region
</p>





Finally, looking at each event showed the overwhelming majority as a very quick connection attempt to verify authentication or command execution rather than spending time interacting with the fake shell. Again, this aligns with automated opportunistic scanning to find and target publicly exposed and vulnerable SSH services.

<img width="1215" height="757" alt="image" src="https://github.com/user-attachments/assets/d2ea5446-61e3-426c-b655-3ef4ff64fd1e" />


### Fake Shell
Activity within the fake shell environment such as executing commands milliseconds after authentication still points to mostly automated behavior. There were however a variety of techniques from different actors, showing not all scans are looking for the same things.

#### Reconnaissance

Many scans were simply for reconnaissance, running commands such as cat /proc/cpuinfo or ifconfig to gather data about the target. They can determine processor, available interfaces, current load, or signed-in users on the system. This reconnaissance is often to determine if a found vulnerable host is actually suitable for malware or further attacks.

<p align="center">
  <img width="1387" height="667" alt="image" src="https://github.com/user-attachments/assets/5704755a-6ecf-4b3c-bc43-1a4d08baca09" />
  Gather info about the "host"
</p>

<p align="center">
  <img width="1390" height="628" alt="image" src="https://github.com/user-attachments/assets/dbd457f1-79ec-4a41-95a8-373f6a8a10bc" />
  Gather info about the network
</p>

<p align="center">
  <img width="1390" height="141" alt="image" src="https://github.com/user-attachments/assets/ee78b60a-41d0-4c28-a2a5-e92e504386c8" />
  Gather info about signed-in users
</p>


#### Persistence

Many attackers attempted to establish persistence by modifying the .ssh/authorized_keys file, injecting their own RSA keys to allow them their own SSH access and maintain remote access after a compromise.

<img width="1389" height="88" alt="image" src="https://github.com/user-attachments/assets/b4d51596-65c1-447f-96a5-f6accddb5a47" />


#### Automated Script Execution

Many sessions attempted to execute local scripts using chained commands, still suggesting automated deployment. The example below tries to use chmod and then run clean.sh and setup.sh, likely trying to remove competing malware or  prepare the target for malware execution. In the emulated Cowrie environment however, these files did not exist, so the commands failed. The attacker likely expected to be able to download those files earlier in the attack chain, but were not able to because of the emulated environment. A human would likely stop after a download fails, but automated systems will try anyway regardless of the errors.

<img width="1383" height="214" alt="image" src="https://github.com/user-attachments/assets/a8a8982c-138a-4835-b598-7c2f421461a4" />

#### Hidden Malware Execution

I found this session particularly interesting, trying to launch a file named sshd in a hidden directory, passing a long list of IP addresses as its arguments. It also attempted to use a utility called nohup, allowing the malware to continue running even after the attacker logged out. This behavior is most consistent with worm-like malware, with the list of IP addresses being downstream targets from somewhere else in the attacker’s chain.

<img width="1390" height="88" alt="image" src="https://github.com/user-attachments/assets/bd462cbf-eabd-436f-a60a-c7557182029e" />


#### Other Behavior

Some sessions still had more unique behavior, for example trying to locate Telegram data or SMS artifacts. This could be for credential theft or to search for infrastructure to recruit for fraud or spam campaigns.

<img width="1387" height="196" alt="image" src="https://github.com/user-attachments/assets/5cea4943-d80e-46f1-a03e-523c691ba19d" />


These recorded terminal sessions overall further point to automated scanning behavior with commands being executed instantly and with no delay. Additionally, multiple sessions were basically identical, suggesting the use of scripted malware frameworks or botnets, not targeted human intrusion attempts.

### Malware Analysis

Cowrie was able to capture six attempted downloads after user authentication. Five of them were very small and incomplete, but one was around 30 MB, with the SHA256 hash

b9e643a8e78d2ce745fbe73eb505c8a0cc49842803077809b2267817979d10b0

<p align="center">
  <img width="1387" height="264" alt="image" src="https://github.com/user-attachments/assets/018cf4f9-2a28-4bdb-9a96-fc6b2bf783bb" />
  Downloaded Payloads
</p>


Searching this on <a href="https://www.virustotal.com/gui/file/b9e643a8e78d2ce745fbe73eb505c8a0cc49842803077809b2267817979d10b0/detection">VirusTotal</a> revealed it as an ELF Executable, and was flagged as malicious by 40/62 antivirus engines as well as a negative community reputation, strongly suggesting it is malware. Most of the engines classified it as a coin miner or trojan, and dynamic analysis from Zenbox mapped 20 <a href="https://attack.mitre.org/">MITRE ATT&CK</a> techniques to this malware.

<p align="center">
  <img width="1390" height="697" alt="image" src="https://github.com/user-attachments/assets/50193a3a-a19f-4ab2-9c57-c64c45e7d817" />
  VirusTotal Results
</p>

## Conclusion

This project demonstrated how much malicious activity there is in the real world, especially automated scanning and simple attacks. In just 72 hours on a residential network, the Cowrie honeypot collected tens of thousands of connection attempts, authentications, terminal sessions, and other malicious activity.

Analyzing the Cowrie logs revealed repeated attempts to use default or common credentials, indicating large-scale automated behavior. The activity also showed coordinated spikes of activity, suggesting automated campaigns and coordinated infrastructure.

The recorded terminal sessions further support this, with many attackers immediately authenticating and executing commands with little or no time between. These commands attempted to perform reconnaissance, establish persistence, execute their own custom malware scripts, or perform other malicious activities. 

One of the strongest findings was the successful capture of a malware payload. VirusTotal analysis later flagged the file as likely malicious, confirming evidence of real malware deployment, not just automated scanning and credential attacks.

Overall, this project gave me practical insight into how much internet traffic is out there and how much malicious activity there is targeting every network. I got practical experience setting up a honeypot and analyzing real data, and learned more about how honeypots are a valuable tool for collecting data about attacker behavior without allowing them to cause damage. The data made it clear how quickly automated malware attacks can identify vulnerable systems, interact with them, and even try to deploy malware.  
