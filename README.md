# Frappe Multi-Version Docker Deployment

Docker-based deployment for running **Frappe Framework v14** and **Frappe Framework v15** together on a single AWS EC2 Ubuntu server.

Both versions run independently with separate database, Redis, volumes, networks, and ports. The containers are managed using **Portainer**.

---

## Live Demo

| Service | URL |
|---|---|
| Frappe v14 | http://184.72.75.158:8014 |
| Frappe v15 | http://184.72.75.158:8015 |
| Portainer | https://184.72.75.158:9443 |

Default Frappe login:

```txt
Username: Administrator
Password: admin
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

## Architecture

```txt
AWS EC2 Ubuntu Server
│
├── Docker
├── Docker Compose
├── Portainer
│
├── Portainer
│   ├── Container: portainer
│   ├── Port: 9443
│   ├── Data Volume: portainer_data
│   └── Docker Socket: /var/run/docker.sock
│
├── Frappe v14
│   ├── Container: frappe-v14
│   ├── Port: 8014
│   ├── Database: mariadb14
│   ├── Redis: redis14
│   ├── Volume: frappe14_bench
│   └── Network: frappe14_net

└── Frappe v15
    ├── Container: frappe-v15
    ├── Port: 8015
    ├── Database: mariadb15
    ├── Redis: redis15
    ├── Volume: frappe15_bench
    └── Network: frappe15_net
```

---

## Project Structure

```txt
frappe-multi-version-docker-deployment/
├── Dockerfile.frappe
├── docker-compose.yml
└── README.md
```

---

## Services and Ports

| Service | Purpose | Port |
|---|---|---:|
| `frappe-v14` | Frappe v14 app | 8014 |
| `frappe-v15` | Frappe v15 app | 8015 |
| `mariadb14` | Database for v14 | Internal 3306 |
| `mariadb15` | Database for v15 | Internal 3306 |
| `redis14` | Redis for v14 | Internal 6379 |
| `redis15` | Redis for v15 | Internal 6379 |
| `portainer` | Docker management UI | 9443 |

Portainer also listens on `8000` for edge agent and tunnel traffic when needed.

---

## Isolation Between Versions

| Component | Frappe v14 | Frappe v15 |
|---|---|---|
| App Container | `frappe-v14` | `frappe-v15` |
| Port | `8014` | `8015` |
| Database | `mariadb14` | `mariadb15` |
| Redis | `redis14` | `redis15` |
| Volume | `frappe14_bench` | `frappe15_bench` |
| Network | `frappe14_net` | `frappe15_net` |

This avoids conflicts between ports, databases, Redis services, volumes, and networks.

## Deployment Steps

1. Install Docker and Docker Compose v2 on the Ubuntu host.
2. Clone this repository onto the server.
3. Start the full stack:

```bash
docker compose up -d --build
```

4. Open Portainer at:

```txt
https://184.72.75.158:9443
```

5. Use Portainer to inspect, start, stop, restart, and remove the containers for Frappe v14 and Frappe v15.
6. If the site is being deployed on a different host, update the site host URLs in `docker-compose.yml` before bringing the stack up.

---

## Setup

Install Docker and Docker Compose v2:

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker ubuntu
```

Logout and SSH again:

```bash
exit
```

Verify installation:

```bash
docker --version
docker compose version
```

---

## Run Project

Clone repository:

```bash
git clone https://github.com/stymrj/frappe-multi-version-docker-deployment.git
cd frappe-multi-version-docker-deployment
```

Build and start containers:

```bash
docker compose up -d --build
```

Check containers:

```bash
docker ps
```

View logs:

```bash
docker logs -f frappe-v14
docker logs -f frappe-v15
docker logs -f portainer
```

---

## Portainer

Portainer is deployed as part of the same compose stack and uses the Docker socket to manage the host.

From Portainer, these containers can be started, stopped, restarted, inspected, and monitored:

- `frappe-v14`
- `frappe-v15`
- `mariadb14`
- `mariadb15`
- `redis14`
- `redis15`

---

## AWS Security Group

Required inbound ports:

| Port | Purpose |
|---:|---|
| 22 | SSH |
| 9443 | Portainer |
| 8014 | Frappe v14 |
| 8015 | Frappe v15 |

---

## Common Commands

Start services:

```bash
docker compose start
```

Stop services:

```bash
docker compose stop
```

Restart services:

```bash
docker compose restart
```

Stop and remove containers:

```bash
docker compose down
```

Stop and remove containers plus named volumes:

```bash
docker compose down -v
```

Full reset with volumes:

```bash
docker compose down -v --remove-orphans
docker compose up -d --build
```

Enter containers:

```bash
docker exec -it frappe-v14 bash
docker exec -it frappe-v15 bash
```

Build assets if UI/CSS does not load:

```bash
docker exec -it frappe-v14 bash -lc "cd /workspace/frappe14-bench && bench build"
docker exec -it frappe-v15 bash -lc "cd /workspace/frappe15-bench && bench build"
docker restart frappe-v14 frappe-v15
```

---

## Screenshots

Uploaded deployment evidence:

- [Screenshot 1](screenshots/Screenshot%202026-07-03%20at%203.13.30%E2%80%AFPM.png)
- [Screenshot 2](screenshots/Screenshot%202026-07-03%20at%203.14.24%E2%80%AFPM.png)
- [Screenshot 3](screenshots/Screenshot%202026-07-03%20at%203.23.31%E2%80%AFPM.png)
- [Screenshot 4](screenshots/Screenshot%202026-07-03%20at%203.24.58%E2%80%AFPM.png)
- [Screenshot 5](screenshots/Screenshot%202026-07-03%20at%203.27.18%E2%80%AFPM.png)
- [Screenshot 6](screenshots/Screenshot%202026-07-03%20at%203.27.44%E2%80%AFPM.png)

If you want these embedded inline instead of linked, I can add a gallery layout as well.

---

## Verification Checklist

- Frappe v14 runs on port `8014`
- Frappe v15 runs on port `8015`
- Both versions run at the same time
- Separate MariaDB containers are configured
- Separate Redis containers are configured
- Separate Docker volumes are configured
- Separate Docker networks are configured
- Portainer is used for container management

---

## Author

**Satyam Raj**
