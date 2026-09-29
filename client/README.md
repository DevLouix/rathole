# Rathole Client Setup Guide

## Prerequisites

- Docker and Docker Compose installed
- VPS public IP address
- Authentication token from server configuration
- Network connectivity to VPS on the configured port

## Configuration

### Update client.toml

Before starting the client, configure the following:

```toml
# Replace with your VPS public IP
server_addr = "YOUR_VPS_PUBLIC_IP:2333"

# Must match the token set in server.toml
token = "your_secret_token_here"
```

**Required Fields:**
- `server_addr` - VPS public IP and port (default port: 2333)
- `token` - Authentication token for server connection

## Starting the Client

### Launch with Docker Compose

```bash
docker compose up -d
```

This starts the rathole client in the background and automatically connects to your VPS server.

### Verify Startup

Check that the client is running:

```bash
docker compose ps
```

## Verification & Troubleshooting

### Check Connection Status

View real-time client logs:

```bash
docker compose logs -f rathole-client
```

**Success indicator:**
```
[INFO] Services handshaked successfully
```

### Connection Issues

| Issue | Solution |
|-------|----------|
| Connection refused | Verify VPS IP and port in client.toml |
| Authentication failed | Check token matches server.toml exactly |
| Network timeout | Ensure firewall allows outbound connection to VPS |
| Service not handshaked | Check server is running on VPS |

### Debug Mode

Enable verbose logging in docker-compose.yml:

```yaml
environment:
  RUST_LOG: debug
```

## Testing the Tunnel

### Test Public API Endpoint

Once connected, access your local services through the VPS tunnel:

#### Example: Ollama API

```bash
curl http://<YOUR_VPS_PUBLIC_IP>:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smollm2:1.7b",
    "messages": [{"role": "user", "content": "Hello via VPS!"}]
  }'
```

#### Generic Service Testing

```bash
# Replace PORT with your service port
curl http://<YOUR_VPS_PUBLIC_IP>:PORT/health

# Or use any HTTP tool
wget http://<YOUR_VPS_PUBLIC_IP>:PORT/
```

## Port Configuration

Services are forwarded through configured ports. Ensure:

1. Local service is running on the specified port
2. Port is properly mapped in client.toml
3. VPS firewall allows inbound traffic on the port

## Stopping the Client

```bash
docker compose down
```

This gracefully closes the tunnel connection.

## Common Ports

| Service | Default Port |
|---------|--------------|
| Ollama | 11434 |
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| Custom | Configurable |

## Performance & Monitoring

- Monitor CPU/memory usage: `docker stats rathole-client`
- Check connection latency in logs
- Reconnection attempts are automatic on connection loss

## Security Considerations

⚠️ **Important:**
- Keep your token secret (treat like a password)
- Use HTTPS/TLS for sensitive data
- Restrict VPS firewall rules to necessary ports only
- Rotate tokens periodically

## Next Steps

- Review server configuration: `../server/README.md`
- Configure port forwarding rules
- Test services through the tunnel
- Monitor logs for production stability
