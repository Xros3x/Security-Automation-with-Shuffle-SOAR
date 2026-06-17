# Workflow Logic — SSH Brute Force Auto-Response

## Node Chain
Webhook → IP Extraction → AbuseIPDB → VirusTotal → [Condition] → Email

## Node Breakdown

### 1. Webhook Trigger
Receives the alert payload POSTed by Splunk when the SSH brute force alert fires. Activated and assigned a unique webhook URI registered in the Splunk alert action.

### 2. IP Extraction (Shuffle Tools)
Parses the incoming payload and references the attacker source IP using the variable path:
$exec.result.src_ip

### 3. AbuseIPDB Enrichment
Looks up the source IP reputation.
- **Action:** Check IP
- **IP field:** `$exec.result.src_ip`
- **Returns:** abuseConfidenceScore, countryCode, totalReports, usageType, isPublic

### 4. VirusTotal Enrichment
Secondary reputation cross-reference.
- **Action:** Get IP address report
- **IP field:** `$exec.result.src_ip`
- **Returns:** IP analysis stats from VirusTotal's threat database

### 5. Conditional Logic
Gates the email notification on confirmed attack detection.
- **Source:** `$exec.result.src_ip`
- **Condition:** contains `.`
- **Result:** Email fires when a valid IP was extracted from a detected attack

### 6. Email Notification
Sends the enriched alert via Gmail SMTP.
- **SMTP:** smtp.gmail.com:587
- **Auth:** Gmail App Password
- **Body:** dynamically populated with attack details and threat intelligence

## Email Body Template
SECURITY ALERT - Automated SOC Response
SSH brute force attack detected and automatically enriched with threat intelligence.
ATTACK DETAILS:

Source IP: $exec.result.src_ip
Failed Attempts: $exec.result.count

THREAT INTELLIGENCE (AbuseIPDB):

Abuse Confidence Score: $abuseipdb_v2_1.body.data.abuseConfidenceScore/100
Country: $abuseipdb_v2_1.body.data.countryCode
Total Reports: $abuseipdb_v2_1.body.data.totalReports
Usage Type: $abuseipdb_v2_1.body.data.usageType

RECOMMENDED ACTION: Review source IP and block if confirmed malicious.
Generated automatically by Shuffle SOAR.

## Design Notes

- AbuseIPDB and VirusTotal run in sequence so both enrichments are available to the email node.
- Private/internal IPs return a score of 0 (no global reputation), while known-malicious public IPs return high scores — validating the logic in both directions.
- The condition uses IP presence rather than reputation score so the SOC is alerted on every detected attack, with reputation data included as context.
