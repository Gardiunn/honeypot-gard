# Wazuh SIEM Integration

I expanded my Cowrie SSH honeypot into a small SOC monitoring setup using Wazuh.

## Overview
The original project was about collecting real-world SSH activity from a public honeypot. This next step was about taking the raw Cowrie logs and automatically turning them into SIEM alerts that could be filtered, reviewed, and analyzed more easily than raw logs, spreadsheets, and graphs.

Over about two days, the Wazuh dashboard recorded 8,989 Cowrie-related alerts. This setup allows me to run the honeypot continuously and collect data without having to manually clean and organize each file afterward.
<p align="center">
  <img width="1262" height="635" alt="Screenshot 2026-06-15 141308" src="https://github.com/user-attachments/assets/ea0c8506-762f-4849-9449-0e041dc6002b" />
  Wazuh Dashboard
</p>

## Ingestion and Custom Rules

A few parts took some learning and troubleshooting. First, I configured Wazuh to ingest Cowrie’s raw JSON logs by pointing it to the path of the cowrie.json log file. After confirming Wazuh could read the file, I realized the Cowrie events needed custom detection rules before they could actually be monitored on the dashboard.

I created custom Wazuh rules for each type of Cowrie event I wanted to monitor and mapped several of them to MITRE ATT&CK techniques, including:

T1133 - External Remote Services

T1110 - Brute Force

T1059.004 - Unix Shell

T1105 - Ingress Tool Transfer

T1090 - Proxy

  <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/6efa800c-0ed8-4367-afb7-a2b1bcb5c4e1" />

  <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/d00f10da-36a3-45eb-8fb6-30a96e2f3b61" />
<p align="center">
  Custom Wazuh Detection Rules
</p>

Custom rules: ([`local_rules.xml`](local_rules.xml))



## Alert Filtering
Once the alerts were being generated, I refreshed the Wazuh index fields so the new fields would be recognized and filtered the dashboard by rule.groups: cowrie, which was defined in the custom rules. From there, I could show fields like source IP, username, commands, rule level, and session ID to monitor activity from one place.

<p align="center">
  <img width="1258" height="401" alt="Screenshot 2026-06-15 141414" src="https://github.com/user-attachments/assets/b65205da-0d7f-4a77-8411-c224a39479df" />
  Events in Wazuh
</p>

## Conclusion
This was an important step from simply collecting data toward a more realistic SOC operation with log ingestion, custom detection rules, alert filtering, dashboards, and detection engineering.
