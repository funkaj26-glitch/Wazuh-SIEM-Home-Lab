# 3. Lessons Learned

This section reflects on what this project reinforced technically and professionally. These are the points most worth raising in an interview, since they demonstrate judgment and problem-solving, not just the ability to follow an installation guide.

---

## 3.1 Technical Lessons

### Networking is the foundation everything else depends on
Nothing else in this lab — enrollment, log collection, FIM — could work until basic connectivity was correct. The NAT-vs-Bridged issue reinforced that **network configuration should be the first thing verified**, not an afterthought, whenever two systems are expected to communicate. A misconfigured virtual network adapter can look identical to a dozen other possible failures until it's specifically ruled out.

### Version compatibility must be treated as a hard requirement, not a suggestion
Assuming a newer Agent version would simply work with an older Manager version caused a real, time-costing failure. This reinforced a habit of **explicitly checking compatibility matrices** before deploying any distributed system component, rather than defaulting to "latest is best."

### Memory sizing matters for data-intensive services
The Wazuh Indexer's OOM crash was a direct reminder that search/indexing engines (built on technologies like OpenSearch/Elasticsearch) are memory-hungry by nature. In a home lab with limited VM resources, this becomes an active constraint to plan around, not just a theoretical best practice from documentation.

### Configuration files must be syntactically valid, not just logically correct
The `ossec.conf` XML error showed that even a conceptually correct configuration change (adding a FIM directory) can break an entire service if the underlying file syntax is invalid. This reinforced the value of **validating configuration files after every manual edit**, especially with strict formats like XML where a single unclosed tag halts the whole file from parsing.

### Always capture generated credentials immediately
The Dashboard login issue was entirely avoidable — the credentials were generated and displayed once during installation. This reinforced a simple but important operational discipline: **capture and securely store any auto-generated secrets at the moment they're created**, rather than assuming they can be easily regenerated or recalled later.

---

## 3.2 Process & Professional Lessons

### Documentation should be honest about the process, not just the outcome
Most polished project write-ups only show the final working state. Documenting the actual failures — and the diagnostic steps taken to resolve them — is far more representative of real security operations work, where troubleshooting is the majority of the job.

### Verifying assumptions with commands beats assuming success
Several issues in this project (the Indexer crash, the Agent failing to start) would have gone unnoticed without actively checking service status with `systemctl status` and `sc query`. This reinforced the habit of **always confirming a service's actual running state**, rather than assuming a command that "completed" also means a service is healthy.

### Root cause analysis is more valuable than a quick fix
For the Indexer OOM issue in particular, simply restarting the service resolved the immediate symptom, but understanding *why* it happened (insufficient memory allocation) was necessary to prevent recurrence — this distinction between "fixing the symptom" and "fixing the cause" is a core SOC analyst mindset.

### A layered environment requires layered troubleshooting
Diagnosing issues in this lab required checking multiple layers independently: the hypervisor's network settings, the OS-level service status, the application-level Wazuh diagnostics, and the configuration file syntax. This reinforced a habit of **isolating which layer a problem lives in** before attempting a fix, rather than guessing at the application layer when the issue is actually at the network or OS layer.

---

## 3.3 How This Maps to a SOC Analyst / Security Engineer Role

| Lab Experience | Real-World Equivalent |
|---|---|
| Diagnosing NAT vs. Bridged networking issues | Troubleshooting connectivity between security tools and monitored assets |
| Resolving Agent-Manager version mismatch | Managing compatibility across a fleet of endpoint agents in an enterprise environment |
| Investigating an OOM-crashed Indexer | Capacity planning and performance troubleshooting for SIEM infrastructure |
| Fixing malformed XML in `ossec.conf` | Debugging configuration-as-code / config management errors |
| Verifying FIM alerts on a test folder | Validating detection logic actually fires before trusting it in production |
| Capturing and securing generated credentials | Credential management and secrets handling discipline |

Continue to [Future Improvements](04-future-improvements.md) to see how this lab is planned to evolve.

