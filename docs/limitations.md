# Limitations, security and costs

1. **Lab-only scope.** The current SPL rule filters to the local test account `soc_test` and Kali address `192.168.10.250`, with a deliberately low threshold of two failures. It is not a comprehensive detection of all brute-force attacks.
2. **Fixed time buckets.** Fifteen-minute bins can split activity across boundaries. The completed-bucket rule also introduces notification delay and has a finite lookback window.
3. **At-least-once versus loss.** A processed key can be remembered before Slack delivery completes. Automatic retries and an error handler help with observability, but exactly-once delivery was not implemented.
4. **AI limitations.** Gemini can produce unsupported conclusions or unsuitable recommendations. The output is triage assistance requiring human verification; no automatic containment occurs.
5. **Splunk licence.** This build used an active Splunk Enterprise trial. Whether the authenticated REST integration continues after switching to Splunk Free was not tested. Splunk Free licence feature restrictions must be assessed before claiming indefinite free operation.
6. **Network and credential security.** The lab uses HTTP for the private n8n service, a self-signed Splunk management certificate exception and Basic Auth stored in n8n. Do not expose these services or allow them on untrusted networks in this configuration.
7. **External free tiers.** Gemini and Slack free-tier access have quotas and usage restrictions and are subject to change. Credentials must never be published in GitHub or screenshots.
8. **Persistence and recovery.** n8n data were stored in a persistent Docker volume. Backups and full disaster recovery were not demonstrated.
9. **No reproducible workflow export yet.** A real, sanitized n8n JSON export should be added later if desired. An invented export would not match the verified workflow.
