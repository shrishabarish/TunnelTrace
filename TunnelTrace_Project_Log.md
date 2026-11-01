# TunnelTrace Project Log

**Project:** TunnelTrace: Evidence-Driven Diagnosis of IPsec VPN Connectivity and Security Failures  
**Platform:** Cisco Packet Tracer only  
**Software Version:** Cisco Packet Tracer 9.0.1.0858  
**Operating System:** Windows  
**Current Phase:** Phase 1 — Topology Creation and Cabling (Phase 0 PASSED)  
**Last Updated:** 2026-10-19  

---

## 1. Selected Devices
| Device Name | Selected Model | Role | Status |
| :--- | :--- | :--- | :--- |
| **R1-HQ** | Cisco 2911 (securityk9) | HQ Gateway & VPN Endpoint | Verified Compatible |
| **R2-ISP** | Cisco 2911 | Simulated WAN / ISP Gateway | Planned |
| **R3-Branch** | Cisco 2911 (securityk9) | Branch Gateway & VPN Endpoint | Verified Compatible |
| **SW1-HQ** | Cisco 2960-24TT | HQ LAN Switch | Planned |
| **SW2-Branch** | Cisco 2960-24TT | Branch LAN Switch | Planned |
| **SW3-External** | Cisco 2960-24TT | External LAN Switch | Planned |
| **PC-HQ** | PC-PT | HQ Test Client | Planned |
| **PC-HQ-Admin** | PC-PT | HQ Admin Client | Planned |
| **PC-Branch** | PC-PT | Branch Test Client | Planned |
| **PC-External** | PC-PT | External Test Client | Planned |
| **Server-HQ** | Server-PT | HQ Web / DNS Server | Planned |
| **Server-External** | Server-PT | External Web Server | Planned |

---

## 2. Enterprise VLAN & Subnet Architecture
### Headquarters (HQ) Campus:
- **VLAN 10 (Admin & Management):** `192.168.10.0/24` | Gateway: `192.168.10.1` (`Gig0/0.10`)
  - Member: `PC-HQ-Admin` (`192.168.10.20`) on `SW1-HQ Fa0/1`
- **VLAN 20 (Engineering & Staff):** `192.168.30.0/24` | Gateway: `192.168.30.1` (`Gig0/0.20`)
  - Member: `PC-HQ` (`192.168.30.10`) on `SW1-HQ Fa0/2`
- **VLAN 30 (Enterprise Server DMZ):** `192.168.40.0/24` | Gateway: `192.168.40.1` (`Gig0/0.30`)
  - Member: `Server-HQ` (`192.168.40.100`) on `SW1-HQ Fa0/4`
- **Trunk Link:** `SW1-HQ Fa0/3` $\longleftrightarrow$ `R1-HQ Gig0/0` (802.1Q Tagged Trunk)

### Branch Campus:
- **VLAN 50 (Branch Operations):** `192.168.20.0/24` | Gateway: `192.168.20.1` (`Gig0/0.50`)
  - Member: `PC-Branch` (`192.168.20.10`) on `SW2-Branch Fa0/1`
- **Trunk Link:** `SW2-Branch Fa0/2` $\longleftrightarrow$ `R3-Branch Gig0/0` (802.1Q Tagged Trunk)

---

## 3. Router Interface & Subinterface Map
| Router | Interface / Subinterface | 802.1Q VLAN | IP Address | Subnet Mask | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1-HQ** | `GigabitEthernet0/0` | Physical Trunk | *(no IP)* | *(no mask)* | 802.1Q Trunk to SW1-HQ Fa0/3 |
| **R1-HQ** | `Gig0/0.10` | VLAN 10 | `192.168.10.1` | `255.255.255.0` | HQ Admin Gateway |
| **R1-HQ** | `Gig0/0.20` | VLAN 20 | `192.168.30.1` | `255.255.255.0` | HQ Staff Gateway |
| **R1-HQ** | `Gig0/0.30` | VLAN 30 | `192.168.40.1` | `255.255.255.0` | HQ Server DMZ Gateway |
| **R1-HQ** | `GigabitEthernet0/1` | Native | `203.0.113.1` | `255.255.255.252` | WAN Link to R2-ISP Gig0/1 |
| **R2-ISP** | `GigabitEthernet0/1` | Native | `203.0.113.2` | `255.255.255.252` | WAN Link to HQ |
| **R2-ISP** | `GigabitEthernet0/2` | Native | `198.51.100.1` | `255.255.255.252` | WAN Link to Branch |
| **R2-ISP** | `GigabitEthernet0/0` | Native | `192.0.2.1` | `255.255.255.0` | External LAN Gateway |
| **R3-Branch**| `GigabitEthernet0/0` | Physical Trunk | *(no IP)* | *(no mask)* | 802.1Q Trunk to SW2-Branch Fa0/2 |
| **R3-Branch**| `Gig0/0.50` | VLAN 50 | `192.168.20.1` | `255.255.255.0` | Branch Operations Gateway |
| **R3-Branch**| `GigabitEthernet0/1` | Native | `198.51.100.2` | `255.255.255.252` | WAN Link to R2-ISP Gig0/2 |

---

## 4. Work Completed & Diagnostic Notes
- **2026-10-19:** Recorded Packet Tracer version `9.0.1.0858`.
- **2026-10-19:** Identified and activated Cisco 2911 `securityk9` evaluation license.
- **2026-10-19 [PHASE 1 & 2 PASSED]:**
  - **Phase 1 (Topology):** All 12 devices placed, cabled, and verified.
  - **Phase 2 (Addressing):** 
    - R1-HQ, R2-ISP, and R3-Branch interfaces configured with exact IPs and verified `up/up`.
    - All 6 end devices configured with static IPv4 addresses, masks, and gateways.
    - Verified local gateway connectivity (`PC-HQ` successfully pinged `192.168.10.1`, 0% loss).
- **2026-10-19 [PHASE 3 PASSED - BASELINE ROUTING]:**
  - Configured static routes across R1-HQ, R2-ISP, and R3-Branch.
  - End-to-end routing successfully established across the simulated ISP.
  - `PC-HQ` -> `PC-Branch` (`192.168.20.10`) ping succeeded (`Reply from 192.168.20.10, TTL=125`).
- **2026-10-19 [ENTERPRISE MULTI-VLAN IPSEC VPN FULLY OPERATIONAL]:**
  - Configured return routes on `R3-Branch` for VLAN 20 (`192.168.30.0/24`) and VLAN 30 (`192.168.40.0/24`).
  - Extended Crypto ACL 101 on both `R1-HQ` and `R3-Branch` to encapsulate multi-department traffic.
  - Extended NAT Exemption ACL 110 on `R1-HQ` to bypass PAT for all internal VLAN subnets.
  - **Verification:** `PC-HQ` (VLAN 20 Staff: `192.168.30.10`) successfully pinged `PC-Branch` (`192.168.20.10`) across the encrypted tunnel (`Reply from 192.168.20.10, TTL=126`).
- **Master Enterprise Baseline Saved:** User instructed to save `TunnelTrace_06_Enterprise_VLAN_VPN.pkt`.
- **Ready for Next Phase:** Dynamic DHCP Gateway Pools OR Phase 7 Controlled Fault-Injection Experiments.
