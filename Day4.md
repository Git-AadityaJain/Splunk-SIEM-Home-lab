# Splunk SIEM Home-Lab: Day 4 (Learning Book Format)- Advanced SIEM & Security Operations

**Date:** November 10, 2025  
**Duration:** 4 hours  
**Status:** ✅ **COMPLETED**  

A comprehensive learning guide covering advanced SIEM analytics, dashboard creation, alert correlation, and SOC operations. This document captures all commands executed, the machines they run on, and their expected outputs.

---

## Chapter 1: Understanding Splunk SIEM Operations

### What is a SIEM?

A Security Information and Event Management (SIEM) system collects, aggregates, and analyzes security data from multiple sources to detect and respond to threats. Splunk serves as the data repository and analysis platform, ingesting logs from:

- **IDS/IPS** (Suricata) - Network threat detection
- **Firewalls** (pfSense) - Network access control
- **Operating Systems** - System logs and audit trails
- **Applications** - Business logic and security events

### Day 4 Learning Objectives

Today you will learn how to transform raw security events into actionable intelligence through:

- Building professional dashboards for visualization
- Extracting meaningful fields from complex data
- Correlating related events into attack campaigns
- Automating threat detection with alerts
- Conducting SOC-style investigations

---

## Chapter 2: Dashboard Creation & Visualization

### Understanding Dashboards

A Splunk dashboard is a collection of visualizations that display your security data in real-time. Unlike searches (which are one-time queries), dashboards provide continuous monitoring of your environment's security posture.

### Building Your First Dashboard

#### Step 1: Access Splunk Web Interface

**Machine:** Your local workstation  
**Browser:** Open `http://192.168.20.11:8000`

**Expected Output:**
```
Splunk Web login page appears
Username: admin (or your configured user)
Password: [Your Splunk admin password]
```

**What to do:** Log in with your admin credentials

---

#### Step 2: Create a New Dashboard

**Machine:** Splunk Web GUI (browser)  
**Navigation:** Top menu → "Dashboards" → "Create New Dashboard"

**Expected Interface:**
```
Dashboard Title:      [text field]
Dashboard Description: [text area]
Permissions:          [dropdown: Shared/Private]
Create button:        [blue button]
```

**Action:** Enter these values:
- **Dashboard Title:** `Suricata Alerts Overview`
- **Description:** `Real-time visualization of IDS alerts from Suricata network monitoring`
- **Permissions:** Select "Shared"
- Click **Create**

**Expected Output:**
```
✓ Dashboard created successfully
Redirected to blank dashboard editor
"Add Panel" button visible at top
Canvas shows: "Click 'Add Panel' to add content"
```

---

### Creating Dashboard Panels

Each panel displays data through a search query and visualization type. We'll create 5 panels that collectively show:
1. Recent individual alerts (table view)
2. Alert trends over time (line chart)
3. Top attacking source IPs (bar chart)
4. Top targeted systems (pie chart)
5. Severity distribution (pie chart)

#### Panel 1: Recent Alerts Table

**Machine:** Splunk Web GUI  
**Location:** Dashboard Editor → Add Panel → New

**Step 1: Enter Search Query**

Copy and paste this exact query:

```splunk
index=main sourcetype=suricata:eve event_type=alert
| table _time, src_ip, dest_ip, alert.signature, alert.severity
| sort - _time
| head 50
```

**Query Explanation:**
- `index=main sourcetype=suricata:eve event_type=alert` - Find all IDS alerts
- `| table ...` - Select only these columns to display
- `| sort - _time` - Sort by timestamp (newest first)
- `| head 50` - Show only first 50 results

**Step 2: Configure Visualization**

- Visualization Type: **Table**
- Panel Title: `Recent Suricata Alerts`
- Click **Save**

**Expected Output (Sample Data):**
```
_time                | src_ip        | dest_ip      | alert.signature              | alert.severity
Nov 10 15:45:30.201  | 192.168.30.15 | 192.168.20.11| ET Policy Suspicious inbound | 2
Nov 10 15:45:28.932  | 192.168.30.15 | 192.168.20.11| ET SQL Injection Attempt     | 3
Nov 10 15:45:27.654  | 192.168.30.15 | 192.168.20.11| ET ATTACK_RESPONSE XSS       | 2
Nov 10 15:45:26.401  | 192.168.30.15 | 192.168.20.11| ET Attack Response - Shell   | 3
```

**What You See:** A table showing the most recent 50 alerts with key details for quick reference

---

#### Panel 2: Alert Count Over Time

**Machine:** Splunk Web GUI  
**Location:** Dashboard Editor → Add Panel → New

**Step 1: Enter Search Query**

```splunk
index=main sourcetype=suricata:eve event_type=alert
| timechart count by alert.signature limit=10
```

**Query Explanation:**
- `| timechart count` - Count alerts grouped by time intervals
- `by alert.signature limit=10` - Split by signature, show top 10 types
- Automatic time bucketing (usually 10-minute intervals)

**Step 2: Configure Visualization**

- Visualization Type: **Line Chart**
- Panel Title: `Alert Count Over Time`
- X-Axis: Time
- Y-Axis: Count
- Series: Different signature types (shown as separate lines)
- Click **Save**

**Expected Output (Visual):**
```
Line Chart showing:
- X-axis: Time (15:00 to 16:00)
- Y-axis: Number of alerts (0 to 50+)
- Multiple colored lines representing different signature types
- Spike around 15:45 where attacks occurred
```

