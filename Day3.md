# Splunk SIEM Home-Lab: Day 3 - Troubleshooting & Detection Demo

**Date:** November 9, 2025  
**Status:** ✅ **COMPLETED**  

---

## Executive Summary

Day 3 focused on **diagnosing and resolving Suricata alert ingestion issues** within Splunk SIEM, validating end-to-end detection pipeline from attack generation to security event visualization.

- ✅ **Permissions Fixed** - Ensured Splunk could read Suricata logs (`eve.json`)
- ✅ **Service Restart** - Reloaded Suricata and Splunk services
- ✅ **Attack Traffic Generated** - Used Kali-Attacker to simulate real-world intrusions
- ✅ **Alerts Monitored** - Verified alert logs in both Ubuntu SIEM terminal and Splunk GUI
- ✅ **SIEM Visualization** - Confirmed that alerts are now indexed and viewable in Splunk Search

**Key Achievement:** Achieved full IDS-to-SIEM integration with validated alert detection on simulated attacks.

---

## Day 3 Objectives vs. Completion

| Objective                               | Status        | Evidence                       |
|------------------------------------------|--------------|--------------------------------|
| Diagnose Splunk GUI alert ingestion      | ✅ Complete   | GUI now displays alerts        |
| Fix file and process permissions         | ✅ Complete   | `chown`/`chmod`/`usermod` done |
| Restart Splunk & Suricata services       | ✅ Complete   | No errors; fresh logs indexed  |
| Generate attack activity (curl, nmap)    | ✅ Complete   | Alerts fired in eve.json       |
| Live alert monitoring (terminal)         | ✅ Complete   | Alerts seen in real-time tail  |
| Confirm end-to-end pipeline in GUI       | ✅ Complete   | `event_type=alert` shows data  |

---

## Lab Execution & Key Commands

### Fix Permissions  
*Ubuntu Splunk-SIEM terminal:*
```
sudo chown root:splunk /var/log/suricata/eve.json
sudo chmod 664 /var/log/suricata/eve.json
sudo usermod -a -G splunk splunk
```

### Restart Services  
*Ubuntu Splunk-SIEM terminal:*
```
sudo systemctl restart suricata
sudo /opt/splunk/bin/splunk restart
```

### Generate Attacks  
*Kali-Attacker terminal:*
```
curl -v -A "sqlmap/1.4" "http://192.168.20.11:8000/"
nmap -sS -p 21,22,80,8000 192.168.20.11
```

### Monitor Alerts  
*Ubuntu SIEM terminal (real-time):*
```
sudo tail -f /var/log/suricata/eve.json | grep alert
```
*Splunk Web GUI (Search):*
```
index=main sourcetype=suricata:eve event_type=alert
```

---

## End-to-End Demo Steps

1. **Simulate attacks:** Send requests from Kali Linux to monitored VM
2. **Check eve.json:** Confirm Suricata generates alerts for attacks
3. **Validate in Splunk:** Ensure new alerts appear in Splunk Search after each attack
4. **Troubleshoot:** If not visible, rerun permissions and restarts and repeat test
5. **Success:** GUI and terminal both show identical, live Suricata alerts

---

## Summary Table

| Step               | Machine           | Command/Action                                    | Outcome                        |
|--------------------|-------------------|---------------------------------------------------|-------------------------------|
| Fix permissions    | Ubuntu Splunk-SIEM| `chown`, `chmod`, `usermod`                       | Splunk reads eve.json         |
| Restart services   | Ubuntu Splunk-SIEM| `systemctl`, `splunk restart`                     | Config reloaded               |
| Attack test        | Kali-Attacker     | `curl`, `nmap`                                    | Suricata triggers alerts      |
| Alert monitoring   | Ubuntu Splunk-SIEM| `tail -f ...eve.json | grep alert`                | Alerts visible in terminal    |
| SIEM check         | Splunk Web GUI    | `index=main sourcetype=suricata:eve event_type=alert` | Alerts visible in GUI         |

---

## Reflection

- Diagnosed ingestion pipeline issues (permissions, ownership)
- Validated blue-team detection scenario for simulated attacks
- Confirmed SIEM is now ready for advanced log parsing, dashboards, and real-world security use cases

**End of Day 3 Report**
