---
layout: post
title: "Homelab Basics: Getting Started with Docker Compose"
categories: [homelab, infrastructure]
tags: [docker, docker-compose, homelab, self-hosting]
---

*This is a draft post template. Move to _posts/ with a date when ready to publish.*

Starting a homelab can be overwhelming. In this post, I'll share a practical approach to self-hosting services using Docker Compose—the foundation of my homelab infrastructure.

## Why Docker Compose?

**Benefits for homelabs:**
- Single configuration file per service
- Easy to version control
- Simple backup and migration
- Isolated environments
- Low resource overhead

## Prerequisites

- Linux server (Ubuntu/Debian recommended)
- Basic command line knowledge
- Domain name (optional but recommended)

## Installation

### Install Docker

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose-plugin
```

Verify installation:
```bash
docker --version
docker compose version
```

## Your First Service: Nginx

Create a directory structure:
```bash
mkdir -p ~/homelab/nginx
cd ~/homelab/nginx
```

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./config:/etc/nginx/conf.d
      - ./html:/usr/share/nginx/html
      - ./certs:/etc/nginx/certs
    networks:
      - homelab

networks:
  homelab:
    driver: bridge
```

Start the service:
```bash
docker compose up -d
```

Check status:
```bash
docker compose ps
docker compose logs -f
```

## Essential Homelab Services

### 1. Portainer (Container Management UI)

```yaml
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

volumes:
  portainer_data:
```

Access: `http://your-server-ip:9000`

### 2. Pi-hole (Network-wide Ad Blocking)

```yaml
version: '3.8'

services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8080:80"
    environment:
      TZ: 'Europe/Warsaw'
      WEBPASSWORD: 'ChangeMe123!'
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d

volumes:
  etc-pihole:
  etc-dnsmasq.d:
```

### 3. Uptime Kuma (Monitoring)

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - uptime-kuma:/app/data

volumes:
  uptime-kuma:
```

## Best Practices

### 1. Directory Structure

```
~/homelab/
├── nginx/
│   ├── docker-compose.yml
│   ├── config/
│   └── html/
├── portainer/
│   └── docker-compose.yml
├── pihole/
│   └── docker-compose.yml
└── scripts/
    ├── backup.sh
    └── update.sh
```

### 2. Backup Script

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/mnt/backups/homelab"
DATE=$(date +%Y%m%d_%H%M%S)

cd ~/homelab

# Backup each service
for service in */; do
    echo "Backing up ${service}..."
    tar -czf "${BACKUP_DIR}/${service%/}_${DATE}.tar.gz" "$service"
done

# Keep only last 7 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete
```

### 3. Update Script

```bash
#!/bin/bash
# update.sh

cd ~/homelab

for service in */; do
    echo "Updating ${service}..."
    cd "$service"
    docker compose pull
    docker compose up -d
    cd ..
done

# Clean up old images
docker image prune -af
```

### 4. Environment Variables

Create `.env` files for sensitive data:

```bash
# .env
MYSQL_ROOT_PASSWORD=secure_password_here
PIHOLE_PASSWORD=another_secure_password
```

Reference in `docker-compose.yml`:
```yaml
environment:
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

**Important**: Add `.env` to `.gitignore`!

## Monitoring and Maintenance

### Check Resource Usage

```bash
docker stats
```

### View Logs

```bash
docker compose logs -f service_name
```

### Restart Services

```bash
docker compose restart
```

### Update and Rebuild

```bash
docker compose pull
docker compose up -d
```

## Networking Tips

### Reverse Proxy Setup

Use Nginx Proxy Manager for easy SSL:

```yaml
version: '3.8'

services:
  nginx-proxy-manager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
      - '81:81'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

Access admin UI: `http://your-server-ip:81`

Default credentials:
- Email: `admin@example.com`
- Password: `changeme`

### Custom Network

```yaml
networks:
  homelab:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

## Common Issues

**Problem**: Container won't start
**Solution**: Check logs: `docker compose logs service_name`

**Problem**: Port already in use
**Solution**: Change external port in compose file: `"8081:80"`

**Problem**: Permission errors
**Solution**: Check volume ownership: `sudo chown -R 1000:1000 ./data`

## Next Steps

- Set up automatic backups
- Implement monitoring with Grafana
- Configure VPN access (WireGuard)
- Add Plex or Jellyfin media server
- Set up Home Assistant

## Resources

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Awesome Selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)
- My homelab setup: [Coming soon to GitHub]

## Conclusion

Docker Compose makes homelab management accessible and maintainable. Start with essential services, document your setup, and expand gradually.

The key is treating your homelab as code—version controlled, reproducible, and well-documented.

---

*Questions about homelabs or self-hosting? Connect on [LinkedIn](https://linkedin.com/in/marcinwernecki) or [GitHub](https://github.com/ch0nkster).*
