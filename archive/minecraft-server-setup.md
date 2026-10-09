# Paper Minecraft Server on Windows 11 (Docker) with Java + Bedrock Crossplay

A complete walkthrough, plus the risks to your PC and how to reduce them.

---

## Part 1: Install Docker on Windows 11

1. **Enable virtualization** in BIOS/UEFI if needed.
   Check: Task Manager → Performance → CPU → "Virtualization: Enabled".
2. **Install WSL 2** (PowerShell as Administrator):
   ```powershell
   wsl --install
   ```
   Reboot when asked.
3. **Install Docker Desktop**, from docker.com or via winget:
   ```powershell
   winget install -e --id Docker.DockerDesktop
   ```
4. Launch Docker Desktop and accept the terms.
   Confirm **Settings → General → "Use the WSL 2 based engine"** is ticked.
5. **Verify:**
   ```powershell
   docker --version
   docker run hello-world
   ```

---

## Part 2: Create the Paper server with Geyser + Floodgate

Geyser lets Bedrock clients (phones, consoles, Windows Bedrock) connect to a Java server. Floodgate lets Bedrock players join without a Java account.

1. Create a folder, e.g. `C:\minecraft`.
2. Inside it, create `docker-compose.yml`:

```yaml
services:
  mc:
    image: itzg/minecraft-server
    container_name: mc
    ports:
      - "25565:25565"          # Java (TCP)
      - "19132:19132/udp"      # Bedrock (UDP)
    environment:
      EULA: "TRUE"
      TYPE: "PAPER"
      VERSION: "LATEST"        # consider pinning, e.g. "1.21.4"
      MEMORY: "4G"
      MOTD: "ghetti's crossplay server"
      ONLINE_MODE: "TRUE"
      ENABLE_WHITELIST: "TRUE"
      WHITELIST: "yourjavaname"
      PLUGINS: |
        https://download.geysermc.org/v2/projects/geyser/versions/latest/builds/latest/downloads/spigot
        https://download.geysermc.org/v2/projects/floodgate/versions/latest/builds/latest/downloads/spigot
    mem_limit: 6g              # hard cap on RAM for the container
    cpus: 4                    # cap CPU use
    volumes:
      - ./data:/data
    stdin_open: true
    tty: true
    restart: unless-stopped
```

3. Start it from that folder:
   ```powershell
   docker compose up -d
   docker logs -f mc
   ```
4. Wait for `Done (...)! For help, type "help"`. Confirm the logs show Geyser and Floodgate loaded and Geyser listening on UDP 19132.

---

## Part 3: Windows Firewall

Run in an Administrator PowerShell. These rules are limited to **Private** networks so they don't open ports on public Wi-Fi:

```powershell
New-NetFirewallRule -DisplayName "Minecraft Java" -Direction Inbound -Protocol TCP -LocalPort 25565 -Action Allow -Profile Private
New-NetFirewallRule -DisplayName "Minecraft Bedrock" -Direction Inbound -Protocol UDP -LocalPort 19132 -Action Allow -Profile Private
```

Make sure your home network is set to **Private**: Settings → Network & internet → your connection → Network profile type.

---

## Part 4: Connect

| Who | How |
|---|---|
| Java, same PC | Add server `localhost` |
| Java, same network | Add server `<your-PC-IPv4>` (run `ipconfig`) |
| Bedrock, same network | Servers → Add Server → IP, port **19132** |
| Consoles | No "add server" button. Use the BedrockConnect DNS method or a friends-list workaround |
| Over the internet | Forward TCP 25565 and UDP 19132 on your router to your PC's local IP (see risks first) |

---

## Part 5: Manage the server

```powershell
docker exec -i mc rcon-cli                 # server console
docker exec mc rcon-cli op yourname        # make a Java player operator
docker exec mc rcon-cli whitelist add friendname
docker exec mc rcon-cli fwhitelist add BedrockName   # Floodgate whitelist (no dot needed)
docker compose stop                        # stop
docker compose start                       # start
docker compose pull; docker compose up -d  # update image (re-downloads plugins on restart)
```

Notes:
- Bedrock players appear with a `.` prefix (e.g. `.Steve`). Use `.Steve` for `op`, or use `fwhitelist` for the whitelist.
- If Bedrock players can't join after a Minecraft update, wait for Geyser to update, then restart the container.
- Geyser config: `data/plugins/Geyser-Spigot/config.yml`.

### Optional: limit WSL 2 memory

Create `C:\Users\<you>\.wslconfig`:
```
[wsl2]
memory=6GB
processors=4
```
Then run `wsl --shutdown` and restart Docker Desktop.

---

## Part 6: Risks to your PC and network

### 1. Exposing your home network (highest risk)
Port forwarding makes a service on your PC reachable by anyone on the internet. Bots constantly scan for open Minecraft ports. A bug in the server, a plugin, or Docker could become an entry point to your PC or home network.

### 2. Your public IP address is revealed
Anyone who connects learns your home IP. This enables targeted DDoS attacks (which can knock your whole home connection offline), and IP-based harassment.

### 3. Server and plugin vulnerabilities
Minecraft servers have had serious remote-code-execution bugs before (for example Log4Shell in 2021). Out-of-date Paper, Geyser, Floodgate or other plugins are the most common way servers get compromised.

### 4. Plugin supply-chain risk
The compose file downloads plugins from URLs using `latest`. If a download source were ever compromised, or you add plugins from unofficial sites, you could run malicious code. Third-party plugins on random forums are a well-known malware vector.