**What You See:** Temporal patterns showing when attacks peaked, making it easy to correlate with incident times

---

#### Panel 3: Top Attacker Source IPs

**Machine:** Splunk Web GUI  
**Location:** Dashboard Editor → Add Panel → New

**Step 1: Enter Search Query**

```splunk
index=main sourcetype=suricata:eve event_type=alert
| top src_ip limit=10
```

**Query Explanation:**
- `| top src_ip` - Rank source IPs by alert frequency
- `limit=10` - Show top 10 attackers
- Automatically includes count and percentage

**Step 2: Configure Visualization**

- Visualization Type: **Bar Chart** (vertical columns)
- Panel Title: `Top Attacker Source IPs`
- Click **Save**

**Expected Output (Sample):**
```
Bar Chart:
192.168.30.15  ████████████████████ 450 (85%)
192.168.20.1   ███ 50 (9%)
192.168.20.100 ██ 25 (5%)
Other          █ 5 (1%)
```

**What You See:** Which external/internal IPs are generating the most alerts, helping identify primary threat actors

---

#### Panel 4: Top Target Destination IPs

**Machine:** Splunk Web GUI  
**Location:** Dashboard Editor → Add Panel → New

**Step 1: Enter Search Query**

```splunk
index=main sourcetype=suricata:eve event_type=alert
| top dest_ip limit=10
```

**Query Explanation:**
- `| top dest_ip` - Rank destination IPs by alert frequency
- Shows which systems are most frequently targeted

**Step 2: Configure Visualization**

- Visualization Type: **Pie Chart**
- Panel Title: `Top Target Destination IPs`
- Click **Save**

**Expected Output (Visual):**
```
Pie Chart showing segments:
- 192.168.20.11 (Splunk):  90% (blue)
- 192.168.20.1 (pfSense):   8% (orange)
- Other:                     2% (gray)
```

**What You See:** Which systems are being attacked most, indicating your most valuable/vulnerable assets

---

#### Panel 5: Alert Severity Distribution

**Machine:** Splunk Web GUI  
**Location:** Dashboard Editor → Add Panel → New

**Step 1: Enter Search Query**

```splunk
index=main sourcetype=suricata:eve event_type=alert
| stats count by alert.severity
```

**Query Explanation:**
- `| stats count by alert.severity` - Count alerts grouped by severity level
- Severity values: 1=Low, 2=Medium, 3=High

**Step 2: Configure Visualization**

- Visualization Type: **Pie Chart**
- Panel Title: `Alert Severity Distribution`
- Click **Save**

**Expected Output (Visual):**
```
Pie Chart:
- High (3):    15% (red)
- Medium (2):  50% (yellow)
- Low (1):     35% (green)
```

**What You See:** Risk assessment at a glance - what percentage of your alerts pose high/medium/low severity threats

---

### Arranging Your Dashboard

**Machine:** Splunk Web GUI  
**Location:** Dashboard Editor → Dashboard Canvas

**Actions:**

Drag panels to arrange them:
```
[Panel 1: Table (Recent Alerts)]              [Panel 2: Line Chart (Timeline)]
[Large span across full width]

[Panel 3: Bar Chart (Top IPs)]  [Panel 4: Pie Chart (Top Targets)]  [Panel 5: Pie Chart (Severity)]
```

**Save Your Dashboard**
- Click **Save** (top right)
- Set time range: "All time" (for testing) or "Real-time" (for live monitoring)
- Dashboard is now live and accessible from Dashboards menu

**Expected Result:**
```
✓ Dashboard saved successfully
5 panels visible and updating with live data
Time picker shows: "All time" or "Real-time"
```

---

## Chapter 3: Understanding Field Extraction

### What Are Fields?

In Splunk, a field is a key-value pair extracted from your data. For example:

```json
{
  "timestamp": "2025-11-10T15:45:30Z",     ← field: timestamp, value: "2025-11-10T15:45:30Z"
  "src_ip": "192.168.30.15",                ← field: src_ip, value: "192.168.30.15"
  "alert.signature": "ET SQL Injection"    ← field: alert.signature, value: "ET SQL Injection"
}
```

### JSON Auto-Extraction

Suricata outputs JSON format (eve.json), so Splunk automatically extracts fields. You don't need to configure anything—just reference them in searches!

**Example:** `alert.signature`, `alert.severity`, `src_ip`, `dest_ip` are all automatically available.

### Custom Field Extraction (Optional Advanced Topic)

If you had unstructured log data (e.g., Apache logs with space-delimited format), you would need to manually extract fields using:

- **Regular Expressions** (regex) patterns
- **Delimiter-based** extraction (space, comma, pipe)
- **Key-value** extraction (key=value pairs)

For Day 4, we use Suricata's JSON, so we leverage auto-extraction.

---

## Chapter 4: Alert Correlation Fundamentals

### What is Correlation?

**Correlation** is linking multiple related events together to identify attack patterns. Individual alerts are often noise, but multiple related alerts tell a story:

```
Event 1: Port Scan from 192.168.30.15       ← Reconnaissance
Event 2: SQL Injection from 192.168.30.15   ← Exploitation attempt
Event 3: Data Exfil from 192.168.30.15      ← Attack success

Correlation Result: This is a MULTI-STAGE ATTACK from 192.168.30.15
Risk Level: HIGH (requires investigation)
```

### The Transaction Command

The **transaction** command groups related events into single records based on shared characteristics (like source IP) and time windows.

**Basic Syntax:**
```splunk
| transaction <grouping_field> maxspan=<time_window>
```

