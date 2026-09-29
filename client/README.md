Update client.toml:

Replace YOUR_VPS_PUBLIC_IP with your actual VPS IP.

Set token to match server.toml.

Start the client:

Bash
docker compose up -d
Verification & Troubleshooting
Check client connection logs:

Bash
docker compose logs -f rathole-client
Look for: [INFO] Services handshaked successfully

Test public API endpoint from outside network:
eg for ollama
Bash
curl http://<YOUR_VPS_PUBLIC_IP>:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "smollm2:1.7b",
    "messages": [{"role": "user", "content": "Hello via VPS!"}]
  }'
