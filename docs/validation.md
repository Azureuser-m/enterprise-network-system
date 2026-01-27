# Validation (Baseline)

This document records baseline validation for the enterprise network build using captured CLI outputs stored in `evidence/`.  
The objective is **repeatable verification of design intent**, not a one-time demonstration.

Where operational state was not captured, the item is explicitly marked **[UNVERIFIED]**.

---

## Evidence index (current set)

### Switches
- VLAN database:
  - [SW1 – show vlan brief](../evidence/switches/SW1%23show%20vlan%20brief.txt)
  - [SW2 – show vlan brief](../evidence/switches/SW2_show-vlan-brief)
- Trunk operational state:
  - [SW1 – show interfaces trunk](../evidence/switches/SW1_show-int-trunk)
  - [SW2 – show interfaces trunk](../evidence/switches/SW2_show-int-trunk)
- EtherChannel state:
  - [SW1 – show etherchannel summary](../evidence/switches/SW1%23show%20etherchannel%20summary)
  - [SW2 – show etherchannel summary](../evidence/switches/SW2%23show-etherchannel-summary)
- Spanning-tree summary (reference):
  - [SW1 – show spanning-tree summary](../evidence/switches/SW1%23show%20spanning-tree%20summary)
  - [SW2 – show spanning-tree summary](../evidence/switches/SW2_show-spanning-tree-summary)

### Routers
- Interface state:
  - [R1 – show ip interface brief](../evidence/routers/R1_show-ip-int-brief.txt)
  - [R2 – show ip interface brief](../evidence/routers/R2_show-ip-int-brief)
- Routing tables:
  - [R1 – show ip route](../evidence/routers/R1_show-ip-route)
  - [R2 – show ip route](../evidence/routers/R2_show-ip-route)
- HSRP:
  - [R1 – show standby brief](../evidence/routers/R1_show-standby-brief)
  - [R2 – show standby brief](../evidence/routers/R2_show-standby-brief)
- ACLs and counters:
  - [R1 – show ip access-lists](../evidence/routers/R1_show-ip-access-lists)
  - [R2 – show ip access-lists](../evidence/routers/R2_show-ip-access-lists)
- Router configs (reference):
  - `configs/routers/R1_running-config.txt`
  - `configs/routers/R2_running-config.txt`

### Firewalls
- ASA01 routing, NAT, VPN split tunnel:
  - [ASA01 validation](../evidence/firewalls/ASA01_validation.txt)
- ASA02 routing and ACLs:
  - [ASA02 validation](../evidence/firewalls/ASA02_validation.txt)

---

## Validation summary (baseline)

### L2 Switching

#### VLAN definition and consistency
**Objective:** Ensure all required VLANs exist, including blackhole VLAN.  
**Evidence:** SW1/SW2 `show vlan brief`  
**Expected:** VLANs 10,20,30,40,50,60,70,80,85,90 and 999 present.  

---

#### Trunk configuration and forwarding
**Objective:** Confirm trunks use native VLAN 999 and forward only required VLANs.  
**Evidence:** SW1/SW2 `show interfaces trunk`  
**Expected:**  
- Native VLAN = 999  
- Allowed VLAN list matches design  

---

#### EtherChannel (LACP) between SW1 and SW2
**Objective:** Confirm inter-switch redundancy and aggregation are operational.  
**Evidence:** SW1/SW2 `show etherchannel summary`  
**Expected:**  
- Port-channel1 present  
- Member links bundled (`P`)  

---

### L3 Routing and Redundancy

#### Inter-VLAN routing interfaces
**Objective:** Confirm router subinterfaces are operational.  
**Evidence:** R1/R2 `show ip interface brief`  
**Expected:** VLAN subinterfaces and uplinks show `up/up`.  

---

#### Default routing toward firewall
**Objective:** Confirm routers forward unknown traffic to the edge firewall.  
**Evidence:**  
- R1 `show ip route`  
- R2 `show ip route`  
**Expected:** Default route present toward respective ASA inside interface.  

---

#### HSRP first-hop redundancy
**Objective:** Verify gateway redundancy and role assignment.  
**Evidence:** R1/R2 `show standby brief`  
**Expected:**  
- Virtual IP = `.1` per VLAN  
- R1 Active, R2 Standby  

---

### DHCP Relay

#### DHCP helper configuration
**Objective:** Ensure user VLANs forward DHCP requests to server VLAN.  
**Evidence:** R1 running config (`ip helper-address` under VLAN subinterfaces)  
**Expected:** Helper addresses pointing to DC01 and DC02 (VLAN 80).  

---

### Segmentation and Access Control

#### Guest VLAN isolation
**Objective:** Prevent guest access to internal enterprise subnets.  
**Evidence:** R1 `show ip access-lists`  
**Expected:**  
- DHCP permitted  
- Guest → 192.168.0.0/16 denied  
- Guest → internet permitted 

---

#### Restricted VLAN policies (Warehouse / Production)
**Objective:** Limit lateral movement while preserving server and internet access.  
**Evidence:** R1 `show ip access-lists`  
**Expected:**  
- Permit DHCP  
- Permit access to server VLAN  
- Deny other internal RFC1918 ranges  

---

#### Management plane protection
**Objective:** Restrict device access to management networks and SSH only.  
**Evidence:**  
- R1/R2 `show ip access-lists`  
- R1/R2 running configs (VTY)  
**Expected:**  
- `access-class MGMT_VTY in`  
- `transport input ssh`  

---

### Firewall Edge Validation

#### ASA01 NAT and routing
**Objective:** Confirm outbound connectivity and NAT translation.  
**Evidence:** ASA01 validation output  
**Expected:**  
- Default route via outside  
- Dynamic PAT configured with translation hits  

---

#### ASA02 routing and NAT policy
**Objective:** Confirm secondary firewall can forward and translate traffic.  
**Evidence:** ASA02 validation output  
**Expected:**  
- Default route via outside  
- Inside route to 192.168.0.0/16  
- NAT policy present  

---

## Summary
This baseline validation confirms that:
- Layer 2 forwarding, aggregation, and loop prevention are operational
- Layer 3 routing and gateway redundancy are functional
- Segmentation and management access controls are enforced
- Edge firewalls provide routed, NATed internet access
