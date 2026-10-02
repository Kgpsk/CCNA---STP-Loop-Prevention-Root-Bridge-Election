# STP Loop Prevention & Root Bridge Election — Cisco Packet Tracer Lab

## 📌 Overview

This lab demonstrates how **Spanning Tree Protocol (STP)** prevents Layer 2 loops in a redundant switched topology and how to **influence root bridge election** using `spanning-tree vlan <id> root primary`.

**Platform:** Cisco Packet Tracer  
**Protocol:** IEEE 802.1D STP  
**VLAN:** 10 (DATA)  
**Subnet:** 192.168.10.0/24

---

## 🗺️ Topology

```
                [ SW2 ]
               /       \
              /         \
             /           \
        [ SW1 ] -------- [ SW3 ]
          |                 |
        [PC0]             [PC1]
```

| Device | Model | Role |
|--------|-------|------|
| SW1 | Cisco 2960-24TT | Access + Root Bridge |
| SW2 | Cisco 2960-24TT | Distribution |
| SW3 | Cisco 2960-24TT | Access |
| PC0 | PC-PT | Host |
| PC1 | PC-PT | Host |

### 🔗 Cabling

| From | Port | To | Port |
|------|------|----|------|
| PC0 | Fa0 | SW1 | Fa0/1 |
| PC1 | Fa0 | SW3 | Fa0/1 |
| SW1 | G0/1 | SW2 | G0/1 |
| SW2 | G0/2 | SW3 | G0/1 |
| SW3 | G0/2 | SW1 | G0/2 |

> The **SW1–SW2–SW3 triangle** creates the Layer 2 loop used to demonstrate STP blocking.

---

## 🎯 Objectives

1. Build the looped topology with 3 switches and 2 PCs.
2. Place all hosts in **VLAN 10**.
3. Configure **802.1Q trunk links** between switches.
4. Observe default STP behavior (`show spanning-tree vlan 10`).
5. Force **SW1** to be the **Root Bridge** for VLAN 10.
6. Verify **Root Port**, **Designated Port**, and **Blocking (Alternate) Port**.
7. Confirm end-to-end connectivity with `ping`.

---

## ⚙️ Configuration

### SW1

```
enable
configure terminal
hostname SW1

vlan 10
 name DATA
exit

interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface range g0/1 - 2
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
exit

interface vlan 10
 no shutdown
exit

spanning-tree vlan 10 root primary

end
write memory
```

### SW2

```
enable
configure terminal
hostname SW2

vlan 10
 name DATA
exit

interface range g0/1 - 2
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
exit

interface vlan 10
 no shutdown
exit

end
write memory
```

### SW3

```
enable
configure terminal
hostname SW3

vlan 10
 name DATA
exit

interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface range g0/1 - 2
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
exit

interface vlan 10
 no shutdown
exit

end
write memory
```

### PCs

| PC | IP Address | Subnet Mask | Gateway |
|----|-----------|-------------|---------|
| PC0 | 192.168.10.10 | 255.255.255.0 | — |
| PC1 | 192.168.10.20 | 255.255.255.0 | — |

---

## 🔍 Verification

### STP on SW1 (Root Bridge)

```
SW1# show spanning-tree vlan 10
```

```
VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    24586
             Address     0000.5811.62AD
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24586  (priority 24576 sys-id-ext 10)
             Address     0000.5811.62AD
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface   Role Sts Cost      Prio.Nbr Type
----------  ---- --- --------- -------- ----
G0/2        Desg FWD 4         128.26   P2p
G0/1        Desg FWD 4         128.25   P2p
Fa0/1       Desg FWD 19        128.1    P2p
```

✅ All ports **Designated / Forwarding** — SW1 is the root.

---

### STP on SW2

```
SW2# show spanning-tree vlan 10
```

```
VLAN0010
  Root ID    Priority    24586
             Address     0000.5811.62AD
             Cost        4
             Port        26(GigabitEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0008.BE18.5008
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface   Role Sts Cost      Prio.Nbr Type
----------  ---- --- --------- -------- ----
G0/2        Root FWD 4         128.26   P2p
G0/1        Desg FWD 4         128.25   P2p
```

- **G0/2** → Root Port (best path to SW1)
- **G0/1** → Designated Port

---

### STP on SW3

```
SW3# show spanning-tree vlan 10
```

```
VLAN0010
  Root ID    Priority    24586
             Address     0000.5811.62AD
             Cost        4
             Port        25(GigabitEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0030.A3C8.6452
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface   Role Sts Cost      Prio.Nbr Type
----------  ---- --- --------- -------- ----
G0/1        Root FWD 4         128.25   P2p
Fa0/1       Desg FWD 19        128.1    P2p
G0/2        Altn BLK 4         128.26   P2p
```

- **G0/1** → Root Port (Forwarding)
- **G0/2** → **Alternate / Blocking** ← 🛑 STP loop prevention

---

## ✅ Connectivity Test

From PC0:

```
C:\> ping 192.168.10.20

Reply from 192.168.10.20: bytes=32 time<1ms TTL=128
Reply from 192.168.10.20: bytes=32 time<1ms TTL=128
Reply from 192.168.10.20: bytes=32 time<1ms TTL=128
Reply from 192.168.10.20: bytes=32 time<1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

✅ End-to-end connectivity works even with one port blocking.

---

## 📊 STP Summary

| Switch | Bridge Priority | Role | Port States |
|--------|----------------|------|-------------|
| SW1 | 24586 | **Root Bridge** | All Desg / FWD |
| SW2 | 32778 | Non-Root | 1 Root FWD, 1 Desg FWD |
| SW3 | 32778 | Non-Root | 1 Root FWD, 1 Desg FWD, **1 Altn BLK** |

---

## 🧠 Key Concepts

| Concept | Explanation |
|---------|-------------|
| **Bridge ID** | Priority (default 32768) + MAC address |
| **Root Bridge** | Lowest Bridge ID — becomes the STP root |
| **Root Port** | Best path to the root bridge (one per non-root switch) |
| **Designated Port** | Forwarding port for each segment |
| **Alternate Port** | Backup port placed in **Blocking** state to prevent loops |
| **root primary** | Sets priority to **24576** (or 4096 lower than current root) |
| **Port Cost** | 4 (Gigabit), 19 (FastEthernet) — 802.1D |

---

## 🛠️ Useful Commands

```
show spanning-tree
show spanning-tree vlan 10
show spanning-tree vlan 10 root
show spanning-tree interface g0/1
show vlan brief
show interfaces trunk
show ip interface brief
show mac address-table vlan 10
```

---

## ⚠️ Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Ping fails | Trunk not allowing VLAN 10 | `switchport trunk allowed vlan 10` |
| VLAN 10 "down" | No active port in VLAN | Connect PC / `no shutdown` |
| No blocking port | Loop broken | Verify all 3 inter-switch links are up |
| Wrong root | Forgot `root primary` | Re-run on SW1 |
| PC unreachable | Wrong VLAN / IP | Check access VLAN and IP config |

---

## 📁 Files

- `STP.pkt` — Cisco Packet Tracer topology

---

## 👤 Author

Networking lab — Week 13  
**Topic:** STP Loop Prevention & Root Bridge Election  
**Tool:** Cisco Packet Tracer

---

## 📜 License

Educational use only.
