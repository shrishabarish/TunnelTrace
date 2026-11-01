# TunnelTrace — Completed Implementation Report
**Project Title:** TunnelTrace: Evidence-Driven Diagnosis of IPsec VPN Connectivity and Security Failures  
**Platform:** Cisco Packet Tracer (v9.0.1.0858)  
**Implementation Status:** Baseline & Enterprise Multi-Branch Infrastructure 100% Complete & Verified  
**Date of Completion:** 2026-10-19  

---

## 1. Executive Summary & Architecture Overview
The foundational network infrastructure of **TunnelTrace** has been successfully designed, cabled, configured, and verified. Beyond the baseline requirement of a single point-to-point VPN, the architecture has been expanded into an **Enterprise Multi-Branch Hub-and-Spoke Topology** featuring:
1. **HQ Campus Enterprise VLAN Segmentation (IEEE 802.1Q Router-on-a-Stick)** with dedicated subinterfaces for Administrative Management, Staff Engineering, and Server DMZ.
2. **Dual-Tunnel Enterprise IPsec VPN Concentrator** on `R1-HQ`, maintaining simultaneous encrypted tunnels to **Branch Office 1** and **Branch Office 2**.
3. **Port Address Translation (PAT/NAT Overload) with Multi-Subnet VPN Exemption**, preventing translation collisions with VPN traffic while allowing outbound internet access.
4. **Application Services Layer** with functional HTTP Web services and DNS hostname resolution (`server.external.com`).
5. **Corporate Office Infrastructure** featuring a Wireless Access Point (`WAP-HQ`) serving wireless clients (`Laptop-Staff`, `Phone-Staff`) and a dedicated Network Printer (`Printer-HQ`).

---

## 2. Implemented Device Inventory
| Device Identifier | Hardware Model | Operational Role | Interface Connections |
| :--- | :--- | :--- | :--- |
| **R1-HQ** | Cisco 2911 ISR (`securityk9`) | HQ Gateway & VPN Concentrator | `Gig0/0` (Trunk), `Gig0/1` (WAN to ISP) |
| **R2-ISP** | Cisco 2911 ISR | Simulated Public WAN / Internet Exchange | `Gig0/1` (to HQ), `Gig0/2` (to Branch 1), `Gig0/0` (to External LAN/Branch 2) |
| **R3-Branch** | Cisco 2911 ISR (`securityk9`) | Branch Office 1 Gateway | `Gig0/1` (WAN to ISP), `Gig0/0` (Branch 1 LAN) |
| **R4-Branch2** | Cisco 2911 ISR (`securityk9`) | Branch Office 2 Gateway | `Gig0/1` (WAN to ISP via SW3), `Gig0/0` (Branch 2 LAN) |
| **SW1-HQ** | Cisco 2960-24TT | HQ Campus Distribution/Access Switch | `Fa0/3` (Trunk to R1), `Fa0/1` (Admin), `Fa0/2` (Staff), `Fa0/4` (Server), `Fa0/5` (WAP), `Fa0/6` (Printer) |
| **SW2-Branch** | Cisco 2960-24TT | Branch 1 LAN Switch | `Fa0/2` (to R3), `Fa0/1` (to PC-Branch) |
| **SW3-External** | Cisco 2960-24TT | Public ISP Edge Switch | `Fa0/2` (to R2), `Fa0/1` (PC-External), `Fa0/3` (Server-External), `Fa0/4` (to R4) |
| **SW4-Branch2** | Cisco 2960-24TT | Branch 2 LAN Switch | `Fa0/1` (to R4), `Fa0/2` (PC-Branch2), `Fa0/3` (Laptop-Branch2) |
| **PC-HQ-Admin** | PC-PT | HQ Admin Workstation (VLAN 10) | `Fa0` to `SW1-HQ Fa0/1` |
| **PC-HQ** | PC-PT | HQ Staff Workstation (VLAN 20) | `Fa0` to `SW1-HQ Fa0/2` |
| **Server-HQ** | Server-PT | Internal DNS & Application Server (VLAN 30) | `Fa0` to `SW1-HQ Fa0/4` |
| **Printer-HQ** | Printer-PT | Corporate Network Printer (VLAN 10) | `Fa0` to `SW1-HQ Fa0/6` |
| **WAP-HQ** | AccessPoint-PT | HQ Wireless Access Point (VLAN 20) | `Port 0` to `SW1-HQ Fa0/5` |
| **Laptop-Staff** | Laptop-PT (`WPC300N`) | HQ Wireless Staff Client (VLAN 20) | Wireless 802.11 association to `WAP-HQ` |
| **Phone-Staff** | Smartphone-PT | HQ Wireless Mobile Client (VLAN 20) | Wireless 802.11 association to `WAP-HQ` |
| **PC-Branch** | PC-PT | Branch Office 1 Client | `Fa0` to `SW2-Branch Fa0/1` |
| **PC-Branch2** | PC-PT | Branch Office 2 Desktop Client | `Fa0` to `SW4-Branch2 Fa0/2` |
| **Laptop-Branch2**| Laptop-PT | Branch Office 2 Laptop Client | `Fa0` to `SW4-Branch2 Fa0/3` |
| **Server-External**| Server-PT | Public Web Server (`192.0.2.10`) | `Fa0` to `SW3-External Fa0/3` |
| **PC-External** | PC-PT | Public Internet Client (`192.0.2.20`) | `Fa0` to `SW3-External Fa0/1` |

