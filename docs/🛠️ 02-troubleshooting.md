# 2. Troubleshooting Guide

This document catalogs every real issue encountered while building this lab. Each entry follows the same structure — **Symptom, Diagnosis, Root Cause, Resolution** — to reflect how an incident would actually be worked and documented in a professional SOC or IT environment.

---

## Issue #1 — VirtualBox Networking Misconfigured as NAT

**Symptom:** The Windows endpoint and Ubuntu server could not communicate with each other. Attempts to reach the Manager's IP address from the Windows Agent failed.

**Diagnosis:** Checked network adapter settings in VirtualBox for both virtual machines.

**Root Cause:** Both VMs were initially configured with **NAT (Network Address Translation)** networking. In NAT mode, each VM is placed behind VirtualBox's own internal virtual router and receives an isolated, private IP address that is not directly reachable from other VMs or from the physical network — each machine effectively sits on its own private "island."

**Resolution:** Changed both VMs' network adapters to **Bridged Adapter** mode. This connects each VM directly to the physical network interface of the host machine, allowing both VMs to obtain IP addresses from the same LAN's DHCP server and communicate with each other exactly as separate physical machines would.

```
VirtualBox → VM Settings → Network → Attached to: Bridged Adapter
```

![Screenshot: VirtualBox Network settings comparing NAT vs Bridged Adapter selection]

---

## Issue #2 — Ubuntu and Windows Received Mismatched IP Addressing

**Symptom:** Even after some initial network changes, the two machines appeared to be on different subnets and still could not reliably reach each other.

**Diagnosis:** Ran the following commands to compare actual assigned addressing on both machines:

```bash
ip a
hostname -I
```
```powershell
ipconfig
```

**Root Cause:** This was a direct downstream effect of Issue #1 — until both adapters were fully switched to Bridged mode and the VMs were restarted/re-leased a DHCP address, one machine was still holding an old NAT-assigned address while the other had already picked up a bridged LAN address, so the two were not on the same network segment.

**Resolution:** After confirming both adapters were set to Bridged Adapter and restarting networking on both VMs, `hostname -I` and `ipconfig` confirmed both machines were now on the same subnet, and connectivity was restored.

![Screenshot: Side-by-side terminal output confirming matching subnet on both machines]

---

## Issue #3 — Wazuh Dashboard Login Failed

**Symptom:** After installation completed, attempting to log into the Wazuh Dashboard web interface with default or guessed credentials failed.

**Diagnosis:** Reviewed the installer's terminal output history and Wazuh's documentation on credential generation.

**Root Cause:** The Wazuh all-in-one installer automatically generates strong, random administrator credentials during setup rather than using a static default password. These credentials are only displayed once during installation and are also stored in a local file — if not captured at that moment, they must be retrieved from disk.

**Resolution:** Located and retrieved the generated Dashboard credentials from the installer's output file, then used them to log in successfully. This was noted as an important operational reminder: **always capture generated credentials immediately during any automated installation.**

![Screenshot: Wazuh Dashboard login page]

---

## Issue #4 — Wazuh Indexer Stopped Due to Out-of-Memory (OOM) Error

**Symptom:** The Wazuh Dashboard became unreachable, and log data appeared to stop flowing entirely.

**Diagnosis:**
```bash
sudo systemctl status wazuh-indexer
```
The service status showed the Indexer in a failed/stopped state, and system logs confirmed the process had been terminated by the Linux kernel's OOM killer.

**Root Cause:** The Wazuh Indexer (built on OpenSearch) is memory-intensive by design, since it handles indexing and search operations across all collected security data. The virtual machine's allocated RAM was insufficient for the Indexer's default memory (JVM heap) requirements under load, causing the kernel to terminate the process to protect overall system stability.

**Resolution:** Restarted the Wazuh Indexer service:
```bash
sudo systemctl restart wazuh-indexer
sudo systemctl status wazuh-indexer
```
For a lasting fix rather than a one-time restart, the lab's VM memory allocation was reviewed, since production-grade guidance recommends dedicating adequate RAM specifically for indexer/JVM heap usage. This issue is tracked further in the [Future Improvements](04-future-improvements.md) document as a candidate for resource tuning.

![Screenshot: systemctl status output showing wazuh-indexer active after restart]

---

## Issue #5 — Windows Agent Version Incompatible with Manager

**Symptom:** The Windows Agent (initially installed as v4.14.7) failed to properly register or communicate with the Wazuh Manager (v4.12.0).

**Diagnosis:** Compared version numbers between the installed Agent and the Manager, and reviewed Wazuh's version compatibility expectations.

**Root Cause:** Wazuh Agents and the Wazuh Manager are designed to run on matching or explicitly supported version pairs. The Agent (v4.14.7) was newer than the Manager (v4.12.0), creating a protocol/feature mismatch that prevented proper communication.

**Resolution:** Uninstalled the mismatched v4.14.7 Agent and installed **Wazuh Agent v4.12.0** to match the Manager version exactly. This resolved the communication issue immediately.

> **Takeaway:** In any distributed monitoring architecture, always verify version compatibility between the central server component and its distributed agents before deployment — do not assume "newer is fine."

---

## Issue #6 — Windows Agent Failed to Start Due to XML Error in `ossec.conf`

**Symptom:** After correcting the version mismatch, the Wazuh Agent service still failed to start on the Windows endpoint.

**Diagnosis:**
```powershell
sc query WazuhSvc
```
The service reported a stopped/failed state. Reviewing the Wazuh Agent's local log file pointed to a parsing error in its configuration file, `ossec.conf`.

**Root Cause:** `ossec.conf` is an XML-based configuration file, and a manual edit (made while configuring FIM directory monitoring) had introduced a malformed tag — an unclosed or mismatched XML element — that caused the Agent's configuration parser to fail on startup.

**Resolution:** Reviewed `ossec.conf` line by line, identified the malformed tag, and corrected the XML structure so every opening tag had a matching closing tag. Restarted the service:
```powershell
sc query WazuhSvc
```
The service reported `RUNNING`, confirming the fix was successful.

![Screenshot: Corrected ossec.conf XML snippet with properly closed tags]

> **Takeaway:** XML configuration files are strict about structure — a single unclosed tag can prevent an entire service from starting. Always validate configuration syntax after manual edits, ideally with an XML linter or careful visual review, before restarting the service.

---

## Issue Summary Table

| # | Issue | Root Cause | Resolution |
|---|---|---|---|
| 1 | VMs couldn't communicate | Network mode set to NAT | Switched to Bridged Adapter |
| 2 | Mismatched IP addressing | Downstream effect of Issue #1 | Re-verified addressing after bridging both VMs |
| 3 | Dashboard login failed | Auto-generated credentials not captured | Retrieved generated credentials from installer output |
| 4 | Indexer stopped (OOM) | Insufficient RAM for Indexer/JVM heap | Restarted service; flagged for resource tuning |
| 5 | Agent-Manager version mismatch | Agent v4.14.7 vs Manager v4.12.0 | Installed matching Agent v4.12.0 |
| 6 | Agent service failed to start | Malformed XML in `ossec.conf` | Corrected XML tag structure |

Continue to [Lessons Learned](03-lessons-learned.md) for the broader technical and professional takeaways from this project.
