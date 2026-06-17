# Setup Guide — Shuffle SOAR Deployment

## Prerequisites
- Ubuntu 24.04 VM 
- Docker and Docker Compose v2
- Splunk Enterprise (existing SIEM)
- AbuseIPDB free API key
- VirusTotal free API key
- Gmail account with App Password

## 1. Install Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io git -y
sudo apt install docker-compose-v2 -y
sudo systemctl start docker
sudo systemctl enable docker
```

> Note: Use `docker compose` (v2 plugin syntax), not the legacy `docker-compose`.

## 2. Deploy Shuffle

```bash
git clone https://github.com/Shuffle/Shuffle
cd Shuffle
sudo mkdir -p shuffle-database
sudo chown -R 1000:1000 shuffle-database
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo docker compose up -d
```

Verify containers:
```bash
sudo docker ps
```

## 3. Access Shuffle

https://[Ubuntu-VM-IP]:3443

Create an admin account on first login.

## 4. Build the Workflow

1. Create a new workflow
2. Add a Webhook trigger and copy its URI
3. Add Shuffle Tools, AbuseIPDB, VirusTotal, and Email nodes
4. Connect them in sequence
5. Configure each node's API authentication and IP field
6. Add the condition on the Email connection

## 5. API Keys

- **AbuseIPDB:** account → API → generate key
- **VirusTotal:** profile → API key

> Security: Treat API keys as secrets. If a key is ever exposed (in a screenshot), regenerate it immediately.

## 6. Gmail SMTP

1. Enable 2-Step Verification on the Google account
2. Generate an App Password (16 characters)
3. Use this password in the Email node, not the account password

## 7. Test

```bash
curl -k -X POST [WEBHOOK-URI] -H "Content-Type: application/json" \
-d '{"result": {"src_ip": "185.220.101.1", "count": "7012"}}'
```

A public IP returns real reputation data, confirming the enrichment works.
