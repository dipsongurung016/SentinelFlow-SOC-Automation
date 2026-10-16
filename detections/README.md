# Detection rules

`failed-logons-lab.spl` contains the **specific lab detection used in the fresh validation**. It intentionally filters on `soc_test` and `192.168.10.250` with a two-event threshold so the controlled test stays below the Windows account-lockout threshold.

It is not a production brute-force detection. Refer to [`../docs/detection-and-analysis.md`](../docs/detection-and-analysis.md) for field mapping and false-positive considerations.
