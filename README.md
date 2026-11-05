# Minecraft Server

## Connect
ssh fabian@192.168.0.221 - SSH into the server on the home network
ssh fabian@homeserver - SSH into the server via Tailscale (works from anywhere)
cd /srv/minecraft - Go to the Minecraft project folder (run docker compose commands from here)

## Files
scp "C:\Users\mrbaz\Downloads\[Data Pack].zip" fabian@homeserver:/srv/minecraft/data/world/datapacks/ - Send a data pack (run on PC)
scp "C:\Users\mrbaz\Downloads\OriginsPack.zip" fabian@homeserver:/srv/minecraft/data/ - Send an updated modpack (run on PC)
scp fabian@homeserver:/srv/minecraft/[file] . - Copy a file from the server to the PC (run on PC)
nano compose.yaml - Edit the compose file (Ctrl+O save, Ctrl+X exit)

## Docker
docker compose up -d - Start containers / apply changes to compose.yaml
docker compose down - Stop and remove containers (data is kept)
docker compose restart mc - Restart only the Minecraft server
docker compose ps - Show whether containers are running or restarting
docker compose logs -f - Follow the log live (Ctrl+C to stop watching)
docker compose logs --tail 50 - Show the last 50 log lines
docker stats - Live CPU and RAM usage per container
docker compose pull && docker compose up -d - Update the container images
docker exec -i minecraft rcon-cli - Open the Minecraft server console (Ctrl+C to exit)

## Server console / in-game (with / in-game)
whitelist add [name] / whitelist remove [name] / whitelist list - Manage the whitelist
op [name] - Make a player operator
list - Show who is online
save-all - Force a world save (do before backups)
kick [name] / ban [name] - Remove a player
chunky radius 3000 - Set pre-generation radius in blocks
chunky start / chunky pause / chunky continue / chunky progress - Control pre-generation
spark tps - Check server performance (MSPT under 50 ms = keeping up)
spark profiler - Find what is causing lag

## Backups
tar -czf ~/backups/world-$(date +%F).tar.gz -C /srv/minecraft/data world - Back up the world to a dated archive (run save-all first)

## System
sudo apt update && sudo apt upgrade - Update Ubuntu
df -h - Disk space
free -h - RAM usage
htop - Live system overview (q to quit)
sensors - CPU temperatures
hostname -I - The server's local IP
tailscale status - Show Tailscale devices and connection status
sudo reboot / sudo poweroff - Restart / shut down the server

## Main PC
wakepc (or wakeonlan 34:5A:60:6F:E8:F4) - Wake up the main PC
