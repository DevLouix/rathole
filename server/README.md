Ensure ports 2333 and 11434 are open in your VPS firewall / Security Groups:

Bash
sudo ufw allow 2333/tcp
sudo ufw allow 11434/tcp
Start the server:

Bash
docker compose up -d
