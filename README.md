# CyberLab 003: SIEM monitoring and custom detection with Wazuh

I built a small security monitoring lab to collect events from Linux, Windows and a simulated firewall source. I used Wazuh to parse the logs, build dashboards and test two custom detection rules.

**Tools:** Wazuh 4.14.10, Ubuntu Server, VirtualBox, Windows 11, Sysmon, Linux PAM/journald, syslog, XML and JSON.

## What I built

- A Wazuh all-in-one server (manager, indexer and dashboard) running on an Ubuntu VM.
- A Linux endpoint forwarding authentication events.
- A Windows host monitored using Sysmon Event ID 1 process-creation telemetry.
- A simulated firewall syslog source, with a custom decoder to extract action, source and destination IPs, protocol and destination port.
- Three dashboards with ten visualisations: Authentication Activity, Endpoint Process Activity and Network Traffic Overview.
- Two custom rules, tested against live endpoint activity.

The Windows endpoint was the physical host, rather than a separate Windows VM, because of available hardware resources. Firewall events were **simulated**, not collected from a deployed network firewall.

## Architecture

The Wazuh server used VirtualBox NAT for updates and a host-only interface for lab communications. The Linux VM and Windows host sent events to the manager. I generated synthetic firewall messages over UDP 514 to test syslog ingestion and parsing.

## Detection rules and results

| Rule | Detection | Validation | Outcome |
| --- | --- | --- | --- |
| `100101` | Five failed authentications against the same account within 300 seconds | Failed `sudo` authentications on the Linux endpoint | **Passed:** Level 10 alert on 21 November 2026 |
| `100102` | PowerShell launching `cmd.exe` | Launched `cmd.exe` from PowerShell | **Passed:** Level 8 alert on 20 November 2026 |
| `100102` negative control | `cmd.exe` launched by Explorer | Opened Command Prompt from Explorer | **Passed:** event recorded, custom alert not triggered |

The five Linux authentication failures occurred within 2 minutes and 42 seconds. The resulting alert included the earlier matching events, allowing me to verify correlation.

## Problems I encountered and how I solved them

**VirtualBox performance and Hyper-V conflict.** I worked around a host virtualisation conflict and reduced the number of concurrently running VMs. This is why I used the Windows host as a monitored endpoint.

**Firewall log parsing.** My first syslog messages used RFC5424 formatting and did not match the decoder as expected. I switched the test messages to RFC3164 and verified that Wazuh extracted the fields correctly.

**Authentication correlation.** My first account-correlation expression did not trigger in controlled tests. I changed it to `<same_user />`, validated the rule, and later confirmed the five-failure threshold using live Linux authentication events.

**Dashboard presentation.** Some charts appeared empty because of a wide time range or a panel that was too small. I adjusted the time window and dashboard layout before capturing evidence.

## Evidence

- [Custom rules](rules/cyberlab_custom_rules.xml)
- [Firewall decoder](decoders/local_decoder.xml)
- [Sanitised test events](evidence/)
- [Dashboard and alert screenshots](screenshots/)
- [Technical report](report/CyberLab_003_SIEM_Report.pdf)

The evidence is sanitised for public sharing. Original raw logs are not published. The firewall events are synthetic.

## What I would improve

I would add a real firewall or reverse-proxy log source, test more authentication edge cases, map custom alerts to MITRE ATT&CK techniques and configure notifications. For a live application, I would also prioritise application-specific events such as privilege changes, suspicious access to records and payment anomalies, while avoiding sensitive patient information in logs.

All tests were performed on systems under my control.
