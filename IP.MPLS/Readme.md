# 🌐 IP/MPLS Service Provider Lab 

> **Platform:** EVE-NG | **Vendor:** Juniper (vMX / vSRX) and Cisco IOS-XE/XR  
> **Complexity:** Advanced | **Focus:** SP Core + L3VPN + L2VPN + BGP RR Architecture

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Lab Topology](#lab-topology)
3. [Network Addressing Table](#network-addressing-table)
4. [Technology Stack](#technology-stack)
5. [Project Structure](#project-structure)
6. [Implementation Guide](#implementation-guide)
   - [Phase 1 — Underlay: OSPF IGP](#phase-1--underlay-ospf-igp)
   - [Phase 2 — MPLS LDP Label Distribution](#phase-2--mpls-ldp-label-distribution)
   - [Phase 3 — BGP Route Reflector Architecture](#phase-3--bgp-route-reflector-architecture)
   - [Phase 4 — L3VPN (MPLS VRF)](#phase-4--l3vpn-mpls-vrf)
   - [Phase 5 — L2VPN (VPLS / EoMPLS)](#phase-5--l2vpn-vpls--eompls)
   - [Phase 6 — Customer Edge: VLANs & Router-on-a-Stick](#phase-6--customer-edge-vlans--router-on-a-stick)
7. [Verification & Troubleshooting](#verification--troubleshooting)
8. [Key Learnings & Design Decisions](#key-learnings--design-decisions)
9. [Screenshots & Evidence](#screenshots--evidence)
10. [How to Reproduce This Lab](#how-to-reproduce-this-lab)
11. [References](#references)

---

## Project Overview

This lab simulates a **production-grade Service Provider network** connecting two customer sites — **Moscow (MSK)** and **Saint Petersburg (SPB)** — through a full IP/MPLS core. The goal is to demonstrate end-to-end service delivery using industry-standard SP technologies.

### Business Scenario

A telecom operator needs to deliver **enterprise connectivity services** to a customer with offices in Moscow and Saint Petersburg. The SP network must provide:

- **Layer 3 VPN (L3VPN)** — routed inter-site connectivity with full route isolation per customer VRF
- **Layer 2 VPN (L2VPN)** — transparent Ethernet service between sites (as if on the same LAN)
- **High availability** — redundant P routers and dual Route Reflectors
- **Scalable BGP design** — RR-based iBGP to avoid full-mesh requirement

### Skills Demonstrated

| Domain | Technologies |
|---|---|
| IGP / Underlay | OSPF Area 0, Loopback reachability |
| MPLS Data Plane | LDP, Label Switched Paths (LSPs) |
| BGP Control Plane | iBGP, Route Reflectors, MP-BGP (VPNv4) |
| VPN Services | L3VPN (VRF-Lite), L2VPN (EoMPLS or VPLS) |
| Customer Edge | VLAN trunking, 802.1Q, Router-on-a-Stick (RoaS) |
| Operations | Verification commands, traffic tracing, fault isolation |

---

## Lab Topology

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              IP/MPLS CORE                                   │
│                                                                             │
│          msk-RR-1                              spb-RR-2                     │
│       Lo: 172.31.50.1                       Lo: 172.31.51.1                 │
│              │  │                                │  │                       │
│              │  └──────────── msk-P-1 ───────────┘  │                      │
│              │             Lo: 172.17.1.1            │                      │
│         msk-PE-1 ──────── msk-P-2 ─────────── spb-PE-1                     │
│       Lo: 172.17.1.2   Lo: 172.17.1.3      Lo: 172.17.1.4                  │
└──────────┬──────────────────────────────────────────────┬───────────────────┘
           │                                              │
    ┌──────▼──────┐                              ┌────────▼──────┐
    │   MOSCOW    │                              │      SPB      │
    │             │                              │               │
    │ msk-router-1│                              │ spb-router-1  │
    │ msk-switch-1│                              │ spb-switch-1  │
    │     VPC     │                              │     VPC       │
    └─────────────┘                              └───────────────┘
```

> 📸 **See:** [`/screenshots/topology/eve-ng-topology.png`](#) — Full EVE-NG canvas screenshot

---

## Network Addressing Table

### Loopback Interfaces (Router-IDs & BGP Peering)

| Device | Loopback0 | Role |
|---|---|---|
| msk-RR-1 | 172.31.50.1/32 | BGP Route Reflector (MSK) |
| spb-RR-2 | 172.31.51.1/32 | BGP Route Reflector (SPB) |
| msk-P-1 | 172.17.1.1/32 | MPLS P Router |
| msk-PE-1 | 172.17.1.2/32 | PE Router — Moscow CE |
| msk-P-2 | 172.17.1.3/32 | MPLS P Router |
| spb-PE-1 | 172.17.1.4/32 | PE Router — SPB CE |

### Point-to-Point Links (OSPF / LDP)

| Link | Subnet | Device A | Interface | Device B | Interface |
|---|---|---|---|---|---|
| PE1↔P1 | 10.10.10.0/30 | msk-PE-1 | em2/ge-0/0/0 | msk-P-1 | em2/ge-0/0/0 |
| PE1↔P2 | 10.10.10.4/30 | msk-PE-1 | em4/ge-0/0/2 | msk-P-2 | em4/ge-0/0/2 |
| *(add all P-to-P and PE-to-RR links)* | | | | | |

### Customer Edge Addressing

| Site | Device | Interface | IP | VLAN |
|---|---|---|---|---|
| Moscow | msk-router-1 | Gi0/1.10 | *(define)* | 10 |
| Moscow | msk-switch-1 | Gi1/3 trunk | — | 10, 20 |
| SPB | spb-router-1 | Gi0/1.10 | *(define)* | 10 |
| SPB | spb-switch-1 | Gi1/2 trunk | — | 10, 20 |

---

## Technology Stack

### EVE-NG Environment

- **Platform:** EVE-NG Community / Professional
- **Node images:** Cisco IOSv / IOS-XE / Juniper vMX *(specify exact image version)*
- **Host OS:** Ubuntu 20.04 LTS
- **RAM per node:** ~512MB–1GB

### Protocols Used

```
┌─────────────────────────────────────────────────┐
│  CONTROL PLANE                                  │
│  ├── OSPF (Area 0) — IGP underlay               │
│  ├── LDP — MPLS label distribution              │
│  ├── iBGP + Route Reflectors                    │
│  └── MP-BGP (VPNv4 / L2VPN EVPN or LDP-based)  │
│                                                 │
│  DATA PLANE                                     │
│  ├── MPLS Label Switching (P routers)           │
│  ├── VRF — customer traffic isolation           │
│  └── 802.1Q VLAN trunking (CE switches)         │
└─────────────────────────────────────────────────┘
```

---

## Project Structure

```
ip.mpls/
├── README.md                        ← This file
├── topology/
│   ├── eve-ng-topology.png          ← Full lab diagram
│   ├── addressing-table.xlsx        ← Complete IP plan
│   └── topology.drawio              ← Editable diagram source
│
├── configs/
│   ├── msk-RR-1/
│   │   └── running-config.txt
│   ├── spb-RR-2/
│   │   └── running-config.txt
│   ├── msk-P-1/
│   │   └── running-config.txt
│   ├── msk-P-2/
│   │   └── running-config.txt
│   ├── msk-PE-1/
│   │   └── running-config.txt
│   ├── spb-PE-1/
│   │   └── running-config.txt
│   ├── msk-router-1/
│   │   └── running-config.txt
│   └── spb-router-1/
│       └── running-config.txt
│
├── screenshots/
│   ├── topology/
│   │   └── eve-ng-topology.png
│   ├── ospf/
│   │   ├── ospf-neighbors-all.png
│   │   └── ospf-database.png
│   ├── mpls/
│   │   ├── ldp-neighbors.png
│   │   ├── mpls-forwarding-table-PE1.png
│   │   └── mpls-forwarding-table-P1.png
│   ├── bgp/
│   │   ├── bgp-summary-RR1.png
│   │   ├── bgp-vpnv4-table.png
│   │   └── bgp-rr-clients.png
│   ├── l3vpn/
│   │   ├── vrf-table-PE1.png
│   │   ├── vrf-table-PE2.png
│   │   └── ping-msk-to-spb.png
│   ├── l2vpn/
│   │   ├── l2vpn-status.png
│   │   └── ping-l2-msk-to-spb.png
│   └── ce/
│       ├── vlan-config-msk-switch.png
│       ├── roas-msk-router.png
│       └── vpc-ping-end-to-end.png
│
└── docs/
    ├── design-rationale.md          ← Why these design choices
    ├── troubleshooting-log.md       ← Issues faced & fixed
    └── traffic-flow-walkthrough.md  ← Packet walk through the MPLS core
```

---

## Implementation Guide

### Phase 1 — Underlay: OSPF IGP

**Objective:** Establish full loopback reachability across all SP routers (RR, P, PE).

All P/PE/RR devices run OSPF Area 0. Only loopbacks and P2P links are redistributed. No customer prefixes in IGP.

**Key configuration snippet:**
```
export OSPF-EXPORT;
import OSPF-IMPORT;
area 0.0.0.0 {
    interface lo0.0;
    interface ge-0/0/1.0 {
        interface-type p2p;
    }
    interface ge-0/0/3.0 {
        interface-type p2p;
    }
}

```

**Verification commands to run & screenshot:**
```bash
show ospf neighbor                  # All neighbors in FULL state
show ospf database                  # LSA database populated
show ospf route                     # All loopbacks reachable via OSPF
ping 172.17.1.4 source 172.31.50.1  # PE1 → spb-PE1 loopback reachability
```

> 📸 **Screenshot:** `screenshots/ospf/ospf-neighbors-all.png`

---

### Phase 2 — MPLS LDP Label Distribution

**Objective:** Enable MPLS label switching on all P2P links in the SP core.

LDP uses loopbacks as router-IDs. Transport addresses must match OSPF reachability.

**Key configuration snippet:**
```
mpls ip
mpls label protocol ldp
mpls ldp router-id Loopback0 force

interface GigabitEthernet0/1
 mpls ip
```

**Verification commands to run & screenshot:**
```bash
show mpls ldp neighbor             # LDP sessions established
show mpls ldp bindings             # Label bindings per prefix
show mpls forwarding-table         # LFIB populated
traceroute mpls ipv4 172.17.1.4/32 # LSP trace MSK-PE1 → SPB-PE1
```

> 📸 **Screenshots:** `screenshots/mpls/ldp-neighbors.png`, `mpls-forwarding-table-PE1.png`

---

### Phase 3 — BGP Route Reflector Architecture

**Objective:** Build scalable iBGP using Route Reflectors instead of full-mesh. Dual RR design (msk-RR-1 + spb-RR-2) for redundancy.

**Design rationale:**

Without RR, N routers need N*(N-1)/2 iBGP sessions. With 6 SP nodes that is 15 sessions. RR reduces this to N client sessions per reflector and eliminates the scaling problem entirely.

**BGP topology:**
```
msk-RR-1 (172.31.50.1) ←→ spb-RR-2 (172.31.51.1)  [RR peering]
       ↕                          ↕
  [RR Clients]               [RR Clients]
  msk-PE-1                   spb-PE-1
  msk-P-1                    msk-P-2
```

**Key configuration snippet (Route Reflector):**
```
router bgp 65000
 bgp router-id 172.31.50.1
 bgp cluster-id 1
 neighbor RR-CLIENTS peer-group
 neighbor RR-CLIENTS remote-as 65000
 neighbor RR-CLIENTS update-source Loopback0
 neighbor RR-CLIENTS route-reflector-client
 address-family vpnv4
  neighbor RR-CLIENTS activate
  neighbor RR-CLIENTS send-community extended
```

**Key configuration snippet (PE client):**
```
router bgp 65000
 bgp router-id 172.17.1.2
 neighbor 172.31.50.1 remote-as 65000
 neighbor 172.31.50.1 update-source Loopback0
 address-family vpnv4
  neighbor 172.31.50.1 activate
  neighbor 172.31.50.1 send-community extended
```

**Verification commands to run & screenshot:**
```bash
show bgp vpnv4 unicast all summary      # All peers Established, prefixes received
show bgp vpnv4 unicast all neighbors X.X.X.X advertised-routes
show bgp vpnv4 unicast all             # Full VPNv4 table on RR
```

> 📸 **Screenshots:** `screenshots/bgp/bgp-summary-RR1.png`, `bgp-rr-clients.png`

---

### Phase 4 — L3VPN (MPLS VRF)

**Objective:** Deliver a routed VPN service. Moscow and SPB customer traffic travels through the MPLS core in isolated VRFs. The VPC in Moscow should ping the VPC in SPB.

**Data plane label stack:**
```
[VPN Label (inner)] [Transport Label (outer)] → IP Packet
     ↑ assigned by PE                 ↑ LDP / RSVP-TE
```

**Key configuration snippet (PE router):**
```
ip vrf CUSTOMER-A
 rd 65000:100
 route-target export 65000:100
 route-target import 65000:100

interface GigabitEthernet0/0
 ip vrf forwarding CUSTOMER-A
 ip address 192.168.10.1 255.255.255.0

router bgp 65000
 address-family ipv4 vrf CUSTOMER-A
  redistribute connected
  redistribute static
```

**Verification commands to run & screenshot:**
```bash
show ip vrf interfaces                           # VRF interface binding
show ip route vrf CUSTOMER-A                     # Customer routes in VRF table
show bgp vpnv4 unicast vrf CUSTOMER-A           # VPNv4 routes with labels
ping vrf CUSTOMER-A 192.168.20.1                # MSK-PE1 pings SPB customer prefix
traceroute vrf CUSTOMER-A 192.168.20.1          # Path through MPLS core
```

> 📸 **Screenshots:** `screenshots/l3vpn/vrf-table-PE1.png`, `ping-msk-to-spb.png`

---

### Phase 5 — L2VPN (VPLS / EoMPLS)

**Objective:** Transparent Layer 2 service — as if both customer switches were connected to the same switch.

**Key configuration snippet (EoMPLS xconnect):**
```
interface GigabitEthernet0/1
 xconnect 172.17.1.4 100 encapsulation mpls
```

Or for VPLS:
```
l2 vfi VPLS-CUSTOMER-A manual
 vpn id 100
 neighbor 172.17.1.4 encapsulation mpls

interface GigabitEthernet0/1
 service instance 100 ethernet
  encapsulation dot1q 100
  rewrite ingress tag pop 1 symmetric
  bridge-domain 100
```

**Verification commands to run & screenshot:**
```bash
show mpls l2transport vc                  # VC up/up
show l2vpn service all                   # L2VPN instance status
ping 192.168.100.2 source 192.168.100.1  # End-to-end L2 ping
```

> 📸 **Screenshot:** `screenshots/l2vpn/l2vpn-status.png`, `ping-l2-msk-to-spb.png`

---

### Phase 6 — Customer Edge: VLANs & Router-on-a-Stick

**Objective:** Simulate real customer sites with VLAN segmentation and inter-VLAN routing via a single uplink (Router-on-a-Stick).

**Moscow site design:**
```
VPC (eth0)
 └── msk-switch-1 (Gi1/3 access VLAN 10)
       └── msk-router-1 (Gi0/0 trunk → subinterfaces)
             └── msk-PE-1 (Gi0/1 → MPLS core)
```

**Key configuration snippet (switch trunk):**
```
interface GigabitEthernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20

interface GigabitEthernet1/3
 switchport mode access
 switchport access vlan 10
```

**Key configuration snippet (Router-on-a-Stick):**
```
interface GigabitEthernet0/0
 no ip address

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip vrf forwarding CUSTOMER-A

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

**Verification commands to run & screenshot:**
```bash
show interfaces trunk                    # Trunk allowed VLANs
show vlan brief                          # VLAN table on switch
show ip interface brief                  # Subinterface IPs up/up
# From VPC:
ping 192.168.10.1 gateway 192.168.10.1  # Default gateway reachable
ping [SPB VPC IP]                        # End-to-end through MPLS
```

> 📸 **Screenshots:** `screenshots/ce/vlan-config-msk-switch.png`, `roas-msk-router.png`, `vpc-ping-end-to-end.png`

---

## Verification & Troubleshooting

### End-to-End Connectivity Test Checklist

```
[ ] 1. OSPF neighbors — all in FULL state
[ ] 2. All loopbacks reachable via OSPF (ping lo0 from all nodes)
[ ] 3. LDP neighbors established on all P2P links
[ ] 4. MPLS forwarding table populated on P and PE routers
[ ] 5. BGP sessions UP between all clients and RRs
[ ] 6. VPNv4 prefixes visible on both PEs
[ ] 7. VRF routing tables have remote customer prefixes
[ ] 8. L3VPN: ping from msk-router-1 to spb-router-1 succeeds
[ ] 9. L2VPN: VC status UP/UP
[ ] 10. End-to-end: VPC (Moscow) pings VPC (SPB)
```

### Common Issues & Fixes

| Symptom | Likely Cause | Fix |
|---|---|---|
| LDP session not forming | Loopback not in OSPF | Add `network` statement |
| BGP stuck in Active | Wrong update-source | Add `update-source Loopback0` |
| VPNv4 routes missing | No `send-community extended` | Add on all VPNv4 peers |
| VRF ping fails | RD/RT mismatch | Verify import/export RT |
| L2VPN VC down | Mismatched VC-ID | Verify `xconnect` peer and ID |
| VLAN not passing | Trunk not configured | Check `switchport trunk allowed vlan` |

> 📄 **Full troubleshooting log:** [`docs/troubleshooting-log.md`](docs/troubleshooting-log.md)

---

## Key Learnings & Design Decisions

### Why Dual Route Reflectors?

A single RR is a single point of failure for BGP control plane. Using `msk-RR-1` and `spb-RR-2` as a redundant pair ensures that if one RR fails, all PE sessions fall over to the surviving RR. Both RRs peer with each other as regular iBGP (non-client).

### Why Separate P and PE Roles?

P routers (msk-P-1, msk-P-2) only perform label switching — they never hold VRF state or VPNv4 routes. This is the **scalability principle** of MPLS VPN: P routers are VPN-unaware and operate purely at the transport layer. Only PE routers maintain customer VRF tables.

### Why LDP Over RSVP-TE in This Lab?

LDP was chosen for simplicity and to focus on VPN service delivery rather than traffic engineering. In production, RSVP-TE with explicit paths would be used for bandwidth guarantees and fast reroute (FRR). This is a planned extension of this lab.

### Router-on-a-Stick Trade-offs

RoaS is used at CE sites to simulate a realistic small-enterprise setup with a single uplink to the SP. In production, a Layer 3 switch would replace this, but RoaS demonstrates VLAN subinterface and trunking concepts clearly.

---

## Screenshots & Evidence

> All screenshots are stored in `/screenshots/` with descriptive filenames.

| # | Screenshot | What It Proves |
|---|---|---|
| 1 | `topology/eve-ng-topology.png` | Lab is built and running in EVE-NG |
| 2 | `ospf/ospf-neighbors-all.png` | Full OSPF adjacency across SP core |
| 3 | `mpls/ldp-neighbors.png` | LDP sessions on all core links |
| 4 | `mpls/mpls-forwarding-table-PE1.png` | LFIB programmed, labels assigned |
| 5 | `bgp/bgp-summary-RR1.png` | All BGP clients connected to RR |
| 6 | `bgp/bgp-vpnv4-table.png` | VPNv4 prefixes with RD/RT visible |
| 7 | `l3vpn/ping-msk-to-spb.png` | L3VPN working — cross-site ping |
| 8 | `l2vpn/l2vpn-status.png` | L2VPN VC up/up |
| 9 | `ce/vpc-ping-end-to-end.png` | Full end-to-end VPC → VPC ping |
| 10 | `ce/vlan-config-msk-switch.png` | VLAN trunking at CE |

---

## How to Reproduce This Lab

### Prerequisites

- EVE-NG Community or Pro installed
- Router image: Cisco IOSv `vios-adventerprisek9-m.vmdk.SPA.156-x` or equivalent
- Switch image: Cisco IOSvL2 `vios_l2-adventerprisek9-m.vmdk`
- Minimum host RAM: 16 GB
- EVE-NG Web UI access

### Steps

```bash
# 1. Import the EVE-NG lab file
File > Import > upload lab-ip-mpls.unl

# 2. Start all nodes (allow ~3 min to boot)
# 3. Apply configs from /configs/ directory to each node
# 4. Verify connectivity using the checklist above
```

### Config Application (EVE-NG CLI method)

```bash
# SSH into EVE-NG host, then telnet to node console
telnet 127.0.0.1 <node-port>
# Paste config from configs/<device>/running-config.txt
```

---

## References

- [RFC 4364 — BGP/MPLS IP Virtual Private Networks (L3VPN)](https://www.rfc-editor.org/rfc/rfc4364)
- [RFC 4761 — Virtual Private LAN Service (VPLS) Using BGP](https://www.rfc-editor.org/rfc/rfc4761)
- [RFC 3031 — Multiprotocol Label Switching Architecture](https://www.rfc-editor.org/rfc/rfc3031)
- [RFC 5036 — LDP Specification](https://www.rfc-editor.org/rfc/rfc5036)
- [Cisco MPLS Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/mp_l3_vpns/configuration/xe-16/mp-l3-vpns-xe-16-book.html)
- Ivan Pepelnjak — *MPLS and VPN Architectures* (Cisco Press)
- EVE-NG Documentation — [eve-ng.net](https://www.eve-ng.net/index.php/documentation/)

---

## Author

**[Your Name]**  
Network Engineer | CCNP / JNCIP candidate  
[LinkedIn](https://linkedin.com/in/yourprofile) · [GitHub](https://github.com/yourusername)

---

*This lab was built for educational and portfolio purposes. All IP addresses are private/RFC-1918 or documentation ranges.*