### 5. Container escape / over-permissive mounts
Docker Desktop containers are isolated, but not perfectly. Mounting broad folders such as `C:\Users\you` into a container gives anything inside it access to your personal files. Running Docker with unnecessary privileges widens the damage if something is compromised.

### 6. Exposed admin interfaces
RCON (port 25565-adjacent default 25575) gives full console control. If it is published to the internet with a weak password, the server is effectively handed over.

### 7. Offline mode / unauthenticated players
Setting `ONLINE_MODE: "FALSE"` lets anyone log in with any username, including an operator's name, and take over the server.

### 8. Griefing and abuse
Open servers get griefed, spammed, and used to host illegal or abusive content, which is your responsibility as the host.

### 9. Performance, heat and hardware wear
A server running 24/7 uses CPU/RAM and can make the PC slow, loud, hot, and shorten the life of fans and drives. WSL 2 can also grab a lot of RAM and starve Windows and your other programs.

### 10. Data loss
Corrupted worlds from power loss, crashes, bad plugins, or a mistaken `docker compose down -v` can destroy your world with no recovery if you have no backups.

### 11. Using your main PC as the host
Your daily PC holds your accounts, browser sessions, and personal files. If it's compromised through the server, all of that is at risk.

---

## Part 7: Safety steps

### Before you open anything to the internet

- [ ] **Test on LAN only first.** Make sure everything works with friends on the same network before port forwarding.
- [ ] **Keep `ONLINE_MODE: "TRUE"`** (never set it to FALSE on a public server).
- [ ] **Turn on the whitelist** (already set in the compose file) so only people you approve can join.
- [ ] **Do not forward or publish RCON** (port 25575). Only forward 25565/TCP and 19132/UDP. If you do use RCON, set a strong unique password and keep it local.
- [ ] **Limit firewall rules to the Private profile** (done in Part 3).
- [ ] **Keep Docker mounts minimal.** Only mount `./data` (or a named volume). Never mount `C:\`, your user folder, Documents, or Desktop.
- [ ] **Do not run containers with `--privileged`** or mount the Docker socket into the Minecraft container.
- [ ] **Set resource limits** (`mem_limit`, `cpus`, and `.wslconfig`) so the server can't starve your PC.

### Reduce what an attacker would find

- [ ] **Use only trusted plugins** from official sources (GeyserMC, PaperMC Hangar, Modrinth). Avoid random jar files from forums or Discord.
- [ ] **Hide your home IP.** Instead of port forwarding, use a tunnel or relay service (for example playit.gg, or a cheap VPS acting as a proxy). Then friends connect to the relay's address, not your home IP.
- [ ] **Alternatively, use a VPN** such as Tailscale or ZeroTier so only invited friends can reach the server, with no public ports at all. This is the safest option for a small friend group.
- [ ] **Use a separate Windows user account** (standard, not administrator) to run Docker and the server if you can.
- [ ] **Consider a dedicated machine** (an old PC, mini PC, or a VM) for hosting rather than your main computer.

### Keep it updated

- [ ] Update regularly:
  ```powershell
  docker compose pull
  docker compose up -d
  ```
- [ ] Update Docker Desktop and Windows when updates are available.
- [ ] Check the Paper and Geyser release notes for security fixes.
- [ ] Consider pinning `VERSION` to a specific release once you're stable, then upgrading deliberately instead of auto-updating on every restart.

### Backups

- [ ] Stop-free backup of the data folder (run from `C:\minecraft`):
  ```powershell
  docker exec mc rcon-cli save-off
  docker exec mc rcon-cli save-all flush
  Compress-Archive -Path .\data -DestinationPath ".\backup-$(Get-Date -Format yyyy-MM-dd-HHmm).zip"
  docker exec mc rcon-cli save-on
  ```
- [ ] Keep backups **outside** the server folder, ideally on a second drive or cloud storage.
- [ ] Test restoring a backup at least once.
- [ ] Never run `docker compose down -v` unless you've backed up, since `-v` deletes volumes.
- Optional: the `itzg/mc-backup` container automates scheduled backups.

### Monitoring and response

- [ ] Check logs now and then: `docker logs mc --tail 200`.
- [ ] Watch for unknown players, unexpected ops, or strange console commands. Review `ops.json` and `whitelist.json` in `data/`.
- [ ] Watch Task Manager for unexpected CPU or network use when no one is playing.
- [ ] If you suspect a compromise: `docker compose down`, remove the router port forwards, change passwords (including any reused ones), scan the PC with Windows Security, and restore from a known-good backup.
- [ ] Set a player limit if you like: `MAX_PLAYERS: "10"` in the compose file.

### Router hygiene (if you port forward)

- [ ] Forward only the two required ports, to a fixed local IP (set a DHCP reservation).
- [ ] Turn off UPnP on the router if you don't need it.
- [ ] Use a strong router admin password and keep firmware updated.
- [ ] Remove the port forwards whenever the server isn't in use.

---

## Quick risk summary

| Setup | Risk level | Notes |
|---|---|---|
| LAN only, whitelist on | Low | Good for testing and local play |
| VPN (Tailscale/ZeroTier) for friends | Low | No public ports, no exposed IP |
| Tunnel/relay service | Low–medium | Hides home IP; trust the provider |
| Direct port forward, whitelist, updated | Medium | Public IP exposed, needs vigilance |
| Direct port forward, no whitelist, offline mode | High | Not recommended |

---

## Command cheat sheet

```powershell
docker compose up -d            # start
docker compose stop             # stop
docker compose start            # start again
docker logs -f mc               # live logs
docker exec -i mc rcon-cli      # console
docker compose pull; docker compose up -d   # update
```
