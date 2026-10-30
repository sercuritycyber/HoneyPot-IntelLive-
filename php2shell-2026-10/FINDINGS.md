# PHP2Shell (CVE-2026-109618 / CVE-2026-109619) — honeypot evidence, Oct 2026

**SecurityCyber honeynet · captured 21–23 October 2026 · published as defensive intel.**

> **Evidence correction, 30 October 2026:** Wazuh recorded five PHP process-start alerts on 21 October, including two clusters that coincided with administrative SSH. Their origin has not been independently established. None was correlated with the external PHP requests described below. A process-start alert is not proof of an externally triggered compromise. The earlier statement that no execution-stage alert fired was incorrect. See [the supplemental investigation](../investigations/2026-10-30-php-payloads.md).

## TL;DR

Our internet-facing PHP honeypot logged in-the-wild PHP2Shell exploitation **attempts**
from **21 October 2026, 23:33 UTC** — the day the fixes shipped, and days before the
first *confirmed successful* RCE was publicly reported. We captured the full exploitation
*technique* and the raw payload bodies. We did **not** observe successful code execution
on our sensor, despite it running the exact version named in public reports (PHP 8.4.26).
This is a **honeypot sighting of attempts**, not a confirmed compromise, and we present it
as such.

## What we observed

- **21 Oct, 23:33 UTC — `176.65.139.151`** sent requests consistent with several parts of an attempted technique in a ~38-second
  burst: PHP session upload-progress abuse (CVE-2026-109619), chunked `multipart/form-data`
  with no `Content-Length` (CVE-2026-109618), repeated worker fatal/OOM crashes, and the
  correlated **crash-then-re-request loop** that is the fingerprint of heap grooming. It is
  the only source in our data to reach that final correlation stage.
- **23 Oct — a wider wave** from multiple sources (`36.255.97.14`, `89.238.150.166`,
  `45.156.128.0/24` cluster, `109.105.210.102`) ran the same patterns. We captured their
  raw multipart heap-spray payload bodies (8–15 MB each) to disk.
- See `alert-timeline.csv` (95 exploitation-relevant alerts) and `captured-payloads.csv`
  (payload **hashes + sizes only**).

## What we did NOT observe

No attacker-triggered code execution or PHP-process dropper fetch was confirmed for the described burst. A shell and other process starts were recorded earlier on 21 October, but their origin remains unconfirmed and has not been linked to this burst. The 21 Oct burst simply
**stopped after ~38 seconds of crash-looping**. Our sensor's entire alert history for this
event tops out at exploitation-*delivery* and crash signals (MITRE **T1190 Initial Access**)
— there is no execution-stage alert correlated to the burst in the reviewed records.

## Assessment (held loosely)

A later inspection observed **PHP 8.4.26** on a replacement instance. The available evidence does not establish that the October 21–23 captures came from that replacement instance, or conclusively determine the earlier sensor version. That public success landed only against a
deliberately favourable **single-worker `mod_php`** profile and was explicit that it does
not measure reliability elsewhere. Taken together, the honest read is that **reliable
exploitation in the wild is currently configuration-dependent, not universal.** That is not
a reason to relax: attempts are real, broad, and ongoing, and the heap-grooming primitive is
sound. Treat every pre-fix `mod_php` host as exposed.

## Source IPs (high-severity, our sensor)

| IP | First seen (UTC) | Notes |
|---|---|---|
| 176.65.139.151 | 2026-10-21 23:33 | full chain incl. crash→re-request correlation |
| 89.238.150.166 | 2026-10-23 20:52 | sustained upload-progress + oversized multipart |
| 36.255.97.14 | 2026-10-23 13:43 | upload-progress + oversized multipart |
| 45.156.128.43 / .41 / .47 / 45.156.129.57 | 2026-10-23 | upload-progress cluster |
| 109.105.210.102 | 2026-10-23 05:11 | upload-progress |

Full IOC set is in the repo's `AbuseIPDB_Bulk_Report.csv` and our live TAXII 2.1 feed at
`https://cti.securitycyber.uk/taxii2/`.

## A deliberate omission

We are **not** publishing the raw captured payload bodies, exploit code, or any
proof-of-concept. The payload hashes in `captured-payloads.csv` exist for correlation only.
We provide IOC, not POC.

## What to do

Patch — it is the only real fix: **PHP 8.4.27, 8.5.12, 8.3.36 or 8.2.35**. The widely-shared
`session.upload_progress.enabled=0` workaround reduces surface but does **not** fully protect
you. If you cannot patch immediately, hunt the signatures in `detections.md`.

---
*Scope: this is honeypot telemetry of exploitation attempts, not a report of a breached
production system. Timeline corroborated by public reporting (watchTowr, Amazon Threat
Intelligence, BleepingComputer, CISA KEV).*