**Example:**
```splunk
| transaction src_ip maxspan=10m
```

**What it does:**
1. Groups all events with the same `src_ip` value
2. Within a 10-minute time window
3. Creates a single "transaction" record for each group

---

### Building Your Correlation Search

This is the most important search you'll create today. It identifies sophisticated attack campaigns.

#### Step 1: Access Splunk Search

**Machine:** Splunk Web GUI  
**Navigation:** Top menu → "Search & Reporting"

**Expected Output:**
```
Search page opens
Search box visible at top
"Search" button ready
```

---

#### Step 2: Enter Correlation Query

Copy and paste this complete query:

```splunk
index=main sourcetype=suricata:eve event_type=alert
| eval attack_type=case(
    match(alert.signature, "(?i)port.*scan|network.*scan|suspicious.*scan"), "Port Scan",
    match(alert.signature, "(?i)sql.*injection|sql.*attempt"), "SQL Injection",
    match(alert.signature, "(?i)cross.*site|xss"), "XSS Attack",
    match(alert.signature, "(?i)directory.*traversal|path.*traversal|\.\.\/"), "Path Traversal",
    match(alert.signature, "(?i)command.*injection|shell.*command"), "Command Injection",
    match(alert.signature, "(?i)brute.*force|login.*attempt"), "Brute Force",
    1=1, "Other"
)
| transaction src_ip maxspan=10m
| where mvcount(attack_type) > 1
| table _time, src_ip, attack_type, alert.signature, alert.severity
| sort - _time
```

**Step 3: Execute Search**

- Click **Search** button
- Wait for results (30-60 seconds)

**Expected Output:**
```
Results showing transactions with multiple attack types:

_time               | src_ip        | attack_type                          | alert.signature           | alert.severity
Nov 10 15:45:30.201 | 192.168.30.15 | Port Scan                           | ET Policy Suspicious...   | 2
                    |               | SQL Injection                        | ET SQL Injection...       | 3
                    |               | XSS Attack                           | ET ATTACK_RESPONSE XSS    | 2
                    |               | Command Injection                    | ET Policy Shell...        | 3

Nov 10 15:02:15.500 | 192.168.30.15 | Brute Force                         | ET Policy SSH Brute...    | 2
                    |               | Command Injection                    | ET Policy Command...      | 3
```

**What You See:** 
- Each row represents a "transaction" (attack campaign from one source IP)
- Columns show: when it occurred, attacker IP, types of attacks, specific signatures
- Multiple rows = multiple distinct attack campaigns

---

### Understanding the Query Logic

**Part 1: Classification**
```splunk
| eval attack_type=case(
    match(alert.signature, "(?i)sql.*injection"), "SQL Injection",
    ...
)
```

This creates a new field `attack_type` by pattern-matching the signature. `(?i)` means case-insensitive.

**Part 2: Grouping**
```splunk
| transaction src_ip maxspan=10m
```

Groups alerts by source IP, within 10-minute windows. Each transaction becomes a single row with multiple values in the `attack_type` field.

**Part 3: Filtering**
```splunk
| where mvcount(attack_type) > 1
```

`mvcount()` counts distinct values. Only keep transactions with 2+ different attack types (multi-stage attacks).

**Part 4: Display**
```splunk
| table _time, src_ip, attack_type, alert.signature, alert.severity
| sort - _time
```

Show relevant columns, sort newest first.

---

## Chapter 5: Automated Threat Detection with Alerts

### Why Automate?

Running manual searches 24/7 is impossible. Alerts automatically:
- Run on a schedule (every 5 minutes)
- Evaluate conditions (if any results found)
- Take actions (send email, create tickets, trigger webhooks)

### Creating an Automated Alert

#### Step 1: Access Alert Settings

**Machine:** Splunk Web GUI  
**Navigation:** Top menu → "Settings" → "Searches, Reports, and Alerts"

**Expected Output:**
```
Settings page opens
Left sidebar shows: Search, Reports & Alerts, Data inputs, etc.
Main panel shows: "New Alert" button (top right)
```

**Action:** Click **"New Alert"** button

---

#### Step 2: Enter Alert Search Query

**Machine:** Splunk Web GUI  
**Location:** Alert creation form → Search query field

**Paste this query** (similar to correlation search, but stricter):

```splunk
index=main sourcetype=suricata:eve event_type=alert alert.severity>=2
| eval attack_type=case(
    match(alert.signature, "(?i)port.*scan|network.*scan"), "Port Scan",
    match(alert.signature, "(?i)sql.*injection"), "SQL Injection",
    match(alert.signature, "(?i)xss|cross.*site"), "XSS",
    match(alert.signature, "(?i)command.*injection|shell"), "Command Injection",
    match(alert.signature, "(?i)path.*traversal|directory.*traversal"), "Path Traversal",
    1=1, "Other"
)
| transaction src_ip maxspan=15m
| where mvcount(attack_type) > 2
| table src_ip, attack_type, alert.severity, alert.signature, _time
```

**Key Differences from Correlation Search:**
- `alert.severity>=2` - Only High/Medium severity (filter noise)
- `maxspan=15m` - Slightly longer window (15 min vs 10 min)
- `mvcount(attack_type) > 2` - Flag when 3+ types detected (vs 2+)
- This is stricter = fewer false positives

---

#### Step 3: Configure Alert Details

**Machine:** Splunk Web GUI  
**Location:** Alert settings form

Fill in these fields:

