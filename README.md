# Splunk SIEM Home-Lab

## Project Overview
This project is a comprehensive cybersecurity home lab designed to simulate a realistic enterprise environment with segmented networks, attacker simulation, and centralized logging using Splunk SIEM. The goal is to learn and practice detection, defense, and attack techniques in a controlled virtual environment.

## Lab Architecture
The lab consists of the following core network segments and components:

- ![Server Icon](./icons/server.png) **SIEM Server:** Ubuntu Server running Splunk Enterprise and Suricata IDS to collect and analyze security logs.  
- ![Firewall Icon](./icons/firewall.png) **Firewall:** pfSense running as the network firewall to enforce segmentation and route traffic between segments. Suricata integrated with pfSense monitors external traffic.  
- ![DMZ Icon](./icons/dmz.png) **DMZ:** Windows Server and Ubuntu Server hosting web applications, with Splunk Universal Forwarder installed to send logs to the SIEM.  
- ![Applications Icon](./icons/applications.png) **Applications:** Ubuntu server running enterprise applications including Docker, Kubernetes, and NGINX.  
- ![Attacker Icon](./icons/attacker.png) **Attacker Machine:** Kali Linux with Metasploit and other offensive tools to simulate insider and external threats.

![Architecture Diagram](./images/splunk_siem_home_lab_diagram.jpg)

## Technologies Used
- ![VMware Icon](./icons/vmware.png) VMware Workstation Pro for virtualization  
- ![pfSense Icon](./icons/pfsense.png) pfSense firewall and Suricata IDS  
- ![Ubuntu Icon](./icons/ubuntu.png) Ubuntu Server and ![Windows Icon](./icons/windows.png) Windows Server  
- ![Splunk Icon](./icons/splunk.png) Splunk Enterprise and Universal Forwarder  
- ![Kali Icon](./icons/kali.png) Kali Linux for attack simulations  
- ![Docker Icon](./icons/docker.png) Docker, ![Kubernetes Icon](./icons/kubernetes.png) Kubernetes, ![NGINX Icon](./icons/nginx.png) NGINX  

## Setup Process & Timeline

| Day        | Activity                                      |
|------------|-----------------------------------------------|
| Day 1      | VMware installation, network segmentation setup |
| Day 2      | pfSense firewall installation and Suricata integration   |
| Day 3      | Splunk Enterprise setup, log ingestion configuration    |
| Day 4      | DMZ servers deployment and Splunk Universal Forwarder setup |
| Day 5      | Application server deployment (Docker, Kubernetes, NGINX) |
| Day 6      | Attacker VM deployment & attack simulation                |
| Day 7      | Integration testing, dashboard creation, alert configuration, and hardening |

## Network Configuration
- **VMnet1 (WAN/Internet-facing):** 192.168.10.0/24  
- **VMnet2 (LAN/Internal SIEM):** 192.168.20.0/24  
- **VMnet3 (DMZ):** 192.168.30.0/24  
- **VMnet4 (Applications):** 192.168.40.0/24  

Each VM is connected to one or more of these segmented networks based on its role.

## Installation & Configuration Instructions
The detailed step-by-step setup instructions for each component are provided in the [`docs/`](./docs/) folder, including:

- VMware network editor customization  
- pfSense installation and Suricata setup  
- Splunk Enterprise and Universal Forwarder configuration  
- Attacker machine setup for offensive testing

## Usage and Testing
- Launch attacks from Kali Linux VM using Metasploit modules.  
- Monitor live alerts, dashboards, and logs in Splunk UI.  
- Analyze Suricata IDS alerts for network anomalies.  
- Test firewall rules and network isolation via pfSense.

## Future Enhancements
- Implement automated response playbooks in Splunk  
- Add Blue Team tooling like ELK Stack or OSSEC  
- Extend attacker scenarios with advanced threat emulation  
- Automate VM deployment using Ansible or Terraform

## Contributing
Contributions and improvements are welcome! Please submit pull requests or open issues for discussion.

## License
This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.

---

Created as part of a student project to build practical cybersecurity skills with SIEM and network defense.
