# SentinelStack---AI-Augmented-SOC-Lab-Future-of-detection-and-response

A self-hosted Security Operations Center built from scratch on local virtualization infrastructure, combining network intrusion detection (Suricata), network security monitoring (Zeek), SIEM/XDR correlation (Wazuh), and a local LLM-based alert triage layer — all at zero infrastructure cost.

This repo documents the full build: architecture decisions, configuration, and — deliberately left in, because this is where the real learning happened — the troubleshooting process for every non-obvious failure encountered along the way.

Why This Exists

Most "SOC in a box" tutorials stop at docker-compose up and a screenshot of a dashboard with zero real alerts on it. This project goes further: real network capture, a genuinely misconfigured detection pipeline diagnosed and fixed from first principles, and a working (if intentionally minimal) AI triage layer reading live alert data — not a mockup.

The goal was to build something that demonstrates actual detection engineering ability, not just the ability to follow an install guide.
