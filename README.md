# Minecraft Home Server
 
A self-hosted Minecraft server built from a repurposed desktop PC, running headless on Ubuntu Server and managed entirely over SSH. This README doubles as my own command reference for running it.
 
## Setup
 
| Hardware and Tools | Details |
|---|---|
| CPU | Intel Core i7-3770K |
| RAM | 16 GB DDR3 |
| Motherboard | MSI Z77A-GD65 |
| Storage | 2 x 128 GB SATA SSD |
| OS | Ubuntu Server 24.04 LTS |
| Containers | Docker + Docker Compose, `itzg/minecraft-server` image |
| Game | Minecraft 1.20.1, Fabric, custom CurseForge modpack |
| Public access | playit.gg tunnels (TCP for the game, UDP for Simple Voice Chat) |
| Remote admin | Tailscale VPN, SSH, Cockpit web console |
| Extras | Wake-on-LAN for waking my main PC remotely |
 
## My setup process
 
- First I found an old unused pc and tried installing Ubuntu Server on it. The motherboard couldn't read all the files that was on the flash drive I used so i opted instead on installing the ISO Image on a SSD.
- After the installation and setup of the OS was done docker was then installed and with that a minecraft server image was added as a container.
- Using the <compose.yaml> file
- **CurseForge API errors (HTTP 403):** confirmed the API key was valid by testing it directly with `curl`, then used debug logs to identify request throttling during the modpack install.
- **Server stopping right after start:** read `latest.log` and used `docker inspect` (exit code, OOM status) to find an incompatible world from an earlier server version.
- **Exposing the server without router access:** used playit.gg tunnels instead of port forwarding, since I don't control the router.
## Connect
 
| Command | Description |
|---|---|
| `ssh fabian@192.168.0.221` | SSH into the server on the home network |
| `ssh fabian@home-server` | SSH into the server via Tailscale (works from anywhere) |
| `cd /srv/minecraft` | Go to the Minecraft project folder (run `docker compose` commands from here) |
 
## Files
 
| Command | Description |
|---|---|
| `scp "C:\Users\mrbaz\Downloads\[Data Pack].zip" fabian@home-server:/srv/minecraft/data/world/datapacks/` | Send a data pack (run on PC) |
| `scp "C:\Users\mrbaz\Downloads\OriginsPack.zip" fabian@home-server:/srv/minecraft/data/` | Send an updated modpack (run on PC) |
| `scp fabian@home-server:/srv/minecraft/[file] .` | Copy a file from the server to the PC (run on PC) |
| `nano compose.yaml` | Edit the compose file (Ctrl+O save, Ctrl+X exit) |
 
## Docker
 
| Command | Description |
|---|---|
| `docker compose up -d` | Start containers / apply changes to compose.yaml |
| `docker compose down` | Stop and remove containers (data is kept) |
| `docker compose restart mc` | Restart only the Minecraft server |
| `docker compose ps` | Show whether containers are running or restarting |
| `docker compose logs -f` | Follow the log live (Ctrl+C to stop watching) |
| `docker compose logs --tail 50` | Show the last 50 log lines |
| `docker stats` | Live CPU and RAM usage per container |
| `docker compose pull && docker compose up -d` | Update the container images |
| `docker exec -i minecraft rcon-cli` | Open the Minecraft server console (Ctrl+C to exit) |
 
## Server console / in-game
 
Use these in the server console, or in-game chat with a `/` in front.
 
| Command | Description |
|---|---|
| `whitelist add [name]` / `whitelist remove [name]` / `whitelist list` | Manage the whitelist |
| `op [name]` | Make a player operator |
| `list` | Show who is online |
| `save-all` | Force a world save (do before backups) |
| `kick [name]` / `ban [name]` | Remove a player |
| `chunky radius 3000` | Set pre-generation radius in blocks |
| `chunky start` / `chunky pause` / `chunky continue` / `chunky progress` | Control pre-generation |
| `spark tps` | Check server performance (MSPT under 50 ms = keeping up) |
| `spark profiler` | Find what is causing lag |
 
## Backups
 
| Command | Description |
|---|---|
| `tar -czf ~/backups/world-$(date +%F).tar.gz -C /srv/minecraft/data world` | Back up the world to a dated archive (run `save-all` first) |
 
## System
 
| Command | Description |
|---|---|
| `sudo apt update && sudo apt upgrade` | Update Ubuntu |
| `df -h` | Disk space |
| `free -h` | RAM usage |
| `htop` | Live system overview (q to quit) |
| `sensors` | CPU temperatures |
| `hostname -I` | The server's local IP |
| `tailscale status` | Show Tailscale devices and connection status |
| `sudo reboot` / `sudo poweroff` | Restart / shut down the server |
 
## Main PC
 
| Command | Description |
|---|---|
| `wakepc` (or `wakeonlan 34:5A:60:6F:E8:F4`) | Wake up the main PC |