---

## 3. Master IPv4 Addressing Table
| Node / Interface | IP Address | Subnet Mask | Default Gateway | VLAN / Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **R1-HQ `Gig0/0`** | Unassigned | None | N/A | 802.1Q Physical Trunk Carrier |
| **R1-HQ `Gig0/0.10`** | `192.168.10.1` | `255.255.255.0` | N/A | VLAN 10 (HQ Admin Gateway) |
| **R1-HQ `Gig0/0.20`** | `192.168.30.1` | `255.255.255.0` | N/A | VLAN 20 (HQ Staff Gateway) |
| **R1-HQ `Gig0/0.30`** | `192.168.40.1` | `255.255.255.0` | N/A | VLAN 30 (HQ Server DMZ Gateway) |
| **R1-HQ `Gig0/1`** | `203.0.113.1` | `255.255.255.252` | N/A | HQ Public WAN Endpoint |
| **R2-ISP `Gig0/1`** | `203.0.113.2` | `255.255.255.252` | N/A | ISP Gateway towards HQ |
| **R2-ISP `Gig0/2`** | `198.51.100.1` | `255.255.255.252` | N/A | ISP Gateway towards Branch 1 |
| **R2-ISP `Gig0/0`** | `192.0.2.1` | `255.255.255.0` | N/A | ISP Public Server & Branch 2 WAN Subnet |
| **R3-Branch `Gig0/1`** | `198.51.100.2` | `255.255.255.252` | N/A | Branch 1 Public WAN Endpoint |
| **R3-Branch `Gig0/0`** | `192.168.20.1` | `255.255.255.0` | N/A | Branch 1 LAN Gateway |
| **R4-Branch2 `Gig0/1`**| `192.0.2.2` | `255.255.255.0` | `192.0.2.1` | Branch 2 Public WAN Endpoint |
| **R4-Branch2 `Gig0/0`**| `192.168.50.1` | `255.255.255.0` | N/A | Branch 2 LAN Gateway |
| **PC-HQ-Admin** | `192.168.10.20` | `255.255.255.0` | `192.168.10.1` | VLAN 10 Member |
| **Printer-HQ** | `192.168.10.50` | `255.255.255.0` | `192.168.10.1` | VLAN 10 Member |
| **PC-HQ** | `192.168.30.10` | `255.255.255.0` | `192.168.30.1` | VLAN 20 Member |
| **Laptop-Staff** | `192.168.30.25` | `255.255.255.0` | `192.168.30.1` | VLAN 20 Wireless Member |
| **Server-HQ** | `192.168.40.100`| `255.255.255.0` | `192.168.40.1` | VLAN 30 DMZ Server |
| **PC-Branch** | `192.168.20.10` | `255.255.255.0` | `192.168.20.1` | Branch 1 Operations Host |
| **PC-Branch2** | `192.168.50.10` | `255.255.255.0` | `192.168.50.1` | Branch 2 Desktop Client |
| **Laptop-Branch2**| `192.168.50.20` | `255.255.255.0` | `192.168.50.1` | Branch 2 Laptop Client |
| **Server-External**| `192.0.2.10` | `255.255.255.0` | `192.0.2.1` | Public Internet Web Server |
| **PC-External** | `192.0.2.20` | `255.255.255.0` | `192.0.2.1` | Public Internet Host |

---

## 4. Phase-by-Phase Completed Work

### Phase 0: Environment & Licensing Compatibility
* **Packet Tracer Version:** Confirmed `9.0.1.0858`.
* **Hardware Licensing:** Discovered that Cisco ISR 2911 defaults to `ipbasek9`. Successfully activated `securityk9` evaluation package via CLI (`license boot module c2900 technology-package securityk9`) on `R1-HQ`, `R3-Branch`, and `R4-Branch2`.
* **Command Verification:** Verified CLI acceptance of `crypto isakmp policy`, `crypto ipsec transform-set`, and `crypto map`.

