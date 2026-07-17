# 🔑 Azure Entra ID Credential Lifecycle Auditor

<img width="966" height="437" alt="Screenshot 2026-07-17 at 2 11 25 PM" src="https://github.com/user-attachments/assets/96efaebb-75e8-4d54-be0d-9c00e6440ddf" />

A CLI tool that audits Azure App Registration credentials — client secrets and certificates — for age, expiration, and rotation health, producing audit-ready evidence for SOC 2 (CC6.1) and NIST 800-53 (IA-4) compliance.

Client secrets and certificates are easy to forget once created: they cause unexpected downtime when they expire, or become a security liability when left active far longer than they should be. This tool scans an Azure tenant's App Registrations, evaluates every credential's lifecycle against a configurable policy, and classifies findings by risk so security teams and auditors can act on stale or soon-to-expire credentials before they become a problem.

---

## ✨ Features

- **Automated Discovery:** Scans all App Registrations across an Azure tenant.
- **Dual-Credential Inspection:** Evaluates both `passwordCredentials` (client secrets) and `keyCredentials` (certificates).
- **Risk Classification:** Groups credentials into Critical, High, Medium, and Low risk based on age and time to expiration.
- **Secretless Authentication:** Uses Azure's `DefaultAzureCredential` — no hardcoded credentials required.
- **Audit-Ready Exports:** Generates a CSV summary for stakeholders and a detailed JSON report for technical review.
- **Configurable Policy:** Lifecycle thresholds (max age, expiration warning window) are adjustable via command-line flags.

---

## 🛠️ Technologies Used

- **Language:** Python 3
- **Cloud Platform:** Microsoft Azure
- **Authentication:** Azure SDK (`azure-identity`)
- **APIs:** Microsoft Graph API (REST)
- **Data Processing:** Native Python JSON and CSV handling

---

## 🏗️ System Architecture

A stateless command-line tool that queries Azure directly — no persistent infrastructure required.

- **Trigger:** Command-line execution
- **Core:** Python-based risk assessment engine
- **External Service:** Microsoft Graph API (`https://graph.microsoft.com`)
- **Authentication:** Azure CLI session / Service Principal context

```text
+-------------------+       +-----------------------+       +------------------------+
|                   |       |                       |       |                        |
|   Security /      |       |  Azure Credential     |       |  Microsoft Graph API   |
|   GRC Engineer    +-----> |  Lifecycle Auditor    +-----> |  (Entra ID App Data)   |
|   (CLI Execution) |       |  (Python Engine)      |       |                        |
|                   |       |                       |       |                        |
+-------------------+       +-----------+-----------+       +------------------------+
                                        |
                                        v
                            +-----------------------+
                            |                       |
                            |  Audit Evidence       |
                            |  (JSON & CSV Reports) |
                            |                       |
                            +-----------------------+
```

---

## 🚀 How It Works

<img width="1568" height="676" alt="image" src="https://github.com/user-attachments/assets/9c26a7ee-400d-4222-ae39-e3993a83615e" />

1. **Initialization:** The script is run from the terminal, optionally with custom risk thresholds.
2. **Authentication:** The tool authenticates using the active Azure CLI session via `DefaultAzureCredential` — no separate API keys needed.
3. **Data Retrieval:** It queries the Microsoft Graph API for all App Registrations and their associated credentials.
4. **Analysis:** Each credential's age and remaining time to expiration are calculated.
5. **Risk Classification:** Credentials violating the configured lifecycle policy are flagged and categorized by urgency.
6. **Reporting:** A summary is printed to the terminal, and detailed findings are saved locally as audit evidence.

<img width="1568" height="348" alt="image" src="https://github.com/user-attachments/assets/e29721e4-709c-4a20-823e-2b4add0e8bce" />

---

## 🔒 Security

- **No Hardcoded Secrets:** `DefaultAzureCredential` removes the need to store API keys or credentials in the environment.
- **Least Privilege:** The tool requires only the `Application.Read.All` Microsoft Graph scope — read-only, with no ability to modify tenant data.
- **Safe Data Handling:** Microsoft Graph never returns plaintext secret values, and neither does this tool — only metadata (creation and expiration dates) is inspected or logged.
<img width="1568" height="517" alt="image" src="https://github.com/user-attachments/assets/3dcb3443-1846-424f-ad9e-e341d5893501" />

- **Secure Communication:** All Microsoft Graph API traffic is enforced over TLS 1.2+.

---

## 🧠 Key Design Decisions

- **Stateless CLI over a background service:** An on-demand script keeps the tool's footprint minimal with nothing persistent to secure or maintain.
- **Graph API Pagination:** Built-in pagination ensures accurate results even in tenants with thousands of applications.
- **Dual-Format Reporting:** Output is split into a CSV (for auditors and stakeholders) and a JSON report (for engineers investigating specific findings).

---

## 📈 Scalability

- **Stateless Design:** No local state makes the tool straightforward to containerize or run in ephemeral environments.
- **Automation Ready:** Can be integrated into Azure Automation Runbooks, Azure Functions, or GitHub Actions for scheduled, hands-off compliance monitoring across subscriptions.

---

## 📁 Project Structure

- `inactive_key_checker.py` — Core Graph API client and risk assessment engine.
- `azure_inactive_key_summary.csv` — Executive-level compliance report output.
- `azure_inactive_key_analysis_report.json` — Detailed technical evidence output.
- `venv/` — Isolated Python virtual environment (not committed).

---

## 💻 Installation

1. Clone the repository:
```bash
   git clone <YOUR_REPO_URL>
   cd <YOUR_REPO_DIRECTORY>
```

2. Set up a Python virtual environment:
```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
   pip install azure-identity requests
```

---

## 🏃 Usage

1. Log in to Azure:
```bash
   az login
```

2. Run the auditor with default policy (90-day max age, 30-day expiration warning):
```bash
   python inactive_key_checker.py
```

3. Run with custom thresholds (e.g., 60-day max age, 15-day warning):
```bash
   python inactive_key_checker.py --key-age-threshold 60 --expiration-threshold 15
```

---

## 🔮 Future Improvements

- **Alerting:** Slack or Microsoft Teams integration to notify teams when a credential reaches `CRITICAL` status.
- **Multi-Tenant Support:** Extend scanning to multiple child tenants via Azure Lighthouse, for MSP-style use cases.
- **Automated Remediation:** An `--enforce` flag to automatically disable credentials that have exceeded policy thresholds.

---

## 🎓 Skills Demonstrated

- Identity & Access Management (IAM) auditing via Microsoft Graph
- Cloud security architecture and credential lifecycle management
- API integration and pagination handling
- Compliance automation and GRC (SOC 2, NIST)
- Secure, secretless authentication design
- Python scripting and CLI tool development
