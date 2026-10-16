# Detection logic and analysis

## Objective

Identify repeated failed Windows logon events and pass matching **groups** to n8n for lab triage. A qualifying group is not equivalent to confirmed malicious activity.

## Evidence source

- Splunk index: `endpoint`
- Host: `target-PC`
- Event ID: `4625` (Windows Security: failed logon)
- Controlled local account: `soc_test`
- Test source: Kali `192.168.10.250`

## SPL

The lab rule is stored in [`../detections/failed-logons-lab.spl`](../detections/failed-logons-lab.spl). It extracts the network source IP from event text, groups failures by host and source IP within fixed 15-minute buckets, and returns groups with at least **2** failures. The query examines completed buckets to avoid evaluating an incomplete period and permits approximately one minute for ingestion.

The workflow's IF node passes any group with `failed_attempts >= 2`; severity is labelled **Lab Test**. The schedule runs every 5 minutes. Remove Duplicates uses an identifier based on alert name, source IP, host and time bucket.

### Interpretation

- **Event ID 4625:** an account failed to log on.
- **Logon Type 3:** network authentication; this is compatible with an RDP NLA stage but does not independently prove that every failed network logon used RDP.
- **Substatus `0xC000006A`:** incorrect password for an existing account (as recorded in the test).
- **Two events:** intentionally generated, low-volume activity; not a production brute-force threshold.

### Known limitations

Fixed buckets can split a cluster of attempts across two adjacent 15-minute windows. There is also an alert delay until bucket completion and a subsequent scheduled run. This is a demonstration-specific fixed-bucket rule, not a fully general sliding-window detection. Source IP fields may be absent or formatted differently in other environments. A failed-logon pattern needs additional context and cannot alone establish attacker intent.

## Possible future improvement (not implemented)

Use an overlapping/rolling interval with well-defined suppression, account-level correlation and a richer context check (account lockout/success following failures). Do not describe this as already implemented.