### Phase 1 & 2: Topology, Cabling & IP Subnetting
* 12 devices initially placed and cabled. All links operational (`up/up`).
* Subnetted private addressing spaces (`192.168.10.0/24`, `192.168.20.0/24`, `192.168.30.0/24`, `192.168.40.0/24`, `192.168.50.0/24`) and public simulated WAN ranges (`203.0.113.0/30`, `198.51.100.0/30`, `192.0.2.0/24`).

### Phase 3: Baseline IP Routing
* Static routes deployed across R1, R2, and R3 to ensure underlying bidirectional routability prior to cryptographic encapsulation.
* Verified PC-to-gateway and cross-WAN client-to-client reachability (`TTL=125`, 0% loss).

### Phase 4: Site-to-Site IKEv1/IPsec VPN (Tunnel 1)
* **IKE Phase 1 (ISAKMP):** AES-128 encryption, SHA-1 integrity, Pre-Shared Key (`vpnsecret123`), DH Group 2, 86400s lifetime.
* **IPsec Phase 2:** Transform set `TS-VPN` (`esp-aes esp-sha-hmac`).
* **Interesting Traffic Selectors:** Mirrored extended ACLs matching HQ local subnets to Branch 1 subnets.
* **Verification Proof:** `show crypto isakmp sa` confirmed `QM_IDLE ACTIVE`. `show crypto ipsec sa` confirmed hardware-level encapsulation (`#pkts encaps: 3, #pkts encrypt: 3, #pkts decaps: 3, #pkts decrypt: 3`).

### Phase 5: NAT/PAT with Multi-Subnet VPN Exemption
* Configured PAT overload on `R1-HQ` `Gig0/1` using ACL 110.
* Implemented **NAT Exemption (no-nat)**: explicitly denied VPN-destined traffic from being translated so private packet headers match the crypto map selectors.
* **Verification Proof:** `show ip nat translations` captured active PAT translation (`192.168.10.10:1031 -> 203.0.113.1:1031`), while simultaneous inter-site pings maintained `QM_IDLE ACTIVE` without tunnel disruption.

### Phase 6: Application Services & DNS Resolution
* Enabled HTTP on `Server-External` (`192.0.2.10`).
* Configured DNS A-record on `Server-HQ`: `server.external.com -> 192.0.2.10`.
* Verified HTTP rendering via Packet Tracer Desktop Web Browser from internal PC clients.

### Enterprise Expansion: 802.1Q VLANs & Dual-Spoke Concentrator
* **VLAN Trunking:** Converted `SW1-HQ Fa0/3` and `R1-HQ Gig0/0` into an 802.1Q trunk carrying VLAN 10 (Admin), VLAN 20 (Staff), and VLAN 30 (Servers). Verified inter-VLAN routing with `TTL=127`.
* **Branch Office 2 Deployment:** Integrated `R4-Branch2` (`192.168.50.0/24`), established second independent IPsec tunnel (Sequence 20 on `R1-HQ`), and verified simultaneous dual-tunnel negotiation.
* **Office Hardware:** Added `WAP-HQ`, `Laptop-Staff`, `Phone-Staff`, and `Printer-HQ`.

---

## 5. Summary of Verified Evidence Captures
1. `00_compatibility_check.png`: Cisco 2911 accepting `crypto isakmp policy 10` after `securityk9` activation.
2. `01_topology_labels.png`: Physical port labels displayed and mapped.
3. `02_baseline_ping.png`: Unencrypted ping from `PC-HQ` to `PC-Branch` (`192.168.20.10`, `TTL=125`).
4. `03_ipsec_active_sa.png`: `show crypto isakmp sa` confirming `QM_IDLE ACTIVE` (conn-id 1093).
5. `04_ipsec_counters.png`: `show crypto ipsec sa` confirming encryption and decryption counters incrementing.
6. `05_nat_translations.png`: `show ip nat translations` proving PAT port translation alongside VPN bypass.
7. `06_http_browser.png`: Successful web render of `http://server.external.com` via DNS resolution.
8. `07_intervlan_routing.png`: Successful inter-VLAN ping between VLAN 10, VLAN 20, and VLAN 30 (`TTL=127`).
9. `08_branch2_tunnel.png`: Successful dual-tunnel negotiation reaching `192.168.50.10` (`TTL=125`).
