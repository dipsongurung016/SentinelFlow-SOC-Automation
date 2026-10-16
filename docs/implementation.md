# Implementation overview

This document records the system components and configuration observed during the build; it is not an exact one-click reproduction script.

## VM and services

- `TARGET-PC`: standalone Windows 10 endpoint using local users; Windows Security event logging, Sysmon and Splunk forwarder.
- Splunk Enterprise: accessible on the lab at `192.168.10.10:8000` (web) and `192.168.10.10:8089` (management API); security telemetry stored in `index=endpoint`.
- `n8n-server`: Ubuntu Server without a desktop, accessed through SSH; Docker Engine and Compose installed; n8n Community Edition runs with a persistent volume.
- n8n performs authenticated calls to Splunk via its HTTP Request node with an account created for the integration.

## n8n node flow

`Schedule Trigger (5m) → HTTP Request (Splunk) → Split Out (results) → IF (count >= 2) → Edit Fields → Remove Duplicates → Basic LLM Chain / Gemini → HTTP Request (Slack)`

A separate n8n Error Trigger workflow sends diagnostic messages to Slack for failed scheduled workflows.

### Splunk search API configuration

- Method: `POST`
- Path: `https://<SPLUNK_IP>:8089/services/search/jobs`
- Body encoding: form URL encoded
- `search`: SPL from [`../detections/failed-logons-lab.spl`](../detections/failed-logons-lab.spl)
- `exec_mode`: `oneshot`
- `output_mode`: `json`
- Credentials: kept in n8n's encrypted credential store, **never committed**.

### Gemini analysis

The AI is instructed to summarise evidence, explain why the log pattern merits review, and propose investigation steps. The prompt explicitly warns the model against claiming that maliciousness or RDP is confirmed without sufficient supporting data.

### Slack integration

The AI summary is sent using a Slack Incoming Webhook stored privately in n8n. This project does not publish the URL, an API key, the encryption key or the credential store.

## Example container configuration

An illustrative Compose file and variable template appear in [`../deployment/`](../deployment/). They reflect the type of setup used but are not an exported copy of the running VM.
