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
 
- First I found an old unused PC and tried installing Ubuntu Server on it. The installer booted but couldn’t find its files on the USB stick, likely a USB compatibility issue with the older board. After ruling out ports, BIOS settings and the ISO, I wrote the installer to a spare SSD, which worked.
- After the installation and setup of the OS was done i installed Docker and with that a Minecraft server image was added as a container.
- Using the `compose.yaml` file I set up the mc service and environment. It was at this step I installed mods on to the server with fabric.
- I used `scp` to send the modpack from my PC to the server and then added the proper API key and CF settings to the environment and tested.
- The server failed with an API key error. The log said dollar signs had to be escaped, so I doubled every $ in the key. It still failed with HTTP 403, so I tested the key directly against the CurseForge API with curl and got 200, which meant the key itself was valid. Debug logging showed that one request in a burst was being refused, so the real cause was rate limiting, made worse by the container’s restart loop. I stopped the container, waited for the limit to reset, and the install went through.
- After the API problem was fixed I went on and installed performance enhancing mods that alleviate some of the work for the cpu and ram. I then sent the new modpack to the server again using `scp` and tested it.
- To let friends join from outside my network I decided to not use portforwarding. Instead I used playit.gg, which tunnels traffic to the server without any router changes. I added the playit agent as a second container in `compose.yaml` with host networking, so it can reach the Minecraft.
- The game and the voice chat use different protocols, so they needed separate tunnels. Minecraft itself runs over TCP on port 25565, while the Simple Voice Chat mod sends audio over UDP on port 24454. I mapped `24454:24454/udp` in `compose.yaml` and created two tunnels in playit.gg: a Minecraft Java tunnel (TCP) and a Simple Voice Chat tunnel (UDP). I then configured `voicechat-server.properties` with `bind_address=*` so the mod listens on all interfaces, and `voice_host` set to the UDP tunnel's public address so players' clients connect through it. My first attempt put the tunnel address in `bind_address`, which is the server's local listening address, so I moved it to `voice_host`.
- Since playit.gg makes the server reachable by anyone who finds the address, I turned on the whitelist (`whitelist on`, `whitelist add [name]`) so only invited players can join.
- I added a datapack (GhastianOrigin) by copying it from my PC to `world/datapacks/` with `scp` and loading it with `reload` in the server console.
- For my own remote access I installed Tailscale on the server. It creates a private, encrypted network between my devices, so I can reach the server's SSH console and the Cockpit web dashboard from anywhere without exposing either to the internet. Neither port is open publicly; they're only reachable through Tailscale.
- I also used the server to solve a separate problem: waking my main PC remotely for game streaming. Wake-on-LAN packets from outside were blocked by the network, but the server is always on and sits on the same local network. I installed `wakeonlan` on the server, found my PC's MAC address with `ipconfig /all`, and created a `wakepc` alias. Now I can SSH into the server over Tailscale and send the magic packet locally.

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

## Docker compose.yaml File
```
services:
  mc:
    image: itzg/minecraft-server:java21
    container_name: minecraft
    ports:
      - "25565:25565"
    environment:
      EULA: "TRUE"
      TYPE: "AUTO_CURSEFORGE"
      CF_API_KEY: "$$2a$$10$$<rest-of-api-key-here>"
      CF_SLUG: "custom"
      CF_FORCE_SYNCHRONIZE: "true"
      CF_MODPACK_ZIP: "/data/OriginsPack.zip"
      MEMORY: "12G"
      VIEW_DISTANCE: "16"
      SIMULATION_DISTANCE: "8"
      DEBUG_HELPER: "true"
    volumes:
      - ./data:/data
    restart: unless-stopped
    tty: true
    stdin_open: true
  playit:
    image: ghcr.io/playit-cloud/playit-agent:latest
```
