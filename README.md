# honeypot-gard

## Project Overview
I deployed a public-facing SSH honeypot on a residential network using Cowrie and Docker to collect and analyze the data from it. My goal for this project was to observe and analyze attacker activity on a normal network to better understand how attackers and automated systems behave in the real world.

During the 72-hour collection period, the honeypot received 36,433 events through Cowrie and 5,672 connected SSH sessions from around the world, trying to sign in, execute commands, or download payloads. I collected this data, then processed and visualized it using spreadsheets and charts to allow a deeper understanding and further analysis.

### Expectations

Before collecting any data, I thought about what I expected to collect to see if my learning matches with what happens in the real world. In general, I expected a lot of automated attacks and scanning, as the system was on a residential network I was not likely to receive highly targeted manual intrusion attempts like a large corporation might. I expected lots of port scanning, brute-force credential attacks, and some commands executed if sign-in was successful. 

## Organization
I ran the honeypot on an old HP Pavilion laptop I have with a dedicated Ubuntu Server installation, and remotely managed it using SSH from a separate desktop. To simplify container deployment, I used Docker Compose to write files that spun up containers for me. The main service used is Cowrie, an open-source SSH/Telnet honeypot that emulates a vulnerable Linux server. It allows attackers to sign in to a fake shell where it can record interactions and data about the attacker. 

Cowrie was configured to expose the fake SSH service on port 2222 and Telnet on port 2223, while the home router was configured to port forward traffic from ports 22 and 23 to the honeypot container, ensuring malicious traffic could only access the emulated environment and not the actual host. 

Cowrie generates JSON logs containing data such as usernames, passwords, source IP, commands, and files downloaded. It also stores terminal activity when attackers sign in and any payloads that are downloaded, allowing deeper analysis on attacker behavior. Simultaneously, a tcpdump was being run in the background using tmux to gather raw network data as a PCAP which could be used later to connect network activity with the Cowrie logs.

<img width="604" height="418" alt="image" src="https://github.com/user-attachments/assets/041fbff0-70d2-4a93-9d5a-712cbe7b8cbc" />

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

When analyzing the results of this project, it is important to keep in mind it is fairly limited. The honeypot was only deployed for 72 hours, a relatively short time. This means traffic collected likely does not represent every type of internet activity a network might see, and deploying for a longer time could significantly change the data.

Additionally, Cowrie is an emulated Linux environment, not an actual vulnerable host. This can cause problems, especially with automated attacker workflows if expected files do not exist or behavior is not followed. The goal of the project is to observe what attackers are trying to do, but not really allow them any access to anything.

### Conpot

I was originally going to include Conpot, another open-source honeypot that emulates industrial control systems, but after a 24-hour test period I got no results. I was able to verify a connection from an external network, but got no hits other than my own. I believe this is because attackers would likely not target residential IP ranges with industrial protocols and systems, so I decided not to include it for the final test.


## Results

### Cowrie Logs

During the 72 hour period, the honeypot collected tens of thousands of unsolicited SSH connection attempts from around the world. Cowrie generated JSON formatted logs containing data about these attempts such as credentials, terminal activity, and files downloaded. Using Python scripts I consolidated this data into a CSV spreadsheet to further analyze it using pivot tables and charts. 

Data shows the activity was not evenly distributed across the collection period, actually appearing largely in bursts, suggesting waves of automated scans.

<img width="1134" height="715" alt="image" src="https://github.com/user-attachments/assets/8158eb28-d113-46af-99de-7f76602530be" />



Analysis of the authentication methods used  revealed the automated behavior I was expecting, most frequently seeing credentials like root, admin, and 123456. Large numbers of sessions from different sources all doing the same thing points to botnets or other malware campaigns automatically scanning with dictionaries containing common credentials.

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/363bbec1-7c92-4891-adb2-eb76240532f1" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8d08e7ff-d00c-4545-b97d-e5f7c859c825" />


Additionally, source IP analysis from each connection attempt showed many attempts from a fairly small number of systems. Each one had a source IP, and using geolocation mapping I was able to show the global infrastructure. It is important to keep in mind this geolocation is not entirely accurate, and very likely reflects the locations of compromised systems, virtual private servers, or a distributed botnet, not the true origin of the attackers.

<img width="1237" height="753" alt="image" src="https://github.com/user-attachments/assets/d1adc16b-a25f-461e-83f4-0e6b2a468f26" />


Finally, looking at each event showed the overwhelming majority as a very quick connection attempt to verify authentication or command execution rather than spending time interacting with the fake shell. Again, this aligns with automated opportunistic scanning to find and target publicly exposed and vulnerable SSH services.

<img width="1215" height="757" alt="image" src="https://github.com/user-attachments/assets/d2ea5446-61e3-426c-b655-3ef4ff64fd1e" />


### Fake Shell
Activity within the fake shell environment such as executing commands milliseconds after sign-in still points to mostly automated behavior. There were however a variety of techniques from different actors, showing not all scans are looking for the same things.

