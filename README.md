# Agentic AI SOC Automation Lab: Wazuh SIEM + Postman API + n8n AI Agent + Atomic Red Team

A self-directed home lab built on an Ubuntu VM (VirtualBox). It deploys the [Wazuh](https://wazuh.com) SIEM and [n8n](https://n8n.io) with Docker, connects a Wazuh agent, generates adversary-style activity with [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team), queries the Wazuh APIs with Postman, and uses an n8n AI agent to summarise recent alerts.

> **Attribution:** This project uses the public repository [Atlantium-AI/wazuh-n8n-lab-setup](https://github.com/Atlantium-AI/wazuh-n8n-lab-setup). This repository documents my own production, with screenshots as evidence.

📄 **Full report with annotated screenshots:** [`SOC_Lab_Report_Complete.docx`](SOC_Lab_Report_Complete.docx)

## Architecture

```
Ubuntu VM (VirtualBox, NAT)
├── Docker
│   ├── Wazuh manager 4.12.0   (agents 1514/1515, API 55000)
│   ├── Wazuh indexer 4.12.0   (9200)
│   ├── Wazuh dashboard 4.12.0 (443 → 5601)
│   └── n8n                    (5678)
├── Wazuh agent 4.12.0-1  (name: SOC-AI, manager 127.0.0.1)
├── auditd
├── PowerShell 7.7.0 + Atomic Red Team (/opt/atomic-red-team)
└── Postman 11.71.7  →  Wazuh API / Indexer API
                     n8n AI agent (Gemini + HTTP Request) → Indexer
```

## Components

| Component | Version | Purpose |
|---|---|---|
| Oracle VirtualBox | VM "SOC Lab AI" (NAT) | Isolated host |
| Docker | 29.10.0 | Container runtime |
| Wazuh manager / indexer / dashboard | 4.12.0 | SIEM / XDR |
| Wazuh agent | 4.12.0-1 | Endpoint telemetry |
| auditd | Ubuntu package | Linux audit events |
| PowerShell / Atomic Red Team | 7.7.0 / GitHub | ATT&CK-mapped test library |
| Postman | 11.71.7 | Calls the Wazuh Management API and Indexer API |
| n8n | `n8nio/n8n` | Automation platform with an AI agent workflow |

## What Was Done

1. **Prepared the VM** with NAT networking in VirtualBox.
2. **Cloned the lab repository** and checked Docker.
3. **Started Wazuh and n8n containers** and verified them with `sudo docker ps`.
4. **Opened the Wazuh dashboard** (no agents registered yet).
5. **Installed the Wazuh agent** using the dashboard's *Deploy new agent* command.
6. **Enabled auditd** and confirmed the agent shows as **Active (1)**.
7. **Installed PowerShell, Invoke-AtomicRedTeam** and cloned the Atomic Red Team repository; launched the atomic tests in the lab VM.
8. **Installed Postman** with the repository's `install-postman.sh`.
9. **Called the Wazuh APIs from Postman:** authenticate (token stored by a post-response script), list agents, and search alerts in the Indexer (`wazuh-alerts-4.x-*`, 1,209 hits at the time).
10. **Built an n8n AI agent workflow** ("Wazuh Agent"): chat trigger → AI Agent (Google Gemini chat model, simple memory, HTTP Request tool that fetches the 20 newest alerts from the Indexer) → written alert summary in the chat.

## Screenshots

| # | Evidence | File |
|---|---|---|
| 1 | VirtualBox network settings (NAT) | [`01-virtualbox-network.png`](screenshots/01-virtualbox-network.png) |
| 2 | git installed and lab repo cloned | [`02-clone-repo.png`](screenshots/02-clone-repo.png) |
| 3 | Repo contents and Docker version | [`03-repo-contents-docker.png`](screenshots/03-repo-contents-docker.png) |
| 4 | Wazuh and n8n containers running | [`04-docker-ps.png`](screenshots/04-docker-ps.png) |
| 5 | Wazuh dashboard, no agents yet | [`05-wazuh-dashboard-no-agents.png`](screenshots/05-wazuh-dashboard-no-agents.png) |
| 6 | Wazuh agent installation | [`06-wazuh-agent-install.png`](screenshots/06-wazuh-agent-install.png) |
| 7 | n8n workspace (initial, empty) | [`07-n8n-workflows-empty.png`](screenshots/07-n8n-workflows-empty.png) |
| 8 | auditd enabled and running | [`08-auditd-enabled.png`](screenshots/08-auditd-enabled.png) |
| 9 | Wazuh shows 1 active agent | [`09-wazuh-agent-active.png`](screenshots/09-wazuh-agent-active.png) |
| 10 | PowerShell + Invoke-AtomicRedTeam installed | [`10-atomic-red-team-install.png`](screenshots/10-atomic-red-team-install.png) |
| 11 | Atomic tests launched | [`11-atomic-tests-run.png`](screenshots/11-atomic-tests-run.png) |
| 12 | Postman installed via lab script | [`12-postman-install.png`](screenshots/12-postman-install.png) |
| 13 | Wazuh API authentication (token redacted) | [`13-postman-wazuh-authenticate.png`](screenshots/13-postman-wazuh-authenticate.png) |
| 14 | List Agents API call | [`14-postman-list-agents.png`](screenshots/14-postman-list-agents.png) |
| 15 | Indexer Search Alerts call | [`15-postman-search-alerts.png`](screenshots/15-postman-search-alerts.png) |
| 16 | n8n AI agent workflow | [`16-n8n-ai-agent-workflow.png`](screenshots/16-n8n-ai-agent-workflow.png) |

![n8n AI agent workflow summarising Wazuh alerts](screenshots/16-n8n-ai-agent-workflow.png)

## Results

- Wazuh 4.12.0 (manager, indexer, dashboard) and n8n run as containers and are reachable locally.
- The agent registered successfully; the dashboard moved from "no agents" to **Active (1)**.
- Both the Wazuh Management API and the Indexer API returned **200 OK** in Postman.
- An n8n AI agent fetched the 20 newest alerts from the Indexer and produced a chat summary.

## Next Steps

- [ ] Run Atomic tests one technique at a time and record the matching Wazuh alert for each
- [ ] Compare the AI agent's summaries with the raw alerts to check accuracy
- [ ] Build an alert-triggered n8n workflow (Wazuh → webhook → notification/ticket)
- [ ] Configure Wazuh to collect auditd logs
- [ ] Add a detection table (technique → rule ID → alert level)

## Skills Practised

Docker-based SIEM deployment · Wazuh agent management · Linux auditing (auditd, systemd) · Adversary emulation setup (Atomic Red Team) · REST API testing with Postman (tokens, environments, Indexer queries) · n8n AI agent automation · MITRE ATT&CK awareness · Lab isolation, redaction and technical documentation

## Author

**Sayed Aslam**: Cybersecurity Enthusiast, Poland ·

## Disclaimer

For educational use in an isolated lab only. Do not run adversary-emulation tools on systems you do not own or have explicit permission to test.
