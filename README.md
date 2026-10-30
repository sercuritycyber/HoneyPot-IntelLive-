Active In-the-Wild Exploitation Intelligence

Live Honeypot Telemetry Gathered by SecurityCyberuk

This repository contains raw, categorized Indicators of Compromise (IOCs) captured via active honeypot engagement between October 21 and October 23, 2026.

Our infrastructure is specifically tuned to capture high-fidelity, actionable threat intelligence rather than general background noise. The data contained in this repository highlights aggressive, automated exploitation of the recently leaked PHP vulnerabilities (CVE-2026-109618 and CVE-2026-109619), alongside observations from SSH, Plex and other service sensors. An alert or successful-looking decoy response does not by itself establish compromise, payload execution, or attribution. See the investigation notes for evidence and limitations.

We believe in a defense-first approach. By publishing this telemetry, we aim to provide blue teams and network defenders with verifiable, active threat data to update firewalls, SIEMs, and blocklists.

👉 View the Categorized IOCs Here

⚠️ DISCLAIMER & RULES OF ENGAGEMENT

The information provided in this repository is strictly for threat intelligence, defensive monitoring, and blocklisting purposes only.

DO NOT navigate to, ping, scan, or interact with any of the IP addresses, URLs, or hashes listed in the IOC files.

Interacting with threat actor infrastructure can expose your organization's location.

It can confirm your vulnerabilities to the attackers.

It invites direct, retaliatory targeting against your infrastructure.

All network indicators have been defanged (e.g., hxxp, [.]) to prevent accidental execution. Internal infrastructure addresses have been [REDACTED] to preserve operational security.

Use this data to defend your networks, not to "hack back."

CTI at securitycyber.uk/cti