| Field | Value |
|-------|-------|
| Alert Name | `High-Risk Multi-Stage Attack Detected` |
| Description | `Alerts when single source launches 3+ attack types in 15 minutes` |
| Search | [Your query above] |
| Run on Schedule | Every 5 minutes |
| Trigger Condition | Greater than 0 results |

**Expected Output:**
```
Form filled with your values
All required fields have ✓ icons
"Next" or "Continue" button visible
```

---

#### Step 4: Configure Alert Actions

**Machine:** Splunk Web GUI  
**Location:** Alert settings → Actions section

**Option A: Create Notable Event** (Recommended for Lab)

Check box: `Create Notable Event`

Settings:
```
Event Title:      Multi-Stage Attack: $result.src_ip$ - $result.attack_type$
Description:      Multi-stage attack from $result.src_ip$. 
                   Types: $result.attack_type$. 
                   Severity: $result.alert.severity$
Severity:         High
Status:           Unreviewed
```

**Option B: Send Email** (Additional)

Check box: `Send email`

Settings:
```
To:       admin@splunk.local
Subject:  ALERT: Multi-Stage Attack Detected!
Message:  Multi-stage attack detected!
          
          Source IP: $result.src_ip$
          Attack Types: $result.attack_type$
          Severity: $result.alert.severity$
          
          Investigate immediately!
```

**Expected Output:**
```
✓ Both action boxes checked
Email and Notable Event configured
Additional options visible below
```

---

#### Step 5: Save Alert

**Machine:** Splunk Web GUI  
**Location:** Alert settings → Bottom

Click **"Save Alert"** button

**Expected Output:**
```
✓ Alert saved successfully
Message: "Alert 'High-Risk Multi-Stage Attack Detected' has been created"
Redirected to alert list view
Your new alert visible in list
```

---

## Chapter 6: Generating Test Traffic - Attack Execution

### Preparing Your Monitoring Environment

Before running attacks, open multiple terminals to observe real-time events:

#### Terminal Setup

**Terminal 1: Kali-Attacker (Attack Execution)**
```
Machine: Kali Linux VM (192.168.30.X)
Purpose: Execute attack commands
Ready: Prepared for attack scripts
```

**Terminal 2: Ubuntu SIEM (Suricata Live Monitoring)**
```
Machine: Ubuntu Splunk-SIEM (192.168.20.11)
Command: sudo tail -f /var/log/suricata/eve.json | grep alert
Purpose: Watch alerts as they're generated in real-time
Expected Output: 
  {"timestamp":"2025-11-10T15:45:30.201","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":1234,...
  {"timestamp":"2025-11-10T15:45:31.342","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":5678,...
```

**Terminal 3: Ubuntu SIEM (Alert Counter)**
```
Machine: Ubuntu Splunk-SIEM (192.168.20.11)
Command: watch -n 5 'sudo tail -100 /var/log/suricata/eve.json | grep alert | wc -l'
Purpose: See alert count updating every 5 seconds
Expected Output:
  Every 5s:
  Alerts in last 100 events: 23
  Alerts in last 100 events: 27
  Alerts in last 100 events: 31
```

**Terminal 4: Browser (Splunk Dashboard)**
```
Machine: Your local workstation
URL: http://192.168.20.11:8000
Location: Dashboards > Suricata Alerts Overview
Purpose: Watch dashboard update in real-time
Expected: Panels refresh every 10-15 seconds showing new data
```

---

### Attack 1: SQL Injection Campaign

**Machine:** Kali-Attacker (192.168.30.X)  
**Location:** Terminal 1  
**Duration:** ~2 minutes

#### Phase 1A: Database Port Reconnaissance

**Command:**
```bash
nmap -sS -p 3306,1433,5432,27017 192.168.20.11 -q
```

**What it does:**
- Scans target (192.168.20.11) for database ports
- `-sS`: TCP SYN (stealth) scan
- `-p`: Specific ports (MySQL 3306, MSSQL 1433, PostgreSQL 5432, MongoDB 27017)
- `-q`: Quiet output

