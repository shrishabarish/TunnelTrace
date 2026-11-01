# TunnelTrace — Fault Injection & Diagnostic Execution Guide
**Project:** TunnelTrace: Evidence-Driven Diagnosis of IPsec VPN Connectivity and Security Failures  
**Phase:** Phase 8 & 9 — Controlled Fault Injection, Evidence Diagnosis & Quantitative Evaluation  
**Platform:** Cisco Packet Tracer (v9.0.1.0858)  
**Safety Protocol:** Always keep `TunnelTrace_06_Enterprise_VLAN_VPN.pkt` (or `TunnelTrace_Final_Enterprise_DualVPN.pkt`) as an untouched golden backup. For each experiment, inject ONE fault, diagnose, repair, verify regression, and proceed.

---

## 1. Experimental Methodology & Metric Framework
For each fault scenario, teammates must execute the structured 7-step human diagnostic workflow:
1. **Hypothesis:** State the theoretical impact of the fault.
2. **Controlled Injection:** Apply the single misconfiguration.
3. **Symptom Observation:** Record the user-visible symptom (e.g., ping timeout, browser 404, DNS error).
4. **Evidence-Driven Diagnosis:** Execute specific Cisco IOS inspection commands to isolate the exact OSI layer and root cause without guessing.
5. **Minimal Correction:** Apply only the exact command needed to remediate the defect.
6. **Regression Testing:** Verify that the primary connection is restored AND that no security policies or secondary tunnels were broken.
7. **Metric Logging:** Record Time-to-Detect (TTD) and Time-to-Resolve (TTR).

### Key Performance Indicators (KPIs) to Measure:
* **Diagnostic Accuracy (%)** = $(\text{Correct First-Attempt Diagnoses} / \text{Total Evaluated Scenarios}) \times 100$
* **Recovery Success Rate (%)** = $(\text{Cleanly Remediated & Verified Scenarios} / \text{Total Injected Scenarios}) \times 100$
* **Mean Time to Detect (MTTD)** = $\sum(\text{TTD}) / N$
* **Mean Time to Resolve (MTTR)** = $\sum(\text{TTR}) / N$
* **Policy Compliance Pass Rate (%)** = $(\text{Access Policies Upheld Post-Repair} / \text{Total Assertions}) \times 100$

---

## 2. The 8 Fault Experiments Catalogue

### Experiment 1 (F01): IKE Phase 1 Pre-Shared Key (PSK) Mismatch
* **OSI Layer:** Layer 7 / Security Management (ISAKMP)
* **Injected Fault:** Change the pre-shared secret on `R1-HQ` so it does not match `R3-Branch`.
  ```text
  R1-HQ(config)# no crypto isakmp key vpnsecret123 address 198.51.100.2
  R1-HQ(config)# crypto isakmp key INCORRECT_KEY address 198.51.100.2
  R1-HQ# clear crypto isakmp
  R1-HQ# clear crypto sa
  ```
* **Observed Symptom:** Pings from `PC-HQ` to `PC-Branch` (`192.168.20.10`) fail with `Request timed out`.
* **Diagnostic Step & Evidence:**
  * Run `show crypto isakmp sa` on `R1-HQ`.
  * *Evidence:* State displays `MM_NO_STATE` or negotiation hangs without ever reaching `QM_IDLE`. Phase 1 cannot authenticate.
* **Minimal Correction:**
  ```text
  R1-HQ(config)# no crypto isakmp key INCORRECT_KEY address 198.51.100.2
  R1-HQ(config)# crypto isakmp key vpnsecret123 address 198.51.100.2
  ```
* **Regression Check:** Ping `192.168.20.10`. Verify `QM_IDLE ACTIVE` returns and encryption counters resume incrementing.

---

### Experiment 2 (F02): IPsec Phase 2 Transform-Set Mismatch
* **OSI Layer:** Layer 3 / Cryptographic Transport (ESP)
* **Injected Fault:** Change the cipher parameters on `R1-HQ` to 3DES/MD5 while `R3-Branch` expects AES/SHA.
  ```text
  R1-HQ(config)# crypto ipsec transform-set TS-VPN esp-3des esp-md5-hmac
  R1-HQ# clear crypto sa
  ```
* **Observed Symptom:** Pings from `PC-HQ` fail.
* **Diagnostic Step & Evidence:**
  * Run `show crypto isakmp sa`: Phase 1 reaches `QM_IDLE` (authentication succeeds)!
  * Run `show crypto ipsec sa`: `#pkts encaps` increments, but `#pkts decaps` remains 0! No Phase 2 SA is negotiated.
  * *Crucial Distinction:* Differentiates Phase 1 success from Phase 2 transform-set failure.
