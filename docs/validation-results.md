# Validation log

| Test | Evidence | Outcome |
| --- | --- | --- |
| n8n running on Ubuntu via Docker Compose | 01 workflow screenshot and earlier setup checks | n8n web UI reachable; self-hosted workflow created |
| Authenticated Splunk REST search | HTTP Request returned JSON containing Windows events | Passed |
| n8n five-minute scheduling | Published workflow and recorded scheduled executions | Passed |
| Detection against historical Hydra activity | Matching failed-logon groups found during initial testing | Passed for test data |
| Gemini generation | Basic LLM Chain produced an investigation summary | Passed |
| Slack delivery | Slack SOC notification received | Passed |
| Duplicate suppression | Running the same inputs again produced no repeat Slack alert | Passed in test |
| Fresh two-attempt validation | Screenshots 02–05 | Passed |
| Error Handler test | Screenshots 06–07 | Deliberately failing workflow generated Slack error notification |
| Reproducibility after Splunk trial expiry | No expiry test supplied | **Not verified** |
| Guaranteed delivery after Slack failure | No delivery-tracking proof | **Not verified** |

## Interpretation notes

The number of matching events in Splunk is **not** automatically the number of individual n8n alerts. Splunk grouped the two failed logons into a single 15-minute detection result. A successful n8n execution with zero Splunk `results` is normal during quiet periods, provided the query itself completed successfully.
