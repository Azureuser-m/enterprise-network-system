# Enterprise Network & Systems (Cisco + Windows Server)

Small-enterprise infrastructure build for **Margielos** focused on **segmentation, high availability, secure administration, and documented validation**.

**Tech:** Cisco IOS (routing/switching), VLANs/802.1Q, HSRPv2, LACP EtherChannel, ACLs, Cisco ASA (NAT), Windows Server (AD DS/DNS/DHCP), Proxmox

## Project at a glance
- **10 VLANs + VLAN 999 (blackhole/native)** across 2 core switches
- **Redundant default gateway:** HSRPv2 across **2 routers**
- **Internet edge:** ASA performing NAT; routers forward default route to firewall transit networks
- **Services:** Dual DCs (AD DS/DNS/DHCP) supporting multi-VLAN DHCP via relay
- **Documentation-first:** diagrams, IP plan, runbooks, validation outputs

## What this demonstrates
- Built **departmental segmentation** + dedicated **Servers (VLAN 80)** and **Infra Mgmt (VLAN 70)** networks.
- Implemented **HSRPv2** with authentication + interface tracking for gateway failover.
- Delivered **inter-VLAN routing** via 802.1Q subinterfaces (router-on-a-stick).
- Centralized **DHCP** using `ip helper-address` to redundant DCs (DC01/DC02).
- Implemented **LACP EtherChannel** between core switches for bandwidth + link resiliency.
- Enforced traffic policy using **router ACLs** (Guest internet-only; restricted east-west for select VLANs).

## Security controls implemented
- **SSH-only management** and restricted management access (VLAN 70 / mgmt subnet)
- **Unused ports** placed in **VLAN 999** and administratively shut down
- **STP edge protections:** PortFast + BPDU Guard on access ports
- **Port-security** (sticky MAC / violation restrict) on key access interfaces

---

## Topology
**Logical topology:**  
<img width="1083" height="510" alt="LOGICAL TOPOLOGY" src="https://github.com/user-attachments/assets/b6a12365-f87f-4a1d-a38d-01b7f931966a" />


<details>
  <summary><strong>Physical topology</strong></summary>
  <br/>
  <img width="1782" height="1021" alt="Physical Topology drawio" src="https://github.com/user-attachments/assets/08cb9a5d-1d92-43af-b869-24a372e7a831" />
</details>

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

Full IP conventions, DHCP scope ranges, and reservations: `docs/Network_Documentation.md`

---

## Validation
- **HSRP:** `show standby brief` (active/standby correct per VLAN)
- **Trunks/EtherChannel:** `show etherchannel summary`, `show interfaces trunk`
- **Segmentation:** ping tests (Guest → blocked internal, allowed Internet)
- **DHCP relay:** client obtains IP/gateway/DNS on each VLAN via DC01/DC02

Evidence outputs: `evidence/`

---

## Repo structure
- `docs/Network_Documentation.md` – full design + IP plan
- `docs/validation.md` – test plan + results
- `docs/runbook.md` – operations & troubleshooting
- `evidence/` – raw show-commands + screenshots

<details>
  <summary><strong>Packet Tracer extension (in progress)</strong></summary>
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

  **Where it will live**
  - `extension-packet-tracer/README.md` – extension overview + scope
  - `extension-packet-tracer/docs/changes-from-baseline.md` – what’s different vs baseline
  - `extension-packet-tracer/docs/validation.md` – test plan + results (extension)
  - `extension-packet-tracer/packet-tracer/enterprise-network-system.pkt` – Packet Tracer file

</details>
