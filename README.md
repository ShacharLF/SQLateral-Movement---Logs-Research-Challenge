# SQLateral-Movement---Logs-Research-Challenge
A blue-team detection engineering dataset capturing a fileless identity-abuse attack chain (WMI, Kerberoasting, Silver Ticket, and DCSync) inside an AD domain environment. Formatted in JSON for Splunk ingestion and training (can be used as ctf).


SQLateral Movement - Detection Engineering Lab Data & Attack Flow

This repository contains a dataset of custom-generated Windows Event Viewer and Sysmon logs (JSON format) capturing a complete, multi-stage Identity-Based and Lateral Movement attack lifecycle within an Active Directory Domain environment. 

This repository is designed as a raw lab infrastructure. Instructors, SOC Managers, and Training Leads can ingest these logs into their own Splunk instance to build customized threat hunting challenges, CTFs, or technical interview assessments.

---

## The Scenario Background
An attacker connects a non-domain-joined rogue asset directly to a physical network port. Armed with compromised domain credentials (`HOSHEN\eliyanko`), the attacker initiates a stealthy, fileless attack chain moving from an endpoint, leveraging Kerberos vulnerabilities, and pivoting into production SQL database infrastructure, before attempting a domain-wide takeover.

## Infrastructure & Ingestion
1. The raw telemetry is stored in JSON format inside the `logs/` directory.
2. Create a dedicated index in your Splunk instance (e.g., `index=sql_lateral`).
3. Ingest the JSON files with proper timestamp extraction and set the sourcetype to `_json`.

---

##  Chronological Attack Flow & Telemetry Mapping
Use this master breakdown to understand the sequence of events and map your own questions/queries:

### Stage 1: Initial Remote Access (Living off the Land)
* **The Action:** The attacker leverages RPC/WMI from a non-domain asset (`192.101.10.1`) to execute remote commands on a domain workstation (`SHACHCHE-014` / `192.101.20.3`) using the compromised credentials of `HOSHEN\eliyanko`.
* **Telemetry Evidence:** Look for network logon events (Logon Type 3) under Event ID 4624, coupled with suspicious parent-child process relationships spawned by WMI services (e.g., `wmiprvse.exe` launching `cmd.exe` or `powershell.exe`).

### Stage 2: Identity Abuse & Credential Harvesting (Kerberoasting)
* **The Action:** Once inside the domain workstation, the attacker executes a Kerberoasting attack to request TGS (Ticket Granting Service) tickets for high-value service accounts, aiming to crack their hashes offline.
* **Telemetry Evidence:** Look for active Directory Service Ticket Requests (Event ID 4769) with weak encryption types (RC4 / 0x17) or unusual volume from a single host.

### Stage 3: Privilege Escalation & Database Pivot (Silver Ticket)
* **The Action:** The attacker forges a Silver Ticket for the SQL Server service account, bypassing traditional authentication boundaries and elevating privileges directly inside the database management system.
* **Telemetry Evidence:** Look for anomalous Event ID 4624 (Successful Logon) to the SQL server with target service names or forged ticket signatures that don't align with a preceding TGT request (Event ID 4768).

### Stage 4: Remote Execution & Persistence
* **The Action:** The attacker uses their elevated database privileges or administrative rights to create a remote scheduled task / WMI subscription to maintain a persistent foothold on key domain infrastructure.
* **Telemetry Evidence:** Look for Scheduled Task creation/modification logs (Event ID 4698) or Sysmon Event IDs 19, 20, 21 (WMI Event Consumers/Filters).

### Stage 5: Domain Compromise (DCSync)
* **The Action:** The final objective—the attacker executes a DCSync attack against the Domain Controller to mimic directory replication requests, extracting the NTLM hash of the highly critical `KRBTGT` account to establish "End Game" domain persistence.
* **Telemetry Evidence:** Look for Directory Service Access events (Event ID 4662) on the Domain Controller, specifically requesting replication rights (`DS-Replication-Get-Changes-All`) from a non-DC computer account.

---

## Suggested Use Cases for Instructors
* **SOC Challenge:** Provide analysts with the logs after inserting it to Splunk and build quistions they need to ansewr through the logs research based on the attack flow.
