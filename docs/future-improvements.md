# 4. Future Improvements

This lab is an evolving project. The improvements below are prioritized based on what would add the most realistic SOC/security engineering value next.

---

## 4.1 Infrastructure & Resource Improvements

- **Right-size VM memory allocation for the Indexer.** Following up on the OOM crash documented in the Troubleshooting Guide, permanently increase the Ubuntu VM's allocated RAM and, if needed, tune the Wazuh Indexer's JVM heap settings so this failure does not recur under normal load.
- **Separate the Wazuh components across multiple VMs.** Currently the Manager, Indexer, and Dashboard run on a single host for simplicity. Splitting them across separate VMs would better reflect a production-style distributed deployment and allow independent scaling/troubleshooting of each component.
- **Add a dedicated syslog-based Linux endpoint.** Enrolling a second agent on a Linux machine (in addition to the current Windows endpoint) would demonstrate cross-platform monitoring capability.

## 4.2 Detection & Monitoring Enhancements

- **Expand File Integrity Monitoring coverage.** Extend FIM beyond a single test folder to include commonly targeted Windows directories (e.g., `System32`, startup folders, and user profile directories) to reflect realistic attack surface monitoring.
- **Enable and tune Windows Security Event Log collection in depth**, mapping collected event IDs to relevant MITRE ATT&CK techniques for more meaningful alerting context.
- **Build custom Wazuh detection rules and decoders** tailored to specific simulated attack scenarios, rather than relying solely on default rule sets.
- **Integrate a threat intelligence feed** (such as an open-source IOC feed) with Wazuh's active response or rule correlation to simulate enrichment of raw events with threat context.

## 4.3 Simulated Attack & Validation Scenarios

- **Run controlled adversary simulation exercises** (e.g., using a tool such as Atomic Red Team) against the Windows endpoint to validate that Wazuh detects and alerts on specific MITRE ATT&CK techniques, and document detection coverage gaps.
- **Test and document Wazuh's Active Response capability**, such as automatically blocking an IP address after repeated failed authentication attempts, to demonstrate automated response, not just passive detection.

## 4.4 Operational Maturity

- **Add centralized log retention and backup planning** for the Wazuh Indexer to reflect real data retention/compliance considerations.
- **Document a formal incident response runbook** based on a simulated alert triggered in this lab, extending the "Lessons Learned" mindset into a repeatable operational process.
- **Add role-based access control (RBAC) configuration** within the Wazuh Dashboard to reflect least-privilege access principles for multiple analyst roles.

## 4.5 Documentation Improvements

- **Add architecture diagrams generated from an actual network capture or topology tool**, rather than a text-based diagram, once the environment is expanded to multiple VMs.
- **Record a short screen-capture walkthrough** of an alert firing end-to-end (test file change → FIM alert → Dashboard visualization) to supplement the written documentation for interview presentations.

---

## Priority Summary

| Priority | Improvement | Value |
|---|---|---|
| High | Fix Indexer memory sizing permanently | Prevents recurring service instability |
| High | Add MITRE ATT&CK-mapped detection testing | Strong interview talking point on detection engineering |
| Medium | Split components across multiple VMs | More realistic distributed architecture |
| Medium | Add a second (Linux) endpoint | Demonstrates cross-platform coverage |
| Lower | RBAC and access control configuration | Reflects operational maturity |

Continue to [References](05-references.md) for the resources used throughout this project.
