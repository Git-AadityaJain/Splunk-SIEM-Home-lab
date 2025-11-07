# Splunk SIEM Home-Lab

## Project Overview  
This project is a comprehensive cybersecurity home lab designed to simulate a realistic enterprise environment with segmented networks, attacker simulation, and centralized logging using Splunk SIEM. The goal is to learn and practice detection, defense, and attack techniques in a controlled virtual environment.

## Lab Architecture  
The lab consists of the following core network segments and components:

- **SIEM Server:** Ubuntu Server running Splunk Enterprise and Suricata IDS to collect and analyze security logs.
- **Firewall:** pfSense running as the network firewall to enforce segmentation and route traffic between segments. Suricata integrated with pfSense monitors external traffic.
- **DMZ:** Windows Server and Ubuntu Server hosting web applications, with Splunk Universal Forwarder installed to send logs to the SIEM.
- **Applications:** Ubuntu server running enterprise applications including Docker, Kubernetes, and NGINX.
- **Attacker Machine:** Kali Linux with Metasploit and other offensive tools to simulate insider and external threats.

## Architecture Diagram  
<img width="1174" height="499" alt="image" src="https://github.com/user-attachments/assets/e32b9480-f809-48f3-bd4a-f74a5d095d34" />


## Technologies Used  
- VMware Workstation Pro for virtualization  
- pfSense firewall and Suricata IDS  
- Ubuntu Server and Windows Server  
- Splunk Enterprise and Universal Forwarder  
- Kali Linux for attack simulations  
- Docker, Kubernetes, NGINX  

## Setup Process & Timeline  

| Day        | Activity                                      |
|------------|---------------------------------------------|
| Day 1      | VMware installation, network segmentation setup         |
| Day 2      | pfSense firewall installation and Suricata integration |
| Day 3      | Splunk Enterprise setup, log ingestion configuration   |
| Day 4      | DMZ servers deployment and Splunk Universal Forwarder setup |
| Day 5      | Application server deployment (Docker, Kubernetes, NGINX) |
| Day 6      | Attacker VM deployment & attack simulation               |
| Day 7      | Integration testing, dashboard creation, alert configuration, and hardening |

## Network Configuration  
- **VMnet1 (WAN/Internet-facing):** 192.168.10.0/24  
- **VMnet2 (LAN/Internal SIEM):** 192.168.20.0/24  
- **VMnet3 (DMZ):** 192.168.30.0/24  
- **VMnet4 (Applications):** 192.168.40.0/24  

Each VM is connected to one or more of these segmented networks based on its role.

## Installation & Configuration Instructions  

### 1. VMware Network Editor Customization  
- Open VMware Workstation > Edit > Virtual Network Editor.  
- Create VMnet1 - VMnet4 for segmentation with subnet assignments above.  
- Disable DHCP on these VMnets to avoid IP conflicts; pfSense will manage IP addressing.  
- Assign NICs for each VM based on network segment roles from architecture.

### 2. pfSense Installation & Suricata Setup  
- Deploy pfSense VM with four NICs (one for each VMnet).  
- Install pfSense from official ISO.  
- Configure interfaces with static IPs per subnet.  
- Install Suricata package in pfSense.  
- Enable Suricata for WAN, DMZ, and LAN interfaces in IDS mode.  
- Configure Suricata logging to syslog for forwarding to Splunk.  
- Create firewall rules to enforce network segmentation per design.

### 3. Splunk Enterprise & Universal Forwarder  
- Deploy Ubuntu Server for Splunk Enterprise on LAN network.  
- Install Splunk Enterprise (free tier recommended).  
- Configure HTTP Event Collector (HEC) for log ingestion.  
- Install Splunk Universal Forwarder on DMZ Ubuntu and Windows server VMs.  
- Configure forwarders to send logs to Splunk over network.

### 4. Attacker Machine Setup  
- Deploy Kali Linux VM on WAN or attacker network.  
- Install Metasploit, Hydra, and scanning tools like Nmap.  
- Verify connectivity to DMZ and Apps networks through firewall.  
- Prepare and run attack simulations, watch Splunk for alerting.

## Usage and Testing  
- Launch attacks from Kali and observe detection in Splunk.  
- Monitor dashboards for Suricata IDS alerts and log data.  
- Test firewall policy invalid access and lateral movement attempts.  
- Tune rules and alerts for optimal detection.

## Future Enhancements  
- Add automated response playbooks in Splunk.  
- Integrate Blue Team tools like ELK Stack or OSSEC.  
- Expand attack scenarios for advanced detection training.  
- Automate VM deployments with Terraform or Ansible.

## Contributing  
Contributions and improvements are welcome! Please open issues or submit pull requests.

## License  
This project is licensed under the MIT License.

---

Created as a student project for practical cybersecurity skills development using Splunk SIEM and network defense.
