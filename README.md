
# Security Alerts (Outlook Emails)  → TheHive Ingestion

Automate the ingestion of **security alert emails** from **Microsoft 365 (Outlook)** into **TheHive**.

This repository contains two Python scripts:

- `cortexxdr.py` — Parses **Palo Alto Cortex XDR** alert emails and creates **TheHive Alerts**.
- `threat_.py` — Parses **Palo Alto Firewall** alert emails and creates **TheHive Alerts**.

Both scripts:
- Authenticate to **Microsoft Graph** using **MSAL (OAuth2)**.
- Read **unread** messages from a target mailbox/folder.
- Parse HTML emails into normalized text and key fields.
- Extract common **IOCs** (IPs, URLs, hashes, emails).
- Create **TheHive Alerts** with **idempotency** via Outlook `internetMessageId` as `sourceRef`.
- Mark emails **read after successful submission** (or optionally move to a processed folder).

---

## Why

Security products frequently notify by email. This automation:
- Prevents missed alerts and manual copy-paste.
- Normalizes data for SOC triage in TheHive.
- Adds IOC artifacts automatically.
- Ensures safe processing (no loss when downstream is unavailable).

---

## Architecture (High-Level)

```
Outlook 365 Mailbox → Microsoft Graph API → Python Ingestor (Parser + IOC Extractor) → TheHive (thehive4py)
```

- **Collection**: Microsoft Graph with MSAL (client credentials).
- **Parsing**: HTML → Text + vendor-specific extraction (Cortex XDR / Palo Alto FW).
- **Artifacts**: IP, URL, SHA-256 hash, email.
- **Idempotency**: `sourceRef = internetMessageId` to avoid duplicates.

---

## Repository Layout

```
.
├─ cortexxdr.py                # Cortex XDR alert ingestion
├─ threat_.py                   # Palo Alto Firewall alert ingestion
├─ config.json                 # Graph/AAD base config (client_id, secret, authority, endpoint)
├─ globals.json                # static mappings for parsers
├─ requirements.txt            # Python dependencies
└─ README.md                   # This file
```

---

## Requirements

### Python Dependencies
Install required packages:

```bash
pip install -r requirements.txt
```

Dependencies include:

```
msal
requests
thehive4py
beautifulsoup4
lxml
pandas
```

---

## Configuration

Use a **config.json** plus **environment variables**.

### Example `config.json`

```json
{
  "client_id": "00000000-0000-0000-0000-000000000000",
  "secret": "YOUR-CLIENT-SECRET",
  "authority": "https://login.microsoftonline.com/<TENANT_ID>",
  "scope": ["https://graph.microsoft.com/.default"],
  "endpoint": "https://graph.microsoft.com/v1.0/users"
}
```
---

## Usage

### Cortex XDR alerts
```bash
python cortexxdr.py config.json
```

### Palo Alto Firewall alerts
```bash
python threat_.py config.json
```

---

