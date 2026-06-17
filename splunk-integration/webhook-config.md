# Splunk Webhook Integration

## Overview
Connects the Splunk SSH brute force alert to the Shuffle webhook so detections trigger the SOAR workflow automatically.

## Alert Search

```spl
index=main host="raspberrypi" "Failed password"
| rex field=_raw "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
```

The `rex` command extracts the source IP into a named `src_ip` field, which Splunk includes in the webhook payload under the `result` object.

## Alert Configuration

| Setting | Value |
|---------|-------|
| Alert Type | Real-time |
| Trigger Condition | Number of Results > 5 in 1 minute |
| Trigger Action | Webhook |
| Webhook URL | Shuffle webhook URI |

## Payload Structure

Splunk wraps the detection data in a `result` object:

```json
{
  "search_name": "SSH Brute Force Attack Detected",
  "result": {
    "src_ip": "192.168.x.x",
    "_raw": "...Failed password for root from 192.168.x.x..."
  }
}
```

This is why Shuffle references the IP as `$exec.result.src_ip` rather than `$exec.src_ip` — the data is nested one level under `result`.

## Key Lesson

When integrating SIEM to SOAR, the payload structure matters. Splunk's webhook nests event data under `result`, so variable paths in the SOAR platform must account for that nesting. Testing with a `curl` command that mirrors the exact payload structure is the fastest way to validate variable paths before relying on live alerts.
