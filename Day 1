# Splunk SIEM Home-Lab - Day 1 Setup

## Objective

Set up the foundational segmentation and routing environment for a cybersecurity SIEM home lab, following enterprise-style architecture using VMware Workstation, pfSense as firewall/router, and Ubuntu Server for Splunk SIEM.

---

## Tasks Completed

- **Installed VMware Workstation Pro (version 25H2 for Windows)**  
  Ensured host had adequate resources for multiple VMs and configured virtualization settings.

- **Configured Custom Virtual Networks (VMnet2 to VMnet5)**  
  - VMnet2 (WAN): 192.168.10.0/24  
  - VMnet3 (LAN): 192.168.20.0/24  
  - VMnet4 (DMZ): 192.168.30.0/24  
  - VMnet5 (Apps): 192.168.40.0/24  
  - Disabled DHCP on all custom networks so pfSense manages IPs.

- **Deployed & Installed pfSense VM**
  - Added four NICs, one for each VMnet segment.
  - Assigned network names and static IPs per segment.
  - Launched pfSense console, assigned interfaces, and set IP addresses.
  - Accessed WebGUI from internal client VM.

- **Configured pfSense Initial Settings via WebGUI**
  - Changed default admin password
  - Set hostname, domain (`lab.local`), and DNS servers (e.g., 8.8.8.8, 1.1.1.1).
  - Assigned correct interfaces under 'Interface Assignments'.

- **Enabled DHCP Server on Internal Segments**
  - Configured DHCP ranges for LAN, DMZ, and Apps interfaces.

- **Created and Applied Firewall Rules for Connectivity Testing**
  - Navigated to Firewall > Rules in pfSense WebGUI.
  - Added “Allow All” rules to LAN, DMZ, and Apps.

- **Started Ubuntu Server VM (“Splunk-SIEM”)**
  - Attached to LAN/SIEM segment (VMnet3).
  - Checked DHCP/static IP and tested connectivity (`ping 192.168.20.1`).

---

## Problems Faced and Solutions

- **Default VMnet1 conflicted with planned WAN segment**
  - VMnet1 was host-only with unwanted DHCP/subnet.
  - *Solution*: Left VMnet1 as is, created new networks VMnet2–VMnet5 for proper segmentation.

- **DHCP Confusion on Custom Networks**
  - VMware auto-enabled DHCP on new VMnets.
  - *Solution*: Disabled DHCP per segment within VMware Virtual Network Editor, then configured all DHCP via pfSense.

- **Accessing pfSense WebGUI from client VM**
  - Initially couldn't reach WebGUI; realized wrong network adapter was assigned.
  - *Solution*: Set Ubuntu Server network adapter to VMnet3 (LAN), verified IP assignment, then successfully logged in with `https://192.168.20.1`.

---

## Screenshots

- pfSense interface assignments  
- Hostname/DNS setup  
- Firewall rules screen  
- Ubuntu connectivity test (ping results)



---

## Conclusion

Successfully segmented the home lab environment, deployed pfSense firewall, and verified initial connectivity. All internal segments are isolated and managed by pfSense. Ubuntu Server is up and reachable, ready for Splunk installation on Day 2.  
All issues resolved effectively during bootstrapping. The lab is now ready for SIEM, IDS, and application/test VM deployments.

