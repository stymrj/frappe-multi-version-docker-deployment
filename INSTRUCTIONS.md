# Deployment Instructions

This file contains the complete step-by-step instructions to deploy and manage the Frappe Multi-Version Docker Deployment on an AWS EC2 Ubuntu server.

---

## 1. Launch AWS EC2 Instance

Recommended configuration:

```txt
AMI: Ubuntu Server 22.04 LTS or Ubuntu Server 24.04 LTS
Instance type: t3.large or higher
Storage: 30 GB minimum
```

Recommended for smooth setup:

```txt
2 vCPU
8 GB RAM
30 GB storage
```

---

## 2. Configure AWS Security Group

Open the following inbound ports:

| Port | Purpose |
|---:|---|
| 22 | SSH |
| 9443 | Portainer |
| 8014 | Frappe Framework v14 |
| 8015 | Frappe Framework v15 |

For testing:

```txt
Source: 0.0.0.0/0
```

For better security:

```txt
Source: Your IP address only
```

---

## 3. SSH Into EC2

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

---

## 4. Update Server

```bash
sudo apt update && sudo apt upgrade -y
```

Install basic utilities:

```bash
sudo apt install -y curl git nano ca-certificates
```

---

## 5. Install Docker and Docker Compose v2

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker ubuntu
```

Logout:

```bash
exit
```

SSH again:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Check installation:

```bash
docker --version
docker compose version
```

Important:

```txt
Use docker compose
Do not use old docker-compose
```

---

## 6. Install Portainer

Create Portainer volume:

```bash
docker volume create portainer_data
```

Run Portainer:

```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:lts
```

Open in browser:

```txt
https://YOUR_EC2_PUBLIC_IP:9443
```

First-time setup:

1. Create admin username and password.
2. Select local Docker environment.
3. Open the Portainer dashboard.

---

## 7. Clone Project Repository

```bash
git clone https://github.com/stymrj/frappe-multi-version-docker-deployment.git
cd frappe-multi-version-docker-deployment
```

---

## 8. Build and Start Deployment

```bash
docker compose up -d --build
```

This command will:

- Build the custom Frappe Docker image
- Start MariaDB for Frappe v14
- Start Redis for Frappe v14
- Start Frappe v14 container
- Start MariaDB for Frappe v15
- Start Redis for Frappe v15
- Start Frappe v15 container
- Create required Docker volumes
- Create required Docker networks

---

## 9. Check Running Containers

```bash
docker ps
```

Expected containers:

```txt
frappe-v14
frappe-v15
mariadb14
mariadb15
redis14
redis15
portainer
```

---

## 10. View Logs

Frappe v14 logs:

```bash
docker logs -f frappe-v14
```

Frappe v15 logs:

```bash
docker logs -f frappe-v15
```

The first run may take time because dependencies and sites are being initialized.

---

## 11. Access Frappe Applications

Replace `YOUR_EC2_PUBLIC_IP` with your actual EC2 public IP.

Frappe v14:

```txt
http://184.72.75.158/:8014
```

Frappe v15:

```txt
http://184.72.75.158/:8015
```

Default login:

```txt
Username: Administrator
Password: admin
```

---

## 12. Access Portainer

```txt
https://184.72.75.158/:9443
```

Use Portainer to:

- View containers
- Start containers
- Stop containers
- Restart containers
- View logs
- Inspect volumes
- Inspect networks

---

## 13. Start Services

```bash
docker compose start
```

---

## 14. Stop Services

```bash
docker compose stop
```

---

## 15. Restart Services

```bash
docker compose restart
```

---

## 16. Stop and Remove Containers

```bash
docker compose down
```

This removes containers and networks but keeps volumes.

---
