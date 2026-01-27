# Network Operations Runbook

## Purpose

This runbook provides day-2 operational guidance for the enterprise network. It is intended for engineers performing routine operations, troubleshooting, and controlled changes. The focus is repeatability, operational clarity, and minimizing blast radius.

---

## Scope and Assumptions

* Single-site enterprise network simulation.
* All routing, switching, and firewalling is managed internally.
* Internet egress is provided by an upstream lab network via DHCP on ASA outside interfaces.
* This runbook is the **authoritative source** for port mapping and operational checks.

---

## Network Overview

* Segmented VLAN architecture (10–90) with role-based isolation.
* Redundant default gateways per VLAN using HSRP (R1 active, R2 standby).
* Dual access switches (SW1, SW2) interconnected via LACP EtherChannel.
* Edge security and NAT provided by dual ASA firewalls.

---

## Device Inventory

| Device | Role                       | Platform / Model            | Management IP | Notes                       |
| ------ | -------------------------- | --------------------------- | ------------- | --------------------------- |
| R1     | Core router (HSRP Active)  | Cisco 2901                  | 192.168.70.2  | Tracks ASA reachability     |
| R2     | Core router (HSRP Standby) | Cisco ISR4321               | 192.168.70.3  | Takes over on R1 failure    |
| SW1    | Access switch              | Cisco Catalyst 2960 | 192.168.70.5  | LACP to SW2                 |
| SW2    | Access switch              | Cisco Catalyst 2960 | 192.168.70.6  | LACP to SW1                 |
| ASA01  | Firewall (primary)         | Cisco ASA 5506-X            | 192.168.80.5  | Primary egress + AnyConnect |
| ASA02  | Firewall (secondary)       | Cisco ASA 5506-X            | [Mgmt unused] | Secondary egress            |

---

## Port Mapping and Cabling Reference

**Purpose:** Authoritative reference for physical and logical port usage. Used for troubleshooting, audits, and change validation.

**Sources of truth:**

* Switch running configs and VLAN tables
* Router interface descriptions and routing tables
* ASA interface, NAT, and routing configuration
* Logical topology diagram

---

### Core Uplinks and Trunks

| Device | Interface         | Connected To    | Mode              | Notes                                                |
| ------ | ----------------- | --------------- | ----------------- | ---------------------------------------------------- |
| SW1    | Gi0/1             | R1 Gi0/1        | 802.1Q trunk      | Native VLAN 999; VLANs 10,20,30,40,50,60,70,80,85,90 |
| SW2    | Gi0/1             | R2 Gi0/0/1      | 802.1Q trunk      | Native VLAN 999; VLANs 10,20,30,40,50,60,70,80,85,90 |
| SW1    | Fa0/1–Fa0/2 (Po1) | SW2 Fa0/1–Fa0/2 | LACP EtherChannel | Inter-switch redundancy                              |

Native VLAN on all trunks: **999 (Blackhole)**

---

### Edge Transit Links (Routers and Firewalls)

| Device | Interface | Connected To    | Purpose            | Addressing           |
| ------ | --------- | --------------- | ------------------ | -------------------- |
| R1     | Gi0/0     | ASA01 Gi1/2     | Inside transit     | 192.168.254.1/30     |
| R2     | Gi0/0/0   | ASA02 Gi1/2     | Inside transit     | 192.168.254.5/30     |
| ASA01  | Gi1/1     | Upstream uplink | Outside / Internet | DHCP + default route |
| ASA02  | Gi1/1     | Upstream uplink | Outside / Internet | DHCP + default route |

---

### Server and Infrastructure Ports (VLAN 80)

| Switch | Interface | Endpoint     | Notes                   |
| ------ | --------- | ------------ | ----------------------- |
| SW1    | Fa0/3     | DC01         | AD / DNS / DHCP         |
| SW1    | Fa0/4     | Proxmox host | Backup + mail workloads |
| SW2    | Fa0/3     | DC02         | AD / DNS / DHCP         |

---

### User and Access Ports

#### VLAN 10 – SALES_CS

| Switch | Interface(s) | Endpoint                |
| ------ | ------------ | ----------------------- |
| SW1    | Fa0/5–Fa0/8  | Sales / CS workstations and Staff AP |

#### VLAN 30 – HR_MGMT

| Switch | Interface(s) | Endpoint   |
| ------ | ------------ | ---------- |
| SW1    | Fa0/9–Fa0/10 | HR systems |

#### VLAN 40 – IT

| Switch | Interface(s)  | Endpoint        |
| ------ | ------------- | --------------- |
| SW1    | Fa0/11–Fa0/12 | IT workstations |

#### VLAN 20 – WAREHOUSE

| Switch | Interface(s) | Endpoint          |
| ------ | ------------ | ----------------- |
| SW2    | Fa0/7–Fa0/8  | Warehouse devices |

#### VLAN 50 – MARKETING_ECOM

| Switch | Interface(s) | Endpoint               |
| ------ | ------------ | ---------------------- |
| SW2    | Fa0/5–Fa0/6  | Marketing / e-commerce |

#### VLAN 60 – PROD_FLOOR

| Switch | Interface(s) | Endpoint         |
| ------ | ------------ | ---------------- |
| SW2    | Fa0/9–Fa0/10 | Production floor |

#### VLAN 85 – FINANCE

| Switch | Interface | Endpoint        |
| ------ | --------- | --------------- |
| SW2    | Fa0/11    | Finance systems |

#### VLAN 70 – INFRA_MGMT

| Switch | Interface | Endpoint                         |
| ------ | --------- | -------------------------------- |
| SW2    | Fa0/14    | Infrastructure admin workstation |

#### VLAN 90 – GUEST

| Switch | Interface | Endpoint          |
| ------ | --------- | ----------------- |
| SW2    | Fa0/4     | Guest wireless AP |

---

### Blackhole and Unused Ports

* VLAN **999** assigned to all unused access ports.
* Unused ports are administratively shut down.
* Purpose: prevent accidental access and reduce attack surface.

---

## Standard Operational Checks

### Verify HSRP State

```
show standby brief
```

Expected: R1 active, R2 standby.

### Verify Trunks and EtherChannel

```
show interfaces trunk
show etherchannel summary
```

### Verify Routing

```
show ip route
```

Confirm default route points toward ASA.

### Verify DHCP Relay

```
show run | include helper-address
```

### Firewall Health

```
show interface ip brief
show route
show nat detail
```

---

## Failure Scenarios

### R1 Failure

* HSRP fails over to R2 automatically.
* End hosts retain default gateway.

### Switch Failure

* Only hosts connected to failed switch are impacted.
* Core routing and firewall remain operational.

### Firewall Failure

* Remaining ASA continues NAT and egress.
* Internal routing remains intact.

---

## Change Management Notes

* Update this runbook for any port reassignment or VLAN change.
* Capture updated `show run` outputs in `evidence/` after changes.
* Validate trunks and HSRP after any core modification.

---

## References

* docs/Network_Documentation.md
* docs/validation.md
* configs/

