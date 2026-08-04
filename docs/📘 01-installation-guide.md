# 1. Installation Guide

This guide walks through how the Wazuh SIEM stack was deployed on Ubuntu and how the Windows 11 endpoint was enrolled as a monitored agent. Each command is explained so the reasoning behind every step is clear, not just the syntax.

---

## 1.1 Lab Environment

| Component | Specification |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Server VM OS | Ubuntu 26.04 LTS |
| Endpoint VM OS | Windows 11 Pro |
| Network Mode | Bridged Adapter (both VMs) |
| SIEM Platform | Wazuh v4.12.0 (Manager, Indexer, Dashboard) |
| Agent | Wazuh Agent v4.12.0 |

**Why Bridged Adapter instead of NAT:** Bridged networking places each virtual machine directly on the physical LAN, giving it its own IP address from the router's DHCP pool. This mirrors how a real server and a real endpoint would communicate on a corporate network, and it allows the Wazuh Manager and the Windows Agent to reach each other directly over ports 1514/1515 without additional port forwarding rules that NAT mode would otherwise require.

<img width="807" height="509" alt="Network setiings" src="https://github.com/user-attachments/assets/fcda950b-fa01-44ba-99f9-dca9128ad60c" />
---

## 1.2 Preparing the Ubuntu Server

Before installing Wazuh, the system package index was refreshed and `curl` was installed, since it is required to retrieve the Wazuh GPG signing key and installation script.

```bash
sudo apt update
sudo apt install curl -y
```

- `sudo apt update` — Refreshes the local list of available packages and their versions from Ubuntu's configured repositories. This ensures the system installs the latest available package metadata before adding new software.
- `sudo apt install curl -y` — Installs `curl`, a command-line tool used to transfer data from or to a server. The `-y` flag automatically confirms the installation prompt.

---

## 1.3 Adding the Wazuh Repository Key

```bash
curl -fsSL https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg
```

This command downloads Wazuh's public GPG signing key and converts it into a binary keyring format that APT can use to verify the authenticity of Wazuh packages before installation.

- `curl -fsSL <url>` — Downloads the key silently (`-s`), following redirects (`-L`), and fails cleanly on server errors (`-f`) without printing progress output (combined effect of the flags).
- `| sudo gpg --dearmor -o ...` — Pipes the downloaded key into `gpg --dearmor`, which converts the human-readable ("ASCII-armored") key into the binary format APT expects, then saves it to the specified keyring path.

This step matters because it establishes a chain of trust — Ubuntu will refuse to install packages from the Wazuh repository unless it can verify their signature against this key.

---