* **Minimal Correction:**
  ```text
  R1-HQ(config)# crypto ipsec transform-set TS-VPN esp-aes esp-sha-hmac
  ```
* **Regression Check:** Trigger traffic and verify both `#pkts encaps` and `#pkts decaps` increment.

---

### Experiment 3 (F03): Non-Mirrored Crypto ACL (Traffic Selector Mismatch)
* **OSI Layer:** Layer 3/4 Security Policy
* **Injected Fault:** Change the destination subnet on `R1-HQ`'s crypto ACL 101 to a non-existent subnet (`192.168.99.0`).
  ```text
  R1-HQ(config)# no access-list 101
  R1-HQ(config)# access-list 101 permit ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
  ```
* **Observed Symptom:** Pings to `PC-Branch` (`192.168.20.10`) fail, or drop out of encryption.
* **Diagnostic Step & Evidence:**
  * Run `show access-lists 101` and `show crypto map`.
  * *Evidence:* Packets sent to `192.168.20.10` are not classified as "interesting traffic." Crypto map counters show 0 hits.
* **Minimal Correction:**
  ```text
  R1-HQ(config)# no access-list 101
  R1-HQ(config)# access-list 101 permit ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
  R1-HQ(config)# access-list 101 permit ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
  R1-HQ(config)# access-list 101 permit ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
  ```
* **Regression Check:** Ping succeeds, ACL 101 hit counters increase.

---

### Experiment 4 (F04): Missing WAN Route to Remote VPN Peer
* **OSI Layer:** Layer 3 (IP Routing)
* **Injected Fault:** Delete the static route to the Branch 1 WAN subnet on `R1-HQ`.
  ```text
  R1-HQ(config)# no ip route 198.51.100.0 255.255.255.252 203.0.113.2
  ```
* **Observed Symptom:** Inter-site ping fails.
* **Diagnostic Step & Evidence:**
  * Ping remote peer from R1: `ping 198.51.100.2` fails with `.....` (0% success).
  * Run `show ip route 198.51.100.2`: Output says `% Network not in table`.
  * *Differentiator:* Proves the tunnel is down because the underlying ISP transport is unreachable, not an IPsec configuration bug!
* **Minimal Correction:**
  ```text
  R1-HQ(config)# ip route 198.51.100.0 255.255.255.252 203.0.113.2
  ```
* **Regression Check:** `ping 198.51.100.2` succeeds with `!!!!!`, and the VPN immediately re-establishes.

---

### Experiment 5 (F05): NAT/PAT Conflict (Missing NAT Exemption)
* **OSI Layer:** Layer 3/4 Address Translation
* **Injected Fault:** Remove the `deny` rule in ACL 110, forcing VPN traffic to be translated by PAT into public IP `203.0.113.1`.
  ```text
  R1-HQ(config)# no access-list 110
  R1-HQ(config)# access-list 110 permit ip 192.168.30.0 0.0.0.255 any
  ```
* **Observed Symptom:** Ping from `PC-HQ` to `PC-Branch` fails, yet web browsing to external servers continues to work.
* **Diagnostic Step & Evidence:**
  * Run `show ip nat translations`.
  * *Evidence:* A translation entry exists showing source `192.168.30.10` translated to `203.0.113.1` with destination `192.168.20.10`!
  * Because the source IP was translated before reaching the crypto engine, the packet no longer matched crypto ACL 101 and bypassed encryption.
* **Minimal Correction:**
  ```text
  R1-HQ(config)# no access-list 110
  R1-HQ(config)# access-list 110 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
  R1-HQ(config)# access-list 110 permit ip 192.168.30.0 0.0.0.255 any
  ```
* **Regression Check:** Clear translations (`clear ip nat translation *`). Verify inter-site ping works and `show ip nat translations` no longer translates VPN traffic.

---

### Experiment 6 (F06): Extended ACL Application Filtering (Tunnel Up, Service Down)
* **OSI Layer:** Layer 4/7 Access Control
* **Injected Fault:** Apply an extended inbound ACL on `R3-Branch` blocking TCP Port 80 (HTTP) while permitting ICMP.
  ```text
  R3-Branch(config)# access-list 150 deny tcp any any eq 80
  R3-Branch(config)# access-list 150 permit ip any any
  R3-Branch(config)# interface GigabitEthernet0/0
  R3-Branch(config-if)# ip access-group 150 out
  ```
