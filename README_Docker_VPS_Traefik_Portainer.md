# 🐳 Dockerized VPS Hosting with Traefik and Portainer

## What Can You Do with a VPS, a Free Day, and an Interest in Docker?

You’ve got a VPS, some spare time, and you’re curious about Docker. Why not build your own modern hosting setup with Docker containers, a Traefik reverse proxy, and Portainer for easy management?

This guide walks you through setting up a containerized hosting environment using:
- **Docker** to run your applications
- **Traefik** for reverse proxy and automatic HTTPS
- **Portainer** for managing your containers through a web UI

---

## 🔧 Prerequisites

- VPS with Ubuntu 20.04/22.04
- Root or sudo access
- Docker & Docker Compose installed
- A domain name (for Traefik + HTTPS)

---

## 🧱 Directory Structure

```
~/docker
├── traefik
│   ├── traefik.yml
│   ├── dynamic.yml
│   └── acme.json
├── portainer
│   └── docker-compose.yml
├── whoami
│   └── docker-compose.yml
```

---

## ⚙️ Setup Steps

### 1. Install Docker & Docker Compose

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose -y
sudo usermod -aG docker $USER
newgrp docker
```

---

### 2. Configure Traefik

**traefik/traefik.yml**

```yaml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

api:
  dashboard: true

providers:
  docker:
    exposedByDefault: false

certificatesResolvers:
  letsencrypt:
    acme:
      email: your-email@example.com
      storage: acme.json
      httpChallenge:
        entryPoint: web
```

**traefik/dynamic.yml**

```yaml
http:
  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
```

**Create and secure the ACME file:**

```bash
touch acme.json
chmod 600 acme.json
```

**traefik/docker-compose.yml**

```yaml
version: "3.8"

services:
  traefik:
    image: traefik:v3.0
    command:
      - --configFile=/etc/traefik/traefik.yml
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.yml:/etc/traefik/traefik.yml
      - ./dynamic.yml:/etc/traefik/dynamic.yml
      - ./acme.json:/acme.json
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`traefik.yourdomain.com`)"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
```

---

### 3. Add a Sample App (Whoami)

**whoami/docker-compose.yml**

```yaml
version: "3.8"

services:
  whoami:
    image: traefik/whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.yourdomain.com`)"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls.certresolver=letsencrypt"
```

---

### 4. Set Up Portainer

**portainer/docker-compose.yml**

```yaml
version: "3.8"

services:
  portainer:
    image: portainer/portainer-ce
    command: -H unix:///var/run/docker.sock
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    ports:
      - "9000:9000"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.yourdomain.com`)"
      - "traefik.http.routers.portainer.entrypoints=websecure"
      - "traefik.http.routers.portainer.tls.certresolver=letsencrypt"

volumes:
  portainer_data:
```

---

## 🚀 Launch Everything

From each service directory:

```bash
docker compose up -d
```

---

## 🌍 Visit Your Apps

- `https://traefik.yourdomain.com` – Traefik Dashboard  
- `https://portainer.yourdomain.com` – Portainer  
- `https://whoami.yourdomain.com` – Demo app

---

## ✅ Done!

You now have a lightweight, production-ready Docker hosting setup with:
- SSL via Let’s Encrypt
- Auto-routing with Traefik
- GUI control via Portainer

Enjoy your new self-hosted infrastructure!