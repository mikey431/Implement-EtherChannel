# Packet Tracer - Implement EtherChannel

## 📌 Project Overview
This repository contains the topology setup guide, configuration commands, and verification steps for the **Implement EtherChannel** Packet Tracer activity. In this lab, a three-switch network (`SWA`, `SWB`, and `SWC`) is built from scratch and configured with static trunk links and aggregated **PAgP** and **LACP** EtherChannel bundles to increase uplink bandwidth and provide link redundancy.

---

## 📋 Topology & Port Channel Table

All connections between switches use **Copper Straight-Through** cables.

| Port-Channel | Connected Devices | Port Connections | Protocol | Negotiation Mode |
| :--- | :--- | :--- | :--- | :--- |
| **Port-Channel 1** | **SWA** $\leftrightarrow$ **SWB** | `G0/1` $\leftrightarrow$ `G0/1`<br>`G0/2` $\leftrightarrow$ `G0/2` | **PAgP** | SWA: `desirable`<br>SWB: `desirable` |
| **Port-Channel 2** | **SWA** $\leftrightarrow$ **SWC** | `F0/21` $\leftrightarrow$ `F0/21`<br>`F0/22` $\leftrightarrow$ `F0/22` | **LACP** | SWA: `active`<br>SWC: `active` |
| **Port-Channel 3** | **SWB** $\leftrightarrow$ **SWC** | `F0/23` $\leftrightarrow$ `F0/23`<br>`F0/24` $\leftrightarrow$ `F0/24` | **LACP** | SWB: `passive`<br>SWC: `active` |

---

## 🛠️ Part 1: Build the Network

1. **Add Switches:** Place three **Cisco 2960** switches into the Packet Tracer workspace.
2. **Set Display Names:** Rename the device display labels below each switch icon to **`SWA`**, **`SWB`**, and **`SWC`** (case-sensitive for Packet Tracer scoring).
3. **Cable Connections:** Connect the switches using **Copper Straight-Through** cables according to the Port Channel Table above.

---

## ⚙️ Part 2: Configure EtherChannel

> ⚠️ **Best Practice:** Shut down physical interfaces before bundling them into a `channel-group` to prevent EtherChannel Misconfig Guard from placing ports into an `err-disabled` state. Configure both the physical member ports and the logical `Port-channel` interfaces as static trunks.

### Switch SWA Configuration
* **Host Configuration & Static Trunks:** Configure `G0/1-2` and `F0/21-22` as static trunks.
* **Port-Channel 1 (PAgP):** Active negotiation (`desirable`) with `SWB`.
* **Port-Channel 2 (LACP):** Active negotiation (`active`) with `SWC`.

```ios
Switch> enable
Switch# configure terminal
Switch(config)# hostname SWA

! Configure static trunks on physical interfaces
SWA(config)# interface range GigabitEthernet 0/1 - 2, FastEthernet 0/21 - 22
SWA(config-if-range)# switchport mode trunk
SWA(config-if-range)# exit

! Configure Port-Channel 1 (PAgP Desirable)
SWA(config)# interface range GigabitEthernet 0/1 - 2
SWA(config-if-range)# shutdown
SWA(config-if-range)# channel-group 1 mode desirable
SWA(config-if-range)# no shutdown
SWA(config-if-range)# exit

SWA(config)# interface port-channel 1
SWA(config-if)# switchport mode trunk
SWA(config-if)# exit

! Configure Port-Channel 2 (LACP Active)
SWA(config)# interface range FastEthernet 0/21 - 22
SWA(config-if-range)# shutdown
SWA(config-if-range)# channel-group 2 mode active
SWA(config-if-range)# no shutdown
SWA(config-if-range)# exit

SWA(config)# interface port-channel 2
SWA(config-if)# switchport mode trunk
SWA(config-if)# exit
```

---

### Switch SWB Configuration
* **Host Configuration & Static Trunks:** Configure `G0/1-2` and `F0/23-24` as static trunks.
* **Port-Channel 1 (PAgP):** Active negotiation (`desirable`) with `SWA`.
* **Port-Channel 3 (LACP):** Passive negotiation (`passive`) — listens for LACP from `SWC` without initiating.

```ios
Switch> enable
Switch# configure terminal
Switch(config)# hostname SWB

! Configure static trunks on physical interfaces
SWB(config)# interface range GigabitEthernet 0/1 - 2, FastEthernet 0/23 - 24
SWB(config-if-range)# switchport mode trunk
SWB(config-if-range)# exit

! Configure Port-Channel 1 (PAgP Desirable)
SWB(config)# interface range GigabitEthernet 0/1 - 2
SWB(config-if-range)# shutdown
SWB(config-if-range)# channel-group 1 mode desirable
SWB(config-if-range)# no shutdown
SWB(config-if-range)# exit

SWB(config)# interface port-channel 1
SWB(config-if)# switchport mode trunk
SWB(config-if)# exit

! Configure Port-Channel 3 (LACP Passive)
SWB(config)# interface range FastEthernet 0/23 - 24
SWB(config-if-range)# shutdown
SWB(config-if-range)# channel-group 3 mode passive
SWB(config-if-range)# no shutdown
SWB(config-if-range)# exit

SWB(config)# interface port-channel 3
SWB(config-if)# switchport mode trunk
SWB(config-if)# exit
```

---

### Switch SWC Configuration
* **Host Configuration & Static Trunks:** Configure `F0/21-24` as static trunks.
* **Port-Channel 2 (LACP):** Active negotiation (`active`) with `SWA`.
* **Port-Channel 3 (LACP):** Active negotiation (`active`) — initiates LACP negotiation with `SWB`.

```ios
Switch> enable
Switch# configure terminal
Switch(config)# hostname SWC

! Configure static trunks on physical interfaces
SWC(config)# interface range FastEthernet 0/21 - 24
SWC(config-if-range)# switchport mode trunk
SWC(config-if-range)# exit

! Configure Port-Channel 2 (LACP Active)
SWC(config)# interface range FastEthernet 0/21 - 22
SWC(config-if-range)# shutdown
SWC(config-if-range)# channel-group 2 mode active
SWC(config-if-range)# no shutdown
SWC(config-if-range)# exit

SWC(config)# interface port-channel 2
SWC(config-if)# switchport mode trunk
SWC(config-if)# exit

! Configure Port-Channel 3 (LACP Active)
SWC(config)# interface range FastEthernet 0/23 - 24
SWC(config-if-range)# shutdown
SWC(config-if-range)# channel-group 3 mode active
SWC(config-if-range)# no shutdown
SWC(config-if-range)# exit

SWC(config)# interface port-channel 3
SWC(config-if)# switchport mode trunk
SWC(config-if)# exit
```

---

## 🔍 Verification Commands

Execute the following commands in privileged EXEC mode on `SWA`, `SWB`, and `SWC` to confirm proper EtherChannel formation and trunking:

1. **Verify EtherChannel Bundle Status:**
   ```ios
   show etherchannel summary
   ```
   *Expected result:* Each `Po` interface should show `(SU)` (Layer 2 and In Use), and all physical member ports should show `(P)` (Bundled in port-channel).

2. **Verify Logical Trunk Links:**
   ```ios
   show interfaces trunk
   ```
   *Expected result:* Logical interfaces `Po1`, `Po2`, and `Po3` appear as active `802.1q` trunks in the `trunking` state.

3. **Verify Spanning Tree Topology:**
   ```ios
   show spanning-tree
   ```
   *Expected result:* Spanning Tree treats each EtherChannel bundle (`Po1`, `Po2`, `Po3`) as a single logical interface rather than individual physical links.