**Expected Output (on Kali):**
```
Nmap 7.92 scan report for 192.168.20.11
Host is up (0.0024s latency).

PORT      STATE  SERVICE
3306/tcp  closed mysql
1433/tcp  closed mssql-s
5432/tcp  closed postgresql
27017/tcp closed mongodb
```

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:45:30.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2026709,"signature":"ET Policy Suspicious inbound to MYSQL port 3306","category":"","severity":2}}
{"timestamp":"2025-11-10T15:45:30.304Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2026708,"signature":"ET Policy Suspicious inbound to MSSQL port 1433","category":"","severity":2}}
```

**Alert Counter (Terminal 3):**
```
Alerts in last 100 events: 5
```

---

#### Phase 1B: SQL Injection Payloads

**Wait 2 seconds, then execute:**

```bash
curl -s "http://192.168.20.11:8000/search?username=admin' --&password=anything" > /dev/null
sleep 2
curl -s "http://192.168.20.11:8000/api/users?id=1' UNION SELECT NULL,username,password FROM users --" > /dev/null
sleep 2
curl -s "http://192.168.20.11:8000/search?id=1' AND SLEEP(5)--" > /dev/null
```

**What it does:**
- Sends HTTP requests with SQL injection payloads embedded
- `curl -s`: Silent HTTP client (no progress bar)
- Payloads: SQL syntax designed to exploit weak input validation
- `> /dev/null`: Discard HTTP response (we only care about IDS detection)

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:45:35.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2003055,"signature":"ET SQL Injection Attempt - UNION SELECT","category":"","severity":3}}
{"timestamp":"2025-11-10T15:45:36.402Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2003057,"signature":"ET SQL Injection Attempt - Time-Based","category":"","severity":3}}
```

**Alert Counter (Terminal 3):**
```
Alerts in last 100 events: 8
```

**Dashboard (Terminal 4):**
- Alert count timeline shows spike
- Severity pie chart shows more High (red) severity

---

#### Phase 1C: Data Exfiltration Simulation

**Wait 2 seconds, then execute:**

```bash
curl -s -X POST "http://192.168.20.11:8000/submit" \
  -d "data=$(echo 'SELECT * FROM users; SELECT * FROM passwords;' | base64)" > /dev/null
sleep 3
```

**What it does:**
- Sends POST request with encoded "sensitive data"
- Simulates data being exfiltrated after SQL injection succeeds
- `base64` encoding simulates obfuscation

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:45:40.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2025999,"signature":"ET Policy Suspicious Data Transfer","category":"","severity":2}}
```

**Total Time:** ~2 minutes  
**Total Alerts Generated:** 8-12 alerts  
**Alert Types:** Port Scan (2-3), SQL Injection (3-4), Data Exfil (1-2)

---

### Attack 2: SSH Brute Force Campaign

**Machine:** Kali-Attacker (192.168.30.X)  
**Location:** Terminal 1  
**Duration:** ~2 minutes

#### Phase 2A: SSH Service Discovery

**Command:**
```bash
nmap -sS -p 22 -sV 192.168.20.11 -q
```

**What it does:**
- Scans for SSH service on port 22
- `-sV`: Probe for service version
- Attempts to identify SSH daemon version

**Expected Output (on Kali):**
```
Nmap scan report for 192.168.20.11
Host is up (0.0015s latency).

PORT   STATE  SERVICE VERSION
22/tcp closed ssh
```

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:47:01.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2025100,"signature":"ET Policy SSH Version Detection","category":"","severity":2}}
```

---

#### Phase 2B: Brute Force Attempt Simulation

**Wait 2 seconds, then execute:**

```bash
PASSWORDS=("password" "admin" "root" "12345" "P@ssw0rd" "letmein" "welcome")

for pass in "${PASSWORDS[@]}"; do
  ssh -o ConnectTimeout=1 admin@192.168.20.11 "echo 'test'" 2>&1 | head -1
  sleep 1
done
```

**What it does:**
- Attempts SSH login with multiple passwords
- `-o ConnectTimeout=1`: Give up after 1 second
- Loops through 7 different passwords
- `2>&1`: Capture error messages
- `| head -1`: Show only first line of output

