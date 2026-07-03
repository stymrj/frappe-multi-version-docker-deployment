# Frappe Multi-Version Docker Deployment

This repository contains a Docker-based deployment setup for running two different versions of the Frappe Framework simultaneously on the same AWS EC2 Ubuntu server.

The deployment includes:

- Frappe Framework v14
- Frappe Framework v15
- Separate MariaDB databases
- Separate Redis services
- Separate Docker volumes
- Separate Docker networks
- Portainer for Docker container management

---

## Assignment Objective

The objective of this DevOps assignment is to create a Docker-based deployment setup containing:

- One container running Frappe Framework v14
- One container running Frappe Framework v15
- Both Frappe versions running simultaneously without conflicts
- Required services such as MariaDB and Redis
- Separate networking and persistent volumes
- Portainer-based container management
- Documentation for deployment, start, stop, restart, and management steps

---

## Architecture

```txt
AWS EC2 Ubuntu Server
│
├── Docker
├── Docker Compose
├── Portainer
│
├── Frappe v14 Environment
│   ├── App Container: frappe-v14
│   ├── Database Container: mariadb14
│   ├── Redis Container: redis14
│   ├── Exposed Port: 8014
│   ├── Volume: frappe14_bench
│   └── Network: frappe14_net
│
└── Frappe v15 Environment
    ├── App Container: frappe-v15
    ├── Database Container: mariadb15
    ├── Redis Container: redis15
    ├── Exposed Port: 8015
    ├── Volume: frappe15_bench
    └── Network: frappe15_net
```

---

## Tech Stack

- AWS EC2 Ubuntu
- Docker
- Docker Compose v2
- Portainer
- Frappe Framework v14
- Frappe Framework v15
- MariaDB 10.6
- Redis
- Custom Docker image based on `frappe/bench`

---

## Repository Structure

```txt
frappe-multi-version-docker-deployment/
│
├── Dockerfile.frappe
├── docker-compose.yml
├── README.md
├── INSTRUCTIONS.md
├── .gitignore
└── screenshots/
```

---

## Services

| Service | Description |
|---|---|
| `frappe-v14` | Frappe Framework v14 application container |
| `frappe-v15` | Frappe Framework v15 application container |
| `mariadb14` | MariaDB database for Frappe v14 |
| `mariadb15` | MariaDB database for Frappe v15 |
| `redis14` | Redis service for Frappe v14 |
| `redis15` | Redis service for Frappe v15 |
| `portainer` | Docker container management UI |

---

## Ports Used

| Service | Port |
|---|---:|
| Portainer | 9443 |
| Frappe v14 | 8014 |
| Frappe v15 | 8015 |
| MariaDB v14 | Internal 3306 |
| MariaDB v15 | Internal 3306 |
| Redis v14 | Internal 6379 |
| Redis v15 | Internal 6379 |

---

## Conflict Avoidance

Both Frappe versions are isolated using separate Docker resources.

| Component | Frappe v14 | Frappe v15 |
|---|---|---|
| App Container | `frappe-v14` | `frappe-v15` |
| Exposed Port | `8014` | `8015` |
| Database | `mariadb14` | `mariadb15` |
| Redis | `redis14` | `redis15` |
| Volume | `frappe14_bench` | `frappe15_bench` |
| Network | `frappe14_net` | `frappe15_net` |

This prevents:

- Port conflicts
- Database conflicts
- Redis conflicts
- Volume/data conflicts
- Network conflicts

---

## Prerequisites

Install Docker and Docker Compose v2 on Ubuntu:

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker ubuntu
```

Logout and login again:

```bash
exit
```

Then SSH again into the server.

Check installation:

```bash
docker --version
docker compose version
```

> Important: Use `docker compose`, not old `docker-compose`.

---

## Portainer Installation

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

Open Portainer:

```txt
https://YOUR_EC2_PUBLIC_IP:9443
```

---

## AWS Security Group Rules

Open the following inbound ports in the EC2 security group:

| Port | Purpose |
|---:|---|
| 22 | SSH |
| 9443 | Portainer |
| 8014 | Frappe v14 |
| 8015 | Frappe v15 |

For testing, the source can be:

```txt
0.0.0.0/0
```

For better security, restrict the source to your own IP address.

---

## Deployment Steps

Clone the repository:

```bash
git clone https://github.com/stymrj/frappe-multi-version-docker-deployment.git
cd frappe-multi-version-docker-deployment
```

Build and start containers:

```bash
docker compose up -d --build
```

Check running containers:

```bash
docker ps
```

View logs:

```bash
docker logs -f frappe-v14
docker logs -f frappe-v15
```

The first build may take time because Frappe dependencies, Python packages, Node dependencies, and site setup are initialized.

---

## Access URLs

Replace `http://184.72.75.158/` with your actual EC2 public IP.

```txt
Frappe v14:
http://184.72.75.158/:8014

Frappe v15:
http://184.72.75.158/:8015

Portainer:
http://184.72.75.158/:9443
```

Default Frappe login:

```txt
Username: Administrator
Password: admin
```

---

## Start, Stop, Restart

Start all services:

```bash
docker compose start
```

Stop all services:

```bash
docker compose stop
```

Restart all services:

```bash
docker compose restart
```

Stop and remove containers:

```bash
docker compose down
```


---

## Managing Through Portainer

Portainer is used to manage the complete Docker environment.

Steps:

1. Open Portainer:

```txt
https://184.72.75.158/:9443
```

2. Select the local Docker environment.
3. Go to **Containers**.
4. Manage the following containers:
   - `frappe-v14`
   - `frappe-v15`
   - `mariadb14`
   - `mariadb15`
   - `redis14`
   - `redis15`
5. From Portainer, containers can be:
   - Started
   - Stopped
   - Restarted
   - Inspected
   - Monitored through logs

---

## Useful Commands

Check running containers:

```bash
docker ps
```

Check all containers:

```bash
docker ps -a
```

Check logs:

```bash
docker logs -f frappe-v14
docker logs -f frappe-v15
```

Enter Frappe v14 container:

```bash
docker exec -it frappe-v14 bash
```

Enter Frappe v15 container:

```bash
docker exec -it frappe-v15 bash
```

Check Frappe version inside container:

```bash
bench version
```

Build assets:

```bash
bench build
```

Clear cache:

```bash
bench --site YOUR_SITE_NAME clear-cache
```

---


### 1. Container is restarting

Check logs:

```bash
docker logs --tail=150 frappe-v14
docker logs --tail=150 frappe-v15
```

### 2. Browser shows connection refused

Check if containers are running:

```bash
docker ps
```

Check local access from EC2:

```bash
curl -I http://localhost:8014
curl -I http://localhost:8015
```

If local access works but browser does not open, check AWS security group inbound rules.

### 3. Assets error or CSS bundle error

Run:

```bash
docker exec -it frappe-v14 bash -lc "cd /workspace/frappe14-bench && bench build && bench --site YOUR_SITE_NAME clear-cache"
docker exec -it frappe-v15 bash -lc "cd /workspace/frappe15-bench && bench build && bench --site YOUR_SITE_NAME clear-cache"
docker restart frappe-v14 frappe-v15
```

### 4. Full reset

Use only when fresh setup is required:

```bash
docker compose down -v --remove-orphans
docker compose up -d --build
```


## Submission

This repository contains:

- Docker Compose configuration
- Custom Frappe Docker image
- Frappe Framework v14 container
- Frappe Framework v15 container
- Separate MariaDB services
- Separate Redis services
- Separate Docker networks
- Separate Docker volumes
- Portainer setup instructions
- Deployment and management documentation

---

## Author

**Satyam Raj**
