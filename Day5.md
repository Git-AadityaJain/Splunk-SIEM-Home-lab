# Day 5: Splunk SIEM Home-Lab - Network Architecture & Attacker Setup

**Date:** November 11, 2025  
**Focus:** Correcting network topology, understanding external attack flow, and preparing for firewall + SIEM testing

---

## Objective Summary

By the end of Day 5, you will have:
- ✅ Understood the correct network architecture for SIEM labs
- ✅ Properly positioned the attacker (Kali) on the WAN-side (not DMZ/OPT)
- ✅ Learned why external attack simulation requires traffic to traverse pfSense
- ✅ Set up the foundation for detecting and logging real attack traffic in Splunk

---

## What You Accomplished Before Today

- Set up **4 VMs**: pfSense, Splunk SIEM (Ubuntu), Kali Attacker (Kali Linux), and ZSecurities Kali GUI.
- Configured pfSense interfaces:
  - **WAN (em0):** External/Internet side
  - **LAN (em1):** Management/internal network → `192.168.10.0/24`
  - **OPT1 (em2):** DMZ segment → `192.168.30.0/24`
  - **OPT2 (em3):** Applications (not yet used)
  - **OPT3 (em4):** Initially set for attacker (NOW BEING CORRECTED)
- Attempted to place Kali Attacker on OPT3/em4, but encountered connectivity issues.

---

## Key Learning: Correct Network Architecture

### Problem with Previous Setup
You had placed **Kali Attacker on OPT3 (em4)**, same as DMZ. This is **incorrect** for realistic external attack simulation because:

1. **No firewall traversal:** Traffic goes directly inside the protected network, not through the perimeter.
2. **No IDS/Suricata inspection:** Attacks bypass the main defensive layer.
3. **Not realistic:** In real-world scenarios, attackers come from *outside* (internet/WAN), not inside DMZ.
4. **Lab goals unmet:** SIEM cannot properly log and detect attacks that don't cross firewall boundaries.

---

### Correct Architecture: External Attack Flow

**Traffic should flow:**
```
Attacker Kali (WAN-side) 
    ↓
pfSense WAN Interface (em0) 
    ↓
pfSense Firewall Rules + Suricata IDS 
    ↓
pfSense DMZ Interface (OPT1/em2) 
    ↓
Splunk SIEM (victim in DMZ)
```

---

## Corrected Lab Topology

### VM Placement Table

| VM Name         | VMware VMnet      | pfSense Interface | Example IP        | Role                     |
|-----------------|-------------------|-------------------|-------------------|--------------------------|
| **pfSense**     | VMnet2 (WAN)      | em0 (WAN)         | 192.168.60.1      | Gateway/Firewall/IDS     |
| **pfSense**     | VMnet3 (LAN)      | em1 (LAN)         | 192.168.10.1      | Management network       |
| **pfSense**     | VMnet4 (DMZ)      | em2 (OPT1/DMZ)    | 192.168.30.1      | DMZ gateway              |
| **Kali Attacker**| VMnet2 (WAN)      | —                 | 192.168.60.10     | **Attacker (External)**  |
| **Splunk SIEM** | VMnet4 (DMZ)      | —                 | 192.168.30.x      | Victim/Target            |
| **ZSec Kali**   | VMnet3 (LAN)      | —                 | 192.168.10.x      | Admin/GUI/Management     |

### Key Changes:
- **Kali Attacker MUST be on VMnet2 (WAN-side)**, same VMnet as pfSense WAN (em0).
- **Splunk SIEM stays on VMnet4 (DMZ)**, same as pfSense OPT1/DMZ.
- This ensures all attack traffic crosses pfSense and is visible to Suricata IDS and SIEM.

---

## Step-by-Step: Fixing the Network

### Step 1: Reconfigure Kali Attacker Network Adapter

1. **Power off Kali Attacker VM** (if running).
2. **Open VMware Player/Workstation:**
   - Edit Kali Attacker VM settings.
   - Go to **Network Adapter**.
   - Change from **VMnet6** to **VMnet2** (or the VMnet assigned to pfSense WAN).
   - Ensure it's set to **"Custom (Host-only)"** or **NAT** (NAT is closer to realistic internet).
   - **Save and power on**.

