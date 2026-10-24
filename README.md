# noyoutipme.com

A satirical tipping interface hosted as a static website.

## Deployment Architecture

```
Internet
    ↓
Cloudflare Tunnel (cloudflared)
    ↓
localhost:8080
    ↓
Docker Container
    ↓
Nginx (port 80 inside container)
    ↓
index.html
```

### Components

1. **Static HTML/CSS/JS** - Single page application (`index.html`)
2. **Nginx** - Web server serving static files (inside Docker)
3. **Docker** - Containerization platform
4. **Cloudflare Tunnel** - Secure proxy to expose local server to internet
   - Tunnel ID: `b8c566a4-cf3c-4186-b397-7993c5872e7d`
   - Domains: `noyoutipme.com` and `www.noyoutipme.com`
   - Local service: `http://localhost:8080`

## Quick Start

### Making Changes and Redeploying

After editing `index.html`, redeploy with:

```bash
cd /home/thickarus/noyoutipme.com
./deploy.sh
```

The script will:
- Stop the old container
- Remove the old container
- Build a new Docker image
- Start a new container on port 8080

### Manual Deployment

If you prefer to run commands manually:

```bash
# Stop and remove old container
sudo docker stop noyoutipme-site
sudo docker rm noyoutipme-site

# Build new image
sudo docker build -t noyoutipme .

# Run new container
sudo docker run -d -p 8080:80 --name noyoutipme-site --restart unless-stopped noyoutipme
```

## Troubleshooting

### 502 Bad Gateway Error

**Symptoms:** Website shows "Bad Gateway" error from Cloudflare

**Common Causes:**

1. **Docker container not running**
   ```bash
   # Check if container is running
   sudo docker ps | grep noyoutipme-site

   # If not running, check all containers
   sudo docker ps -a

   # Restart the container
   ./deploy.sh
   ```

2. **Container running but nginx not responding**
   ```bash
   # Check container logs
   sudo docker logs noyoutipme-site

   # Test if nginx is responding
   curl http://localhost:8080
   ```

3. **Wrong port mapping**
   ```bash
   # Verify port 8080 is listening
   ss -tulpn | grep 8080

   # Should show docker-proxy listening on 8080
   ```

4. **Cloudflared tunnel not running**
   ```bash
   # Check if cloudflared is running
   ps aux | grep cloudflared

   # Check tunnel config
   cat /home/thickarus/.cloudflared/config.yml

   # Restart tunnel if needed
   killall cloudflared
   /home/thickarus/.local/bin/cloudflared tunnel --config /home/thickarus/.cloudflared/config.yml run noyoutipme
   ```

### Docker Issues

**Docker not installed:**
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

**Permission denied:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and back in, or use:
newgrp docker
```

**Container won't start:**
```bash
# Check Docker logs
sudo docker logs noyoutipme-site

# Check if port 8080 is already in use
sudo lsof -i :8080

# Inspect container
sudo docker inspect noyoutipme-site
```

### Cloudflare Tunnel Issues

**Check tunnel status:**
```bash
# View running tunnels
ps aux | grep cloudflared

# Check tunnel connectivity
sudo /home/thickarus/.local/bin/cloudflared tunnel info noyoutipme
```

**Restart tunnel:**
```bash
# Kill existing tunnel
pkill cloudflared

# Start tunnel
/home/thickarus/.local/bin/cloudflared tunnel --config /home/thickarus/.cloudflared/config.yml run noyoutipme
```

### Nginx Issues

**Test nginx inside container:**
```bash
# Execute command in running container
sudo docker exec noyoutipme-site nginx -t

# Check nginx status
sudo docker exec noyoutipme-site ps aux | grep nginx

# View nginx logs
sudo docker exec noyoutipme-site cat /var/log/nginx/error.log
```

## Useful Commands

### Docker Management

```bash
# View running containers
sudo docker ps

# View all containers (including stopped)
sudo docker ps -a

# View container logs
sudo docker logs noyoutipme-site

# Follow logs in real-time
sudo docker logs -f noyoutipme-site

# Stop container
sudo docker stop noyoutipme-site

# Start existing container
sudo docker start noyoutipme-site

# Restart container
sudo docker restart noyoutipme-site

# Remove container
sudo docker rm noyoutipme-site

# View images
sudo docker images

# Remove image
sudo docker rmi noyoutipme

# Execute command in container
sudo docker exec -it noyoutipme-site /bin/sh
```

### Testing

```bash
# Test local nginx response
curl http://localhost:8080

# Test with headers
curl -I http://localhost:8080

# Check what's listening on port 8080
sudo lsof -i :8080
sudo ss -tulpn | grep 8080

# Test Cloudflare tunnel routing
curl https://noyoutipme.com
curl https://www.noyoutipme.com
```

## File Structure

```
/home/thickarus/noyoutipme.com/
├── index.html          # Main website file
├── Dockerfile          # Docker configuration
├── deploy.sh           # Deployment script
└── README.md           # This file
```

## Environment Setup

### System Requirements
- Ubuntu/Debian Linux
- Docker installed and running
- Cloudflared installed at `/home/thickarus/.local/bin/cloudflared`

### Cloudflare Tunnel Configuration
- Config file: `/home/thickarus/.cloudflared/config.yml`
- Credentials: `/home/thickarus/.cloudflared/b8c566a4-cf3c-4186-b397-7993c5872e7d.json`

## Development Workflow

1. **Edit** `index.html` with your changes
2. **Test locally** (optional): `python3 -m http.server 8080`
3. **Deploy**: `./deploy.sh`
4. **Verify**: Visit https://noyoutipme.com

## Important Notes

- **Always redeploy after changes** - Docker containers are immutable; changes to `index.html` won't appear until you rebuild
- **Keep Cloudflared running** - The tunnel must be active for the site to be accessible
- **Port 8080 must be free** - Ensure nothing else is using this port
- **Docker must be running** - Check with `sudo systemctl status docker`