## 1.4 Installing the Wazuh Stack

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash ./wazuh-install.sh -a -i
```

- `curl -sO <url>` — Downloads the official Wazuh all-in-one installation script and saves it locally using its original filename (`-O`), suppressing progress output (`-s`).
- `sudo bash ./wazuh-install.sh -a -i` — Executes the installer with two flags:
  - `-a` installs all core components (Wazuh Manager, Wazuh Indexer, and Wazuh Dashboard) on the same host, which is appropriate for a single-server home lab.
  - `-i` runs the installation in interactive/individual mode, allowing visibility into each stage of the process rather than a fully silent install.

This single script handles dependency resolution, service creation, and initial configuration for all three components, which significantly simplifies a process that would otherwise require manually installing and wiring together three separate services.

<img width="1853" height="982" alt="Wazuh Dashboard Status" src="https://github.com/user-attachments/assets/af2836b7-c98c-4794-bbb2-77e50ce33ae1" />
> **Note:** During this stage, the installer generates and displays the initial admin credentials for the Wazuh Dashboard. These credentials must be captured immediately, as retrieving them later requires locating the password file on disk (covered in the [Troubleshooting Guide](02-troubleshooting-guide.md)).

---

## 1.5 Verifying the Core Services

Once installation completes, each Wazuh service was checked individually to confirm it was active and running before moving on to agent enrollment.

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

`systemctl status <service>` queries `systemd` (Ubuntu's service manager) for the current run state of a service, showing whether it is `active (running)`, `failed`, or `inactive`, along with recent log entries. Checking all three services independently — rather than assuming a successful install means everything is running — is a good operational habit, since it was this exact check that revealed the Indexer had stopped due to a memory issue (see Troubleshooting Guide, Issue #4).

<img width="1853" height="982" alt="Wazuh Manager" src="https://github.com/user-attachments/assets/9e554789-9961-4051-acd3-e788e42d240b" />
---

## 1.6 Confirming Network Listeners

```bash
sudo ss -tulnp | grep -E '1514|1515'
```

This command confirms the Wazuh Manager is actively listening for agent connections on its two key ports:

- **Port 1514** — Used for the ongoing transmission of events and logs from enrolled agents to the Manager.
- **Port 1515** — Used specifically during the agent enrollment/registration handshake.

Breaking down the command:
- `ss -tulnp` — Lists active network sockets: TCP (`-t`) and UDP (`-u`) listening (`-l`) ports, showing numeric addresses (`-n`) and the process holding each port (`-p`).
- `grep -E '1514|1515'` — Filters the output to only display lines containing either port number, using extended regular expression matching (`-E`).

Confirming these ports are open and bound to the Manager process before attempting enrollment saves time — if enrollment fails later, this check rules out "the Manager isn't listening" as the cause.

---

## 1.7 Checking Overall Manager Health

```bash
sudo /var/ossec/bin/wazuh-control info
```

This is a Wazuh-specific diagnostic utility (separate from `systemctl`) that reports the installed Wazuh version and the internal status of Wazuh's own sub-processes (such as `wazuh-analysisd`, `wazuh-remoted`, and `wazuh-execd`). It is useful because a service can appear "active" at the `systemd` level while an internal Wazuh component is still misbehaving — this command gives visibility one layer deeper.

---

## 1.8 Identifying Network Addressing

To connect the Windows endpoint to the correct Manager address, the actual IP addresses of both machines had to be confirmed — this became especially important after the earlier networking misconfiguration (see Troubleshooting Guide, Issue #1).

**On Ubuntu:**
```bash
ip a
hostname -I
```
- `ip a` — Displays all network interfaces and their assigned IP addresses in detail.
- `hostname -I` — A quicker, single-line alternative that prints just the IP address(es) assigned to the host.

**On Windows:**
```powershell
ipconfig
```
- `ipconfig` — Displays the IP configuration for all network adapters on the Windows machine, used here to confirm the endpoint had received a valid LAN IP address from the same network segment as the Ubuntu server.

![Screenshot: Terminal output of hostname -I next to Windows ipconfig output, showing matching subnet]

---

## 1.9 Enrolling the Windows Agent

### Step 1 — Register the Agent on the Manager

```bash
sudo /var/ossec/bin/manage_agents
```

This launches an interactive Wazuh utility used to add, list, and remove agents directly from the Manager side. It was used here to create a new agent entry for the Windows endpoint (assigning it a name and ID) and to generate the authentication key required for that endpoint to enroll.

### Step 2 — Install the Matching Agent Version on Windows

The Wazuh Agent installer (matching version **v4.12.0**, to align with the Manager) was installed on the Windows 11 Pro endpoint. Version alignment between Agent and Manager is critical — this is expanded on in the Troubleshooting Guide, since a version mismatch was one of the real issues encountered during this build.

### Step 3 — Configure and Register the Agent

The agent's configuration file (`ossec.conf`) was updated to point to the Ubuntu server's IP address as its Manager, and the generated key from Step 1 was used to complete enrollment.

### Step 4 — Confirm the Windows Service Is Running

```powershell
sc query WazuhSvc
```

`sc query` interacts with the Windows Service Control Manager to report the current state of a named service — in this case `WazuhSvc`, the Windows service that runs the Wazuh Agent. A `RUNNING` state confirms the agent process is active on the endpoint.

<img width="637" height="174" alt="Query" src="https://github.com/user-attachments/assets/e90e569e-2856-4405-920e-bc5a35cf239e" />
---

## 1.10 Enabling File Integrity Monitoring (FIM)

A specific folder on the Windows endpoint was added to the agent's FIM configuration so that Wazuh would monitor it for file creation, modification, and deletion events. This was verified by creating, editing, and removing test files inside the monitored folder and confirming that corresponding alerts appeared on the Wazuh Dashboard.

<img width="1853" height="982" alt="File Intergity" src="https://github.com/user-attachments/assets/d0d53a44-d5ef-4137-a788-1c61f973f63a" /><img width="1853" height="982" alt="Wazuh Manager" src="https://github.com/user-attachments/assets/ad46e378-a86e-4e87-ad32-f6143115b35b" />


Full verification steps are documented in the [Verification Guide](06-verification-guide.md).

---

## 1.11 Installation Summary Table

| Step | Purpose | Verified By |
|---|---|---|
| Install curl & add GPG key | Establish trusted package source | Key file present in keyring directory |
| Run wazuh-install.sh -a -i | Deploy Manager, Indexer, Dashboard | Installer completion output |
| systemctl status checks | Confirm services are running | `active (running)` status for all three |
| ss -tulnp check | Confirm Manager is listening | Ports 1514/1515 shown as LISTEN |
| manage_agents | Register Windows endpoint | Agent key generated |
| Install Agent v4.12.0 on Windows | Match Manager version | `sc query WazuhSvc` shows RUNNING |
| Configure FIM | Enable file monitoring | Test file alerts appear on Dashboard |

Continue to the [Troubleshooting Guide](02-troubleshooting-guide.md) to see the real issues encountered during this process and how each was resolved.
