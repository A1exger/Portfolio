# Aleksandr Gerasimov
 
**Network Engineer** · IP/MPLS · BGP · OSPF · L2/L3VPN · high-availability networks
 
Network engineer with 6+ years in large telecom environments (MegaFon, MTS, Rostelecom) and recent work on enterprise. I build and operate IP/MPLS, BGP and OSPF networks, troubleshoot incidents in high-load environments, and am expanding into network automation with Ansible, Python and NetBox. Hands-on with Cisco, Juniper, Huawei and MikroTik. Junos Intermediate Routing (JIR) certified.
 
Open to remote roles in ISP and telecom network operations.
 
[LinkedIn](https://linkedin.com/in/a1exger) · alexandr.gerasimov.94@gmail.com
 
---
 
## Projects
 
### [IP/MPLS Service Provider Lab](IP.MPLS/Readme.md)
 
A two-site service provider network on Juniper vMX in EVE-NG, built and debugged end to end. One MPLS core delivers three services to a customer with offices in Moscow and Saint Petersburg, and each service carries real traffic between the sites.
 
[![IP/MPLS lab topology](IP.MPLS/screenshots/topology/topology.png)](IP.MPLS/Readme.md)
 
- **Core:** OSPF, LDP and a BGP-free P layer; route reflectors with Route Target Constraint and BFD.
- **Services:** L3VPN (VRF, RD/RT), L2VPN (LDP-signalled pseudowire) and Internet access over eBGP with community-based route filtering.
- **Customer edge:** Cisco IOSv routers and access switches; VPCs in four LANs test the L2VPN and the L3VPN end to end.
- **Security:** Routing Engine protection filter with policers, counters and logging.
- **Troubleshooting:** four case studies, from a VRF export policy and Route Target Constraint to a lo0 filter bug on vMX and LDP session roles.
- **Evidence:** configurations of all 12 devices, verification outputs and screenshots.
---
 
## Skills
 
| Area | Technologies |
|---|---|
| Networking | IP/MPLS, BGP, OSPF, LDP, L2VPN, L3VPN, VLAN, DHCP, IP transit, transport networks |
| Security | IPsec and GRE tunnels, TACACS+ with LDAP authentication, Check Point firewalls |
| Vendors | Cisco, Juniper, Huawei, MikroTik, Eltex, Check Point, Ubiquiti |
| Automation | Ansible, Python, NetBox, Oxidized, Huawei NCE-IP |
| Operations | Linux, VMware, Zabbix, Grafana, Cacti, technical documentation |
 
---
 
## Experience
 
| Period | Company | Role |
|---|---|---|
| May 2026 – present | Seldon PRO | Network Engineer |
| Jul 2025 – Mar 2026 | StormWall | Network Engineer |
| Feb 2024 – Jun 2025 | Digital Fleet LLC | Network Engineer |
| Mar 2023 – Feb 2024 | MegaFon | Senior Engineer, Transport Network Administration |
| Jul 2022 – Mar 2023 | MegaFon | Network Engineer, Transport Network Administration |
| Feb 2020 – Jun 2022 | MegaFon | Technical Solutions Development Engineer |
| Jun 2019 – Jan 2020 | MTS | Engineer, Fixed Telephony Monitoring & Control |
| Apr 2017 – Jun 2019 | Rostelecom | Customer Support Technician |
 
Selected work:
 
- Led a security audit of a multi-vendor enterprise and data-center network: Juniper SRX, Cisco Nexus and IOS, MikroTik, Huawei, HP/H3C.
- Built a network management platform on Linux: NetBox for DCIM and IPAM, Oxidized for git-based configuration backups, nginx with Authelia 2FA, and the first Ansible automation on NetBox dynamic inventory.
- Rolled out TACACS+ with AD/LDAP integration and role-based access across six platforms.
- Fixed an SSD field-notice defect on a Cisco Nexus 9000 vPC core pair with a planned non-ISSU NX-OS upgrade and controlled firewall failover.
- Operated a transport network of 1,000+ nodes at MegaFon and built new interconnection points between organisational units and the internet egress, with redundant switching and BGP.
- Maintained Cisco, Juniper and MikroTik equipment and IP transit services in a high-load DDoS mitigation network at StormWall.
---
 
## Certifications and education
 
- Junos Intermediate Routing (JIR) — Diona Masterlab, 2021
- Network Engineer — OTUS, 2023
- Network Engineer Basic — OTUS, 2022
- MikroTik Basic — Voxlink, 2019
- Infocommunication Technologies and Communication Systems, in progress (expected 2028) — Moscow Technical University of Communications and Informatics, Volga-Vyatka branch
- Telecommunications Technician — Irkutsk State University of Railway Transport, 2014
Languages: Russian (native), English (B1).