### Step 2: Assign Static IP to Kali Attacker

On Kali Attacker, run:
```
sudo nano /etc/network/interfaces
```

Add or modify to:
```
auto eth0
iface eth0 inet static
  address 192.168.60.10
  netmask 255.255.255.0
  gateway 192.168.60.1
  dns-nameservers 8.8.8.8
```

Save and apply:
```
sudo systemctl restart networking
```

Or for temporary configuration:
```
sudo ifconfig eth0 192.168.60.10 netmask 255.255.255.0 up
sudo route add default gw 192.168.60.1
```

### Step 3: Verify Kali IP and Connectivity

On Kali:
```
ip addr show
ip route show
ping 192.168.60.1
```

**Expected output:**
- IP: `192.168.60.10/24`
- Gateway: `192.168.60.1`
- Ping to gateway: Should get replies ✓

### Step 4: Verify pfSense WAN Configuration

1. Access **pfSense Web GUI** (from management VM or console).
2. Go to **Interfaces > WAN**.
3. Confirm:
   - **IPv4 Address:** `192.168.60.1/24` (or your WAN subnet)
   - **Status:** "Up"
4. Go to **Status > Interfaces** and verify WAN shows **"UP"** and the correct IP.

### Step 5: Test Attack Traffic Path

From Kali Attacker:
```
# Ping pfSense WAN (should succeed)
ping 192.168.60.1

# Ping Splunk in DMZ (should be blocked unless you allow)
ping 192.168.30.x

# Try nmap to DMZ (realistic attack)
nmap -sV 192.168.30.x
```

**Expected Behavior:**
- ✓ Ping to pfSense WAN: **SUCCESS** (direct neighbor)
- ✗ Ping to DMZ: **TIMEOUT** (firewall blocks by default, good!)
- ✓ Nmap sees firewall rules in action

### Step 6: Enable Firewall Rules to Allow Test Traffic

In **pfSense Web GUI:**
1. Go to **Firewall > Rules > WAN**.
2. Add a rule to allow traffic from WAN to DMZ (for testing):
   - **Action:** Pass
   - **Source:** WAN net
   - **Destination:** DMZ net (192.168.30.0/24)
   - **Protocol:** TCP/UDP
   - **Port:** Specific service port (e.g., 8000 for Splunk)
3. **Apply**.

Now, nmap and attacks from Kali should reach Splunk in DMZ and be logged.

---

## Why This Architecture Matters for SIEM

### Suricata IDS Will Inspect:
- All packets entering pfSense WAN from attacker
- All packets being routed to DMZ
- Protocol anomalies, payloads, and known attack signatures

### Splunk Will Log:
- **Firewall rules triggered** (blocked/allowed)
- **IDS alerts** (Suricata detections)
- **Attack payload details** if forwarded by pfSense
- **Network flow data** (source, destination, ports, bytes)

### Real-World Scenario:
- External attacker (internet) → Firewall (perimeter defense) → Protected DMZ
- This is what your lab now simulates.

---

## Common Mistakes to Avoid

| ❌ Mistake | ✅ Correct |
|-----------|-----------|
| Attacker on OPT/DMZ | Attacker on WAN-side (external) |
| Direct VM-to-VM communication | All traffic through pfSense |
| No firewall rules | Explicit allow/deny rules |
| Ignoring IDS alerts | Monitor and log all detections |

---

## Quick Validation Checklist

- [ ] Kali Attacker IP: `192.168.60.10`, Gateway: `192.168.60.1`
- [ ] pfSense WAN IP: `192.168.60.1`, Status: UP
- [ ] Kali can ping pfSense WAN
- [ ] Kali cannot ping DMZ (blocked by firewall, expected)
- [ ] Firewall rule added to allow test traffic from WAN to DMZ
- [ ] Splunk SIEM is on DMZ (`192.168.30.x`)
- [ ] Suricata is running on pfSense WAN interface
- [ ] Splunk is receiving pfSense logs and IDS alerts

---
