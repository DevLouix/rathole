# Rathole Server Setup Guide

## Overview

The Rathole server runs on your VPS and receives incoming connections from clients on your local network. It acts as a relay for tunneling traffic from public internet to your private services.

## Prerequisites

- VPS with public IP address
- Docker and Docker Compose installed
- Root or sudo access (for firewall configuration)
- Minimum 512MB RAM recommended
- Open internet connection

## Architecture

```
Internet → VPS Server (Port 2333) → Client Connection → Local Service
                    ↓
             [Reverse Tunnel]
```

## Initial Setup

### 1. Configure server.toml

Set the authentication token and listening ports:

```toml
# Listen address and port for client connections
bind_addr = "0.0.0.0:2333"

# Authentication token (must match client.toml)
token = "your_secret_token_here"

# Service routing (example for Ollama)
[services.ollama]
listen_addr = "0.0.0.0:11434"
backend_addr = "127.0.0.1:11434"  # Local service address from client
```

**Key Settings:**
- `bind_addr` - Where server listens for client connections (keep as 0.0.0.0:2333)
- `token` - Secret key for authentication (alphanumeric, 16+ characters recommended)
- `services` - Maps public ports to local service ports

### 2. Firewall Configuration

Open required ports on your VPS:

#### Using UFW (Ubuntu/Debian)

```bash
# Control port (client connections)
sudo ufw allow 2333/tcp

# Service ports (adjust based on your services)
sudo ufw allow 11434/tcp  # Ollama
sudo ufw allow 80/tcp     # HTTP
sudo ufw allow 443/tcp    # HTTPS

# Reload firewall
sudo ufw reload
```

#### Using AWS Security Groups

Create inbound rules:

| Protocol | Port | Source |
|----------|------|--------|
| TCP | 2333 | 0.0.0.0/0 (or restrict to your IP) |
| TCP | 11434 | 0.0.0.0/0 (or restrict to your IP) |
| TCP | 80 | 0.0.0.0/0 |
| TCP | 443 | 0.0.0.0/0 |

#### Using GCP Firewall

```bash
gcloud compute firewall-rules create rathole-server \
  --allow=tcp:2333,tcp:11434,tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0
```

#### Using DigitalOcean

Via Control Panel → Networking → Firewalls → Add Rules

## Starting the Server

### Launch with Docker Compose

```bash
docker compose up -d
```

Verify the server is running:

```bash
docker compose ps
```

Expected output:
```
rathole-server   Up (healthy)
```

## Verification & Monitoring

### Check Server Logs

```bash
docker compose logs -f rathole-server
```

**Success indicators:**
```
[INFO] Listening at 0.0.0.0:2333
[INFO] Client connected from <IP>
```

### Monitor Real-Time Activity

```bash
# Continuous log monitoring
docker compose logs -f rathole-server --tail=100

# Resource usage
docker stats rathole-server
```

### Connection Status

Check if clients are successfully connected:

```bash
docker compose logs rathole-server | grep -i "connected\|handshake"
```

## Troubleshooting

### Server Won't Start

**Error:** `Address already in use`
```bash
# Find process on port 2333
sudo lsof -i :2333

# Kill process if needed
sudo kill -9 <PID>
```

**Error:** `Permission denied`
- Run with `sudo docker compose up -d`
- Or add user to docker group: `sudo usermod -aG docker $USER`

### Firewall Issues

**Symptom:** Clients can't connect

```bash
# Test if port is listening
sudo netstat -tulpn | grep 2333

# Test from external machine
telnet <YOUR_VPS_IP> 2333

# Check firewall status
sudo ufw status
```

### Connection Handshake Fails

**Check:**
1. Token matches between server.toml and client.toml
2. Client IP can reach VPS IP:2333
3. Server is running: `docker compose ps`
4. Firewall allows port 2333

**Enable Debug Logs:**

Update docker-compose.yml:
```yaml
environment:
  RUST_LOG: debug
```

Then restart:
```bash
docker compose down
docker compose up -d
```

## Service Configuration

### Multiple Services Example

```toml
# Protocol mapping
bind_addr = "0.0.0.0:2333"
token = "secure_token_123"

# Service 1: Ollama API
[services.ollama]
listen_addr = "0.0.0.0:11434"
backend_addr = "127.0.0.1:11434"

# Service 2: Web Application
[services.webapp]
listen_addr = "0.0.0.0:8080"
backend_addr = "127.0.0.1:8080"

# Service 3: SSH Access
[services.ssh]
listen_addr = "0.0.0.0:2222"
backend_addr = "127.0.0.1:22"
```

### Port Forwarding Rules

Each service needs:
- **listen_addr** - Public port on VPS
- **backend_addr** - Private service address (usually localhost)

## Testing Server Connectivity

### Verify Ports Are Open

```bash
# Test from VPS
sudo ss -tulpn | grep -E '2333|11434'

# Test from external machine
curl http://<YOUR_VPS_PUBLIC_IP>:11434/v1/models  # For Ollama
```

### Test Client Connection

Once a client connects, verify in logs:

```bash
docker compose logs rathole-server | grep -i "client"
```

Expected output:
```
[INFO] Client connected from 192.168.1.100
[INFO] Services handshaked successfully
```

## Maintenance

### Update Server

```bash
docker compose pull
docker compose up -d
```

### Backup Configuration

```bash
cp server.toml server.toml.backup
```

### Restart Server

```bash
docker compose restart rathole-server
```

### View Server Stats

```bash
docker stats --no-stream rathole-server
```

## Performance Optimization

### Increase Connection Limits

In docker-compose.yml:
```yaml
ulimits:
  nofile:
    soft: 65536
    hard: 65536
```

### Enable Compression

In server.toml (if supported by your rathole version):
```toml
compression = true
```

## Security Best Practices

⚠️ **Critical:**

1. **Token Security**
   - Use strong, random tokens (32+ characters)
   - Never commit to version control
   - Rotate tokens periodically

2. **Firewall Hardening**
   - Restrict port 2333 to known client IPs if possible
   - Use VPS security groups for additional protection
   - Block unused ports

3. **Network Security**
   - Use VPN for admin access to VPS
   - Monitor logs for unauthorized connection attempts
   - Set up alerts for unusual activity

4. **Data Protection**
   - Use HTTPS for web services
   - Encrypt sensitive data in transit
   - Consider TLS termination on server

## Logs and Diagnostics

### Export Logs

```bash
docker compose logs rathole-server > server.log
```

### Monitor for Errors

```bash
docker compose logs rathole-server | grep -i "error\|warn"
```

### Connection Timeline

```bash
docker compose logs rathole-server | grep -E "connected|handshake|closed"
```

## Related Documentation

- **Client Setup:** `../client/README.md`
- **Configuration Reference:** `./server.toml`
- **Docker Compose:** `./docker-compose.yml`

## Support

For issues:
1. Check logs: `docker compose logs -f rathole-server`
2. Verify firewall configuration
3. Confirm token matches client configuration
4. Test connectivity: `telnet <VPS_IP> 2333`
