# Enterprise SOC & Automated Threat Response Architecture (SOAR)

## 📌 Project Overview
This project demonstrates a comprehensive, self-hosted Security Orchestration, Automation, and Response (SOAR) playbook designed to streamline Level 1 SOC operations. Built on a Debian environment using n8n, this architecture automatically ingests Wazuh alerts, enriches them with threat intelligence, leverages Gemini AI for triage, and escalates verified threats to TheHive while notifying the security team.

By automating the tedious triage process, this workflow significantly reduces alert fatigue and accelerates Incident Response times.

## 🏗 Full Architecture & Workflow
![Full Architecture](architecture.png)

### 1. Ingestion & Filtering
The workflow is triggered by a schedule node that continuously queries **TheHive 5** (which syncs with **Wazuh**) for newly generated alerts. A filter is applied to drop already-processed alerts, ensuring only fresh security events (e.g., `sshd: Authentication failed from a public IP`) are pushed down the pipeline.
![Ingestion Phase](ingestion.png)

### 2. Extraction & Threat Intel Enrichment
Once a critical alert is caught, the workflow parses the raw description and extracts the Indicators of Compromise (IOCs), specifically the attacker's IP address.
This IP is then automatically queried against two independent threat intelligence platforms:
* **AbuseIPDB:** To retrieve the global abuse confidence score.
* **MISP (Malware Information Sharing Platform):** To check for existing local or community-shared threat events related to the IP.
![Enrichment Phase](threat-intel-enrichment.png)

### 3. AI Triage (Gemini 2.0 Flash)
The enriched data is forwarded to **Google Gemini**. A highly specific system prompt assigns Gemini the role of a *Level 1 SOC Analyst*. It analyzes the AbuseIPDB score alongside the MISP result count and outputs a strict binary decision: `CASE` (High-Risk) or `IGNORE` (Low-Risk/False Positive).
![AI Triage Phase](gemini-ai-triage.png)

### 4. High-Risk vs. Low-Risk Routing
Based on the AI's verdict, the workflow branches into two distinct paths:
* **Low-Risk (False Path):** If the alert is benign, it bypasses case creation. The alert in TheHive is simply updated with an `n8n_low_priority` tag and closed, saving human analysts valuable time.
* **High-Risk (True Path):** If validated as a true threat, the alert is instantly promoted to a formal **Case** in TheHive, complete with all enriched observables, ready for the Blue Team to investigate.
![Case Creation & Routing](promote-to-case.png)

### 5. Multi-Channel Notifications
For high-risk cases, real-time alerting is critical. The workflow utilizes:
* **Telegram Bot API:** Dispatches a critical alert directly to the SOC team's secure chat.
* **Transactional Email (Brevo):** Sends a detailed HTML email via the custom domain `projectcyber.online` to ensure compliance and record-keeping.
![Notifications](email-notification.png)

## 🛠 Technologies Used
* **SIEM / Endpoint Security:** Wazuh
* **Incident Response Platform:** TheHive 5
* **Automation Engine:** n8n (Self-hosted on Debian)
* **Threat Intelligence:** AbuseIPDB API, Custom MISP Instance
* **AI Engine:** Google Gemini 2.0 Flash API
* **Notifications:** Telegram API, Brevo SMTP 

## 🚀 Deployment & Security Considerations
* All sensitive parameters (API Keys, internal IPs, Telegram Chat IDs) have been sanitized from the public workflow.
* To deploy this in your own environment, import the `workflow.json` into n8n and replace the `<PLACEHOLDERS>` with your localized credentials and API keys.

## 👤 Author
**Anar Musayev**
* [GitHub Profile](https://github.com/Musa3w)
* [LinkedIn Profile](https://www.linkedin.com/in/anarmusayev-/)
