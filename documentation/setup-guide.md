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
