# SentinelFlow — n8n Workflows

This folder contains the **sanitised, importable n8n workflows** behind [SentinelFlow](../README.md), an AI-assisted Security Operations Centre (SOC) automation lab.

SentinelFlow monitors Windows failed-logon events through Splunk, uses n8n to identify repeat attempts, asks Google Gemini to draft an analyst-facing triage summary, and sends the finding to Slack. A separate workflow reports automation errors.

> **Scope:** These are real exports from a controlled home lab, not production-ready detection content. The public JSON copies have had secrets and instance-specific credential references removed.

## Files

| File | Purpose |
| --- | --- |
| [`sentinelflow-main.json`](sentinelflow-main.json) | Scheduled Splunk detection, alert processing, AI analysis and Slack notification. |
| [`sentinelflow-error-handler.json`](sentinelflow-error-handler.json) | Catches failed workflow executions and posts diagnostic details to Slack. |

## Main workflow

```text
Schedule Trigger (every 5 minutes)
  -> HTTP Request (Splunk search API)
  -> Split Out (one item per result)
  -> IF (at least 2 failures)
  -> Edit Fields (structured lab alert)
  -> Remove Duplicates
  -> Basic LLM Chain + Google Gemini Chat Model
  -> HTTP Request (Slack Incoming Webhook)
```

The Splunk search reads Windows Security **Event ID 4625** from the `endpoint` index. In the saved **lab-test configuration**, it filters for `host="target-PC"`, account `soc_test`, and source IP `192.168.10.250`. It groups failures by source, destination host, and **fixed 15-minute time bucket**, reports groups with **2 or more** failures, and considers completed buckets after a short ingestion buffer. The 2-failure threshold was chosen for a controlled test, not as a general SOC policy.

The **Remove Duplicates** node keys each finding by alert name, source IP, destination host and time bucket. Gemini receives structured alert fields, not full Windows logs, and drafts an investigation summary. The final HTTP Request node sends the summary to Slack.

**Verified lab result:** Two deliberately incorrect RDP password attempts from Kali produced two Windows `4625` events on `TARGET-PC`; a scheduled n8n execution processed the finding and posted the AI-assisted alert to Slack. See the [investigation case study](../docs/investigation-case-study.md) and [screenshots](../screenshots/).

## Error-handler workflow

```text
Error Trigger -> HTTP Request (Slack Incoming Webhook)
```

When assigned as the main workflow's **Error Workflow**, it sends the workflow name, failed node, error message and execution URL to Slack. This was verified using a separate test workflow that intentionally threw an exception. It does **not** independently detect a stopped n8n server or guarantee that an alert reached Slack.

## Import and configure

1. Run your own n8n instance. Import **both JSON files** using the editor's **Import from File** option.
2. In the main workflow's **Splunk HTTP Request** node, provide your Splunk management API address (lab export: `https://192.168.10.10:8089`) and configure a Splunk **HTTP Basic Auth** credential with appropriate search permissions. Update index, host, user and source-IP filters for your lab.
3. In **Google Gemini Chat Model**, connect your own Gemini API credential and select an available model. API eligibility, quotas and charges depend on your account and provider.
4. In **both Slack HTTP Request nodes**, replace `https://example.invalid/REPLACE_WITH_YOUR_SLACK_WEBHOOK_URL` with your **own private** Slack Incoming Webhook (or an equivalent secret-backed configuration).
5. Open **main workflow → Settings → Error Workflow** and select your newly imported error-handler workflow. The original instance-specific linkage is intentionally absent.
6. Verify the schedule, timezone, search response, node mappings and Slack destination. Execute a controlled test **before** publishing. Both shared workflows are intentionally **inactive** on import.

For a successful low-volume test, the Splunk search must return matching results, the IF condition must pass, and the alert must not already have been marked as seen by Remove Duplicates. An empty `results: []` is normal when nothing matches.

## Security and limitations

- **No secrets are supplied.** The published JSON files exclude the original Slack webhook URL and credential references. **The webhook used in the original private exports should be revoked and replaced**; do not reuse it.
- The Splunk HTTPS node currently allows a **self-signed certificate without verification** for the isolated lab. Configure trusted TLS for any non-lab deployment. Never expose Splunk management port `8089` publicly.
- Duplicate suppression happens **before** Slack acknowledges delivery. Retries and the error handler reduce visibility gaps but do not provide durable, exactly-once alert delivery.
- The search uses **fixed** 15-minute buckets, not a sliding 15-minute window. Events near bucket boundaries and delayed ingestion can affect detection.
- The lab workflow depends on Splunk API/search access; continued use after an Enterprise trial expires depends on the available licence and API capabilities.
- These exports document authorised simulation and triage. They are not evidence of a production-ready SOC or automatic incident containment.

For setup details and technical context, return to the [SentinelFlow project README](../README.md).
