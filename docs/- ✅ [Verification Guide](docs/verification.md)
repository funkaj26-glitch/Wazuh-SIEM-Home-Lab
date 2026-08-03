# 6. Verification Guide

This guide documents how the completed Wazuh SIEM lab was verified end-to-end — confirming that each core service is running, the Windows endpoint is properly enrolled, and File Integrity Monitoring is actually generating alerts, not just configured on paper.

---

## 6.1 Verify Core Wazuh Services Are Running

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

**Expected result:** All three services report `active (running)`. This confirms the core SIEM stack — event processing (Manager), data storage/search (Indexer), and the web UI (Dashboard) — is fully operational.

![Screenshot: Terminal showing all three services active]

---

## 6.2 Verify the Manager Is Listening for Agents

```bash
sudo ss -tulnp | grep -E '1514|1515'
```

**Expected result:** Both port 1514 (event traffic) and port 1515 (enrollment) appear in the output as `LISTEN`, bound to the Wazuh Manager process. This confirms the Manager is ready to accept agent connections.

---

## 6.3 Verify Manager-Level Health

```bash
sudo /var/ossec/bin/wazuh-control info
```

**Expected result:** Output confirms the installed Wazuh version (v4.12.0) and shows internal Wazuh daemons (e.g., `wazuh-analysisd`, `wazuh-remoted`) in a running state.

---

## 6.4 Verify Network Connectivity Between Server and Endpoint

```bash
ip a
hostname -I
```
```powershell
ipconfig
```

**Expected result:** Both the Ubuntu server and Windows endpoint show IP addresses within the same subnet, confirming Bridged Adapter networking is functioning correctly and both machines can route to each other.

![Screenshot: Side-by-side IP address confirmation between Ubuntu and Windows]

---

## 6.5 Verify the Windows Agent Service Is Running

```powershell
sc query WazuhSvc
```

**Expected result:** The `STATE` field reports `RUNNING`, confirming the Wazuh Agent process is active on the Windows endpoint.

![Screenshot: sc query output showing WazuhSvc RUNNING]

---

## 6.6 Verify Agent Enrollment on the Dashboard

Steps:
1. Log in to the Wazuh Dashboard.
2. Navigate to the **Agents** section.
3. Confirm the Windows endpoint appears in the agent list with a status of **Active**.

**Expected result:** The Windows endpoint is listed by name/ID with an "Active" (green) status indicator, confirming it is successfully communicating with the Manager.

![Screenshot: Wazuh Dashboard Agents page showing Windows endpoint as Active]

---

## 6.7 Verify File Integrity Monitoring (FIM) Is Functioning

This is the most important functional test in the lab, since it confirms the Manager and Agent aren't just "connected," but are actually detecting and reporting real security-relevant events end-to-end.

**Test procedure:**
1. On the Windows endpoint, navigate to the folder configured for FIM monitoring.
2. Create a new test file.
3. Modify the contents of that test file.
4. Delete the test file.
5. On the Wazuh Dashboard, navigate to the **File Integrity Monitoring** module and search for events related to the monitored folder.

**Expected result:** Three distinct alerts appear on the Dashboard corresponding to the file's creation, modification, and deletion — confirming FIM is actively monitoring the configured directory in real time.

![Screenshot: Wazuh Dashboard FIM events showing file added, modified, and deleted alerts]

---

## 6.8 End-to-End Verification Checklist

| # | Check | Command / Action | Expected Result | Status |
|---|---|---|---|---|
| 1 | Manager service running | `systemctl status wazuh-manager` | `active (running)` | ☐ |
| 2 | Indexer service running | `systemctl status wazuh-indexer` | `active (running)` | ☐ |
| 3 | Dashboard service running | `systemctl status wazuh-dashboard` | `active (running)` | ☐ |
| 4 | Manager listening on 1514/1515 | `ss -tulnp \| grep -E '1514\|1515'` | Both ports LISTEN | ☐ |
| 5 | Manager internal health | `wazuh-control info` | Daemons running, correct version | ☐ |
| 6 | Server/endpoint same subnet | `hostname -I` / `ipconfig` | Matching subnet | ☐ |
| 7 | Windows Agent service running | `sc query WazuhSvc` | `RUNNING` | ☐ |
| 8 | Agent shown as Active on Dashboard | Dashboard → Agents | Status: Active | ☐ |
| 9 | FIM create/modify/delete alerts | Dashboard → FIM module | 3 alerts generated | ☐ |

Once every item above is checked, the environment can be considered fully deployed, enrolled, and functionally validated.

---

This concludes the documentation set for the Wazuh SIEM Home Lab. Return to the [README](../README.md) for the full project index.

