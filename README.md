# Enterprise Network & Systems (Cisco IOS + ASA + Windows Server)

Self-directed small enterprise infrastructure build for **Margielos**, focused on segmentation, high availability, secure administration, and repeatable validation.

## Documentation
- Full design and implementation notes: [docs/Network_Documentation.md](docs/Network_Documentation.md)
- Validation plan and results: [docs/validation.md](docs/validation.md)
- Operations and troubleshooting: [docs/runbook.md](docs/runbook.md)
- Device configurations: [configs/](configs/)
- Evidence outputs (show commands, screenshots): [evidence/](evidence/)

---

## What I built
- **10 routed VLANs plus VLAN 999 (native/blackhole)** on a collapsed-core dual-switch design
- **Redundant default gateways** using **HSRPv2** across two routers (VIP `.1` per VLAN)
- **Inter-VLAN routing and policy** via 802.1Q subinterfaces and router ACLs
- **Internet edge** using ASA PAT/NAT, with routers default-routing toward ASA transit networks
- **Central services** via dual Windows DCs providing AD DS, DNS, and DHCP with `ip helper-address`
- **Operational readiness**: validation artifacts, evidence outputs, and a troubleshooting runbook

---

## Environment and constraints
- Single-site small enterprise simulation (about 15 users) built in a lab environment
- Internet egress provided via upstream lab uplink; ASA outside addressing obtained via DHCP
- Design prioritizes realistic enterprise patterns and repeatable validation over provider-grade routing complexity

---

## Topology
**Logical topology**  
<img width="1083" height="510" alt="LOGICAL TOPOLOGY" src="https://github.com/user-attachments/assets/b6a12365-f87f-4a1d-a38d-01b7f931966a" />

<details>
  <summary><strong>Physical topology</strong></summary>
  <br/>
  <img width="1782" height="1021" alt="Physical Topology drawio" src="https://github.com/user-attachments/assets/08cb9a5d-1d92-43af-b869-24a372e7a831" />
</details>

---

## VLANs and IP plan (summary)
HSRP addressing convention per routed VLAN `X`:
- VIP (default gateway): `192.168.X.1`
- R1: `192.168.X.2`
- R2: `192.168.X.3`

| VLAN | Name               | Subnet           | Default Gateway (VIP) |
|------|--------------------|------------------|------------------------|
| 10   | SALES_CS           | 192.168.10.0/24  | 192.168.10.1           |
| 20   | WAREHOUSE          | 192.168.20.0/24  | 192.168.20.1           |
| 30   | HR_MGMT            | 192.168.30.0/24  | 192.168.30.1           |
| 40   | IT                 | 192.168.40.0/24  | 192.168.40.1           |
| 50   | MARKETING_ECOM     | 192.168.50.0/24  | 192.168.50.1           |
| 60   | PROD_FLOOR         | 192.168.60.0/24  | 192.168.60.1           |
| 70   | INFRA_MGMT         | 192.168.70.0/24  | 192.168.70.1           |
| 80   | SERVERS            | 192.168.80.0/24  | 192.168.80.1           |
| 85   | FINANCE            | 192.168.85.0/24  | 192.168.85.1           |
| 90   | GUEST              | 192.168.90.0/24  | 192.168.90.1           |
| 999  | BLACKHOLE / Native | N/A              | N/A                    |

For full IP conventions, DHCP scope ranges, and routing policies, see:
- [docs/Network_Documentation.md](docs/Network_Documentation.md)

---

## Security controls
- **Segmentation policy with router ACLs**
  - Guest VLAN is internet-only
  - Warehouse and Production are restricted east-west, with only required service access permitted
- **Switch hardening**
  - Native VLAN 999 on trunks
  - Unused ports placed in VLAN 999 and administratively shut
  - PortFast + BPDU Guard on access ports
  - Port-security on key access interfaces
- **Secure administration**
  - SSH-only management with restricted access patterns

---

## High availability
- HSRPv2 per VLAN with:
  - R1 preferred Active (priority 110, preempt enabled)
  - R2 standby (priority 100)
  - Upstream tracking on R1 for WAN failure failover behavior
  - MD5 HSRP authentication on groups

---

## Remote access (ASA AnyConnect)
AnyConnect remote-access VPN is documented with lab environment constraints (outside interface reachability varies due to DHCP-provided upstream addressing).

Out-of-scope note: a Tailscale overlay was used during development for remote lab access continuity and is not part of the baseline network design or validation evidence.

---

## Validation and evidence
Validation outputs are documented and supported by evidence artifacts, including:
- HSRP state and failover checks
- Trunks and EtherChannel verification
- ACL segmentation behavior tests
- DHCP relay behavior across VLANs
- NAT/PAT behavior at the edge

See:
- [docs/validation.md](docs/validation.md)
- [evidence/](evidence/)

<details>
  <summary><strong>Quick “show” commands used in validation</strong></summary>

- `show standby brief`
- `show etherchannel summary`
- `show interfaces trunk`
- `show ip interface brief`
- `show ip route`
- `show access-lists`
- `show nat` (ASA)

Exact command outputs are stored in [evidence/](evidence/).
</details>

---
## Packet Tracer extension ( in progress)
<details>
  <summary><strong>Show details</strong></summary>
  <br/>

  The baseline build was implemented on real Cisco/VM infrastructure (IOS + Windows Server + ASA).  
  This **Packet Tracer extension** is a **planned replica** used to demonstrate and validate *additional network controls* (“extras”) **without changing the completed baseline environment**.

  ## Roadmap (Enhancements) 
  ### Packet Tracer extension (network controls) 
  - Implement centralized DHCP in Packet Tracer (Server-PT/router DHCP) and validate **DHCP relay** across VLANs.
  - Implement **DHCP Snooping** with a clear trust boundary (trusted trunk/uplink, untrusted access).
  - Implement **Dynamic ARP Inspection (DAI)** using DHCP Snooping bindings and validate against ARP spoofing.
  - Add a basic **QoS policy** (classification + prioritization) and document verification outputs.
  - Add a simulated **branch LAN** to meaningfully evaluate **OSPF** route exchange and convergence.

Where it will live:
- [extension-packet-tracer/README.md](extension-packet-tracer/README.md)
- [extension-packet-tracer/docs/changes-from-baseline.md](extension-packet-tracer/docs/changes-from-baseline.md)
- [extension-packet-tracer/docs/validation.md](extension-packet-tracer/docs/validation.md)
- [extension-packet-tracer/packet-tracer/enterprise-network-system.pkt](extension-packet-tracer/packet-tracer/enterprise-network-system.pkt)

</details>
