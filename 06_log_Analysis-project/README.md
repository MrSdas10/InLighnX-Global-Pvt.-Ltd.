# 🔍 Log Analysis & SIEM-Based Threat Detection

**Defensive Cybersecurity Project | SOC Operations | InLighnX Global Pvt. Ltd. Internship**

---

## 📌 Project Overview

This project simulates a real-world **SSH brute-force attack** in a controlled virtual lab environment and demonstrates how a SOC analyst detects, investigates, and documents such attacks using **log [...]

An attacker machine (Kali Linux) launches automated brute-force attacks against a victim machine (Ubuntu), and all authentication events are captured, analyzed, and visualized through a Splunk dashboa[...]

---

## 🧪 Lab Environment

| Component | Details |
|-----------|---------|
| **VM1 — Victim** | Ubuntu 22.04 LTS — IP: `192.168.56.101` |
| **VM2 — Attacker** | Kali Linux — IP: `192.168.56.102` |
| **Network** | VirtualBox Host-Only Adapter (`192.168.56.0/24`) |
| **Attack Protocol** | SSH — TCP Port 22 |
| **Virtualization** | VirtualBox |

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Hydra v9.6** | SSH brute-force attack simulation |
| **grep / awk / sort / uniq** | Linux CLI log analysis |
| **Splunk Enterprise (Trial)** | SIEM — log ingestion & visualization |
| **Docker** | Splunk container deployment |
| **/var/log/auth.log** | Primary log source |

---

## 📁 Project Structure

```text
Log-Analysis-Project/
│
├── collected_logs/
│   └── auth_logs.txt               # Raw authentication log from victim machine
│
├── analysis_results/
│   ├── failed_logins.txt           # Filtered failed SSH login entries
│   ├── successful_logins.txt       # Filtered successful login entries
│   └── suspicious_ips.txt          # IP address failure count summary
│
├── screenshots/
│   ├── 01_failed_logins.png
│   ├── 02_count_of_failure_per_ip.png
│   ├── 03_which_usernames_were_targeted.png
│   ├── sucessfull_logins.png
│   ├── 01_ssh_brute_force.png
│   ├── 01-1_ssh_brute_force.png
│   ├── 02_brute_force_with_multiple_user.png
│   ├── 03_Findings_of_password.png
│   ├── 04_login_into_john.png
│   ├── 04_login_into_ram.png
│   └── splunk_dashboard/
│       ├── Screenshot_2026-07-04_224209.png    # Total brute force attempts
│       ├── Screenshot_2026-07-04_224221.png    # Attack timeline
│       ├── Screenshot_2026-07-04_224234.png    # Targeted usernames
│       ├── Screenshot_2026-07-04_224245.png    # Top attacking IPs
│       └── Screenshot_2026-07-04_224254.png    # Failed vs successful logins
│
├── reports/
│   └── Log_Analysis_SIEM_Report.docx
│
└── README.md
```

---

## ⚔️ Attack Summary

### Phase 1 — Single Username Attack

```bash
hydra -l john -P password.txt ssh://192.168.56.101 -t 4 -V
```

- Targeted user `john` with a 207-entry password wordlist
- **Result:** Password `john123` cracked at attempt 188

### Phase 2 — Multi-Username Attack

```bash
hydra -L username.txt -P password.txt ssh://192.168.56.101:22 -t 4
```

- Targeted 8 usernames simultaneously — 1,656 total attempts
- **Result:** 3 accounts cracked — `john:john123`, `ram:ram123`, `sita:sita123`

---

## 🔎 Key Findings

| Finding | Value |
|---------|-------|
| Total failed login attempts | **764** |
| Primary attacker IP | **192.168.56.102** |
| Accounts compromised | **3** (`john`, `ram`, `sita`) |
| Attack duration | **~14 minutes** (10:19 – 10:33 AM, July 1, 2026) |
| Attack rate | **~60 attempts/minute** |
| Usernames targeted | `john`, `root`, `ram`, `sita`, `admin`, `lakhnam`, `ubuntu` |

---

## 📊 Log Analysis Commands Used

```bash
# View full authentication log
sudo cat /var/log/auth.log

# Extract all failed login attempts
sudo grep "Failed password" /var/log/auth.log > failed_logins.txt

# Count failed attempts per IP address
sudo grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn > suspicious_ips.txt

# Identify targeted usernames
sudo grep "Failed password" /var/log/auth.log | awk '{print $9}' | sort | uniq -c | sort -rn

# Extract successful logins
sudo grep "Accepted password" /var/log/auth.log > successful_logins.txt
```

---

## 📈 Splunk SIEM Dashboard

Splunk was deployed via Docker and the auth log was ingested for visualization.

```powershell
# Run Splunk via Docker
docker run -d `
  --name splunk `
  -p 8000:8000 `
  -e SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com `
  -e SPLUNK_START_ARGS=--accept-license `
  -e SPLUNK_PASSWORD='Admin1234!' `
  splunk/splunk:latest
```

**Dashboard:** `SSH Brute Force Investigation`  
**Access:** `http://localhost:8000`

### SPL Queries Used

```spl
# Total brute force attempts
source="auth_logs.txt" "Failed password" | stats count as "Total Failed Attempts"

# Attack timeline
source="auth_logs.txt" "Failed password" | timechart count

# Top attacking IPs
source="auth_logs.txt" "Failed password" | rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" | stats count by src_ip | sort -count

# Targeted usernames
source="auth_logs.txt" "Failed password" | rex "Failed password for (?<username>\w+)" | stats count by username

# Failed vs successful logins
source="auth_logs.txt" ("Failed password" OR "Accepted password")
| eval status=if(searchmatch("Failed password"), "Failed", "Successful")
| stats count by status
```

---

## 🚨 Indicators of Compromise (IoCs)

| Indicator | Value | Severity |
|-----------|-------|----------|
| Attacker IP | `192.168.56.102` | 🔴 HIGH |
| Failed login volume | 764 attempts in 14 minutes | 🔴 HIGH |
| Attack tool | Hydra v9.6 | 🔴 HIGH |
| Compromised account | `john` — password: `john123` | 🔴 HIGH |
| Compromised account | `ram` — password: `ram123` | 🔴 HIGH |
| Compromised account | `sita` — password: `sita123` | 🔴 HIGH |
| Attack protocol | SSH — TCP Port 22 | 🟡 MEDIUM |

---

## 🛡️ Security Recommendations

1. **Install Fail2Ban** — auto-block IPs after 5 failed attempts.
2. **Disable password authentication** — enforce SSH key-based authentication only.
3. **Enforce strong password policy** — minimum 12 characters with complexity.
4. **Disable root SSH login** — set `PermitRootLogin no`.
5. **Restrict SSH with firewall rules** — allow trusted IPs only.
6. **Change default SSH port** — move from 22 to a non-standard port.
7. **Set real-time SIEM alerts** — alert when failures exceed 10/minute.
8. **Enable MFA** — use two-factor authentication (e.g., Google Authenticator).

---

## 📚 Skills Demonstrated

- Linux log analysis (`grep`, `awk`, `sort`, `uniq`)
- SSH brute-force simulation with Hydra
- SIEM log ingestion and dashboard creation (Splunk)
- Indicator of Compromise (IoC) identification
- SOC-style security investigation and reporting
- Docker-based tool deployment

---
