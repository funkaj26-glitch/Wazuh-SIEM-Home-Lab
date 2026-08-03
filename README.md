# 🛡️ Wazuh SIEM Home Lab

## 📌 Project Overview

This project demonstrates the deployment of a Security Information and Event Management (SIEM) Home Lab using Wazuh.

The lab was built using Ubuntu 26.04 LTS running inside Oracle VirtualBox as the Wazuh Server and Windows 11 as the monitored endpoint. The Windows endpoint is configured with the Wazuh Agent to collect security events and send them securely to the Wazuh Manager.

The project also documents the troubleshooting process involved in configuring the environment, resolving networking issues, fixing agent compatibility problems, and correcting configuration errors.

---

## 🎯 Objectives

- Deploy Wazuh SIEM from scratch
- Configure a Windows endpoint
- Install and configure the Wazuh Agent
- Monitor Windows Security Events
- Configure File Integrity Monitoring (FIM)
- Understand Wazuh Manager-Agent communication
- Troubleshoot deployment issues

---

## 🖥️ Lab Environment

| Component | Technology |
|------------|------------|
| Operating System | Ubuntu 26.04 LTS |
| Virtualization | Oracle VirtualBox |
| SIEM Platform | Wazuh 4.12 |
| Endpoint | Windows 11 |
| Agent | Wazuh Agent 4.12 |

---

## 🌐 Network Configuration

- VirtualBox Network Mode: Bridged Adapter
- Ubuntu IP Address: 192.168.1.10
- Windows Host: Same local network
- Communication Ports:
  - TCP 1514
  - TCP 1515

---

## 🔍 Features Implemented

- Wazuh Manager Installation
- Wazuh Dashboard
- Wazuh Indexer
- Windows Agent Enrollment
- Windows Event Collection
- File Integrity Monitoring (FIM)
- Security Configuration Assessment (SCA)

---

## 🛠️ Challenges Faced

During the implementation, multiple real-world issues were encountered and resolved.

- NAT vs Bridged networking
- Windows unable to reach Ubuntu
- Manager and Agent version mismatch
- Wazuh Agent connection issues
- XML configuration error in ossec.conf
- Windows service troubleshooting

---

## ✅ Outcome

Successfully deployed a working Wazuh SIEM Home Lab capable of monitoring a Windows endpoint, collecting security logs, and performing File Integrity Monitoring.

---

## 📚 Skills Demonstrated

- Linux Administration
- Windows Administration
- Networking
- SIEM Deployment
- Log Analysis
- Endpoint Monitoring
- File Integrity Monitoring
- Troubleshooting