**Expected Output (on Kali - all will fail):**
```
ssh: connect to host 192.168.20.11 port 22: Connection refused
ssh: connect to host 192.168.20.11 port 22: Connection refused
...
```

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:47:05.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2025200,"signature":"ET Policy SSH Brute Force Attempt","category":"","severity":2}}
{"timestamp":"2025-11-10T15:47:06.302Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2025200,"signature":"ET Policy Multiple Failed SSH Connections","category":"","severity":2}}
```

---

#### Phase 2C: Command Execution Simulation

**Wait 2 seconds, then execute:**

```bash
curl -s "http://192.168.20.11:8000/cmd?exec=whoami" > /dev/null
curl -s "http://192.168.20.11:8000/cmd?exec=cat%20/etc/passwd" > /dev/null
curl -s "http://192.168.20.11:8000/cmd?exec=id" > /dev/null
curl -s "http://192.168.20.11:8000/cmd?exec=uname%20-a" > /dev/null
sleep 2
```

**What it does:**
- Simulates shell commands executed post-exploitation
- `whoami`: Show current user
- `cat /etc/passwd`: Dump password file
- `id`: Show user identity/groups
- `uname -a`: Show system information
- URL encoding: spaces as `%20`, forward slashes escaped

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:47:12.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2025300,"signature":"ET Policy Remote Command Execution","category":"","severity":3}}
{"timestamp":"2025-11-10T15:47:13.302Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2025301,"signature":"ET Policy Shell Command Detected","category":"","severity":3}}
```

**Total Time:** ~2 minutes  
**Total Alerts Generated:** 10-15 alerts  
**Alert Types:** SSH Brute Force (3-4), Command Injection (3-4)

---

### Attack 3: Web Application Exploit Chain

**Machine:** Kali-Attacker (192.168.30.X)  
**Location:** Terminal 1  
**Duration:** ~2 minutes

#### Phase 3A: Web Server Enumeration

**Command:**
```bash
curl -s -I "http://192.168.20.11:8000/" > /dev/null
curl -s -I "http://192.168.20.11:8000/admin" > /dev/null
curl -s -I "http://192.168.20.11:8000/login" > /dev/null
curl -s -I "http://192.168.20.11:8000/api" > /dev/null
curl -s -I "http://192.168.20.11:8000/.htaccess" > /dev/null
curl -s -I "http://192.168.20.11:8000/.env" > /dev/null
```

**What it does:**
- Sends HTTP HEAD requests to enumerate web resources
- `-I`: HEAD request (retrieve headers only, no body)
- Tests for common admin paths, config files, API endpoints
- Simulates vulnerability scanning tools like Nikto

**Expected Output (on Kali):**
```
HTTP/1.1 200 OK
HTTP/1.1 403 Forbidden
HTTP/1.1 404 Not Found
HTTP/1.1 401 Unauthorized
HTTP/1.1 404 Not Found
HTTP/1.1 403 Forbidden
```

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:48:20.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2027100,"signature":"ET Policy HTTP Methods","category":"","severity":1}}
{"timestamp":"2025-11-10T15:48:21.302Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2027101,"signature":"ET Policy Suspicious File Access","category":"","severity":2}}
```

---

#### Phase 3B: XSS Payload Testing

**Command:**
```bash
XSS_PAYLOADS=(
  "<script>alert('xss')</script>"
  "<img src=x onerror='alert(1)'>"
  "<svg/onload=alert('xss')>"
)

for payload in "${XSS_PAYLOADS[@]}"; do
  curl -s "http://192.168.20.11:8000/search?q=$(echo $payload | sed 's/ /%20/g')" > /dev/null
  sleep 1
done
```

**What it does:**
- Tests multiple XSS (Cross-Site Scripting) payloads
- `sed 's/ /%20/g'`: URL-encode spaces
- XSS payloads attempt to execute JavaScript in browser
- Simulates attacker testing web app input validation

**Expected Output (Suricata Alert - Terminal 2):**
```
{"timestamp":"2025-11-10T15:48:25.201Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2027200,"signature":"ET ATTACK_RESPONSE JavaScript Obfuscation","category":"","severity":2}}
{"timestamp":"2025-11-10T15:48:26.302Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2027201,"signature":"ET ATTACK XSS Attempt","category":"","severity":2}}
{"timestamp":"2025-11-10T15:48:27.403Z","event_type":"alert","alert":{"action":"alert","gid":1,"signature_id":2027202,"signature":"ET Policy Script Injection","category":"","severity":2}}
```

**Total Time:** ~2 minutes  
**Total Alerts Generated:** 10-15 alerts  
**Alert Types:** File Access (2-3), HTTP Methods (1-2), XSS (3-4)

---

## Chapter 7: Verifying Correlation Search Results

### Viewing Multi-Stage Attack Correlations

After running all attacks, return to Splunk and run your correlation search:

**Machine:** Splunk Web GUI  
**Location:** Search & Reporting

**Command (Copy & Paste):**
```splunk
index=main sourcetype=suricata:eve event_type=alert
| eval attack_type=case(
    match(alert.signature, "(?i)port.*scan|network.*scan"), "Port Scan",
    match(alert.signature, "(?i)sql.*injection"), "SQL Injection",
    match(alert.signature, "(?i)xss|cross"), "XSS",
    match(alert.signature, "(?i)command.*injection"), "Command Injection",
    1=1, "Other"
)
| transaction src_ip maxspan=15m
| where mvcount(attack_type) > 1
| stats values(attack_type) as "Attack Types", count as "Alert Count" by src_ip
```

**Expected Output:**

```
src_ip        | Attack Types                              | Alert Count
192.168.30.15 | Port Scan, SQL Injection, Command Inj... | 28
192.168.30.15 | SSH Brute Force, Command Injection, XSS  | 15
192.168.30.15 | Web Enum, XSS Attack                      | 12
```

**What This Shows:**
- 192.168.30.15 is performing multiple distinct attack types
- This is a **multi-stage attack campaign**, not random scanning
- Total of 28+15+12 = 55 related alerts from single attacker
- Each row represents a distinct attack within the 15-minute window

---

## Chapter 8: Verifying Your Automated Alert

### Check if Alert Fired

**Machine:** Splunk Web GUI  
**Location:** Settings > Searches, Reports, and Alerts

**Steps:**
1. Find your alert: "High-Risk Multi-Stage Attack Detected"
2. Click on it to view details
3. Check: "Last triggered" timestamp
4. Expected: Should show time close to when you ran attacks

**Expected Output:**
```
Alert Name:        High-Risk Multi-Stage Attack Detected
Last Triggered:    Mon Nov 10, 2025 3:48 PM
Trigger Count:     3 times
Status:            ✓ Active
Next Scheduled:    Mon Nov 10, 2025 3:53 PM (in 5 minutes)
```

### View Notable Events

**Machine:** Splunk Web GUI  
**Location:** Incidents > Notable Events (or check email)

**Expected Output:**
```
Notable Event List:
- Multi-Stage Attack: 192.168.30.15 - Port Scan, SQL Injection, Command Injection
  Status: Unreviewed | Severity: High | Time: 15:48:27

- Multi-Stage Attack: 192.168.30.15 - SSH Brute Force, Command Injection
  Status: Unreviewed | Severity: High | Time: 15:49:12
```

**Each row represents an alert that was automatically created** by Splunk matching your correlation criteria.

---

## Chapter 9: Dashboard Verification

### Live Dashboard View

**Machine:** Splunk Web GUI  
**Location:** Dashboards > Suricata Alerts Overview

**Expected Panels (After Attacks):**

**Panel 1: Recent Alerts Table**
```
Shows 50+ rows with timestamps from attack period
All have src_ip = 192.168.30.15
Mix of alert.severity = 2 and 3
```

**Panel 2: Alert Count Over Time**
```
Line chart with 3 major spikes:
- Spike 1 (~15:45): SQL Injection phase
- Spike 2 (~15:47): SSH Brute Force phase
- Spike 3 (~15:48): XSS/Web Exploit phase
Each spike: 8-15 alerts per minute during attack
```

**Panel 3: Top Attacker Source IPs**
```
Bar chart (from tallest to shortest):
192.168.30.15:  ███████████████████ 55 alerts (92%)
192.168.20.1:   ██ 4 alerts (7%)
Other:          █ 1 alert (1%)
```

**Panel 4: Top Target Destination IPs**
```
Pie chart:
192.168.20.11 (Splunk):   96% (blue)
192.168.20.1 (pfSense):    3% (orange)
Other:                     1% (gray)
```

**Panel 5: Alert Severity Distribution**
```
Pie chart:
High (3):        35% (red)
Medium (2):      55% (yellow)
Low (1):         10% (green)
```

---

## Chapter 10: SOC Investigation Workflow

### Real-World Incident Response Process

When your alert fires, a SOC analyst follows this workflow:

#### Step 1: Alert Acknowledgment
```
Receive notification (email or console)
Read: "High-Risk Multi-Stage Attack Detected"
Severity: HIGH
Time: 15:48:27 PM
Source IP: 192.168.30.15
```

#### Step 2: Investigation Search

**Machine:** Splunk Web GUI

**Command:**
```splunk
index=main sourcetype=suricata:eve src_ip=192.168.30.15
| stats count as "Alert Count", 
         min(_time) as "First_Alert", 
         max(_time) as "Last_Alert",
         values(alert.signature) as "All_Signatures"
```

**Expected Output:**
```
Alert Count:    55
First_Alert:    2025-11-10 15:45:30 UTC
Last_Alert:     2025-11-10 15:49:15 UTC
All_Signatures: ET Policy Suspicious..., ET SQL Injection..., ET Command Injection..., [24 more]

Interpretation: Single IP, 55 alerts over 4 minutes = coordinated attack
```

#### Step 3: Risk Assessment

From the data, analyst determines:
- **Threat Actor:** Unknown (external IP)
- **Attack Type:** Multi-stage exploitation attempt
- **Target:** Splunk system (192.168.20.11)
- **Techniques:** SQL injection, command execution, web exploitation
- **Success Rate:** Likely unsuccessful (if systems hardened)
- **Risk Level:** HIGH - Multiple attack vectors attempted

#### Step 4: Containment Actions

1. **Block at Firewall**
   - Machine: pfSense (192.168.20.1)
   - Action: Add rule to block 192.168.30.15

2. **Check System Logs**
   - Review Splunk access logs
   - Check for successful exploitations

3. **Document Incident**
   - Ticket: INC-2025-1234
   - Timeline, evidence, actions taken

4. **Improve Defenses**
   - Patch vulnerabilities
   - Update IDS rules

---

## Chapter 11: Key Learnings Summary

### Understanding SIEM Operations

You have successfully completed a professional-grade SIEM exercise:

**Dashboard Creation**
- Built interactive visualizations of security data
- Displayed real-time threat intelligence
- Used multiple visualization types (tables, charts, pie charts)

**Field Extraction & Data Parsing**
- Leveraged Suricata's JSON output for automatic field extraction
- Referenced fields in searches (src_ip, alert.signature, alert.severity)
- Used regex patterns to classify attack types

**Alert Correlation**
- Used `transaction` command to group related events
- Implemented multi-valued field counting (`mvcount()`)
- Detected multi-stage attacks from single sources
- Distinguished attack campaigns from random noise

**Automated Threat Detection**
- Created scheduled searches that run 24/7
- Configured actions (notable events, emails)
- Set appropriate thresholds to reduce false positives

**SOC Operations**
- Understood incident investigation workflow
- Performed root cause analysis
- Documented findings and actions
- Improved defenses based on intelligence

### Critical Splunk Skills Gained

| Skill | Used In | Outcome |
|-------|---------|---------|
| Dashboard design | Panel 1-5 | Real-time security visibility |
| SPL queries | All searches | Data transformation & analysis |
| Regex patterns | `eval attack_type=case()` | Classification & correlation |
| `transaction` command | Correlation search | Attack pattern detection |
| `mvcount()` function | Where clause | Multi-stage attack filtering |
| Alert scheduling | Automated monitoring | 24/7 threat detection |
| Time windowing | `maxspan=15m` | Temporal correlation |
| `stats` & `top` commands | Pattern analysis | Threat actor profiling |

---

## Chapter 12: Complete Command Reference

### All Commands Executed During Day 4

#### Commands for Kali-Attacker (192.168.30.X)

**Attack 1: SQL Injection Campaign**
```bash
# Phase 1A: Database Port Scan
nmap -sS -p 3306,1433,5432,27017 192.168.20.11 -q
sleep 2

# Phase 1B: SQL Injection Payloads
curl -s "http://192.168.20.11:8000/search?username=admin' --&password=anything" > /dev/null
sleep 2
curl -s "http://192.168.20.11:8000/api/users?id=1' UNION SELECT NULL,username,password FROM users --" > /dev/null
sleep 2
curl -s "http://192.168.20.11:8000/search?id=1' AND SLEEP(5)--" > /dev/null
sleep 2

# Phase 1C: Data Exfiltration
curl -s -X POST "http://192.168.20.11:8000/submit" \
  -d "data=$(echo 'SELECT * FROM users; SELECT * FROM passwords;' | base64)" > /dev/null
```

**Attack 2: SSH Brute Force Campaign**
```bash
# Phase 2A: SSH Service Discovery
nmap -sS -p 22 -sV 192.168.20.11 -q
sleep 2

# Phase 2B: Brute Force Attempts
PASSWORDS=("password" "admin" "root" "12345" "P@ssw0rd" "letmein" "welcome")
for pass in "${PASSWORDS[@]}"; do
  ssh -o ConnectTimeout=1 admin@192.168.20.11 "echo 'test'" 2>&1 || true
  sleep 1
done

# Phase 2C: Command Execution
curl -s "http://192.168.20.11:8000/cmd?exec=whoami" > /dev/null
curl -s "http://192.168.20.11:8000/cmd?exec=cat%20/etc/passwd" > /dev/null
curl -s "http://192.168.20.11:8000/cmd?exec=id" > /dev/null
curl -s "http://192.168.20.11:8000/cmd?exec=uname%20-a" > /dev/null
```

**Attack 3: Web Application Exploitation**
```bash
# Phase 3A: Web Enumeration
curl -s -I "http://192.168.20.11:8000/" > /dev/null
curl -s -I "http://192.168.20.11:8000/admin" > /dev/null
curl -s -I "http://192.168.20.11:8000/login" > /dev/null
curl -s -I "http://192.168.20.11:8000/.htaccess" > /dev/null
curl -s -I "http://192.168.20.11:8000/.env" > /dev/null

# Phase 3B: XSS Payloads
curl -s "http://192.168.20.11:8000/search?q=<script>alert('xss')</script>" > /dev/null
curl -s "http://192.168.20.11:8000/search?q=<img%20src=x%20onerror='alert(1)'>" > /dev/null
curl -s "http://192.168.20.11:8000/search?q=<svg/onload=alert('xss')>" > /dev/null
```

#### Commands for Ubuntu SIEM (192.168.20.11)

**Live Monitoring (Run in Background)**
```bash
# Monitor Suricata alerts in real-time
sudo tail -f /var/log/suricata/eve.json | grep alert

# Count recent alerts (update every 5 seconds)
watch -n 5 'sudo tail -100 /var/log/suricata/eve.json | grep alert | wc -l'

# Monitor Splunk indexing
sudo tail -f /var/log/splunk/splunkd.log | grep suricata
```

#### Splunk Queries (Splunk Web GUI)

**Dashboard Panel 1: Recent Alerts**
```splunk
index=main sourcetype=suricata:eve event_type=alert
| table _time, src_ip, dest_ip, alert.signature, alert.severity
| sort - _time | head 50
```

**Dashboard Panel 2: Alert Count Over Time**
```splunk
index=main sourcetype=suricata:eve event_type=alert
| timechart count by alert.signature limit=10
```

**Dashboard Panel 3: Top Attackers**
```splunk
index=main sourcetype=suricata:eve event_type=alert
| top src_ip limit=10
```

**Dashboard Panel 4: Top Targets**
```splunk
index=main sourcetype=suricata:eve event_type=alert
| top dest_ip limit=10
```

**Dashboard Panel 5: Severity Distribution**
```splunk
index=main sourcetype=suricata:eve event_type=alert
| stats count by alert.severity
```

**Correlation Search**
```splunk
index=main sourcetype=suricata:eve event_type=alert
| eval attack_type=case(
    match(alert.signature, "(?i)port.*scan|network.*scan"), "Port Scan",
    match(alert.signature, "(?i)sql.*injection"), "SQL Injection",
    match(alert.signature, "(?i)xss|cross"), "XSS",
    match(alert.signature, "(?i)command.*injection"), "Command Injection",
    1=1, "Other"
)
| transaction src_ip maxspan=15m
| where mvcount(attack_type) > 1
| table _time, src_ip, attack_type, alert.signature, alert.severity
| sort - _time
```

**Alert Detection Query**
```splunk
index=main sourcetype=suricata:eve event_type=alert alert.severity>=2
| eval attack_type=case(
    match(alert.signature, "(?i)port.*scan"), "Port Scan",
    match(alert.signature, "(?i)sql.*injection"), "SQL Injection",
    match(alert.signature, "(?i)xss|cross"), "XSS",
    match(alert.signature, "(?i)command.*injection"), "Command Injection",
    1=1, "Other"
)
| transaction src_ip maxspan=15m
| where mvcount(attack_type) > 2
```

**Investigation Deep-Dive**
```splunk
index=main sourcetype=suricata:eve src_ip=192.168.30.15
| stats count, min(_time), max(_time), values(alert.signature) by src_ip
```

---

## Chapter 13: Expected Artifacts & Documentation

### Files Created During Day 4

**Splunk Dashboards**
- ✓ Suricata Alerts Overview (5 panels)
- ✓ Dashboard saved in Splunk (accessible from Dashboards menu)

**Saved Searches**
- ✓ Multi-Stage Attack Detection (correlation search)
- ✓ Saved in Settings > Searches, Reports, and Alerts

**Automated Alerts**
- ✓ High-Risk Multi-Stage Attack Detected (scheduled alert)
- ✓ Configured to fire every 5 minutes
- ✓ Actions: Create Notable Event + Send Email

**Data Generated**
- ✓ 55+ Suricata alerts from attack simulations
- ✓ 3+ Notable Events created by automation
- ✓ Email notifications sent (if configured)

---

## Conclusion

**Day 4 Accomplishments:**

You have successfully completed advanced SIEM operations training including:

✅ **Dashboard Design** - Created 5-panel security overview  
✅ **Data Visualization** - Used tables, charts, pie charts for insights  
✅ **Alert Correlation** - Implemented multi-event detection logic  
✅ **Automation** - Set up 24/7 threat monitoring  
✅ **Attack Simulation** - Generated realistic threat data  
✅ **Investigation** - Performed SOC-style incident response  
✅ **Documentation** - Captured commands and outputs  

**You now understand:**
- How professional SOCs use SIEM dashboards
- How to correlate security events into actionable intelligence
- How to automate threat detection at scale
- How to respond to security incidents efficiently
