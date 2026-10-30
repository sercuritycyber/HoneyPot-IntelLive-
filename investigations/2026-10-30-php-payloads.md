# PHP honeypot payload investigation — 21–23 October 2026

**Assessment:** The sensor recorded attempted payload delivery. The reviewed evidence does not prove that either downloader below executed or that its payload was retrieved. Indicators are time-bound observations, not attribution or a standalone block recommendation.

## PHP-CGI-style downloader — 23 October

At **22:02:48 UTC**, source `89.247.33[.]110` sent a request to `/hello.world` containing PHP-CGI argument-injection-shaped parameters, including `auto_prepend_file=php://input`, and a PHP body with a base64-encoded shell command. Static decoding showed an attempt to use an embedded SSH key to retrieve a script from `217.60.102[.]5`, with an HTTPS download fallback. The command contained a `cve_2024_4577` label; that label is attacker-supplied and does not prove the vulnerability was exploited.

The web log records an HTTP 200 POST from that source. This establishes a response, not execution of the body. The SHA-256 of the **HTTP request body**, not a malware binary, is `9b7215fd7907a07b7b6601370623a5bcda2ae6f5f896cbfc1ddf5e3e3096dc49`. No retrieved executable or binary hash was verified.

## Upload-progress downloader — 21 October

At **23:33:09 UTC**, source `176.65.139[.]151` submitted a `PHP_SESSION_UPLOAD_PROGRESS` value containing instructions to retrieve `hxxp://176.65.139[.]151/bot-x86_64`, save it as `/tmp/.ak` and start it. The PHP error logs show memory exhaustion and request input-limit warnings during the burst. These are observable effects, but do not establish command execution. Whether the source was authorized testing has not been independently determined.

## Process evidence and limits

A wider Wazuh audit review found five web-user process-start alerts earlier on 21 October, in two clusters near administrative SSH sessions. The commands and timing are consistent with validation activity, but their origin is unconfirmed. None is tied to the two requests above. The previous blanket statement that execution alerts never fired was incorrect. `execve` success means that a process started; it does not prove that a network connection, file operation or malicious payload succeeded.

The archived PHP captures end on 24 October, and retained SIEM history has gaps. The subsequent replacement instance must not be assumed to be the same sensor that produced the earlier events. Raw request bodies and embedded credentials are intentionally not reproduced.

## Defensive use

Look for the network destinations and paths in [the indicator table](2026-10-30-php-iocs.csv), then correlate with web requests, process ancestry, file creation and outbound connection evidence. Do not assume an IP is malicious solely because it appeared in a honeypot log; infrastructure can be shared, reassigned, or used for authorized testing.
