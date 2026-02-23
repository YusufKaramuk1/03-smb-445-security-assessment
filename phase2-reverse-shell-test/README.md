# Phase 2 – Client-Side Attack Simulation & Reverse Connection (Reverse Shell) Validation

> **Scope & Ethics Notice:** This phase was executed **only** in an isolated VirtualBox lab environment on local VMs for internship learning purposes. No external systems were targeted.

---

## Objective

Phase 1 showed that **inbound access** to common services on the Windows host (e.g., SMB/RDP/WinRM) is not possible because ports were **filtered** by firewall rules.

**Phase 2 goal:** Validate whether **host-level access and file retrieval** can still be achieved through a **client-side attack (user interaction)** that creates an **outbound reverse connection**, even when inbound services remain blocked.

---

## Lab Environment

| Role | OS / Hostname | IP Address | Notes |
|------|----------------|------------|------|
| Attacker | Kali Linux | `192.168.56.10` | Metasploit Framework + local HTTP server |
| Victim | Windows 10 (EVAL) | `192.168.56.11` | Windows Firewall enabled, inbound filtered |
| Network | VirtualBox Host-Only | `192.168.56.0/24` | Offline / isolated segment |

---

## Phase 2 Scenario (Realistic Simulation)

Instead of exploiting an exposed service port, this phase simulates a **user-assisted compromise**:

1. A **fake IT update portal** was created (phishing-style delivery).
2. The victim (Windows) **visited the page**, **downloaded** a file, and **executed** it.
3. Execution triggered an **outbound reverse connection** back to the attacker.
4. Due to **endpoint defense interference**, a **smash-and-grab** approach was used to retrieve a known file quickly before the session dropped.

---

## Evidence Index (Screenshots)

All screenshots are stored under:

- `phase2-reverse-shell-test/evidence/`

| # | Evidence | Filename |
|---|----------|----------|
| 1 | Payload generation | `01_payload_generation.png` |
| 2 | Handler setup | `02_handler_setup.png` |
| 3 | Fake portal + HTTP server | `03_http_server_and_html.png` |
| 4 | Victim download interaction | `04_victim_download.png` |
| 5 | Reverse connection / session opened | `05_session_opened.png` |
| 6 | File retrieval command (meterpreter download) | `06_file_download_from_target.png` |
| 7 | Proof on Kali (file content + attacker identity) | `07_proof_file_on_kali.png` |

---

## Step-by-Step Execution (Chronological)

### 1) Payload Preparation (Attacker)

A Windows executable was created to establish an outbound reverse connection to the attacker machine.

**Purpose**
- Bypass inbound firewall filtering
- Force the victim to initiate the connection (outbound)

📸 Evidence:  
`evidence/01_payload_generation.png`

---

### 2) Reverse Handler / Listener Configuration (Attacker)

A handler was configured to receive the reverse connection from the victim.

**Purpose**
- Prepare the attacker-side listener
- Accept an incoming reverse session once the victim executes the file

📸 Evidence:  
`evidence/02_handler_setup.png`

---

### 3) Delivery Vector – Fake IT Portal + Local HTTP Hosting (Attacker)

To simulate a realistic enterprise scenario, a simple internal IT update page was prepared and served to the victim using a local HTTP server (port 80).

**Purpose**
- Simulate phishing / social engineering delivery
- Make the payload available via a believable user-facing page

📸 Evidence:  
`evidence/03_http_server_and_html.png`

---

### 4) Victim-Side Interaction (Simulation)

From the Windows host, the victim user:
- visited the attacker-hosted page
- clicked the download button/link
- downloaded the executable
- executed the file (user-assisted execution)

**Purpose**
- Demonstrate a client-side compromise flow
- Show how user actions can bypass network-level protections

📸 Evidence:  
`evidence/04_victim_download.png`

---

### 5) Reverse Connection Established (Session Opened)

After execution, the victim system initiated an outbound connection to the attacker, and a meterpreter session opened successfully.

During the session:
- host information was gathered (system identity)
- user context was verified

**Purpose**
- Confirm host-level access was achieved
- Validate that reverse connection works even with inbound filtering

📸 Evidence:  
`evidence/05_session_opened.png`

---

### 6) Defensive Behavior Observed (Endpoint Interference)

While attempting post-exploitation actions, the following issues were encountered:

- Session stability was inconsistent
- Windows Defender / security controls interfered
- Broad searching triggered defensive reactions
- Process migration was attempted to improve stability, but interruptions continued
- Sessions were dropped unexpectedly (limited dwell time)

**Impact**
- Long, interactive enumeration became unreliable
- A faster approach was required to achieve the objective (file retrieval)

(These observations are reflected in the overall session behavior shown across the evidence.)

---

### 7) “Smash-and-Grab” File Retrieval (Known Path)

Because the target file location was known (Desktop), the workflow shifted from broad discovery to a fast **smash-and-grab** action:

- Instead of searching for files (higher detection risk),
- the file was retrieved directly using its known full path.

**Purpose**
- Minimize time spent in-session
- Retrieve evidence before the session is terminated
- Demonstrate realistic attacker adaptation under defensive pressure

📸 Evidence:  
`evidence/06_file_download_from_target.png`

---

### 8) Verification on Attacker (Kali)

After retrieval, the downloaded file was verified on Kali:

- File content was displayed to confirm successful retrieval
- Attacker machine identity (network interface/IP) was shown for evidence integrity

**Purpose**
- Prove the file exists on the attacker machine
- Demonstrate end-to-end success: victim → attacker file transfer

📸 Evidence:  
`evidence/07_proof_file_on_kali.png`

---

## Results & Observations

| Topic | Outcome |
|------|---------|
| Inbound firewall filtering | Still effective against direct service access |
| Reverse connection (outbound) | **Successful** (victim initiated connection) |
| Host-level access | **Obtained** via client-side execution |
| AV/Defender effect | Reduced stability, caused session drops |
| Enumeration approach | Broad search increased interference risk |
| Data retrieval approach | Known-path “smash-and-grab” was effective |
| File retrieval | **Successful** (file obtained and verified on Kali) |

---

## Key Security Takeaways

- **Inbound firewall rules are not sufficient alone.**
- **Outbound connections (egress)** can enable compromise and data extraction even when inbound ports are filtered.
- **User interaction** remains a major risk vector (social engineering).
- **Endpoint defenses** may reduce session stability, but fast attacker actions can still succeed.
- Strong security requires **layered controls**:
  - egress filtering
  - endpoint monitoring/EDR
  - application control/whitelisting
  - user awareness

---

## Conclusion

Phase 2 confirmed that:

Even with inbound ports filtered by firewall, a client-side execution path can result in:
- reverse connection establishment,
- host-level access,
- and rapid file retrieval (exfiltration) under defensive pressure.

This highlights the importance of **egress control and endpoint visibility** in addition to perimeter firewalling.
