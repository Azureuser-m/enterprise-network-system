# Enterprise Network & Systems (Cisco + Windows Server)

A small-enterprise network build for Margielos focused on segmentation, redundancy, secure management, and documented validation.  
Core features include multi-VLAN routing, redundant default gateways (HSRP), centralized DHCP via relay, and two ASA-based internet edges.

## Highlights (What this demonstrates)
- **Network segmentation** using departmental VLANs and a dedicated **Servers VLAN** and **Management VLAN**. 
- **Gateway redundancy** with **HSRPv2** across VLANs, using **MD5 authentication** and **uplink tracking** to fail over when the primary path drops. 
- **Router-on-a-stick** inter-VLAN routing (subinterfaces per VLAN on both routers). 
- **Centralized DHCP** using `ip helper-address` to redundant domain controllers (DC01/DC02). 
- **Internet edge via ASA** with router default routes pointing to the firewall transit networks. 
- **Security controls** using router ACLs applied per VLAN where required (example: inbound ACL on VLAN 20).

---

## Topology

- **Logical topology:** <img width="1083" height="510" alt="LOGICAL TOPOLOGY" src="https://github.com/user-attachments/assets/2e02e865-167a-4069-91a5-63b447439a07" />

- **Physical topology:** <img width="1782" height="1021" alt="Physical Topology drawio" src="https://github.com/user-attachments/assets/2fa99fb8-df84-4060-9282-b5004ed8445f" />


---

## VLANs and IP Plan (Summary)
| VLAN | Name              | Subnet            | Gateway (VIP) |
|------|-------------------|-------------------|---------------|
| 10   | SALES_CS          | 192.168.10.0/24   | 192.168.10.1  |
| 20   | WAREHOUSE         | 192.168.20.0/24   | 192.168.20.1  |
| 30   | HR_MGMT           | 192.168.30.0/24   | 192.168.30.1  |
| 40   | IT                | 192.168.40.0/24   | 192.168.40.1  |
| 50   | MARKETING_ECOM    | 192.168.50.0/24   | 192.168.50.1  |
| 60   | PROD_FLOOR        | 192.168.60.0/24   | 192.168.60.1  |
| 70   | INFRA_MGMT        | 192.168.70.0/24   | 192.168.70.1  |
| 80   | SERVERS           | 192.168.80.0/24   | 192.168.80.1  |
| 85   | FINANCE           | 192.168.85.0/24   | 192.168.85.1  |
| 90   | GUEST             | 192.168.90.0/24   | 192.168.90.1  |
| 999  | BLACKHOLE / Native| 192.168.199.0/24  | N/A           |

Full addressing + DHCP ranges: see `/docs/ip-plan.md`. 

---

## How traffic flows (High level)
1. Clients use the **HSRP virtual IP (.1)** as the default gateway.
2. The **active router** routes:
   - to other VLANs (internal)
   - or to the **ASA transit / default route** for internet traffic. 

---

## Repository Structure
- `/docs/` — design + implementation docs
- `/configs/` — sanitized device configurations
- `/evidence/` — validation outputs (`show ...` command captures)
- `/diagrams/` — logical + physical diagrams

---

## Validation / Evidence
See `/docs/validation.md` and the raw command outputs in `/evidence/`.

---

## Notes / Limitations
- The build uses **static default routing** toward the firewall and does not run internal OSPF/EIGRP.
- Future improvements are tracked in `/docs/future-improvements.md`.

---

## Future Improvements (Optional Extensions)
- Add a **Packet Tracer replica** for a fully shareable demo environment.
- Extend with a simulated **branch office + site-to-site VPN** and dynamic routing (OSPF) as a separate enhancement.
