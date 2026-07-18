# 🔍 Log Analysis & SIEM-Based Threat Detection

**Defensive Cybersecurity Project | SOC Operations | InLighnX Global Pvt. Ltd. Internship**

---

## 📌 Project Overview

This project simulates a real-world **SSH brute-force attack** in a controlled virtual lab environment and demonstrates how a SOC analyst detects, investigates, and documents such attacks using **log analysis** and **SIEM (Splunk)** tools.

An attacker machine (Kali Linux) launches automated brute-force attacks against a victim machine (Ubuntu), and all authentication events are captured, analyzed, and visualized through a Splunk dashboard.

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
