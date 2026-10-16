# Architecture and network

The lab reuses elements of a previously completed Windows detection lab but runs without its original Active Directory Domain Controller. `TARGET-PC` was removed from the domain and configured with a local Windows account for this project.

| Virtual machine | IP address (observed) | Primary purpose |
| --- | --- | --- |
| Kali Linux | `192.168.10.250` | Authorised controlled RDP authentication attempts |
| `TARGET-PC` (Windows 10) | `192.168.10.100` | Windows Security event generation and log forwarding |
| Splunk Ubuntu Server | `192.168.10.10` | Ingest and query `index=endpoint`, API on `8089` |
| `n8n-server` (Ubuntu Server, CLI only) | `192.168.10.8` | Docker-hosted n8n on port `5678` |

VirtualBox NAT Network was used for VM-to-VM access. Splunk receives Windows Security and Sysmon events from the endpoint; n8n periodically queries Splunk via an authenticated REST request. No paid cloud VM is used. Gemini and Slack are external online APIs, so the environment depends on internet connectivity for analysis and messaging.

The system is an **assisted investigation workflow**, not autonomous containment. An analyst must verify the event context and approve responses.
