# PHP2Shell — detection & hardening (defensive)

Vendor-agnostic signatures for CVE-2026-109618 / CVE-2026-109619. These detect the
*technique*; they contain no exploit code.

## Log heuristics (web server / PHP)

- **Crash-then-re-request loop:** a PHP worker fatal/segfault/OOM followed within seconds by
  a further request from the **same source IP** — repeated. This pattern can support investigation but also occurs for other causes; it does not prove heap grooming or exploitation.
- **Upload-progress abuse:** bursts of requests carrying `PHP_SESSION_UPLOAD_PROGRESS` with a
  `PHPSESSID` cookie, especially interleaved with worker crashes.
- **Chunked multipart, no Content-Length:** `Transfer-Encoding: chunked`
  `multipart/form-data` POSTs with absent/invalid `Content-Length`.
- **Oversized multipart bodies:** single-source multipart uploads of **1 MB+** (observed:
  8–15 MB) that do not correspond to a legitimate upload feature.
- **Pointer leakage:** `Set-Cookie: PHPSESSID=` values containing bytes in the `0x7f`+ range
  (leaked heap/libphp pointers).

## Suricata-style idea

Alert on `multipart/form-data` requests whose boundary is a single repeated character,
combined with `PHP_SESSION_UPLOAD_PROGRESS` and a `PHPSESSID` cookie containing null bytes.
Rate-correlate per source IP with 5xx/worker-crash responses.

## Endpoint / EDR

A web-server worker process (`apache2`/`httpd`/`php-fpm`/`php`) spawning a shell
(`/bin/sh`, `/bin/bash`) is a high-priority process-start signal. Correlate the parent process, request, operator activity and outcome before concluding an attacker achieved RCE. The sensor did record process-start alerts on 21 October; none has been linked to the external burst described in FINDINGS.md.

## Hardening that breaks the chain

- Patch to a fixed release (8.4.27 / 8.5.12 / 8.3.36 / 8.2.35) — the only complete fix.
- `session.use_strict_mode = 1`
- `session.upload_progress.enabled = 0` (partial mitigation only — not a substitute for patching)
- `display_errors = Off` in production (denies the attacker crash/leak feedback)
- Avoid single-worker prefork `mod_php` where practical (the profile most readily exploited).
