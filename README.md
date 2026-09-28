# SentinelStack : AI-Augmented SOCLab 

A self-hosted Security Operations Center built from scratch on local virtualization infrastructure, combining network intrusion detection (Suricata), network security monitoring (Zeek), SIEM/XDR correlation (Wazuh), and a local LLM-based alert triage layer - all at zero infrastructure cost.

This repo documents the full build: architecture decisions, configuration, and - deliberately left in, because this is where the real learning happened - the troubleshooting process for every non-obvious failure encountered along the way.

**Architecture**


**Design principle**: every VM carries three NICs - NAT (internet access for package installs), an Internal Network in promiscuous mode (the "wire" Suricata and Zeek watch - this is the free, local equivalent of AWS Traffic Mirroring), and a Host-only adapter (for management access from the host). This gives full intra-lab traffic visibility without any paid cloud feature.

<img width="712" height="627" alt="image" src="https://github.com/user-attachments/assets/2dd3a03a-a7dc-4407-bc3c-98b1bc7fda3a" />



**Why This Exists**

Most "SOC in a box" tutorials stop at **docker-compose up** and a screenshot of a dashboard with zero real alerts on it. This project goes further: real network capture, a genuinely misconfigured detection pipeline diagnosed and fixed from first principles, and a working (if intentionally minimal) AI triage layer reading live alert data — not a mockup.

The goal was to build something that demonstrates actual detection engineering ability, not just the ability to follow an install guide.

**Design principle:** every VM carries three NICs — NAT (internet access for package installs), an Internal Network in promiscuous mode (the "wire" Suricata and Zeek watch — this is the free, local equivalent of AWS Traffic Mirroring), and a Host-only adapter (for management access from the host). This gives full intra-lab traffic visibility without any paid cloud feature.


<img width="685" height="511" alt="image" src="https://github.com/user-attachments/assets/d202b5c6-9856-4c30-92a7-987cabbdb15d" />



**What Was Actually Built**

->VirtualBox networking topology with three isolated network segments per VM, correctly scoped so Suricata/Zeek get full traffic visibility without exposing the lab to the host's home network

->Wazuh single-node deployment (manager, indexer, dashboard) via Docker Compose

->Suricata deployed in host-mode packet capture, feeding eve.json into Wazuh via agent log collection

->Zeek deployed alongside it, JSON-logging enabled, feeding **conn.log/http.log/dns.log** into the same pipeline

->Wazuh agents enrolled on both the detection host and the attack surface host, giving host-level (FIM, rootcheck, syscollector) visibility in addition to network-level detection

->A Python script that authenticates to the Wazuh indexer's REST API, pulls recent alerts filtered by severity, and passes them to a locally-running LLM for summarization of otherwise illegible alert fields.



<img width="771" height="162" alt="image" src="https://github.com/user-attachments/assets/d2676b71-8463-4929-8a56-20120867a74a" />

Attempting promiscuous (recon) activity onto the machine to generate alerts using basic Nmap.

<img width="1190" height="898" alt="image" src="https://github.com/user-attachments/assets/dea2b927-af2c-485a-b15b-dfbbb2e6a9cd" />

The alerts and fired rules as displayed over at Wazuh dashboard as a result of our deployed agent.



**Sample Output - AI Triage Layer**

<img width="1790" height="327" alt="image" src="https://github.com/user-attachments/assets/477a550a-0183-4d3e-917d-d027628a1f56" />

A working example of the model correctly summarizing raw Wazuh alert JSON into analyst-readable language, although I would be honest and document a limitation worth noting: a small local model without baseline context flags routine admin activity as noteworthy, since it has no concept of "expected" behavior on this specific host. Real deployments would need to feed additional context (known-admin allowlists, historical baselines) to reduce this kind of **over-flagging** - a genuine, first-hand observation about the limits of naive LLM-based triage, remediation of which obviously is included in my future scope.

**Implemented technologies and frameworks**


**Network architecture** - virtual network segmentation, promiscuous-mode traffic mirroring without paid cloud tooling.

**IDS/NSM configuration and debugging** - diagnosed checksum offload issues, rule-path mismatches, and ruleset scoping logic from raw logs rather than surface-level symptoms.

**SIEM deployment and log pipeline engineering** - Docker-based Wazuh deployment, agent-based log shipping architecture, custom ossec.conf log source configuration.

**Detection engineering fundamentals** - understanding of **HOME_NET/EXTERNAL_NET** scoping, signature-based detection logic, MITRE ATT&CK mapping.

**Systems administration** - netplan/cloud-init troubleshooting, systemd service debugging, Linux networking (multiple NICs, static/DHCP configuration).

**API integration** -  authenticated REST queries against both Wazuh's manager API and its OpenSearch-based indexer.

**LLM integration** - integrating a local LLM in form of Llama 3.2 into a real data pipeline, with honest evaluation of its limitations rather than uncritical acceptance of its output.






**Possible Extensions**

->Slack/webhook alerting for real-time notification instead of on-demand querying.

->Expanded prompt context (baseline "normal" behavior) to reduce AI false-positive flagging.

->File integrity monitoring tuning on the attack surface to catch post-exploitation artifacts and assist deeper triage.

->Migration to a cloud provider for scaling purposes.

->Include dynamic analysis tooling as an accessory for streamlining analyst workflow.

->Introducing a more proactive detection methodology/logic for the deployed agent.






