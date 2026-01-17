# Enterprise Network & Systems (Cisco + Windows Server)

A small-enterprise infrastructure build for “Margielos” focused on segmentation, redundancy, secure administration, and documented validation.  
Core features include multi-VLAN routing, redundant default gateways (HSRP), centralized DHCP via relay to domain controllers, and an ASA-based Internet edge.

## Highlights (What this demonstrates)
- **Network segmentation** using departmental VLANs plus dedicated **Servers** and **Infrastructure Management** networks.
- **High availability at the default gateway** using **HSRPv2** with **MD5 authentication** and **interface tracking** to support failover when the primary path degrades.
- **Inter-VLAN routing (router-on-a-stick)** using 802.1Q subinterfaces across the core routers.
- **Centralized DHCP services** with `ip helper-address` to redundant domain controllers (DC01/DC02) for address assignment across VLANs.
- **Secure Internet edge** using ASA firewalls for NAT, with routers forwarding Internet-bound traffic via default routes toward the firewall transit networks.
- **Traffic control between VLANs** using router ACLs to enforce segmentation requirements (example: limiting Warehouse/Production lateral access and keeping Guest Internet-only).

---

## Topology
- **Logical topology:**  
  <img width="1083" height="510" alt="LOGICAL TOPOLOGY" src="https://github.com/user-attachments/assets/2e02e865-167a-4069-91a5-63b447439a07" />

- **Physical topology:**  
  <img width="1782" height="1021" alt="Physical Topology drawio" src="https://github.com/user-attachments/assets/2fa99fb8-df84-4060-9282-b5004ed8445f" />

---

## VLANs and IP Plan (Summary)

| VLAN | Name              | Subnet            | Default Gateway (VIP) |
|------|-------------------|-------------------|------------------------|
| 10   | SALES_CS          | 192.168.10.0/24   | 192.168.10.1           |
| 20   | WAREHOUSE         | 192.168.20.0/24   | 192.168.20.1           |
| 30   | HR_MGMT           | 192.168.30.0/24   | 192.168.30.1           |
| 40   | IT                | 192.168.40.0/24   | 192.168.40.1           |
| 50   | MARKETING_ECOM    | 192.168.50.0/24   | 192.168.50.1           |
| 60   | PROD_FLOOR        | 192.168.60.0/24   | 192.168.60.1           |
| 70   | INFRA_MGMT        | 192.168.70.0/24   | 192.168.70.1           |
| 80   | SERVERS           | 192.168.80.0/24   | 192.168.80.1           |
| 85   | FINANCE           | 192.168.85.0/24   | 192.168.85.1           |
| 90   | GUEST             | 192.168.90.0/24   | 192.168.90.1           |
| 999  | BLACKHOLE / Native| 192.168.199.0/24  | N/A                    |

Full IP addressing conventions, DHCP scope ranges, and server reservations are documented in `docs/Network_Documentation.md`.

---

## How traffic flows (High level)
1. **Clients use the HSRP virtual IP (.1)** in their VLAN as the default gateway (example: VLAN 10 uses 192.168.10.1).
2. **The active HSRP router** forwards traffic based on destination:
   - **Internal traffic:** routed directly to the destination VLAN via the router subinterfaces.
   - **Server access:** routed toward VLAN 80 (Servers) as required by the segmentation policy.
   - **Internet-bound traffic:** forwarded via the router’s **default route** to the ASA transit network, where the ASA performs **NAT** and forwards to the Internet.

---

## Documentation

### Baseline (Completed Build)
- Full documentation: `docs/Network_Documentation.md`
- Validation report: `docs/validation.md`
- Operations runbook: `docs/runbook.md`
- Evidence outputs (raw command outputs): `evidence/`

### Extension (Packet Tracer Replica)
- Extension overview: `extension-packet-tracer/README.md`
- Changes from baseline: `extension-packet-tracer/docs/changes-from-baseline.md`
- Extension validation: `extension-packet-tracer/docs/validation.md`
- Packet Tracer file: `extension-packet-tracer/packet-tracer/enterprise-network-system.pkt`

---

## Notes / Limitations
- This build uses **HSRP for first-hop redundancy** and **static default routing** toward the firewall (no internal OSPF/EIGRP in the baseline).
- The Packet Tracer extension is used to replicate the environment and explore enhancements without modifying the completed baseline build.

---

## Roadmap (Enhancements)
- Packet Tracer replica for a shareable demo environment and repeatable testing.
- Add a simulated **branch office** and evaluate **dynamic routing (OSPF)** and/or site-to-site connectivity as an enhancement.
- Implement **DHCP Snooping** and define a clear trust boundary (trusted trunk/uplink ports, untrusted access ports).
- Implement **Dynamic ARP Inspection (DAI)** using DHCP Snooping bindings and validate mitigation of ARP spoofing attempts.
- Add a basic **QoS policy** (classification and prioritization) and document the trust boundary and verification outputs.
- Expand validation coverage with test cases and evidence (HSRP failover behavior, EtherChannel resiliency, ACL hit counts, DHCP relay lease verification, Snooping/DAI verification).