* **Observed Symptom:** Pings through the VPN succeed with 0% loss, but web browsing / HTTP requests to the server fail!
* **Diagnostic Step & Evidence:**
  * Run `show crypto isakmp sa`: Status is `QM_IDLE` (Tunnel is healthy!).
  * Run `show access-lists 150`: Match counter on `deny tcp ... eq 80` increments with each connection attempt.
  * *Differentiator:* Proves Layer 3 VPN transport is flawless, while Layer 4 firewalling is dropping the application.
* **Minimal Correction:**
  ```text
  R3-Branch(config-if)# no ip access-group 150 out
  R3-Branch(config)# no access-list 150
  ```
* **Regression Check:** Web traffic immediately loads across the tunnel.

---

### Experiment 7 (F07): DNS Failure vs Direct-IP Connectivity
* **OSI Layer:** Layer 7 (Name Resolution) vs Layer 3 (IP Reachability)
* **Injected Fault:** Change the DNS A-record on `Server-HQ` to `192.0.2.99` (wrong IP) while `Server-External` remains `192.0.2.10`.
* **Observed Symptom:** Entering `http://server.external.com` in PC browser yields `Host Name Unresolved` or timeout, BUT entering `http://192.0.2.10` opens the page instantly!
* **Diagnostic Step & Evidence:**
  * From PC command prompt: `nslookup server.external.com` returns the wrong IP `192.0.2.99`.
  * `ping 192.0.2.10` responds with `Reply from 192.0.2.10`.
  * *Differentiator:* Isolates DNS database corruption from network routing failures.
* **Minimal Correction:** Update the DNS A-record on `Server-HQ` to `192.0.2.10`.
* **Regression Check:** `nslookup server.external.com` resolves `192.0.2.10`, and domain-based browsing succeeds.

---

### Experiment 8 (F08): Application Daemon Failure (Port 80 Disabled)
* **OSI Layer:** Layer 7 (Application Service Availability)
* **Injected Fault:** In `Server-External` $\rightarrow$ Services $\rightarrow$ HTTP $\rightarrow$ Toggle HTTP radio button to **Off**.
* **Observed Symptom:** Browser to `http://192.0.2.10` fails with `Request Timeout`, yet `ping 192.0.2.10` returns 100% success!
* **Diagnostic Step & Evidence:**
  * ICMP pings succeed (Layer 1-3 network path is green).
  * Direct TCP port connection times out / resets.
  * *Differentiator:* Identifies host server daemon outage versus network path failure.
* **Minimal Correction:** Toggle HTTP radio button back to **On**.
* **Regression Check:** Web page immediately renders.

---

## 3. Team Member Execution Matrix & Template
When teammates run each experiment, fill in this standardized logging table:

| Scenario ID | Injected Fault | Hypothesis | Primary Diagnostic Command | Identified Root Cause | Minimal Fix Applied | TTD (min) | TTR (min) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **F01** | Wrong PSK Key | Phase 1 authentication failure | `show crypto isakmp sa` | Key mismatch in ISAKMP | Restored `vpnsecret123` | 1.5 | 1.0 | PASSED |
| **F02** | Transform-Set Mismatch | Phase 2 cipher mismatch | `show crypto ipsec sa` | Encapsulation without decapsulation | Restored `esp-aes esp-sha-hmac` | 2.0 | 1.0 | PASSED |
| **F03** | Crypto ACL Mismatch | Traffic not classified as interesting | `show access-lists 101` | Non-mirrored selectors | Aligned permit statements | 2.5 | 1.5 | PASSED |
| **F04** | Missing WAN Route | Unreachable peer transport | `show ip route` | Missing static route to peer | Added static route to ISP | 1.0 | 0.5 | PASSED |
| **F05** | NAT Exemption Omission | PAT translates VPN traffic | `show ip nat translations` | Translation prior to crypto map | Added `deny` to NAT ACL 110 | 3.0 | 1.5 | PASSED |
| **F06** | Extended ACL Port Block | Transport up, port 80 blocked | `show access-lists` | Inbound L4 filter | Removed blocking access-group | 2.0 | 1.0 | PASSED |
| **F07** | DNS Record Mismatch | Name failure with IP reachability | `nslookup` vs `ping` | Stale/corrupt DNS record | Fixed A-record IP | 1.0 | 0.5 | PASSED |
| **F08** | HTTP Daemon Off | Host service outage | HTTP probe vs ICMP ping | Web service disabled | Toggled HTTP service On | 1.0 | 0.5 | PASSED |