#### Reconnaissance

Many scans were simply for reconnaissance, running commands such as cat /proc/cpuinfo or ifconfig to gather data about the target. They can determine processor, available interfaces, current load, or signed-in users on the system. This reconnaissance is often to determine if a found vulnerable host is actually suitable for malware or further attacks.

<img width="1387" height="667" alt="image" src="https://github.com/user-attachments/assets/5704755a-6ecf-4b3c-bc43-1a4d08baca09" />

<img width="1390" height="628" alt="image" src="https://github.com/user-attachments/assets/dbd457f1-79ec-4a41-95a8-373f6a8a10bc" />

<img width="1390" height="141" alt="image" src="https://github.com/user-attachments/assets/ee78b60a-41d0-4c28-a2a5-e92e504386c8" />


#### Persistence

Many attackers attempted to establish persistence by modifying the .ssh/authorized_keys file, injecting their own RSA keys to allow them their own SSH access and maintain remote access after a compromise.

<img width="1389" height="88" alt="image" src="https://github.com/user-attachments/assets/b4d51596-65c1-447f-96a5-f6accddb5a47" />


#### Automated Script Execution

Many sessions attempted to execute local scripts using chained commands, still suggesting automated deployment. The example below tries to use chmod and then run clean.sh and setup.sh, likely trying to remove competing malware or  prepare the target for malware execution. In the emulated Cowrie environment however, these files did not exist, so the commands failed. The attacker likely expected to be able to download those files earlier in the attack chain, but were not able to because of the emulated environment. A human would likely stop after a download fails, but automated systems will try anyway regardless of the errors.

<img width="1383" height="214" alt="image" src="https://github.com/user-attachments/assets/a8a8982c-138a-4835-b598-7c2f421461a4" />

#### Hidden Malware Execution

I found this session particularly interesting, trying to launch a file named sshd in a hidden directory, passing a long list of IP addresses as its arguments. It also attempted to use a utility called nohup, allowing the malware to continue running even after the attacker logged out. This is most consistent with worm type malware, with the list of IP addresses being downstream targets from somewhere else in the attacker’s chain.

<img width="1390" height="88" alt="image" src="https://github.com/user-attachments/assets/bd462cbf-eabd-436f-a60a-c7557182029e" />


#### Other Behavior

Some sessions still had more unique behavior, for example trying to locate Telegram data or SMS artifacts. This could be for credential theft or to search for infrastructure to recruit for fraud or spam campaigns.

<img width="1387" height="196" alt="image" src="https://github.com/user-attachments/assets/5cea4943-d80e-46f1-a03e-523c691ba19d" />


These recorded terminal sessions overall further point to automated scanning behavior with commands being executed instantly and with no delay. Additionally, multiple sessions were basically identical, suggesting the use of scripted malware frameworks or botnets, not targeted human intrusion attempts.

### Malware Analysis

Cowrie was able to capture six attempted downloads after user authentication. Five of them were very small and incomplete, but one was around 30 MB, with the SHA256 hash

b9e643a8e78d2ce745fbe73eb505c8a0cc49842803077809b2267817979d10b0

<img width="1387" height="264" alt="image" src="https://github.com/user-attachments/assets/018cf4f9-2a28-4bdb-9a96-fc6b2bf783bb" />

Searching this on VirusTotal revealed it as an ELF Executable, and was flagged as malicious by 40/62 antivirus engines as well as a negative community reputation, strongly suggesting it is malware. Most of the engines classified it as a coin miner or trojan, and dynamic analysis from Zenbox mapped 20 MITRE ATT&CK techniques to this malware.

<img width="1390" height="697" alt="image" src="https://github.com/user-attachments/assets/50193a3a-a19f-4ab2-9c57-c64c45e7d817" />

## Conclusion

This project showed me how much malicious activity there is in the real world, especially automated scanning and attack systems. In just 72 hours on a residential network, my Cowrie honeypot collected tens of thousands of connection attempts, authentications, and terminal sessions to search for malware or other malicious activity.

Analyzing the logs Cowrie created revealed repeated attempts to use default or common credentials, indicating large-scale automated behavior. They also showed coordinated spikes of activity, suggesting automated campaigns that could use botnets.

Looking at recorded terminal sessions, automated activity is again clear through attackers immediately signing in and executing commands with little or no time between. They attempted to use these commands to do reconnaissance, establish persistence, execute their own custom malware scripts, or perform other malicious activities. 

A malware payload attempted to be downloaded was successfully captured by Cowrie, and after analysis through VirusTotal I discovered it was very likely malicious.

Overall, this project provided a lot of practical insight for me into how much internet traffic is out there and how much malicious activity especially there is targeting every network. Honeypots are a valuable tool for collecting data about attacker behavior without allowing them to cause damage. The data I collected made it clear how automated malware attacks can identify vulnerable systems and interact with them, ultimately deploying a custom malware payload on the system. 
