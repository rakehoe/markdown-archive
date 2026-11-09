# Transition Plan: Paper to Fabric, Adding Mods, and Upgrading RAM

Server: Minecraft in Docker on Windows 11, Java + Bedrock crossplay, exposed through playit.gg.
Container name: **`reykc`** (all `docker` commands below use this name).

> **Container name vs. service name:** `docker ps`, `docker logs`, `docker exec` use the **container name** (`reykc`).
> `docker compose ...` commands use the **service name** from `docker-compose.yml`. If you renamed only `container_name`, the service name may still be `mc`. Check with `docker compose config --services`.

---

## Part 1: Transition from Paper to Fabric

### What changes

- Paper plugins do **not** work on Fabric. Fabric uses **mods** (in a `mods` folder).
- Geyser, Floodgate and spark all have Fabric versions, so crossplay and monitoring still work.
- Paper's performance optimizations and config files go away. You can add performance mods (Lithium, FerriteCore, Krypton) to compensate.
- Whitelist, ops, and playit tunnels carry over, because they live in `data/` and on the host.

### Step 1: Back up everything

```powershell
cd C:\minecraft
docker compose stop
Compress-Archive -Path .\data -DestinationPath ".\backup-before-fabric-$(Get-Date -Format yyyy-MM-dd-HHmm).zip"
```

Copy the zip to a second drive or cloud storage. Do not continue until the backup exists.

### Step 2: Convert the world folders (to keep your world)

Paper keeps the Nether and End in separate folders. Fabric (like vanilla) keeps them inside the main world folder.

```powershell
cd C:\minecraft\data
Move-Item .\world_nether\DIM-1 .\world\DIM-1
Move-Item .\world_the_end\DIM1 .\world\DIM1
```

- If `DIM-1` or `DIM1` already exists inside `world`, rename or delete that one first.
- Skip this step if you are starting a brand-new world.

### Step 3: Move old Paper plugins out of the way

```powershell
cd C:\minecraft\data
Move-Item .\plugins .\plugins-paper-old
```

Delete `plugins-paper-old` later, once everything works. Keep it until then, since it holds your old Geyser config.

### Step 4: Edit docker-compose.yml

Change `TYPE`, remove the `PLUGINS:` block, and add mods through Modrinth:

```yaml
services:
  mc:                                   # service name (yours may differ)
    image: itzg/minecraft-server
    container_name: reykc
    ports:
      - "25565:25565"
      - "19132:19132/udp"
    environment:
      EULA: "TRUE"
      TYPE: "FABRIC"
      VERSION: "1.21.4"                 # pin to a version your mods support
      MEMORY: "4G"
      USE_AIKAR_FLAGS: "true"
      ONLINE_MODE: "TRUE"
      ENABLE_WHITELIST: "TRUE"
      MODRINTH_PROJECTS: |
        fabric-api
        geyser
        floodgate
        spark
      MODRINTH_DOWNLOAD_DEPENDENCIES: "required"
    mem_limit: 6g
    cpus: 4
    volumes:
      - ./data:/data
    stdin_open: true
    tty: true
    restart: unless-stopped
```

Keep your other existing settings (RCON, MOTD, whitelist, and so on).

Notes:
- **Pin `VERSION`.** Mods are built for specific Minecraft versions. `LATEST` can break them after an update.
- **Fabric API** is required by most mods, so keep it in the list.
- **Whitelist names** in `WHITELIST:` must be real Java usernames only. Bedrock names go in through `fwhitelist`.

### Step 5: Start and verify

```powershell
cd C:\minecraft
docker compose up -d --force-recreate
docker logs -f reykc
```

Look for:
- the Fabric loader starting,
- your mods loading with no errors,
- Geyser listening on UDP 19132,
- `Done (...)! For help, type "help"`.

Then:

```powershell
docker ps
docker exec reykc rcon-cli list
docker exec reykc mc-monitor status
```

`docker ps` should show `reykc` as `Up ... (healthy)`.

### Step 6: Test crossplay

- Join as a Java player (with a Fabric client on the same Minecraft version).
- Join as a Bedrock player on port 19132.
- Check the playit dashboard shows the agent online, with the Java tunnel pointing to `127.0.0.1:25565` and the Bedrock tunnel to `127.0.0.1:19132`.

### Rollback plan (if something goes wrong)

```powershell
cd C:\minecraft
docker compose down
Rename-Item .\data .\data-failed-fabric
Expand-Archive .\backup-before-fabric-<timestamp>.zip -DestinationPath .
```

Then change `TYPE` back to `PAPER`, restore the `PLUGINS:` block, and run `docker compose up -d`. **Do not use `down -v`.**

---

## Part 2: Adding mods

### Before you add anything

1. **Check the Minecraft version.** Each mod page on Modrinth lists supported versions. It must match your `VERSION`.
2. **Check the loader.** The mod must support **Fabric** (not Forge, NeoForge, or Quilt-only).
3. **Check where it runs.** Mods are marked Server, Client, or both:
   - **Server-side only** mods (performance, admin tools): nothing needed on players' computers.
   - **Both sides** (new blocks, items, mobs): every Java player needs the same mod.
4. **Check dependencies.** Many mods need Fabric API or other library mods.
5. **Only download from Modrinth or CurseForge** pages of known authors. Mods run with full access inside the container, so random jar files are a malware risk.

### Method A: Modrinth slugs (recommended)

The slug is the last part of the mod's URL: `modrinth.com/mod/lithium` → `lithium`.

1. Open `docker-compose.yml`.
2. Add the slug on a new line under `MODRINTH_PROJECTS`:
   ```yaml
   MODRINTH_PROJECTS: |
     fabric-api
     geyser
     floodgate
     spark
     lithium
     ferrite-core
   ```
3. Apply it:
   ```powershell
   cd C:\minecraft
   docker compose up -d --force-recreate
   docker logs -f reykc
   ```

Required dependencies download automatically because `MODRINTH_DOWNLOAD_DEPENDENCIES` is set to `required`.

### Method B: Manual jar files

1. Download the correct `.jar` from the mod's official page.
2. Stop the server:
   ```powershell
   docker compose stop
   ```
3. Put the file in `C:\minecraft\data\mods\`.
4. Start it:
   ```powershell
   docker compose start
   docker logs -f reykc
   ```

### Removing a mod

- **Method A:** delete the slug from `MODRINTH_PROJECTS`, then delete the old jar from `data\mods\` and recreate the container.
- **Method B:** stop the server and delete the jar from `data\mods\`.
- Removing a content mod that added blocks or items can delete those blocks from your world. Back up first.

### Recommended starter mods

| Mod (Modrinth slug) | Side | Purpose |
|---|---|---|
| `fabric-api` | Both | Required by most mods |
| `lithium` | Server | General performance (game logic) |
| `ferrite-core` | Server | Lower memory use |
| `krypton` | Server | Network performance |
| `spark` | Server | Profiling, TPS, player ping |
| `geyser` / `floodgate` | Server | Bedrock crossplay |

### Getting players set up (Java)

1. Install Minecraft Java on the **same version** as the server.
2. Install the **Fabric loader** (from fabricmc.net) for that version.
3. Put required client mods and Fabric API in their `mods` folder.
4. Join using the playit address.

### Mods and Bedrock players

Bedrock players cannot run Java mods. Server-side mods that don't change gameplay content work fine for them. Mods that add custom blocks, items, or mobs generally will **not** appear correctly for Bedrock players. Plan your mod list with this limit in mind.

### Troubleshooting mods

| Symptom | Likely cause | Fix |
|---|---|---|
| Container restarts in a loop | Mod for the wrong version, or a missing dependency | `docker logs reykc --tail 60`, then remove or update the named mod |
| "Incompatible mods" when joining | Client and server mod lists differ | Match the client mods to the server |
| Low TPS after adding a mod | The mod is heavy | `spark` profiler: `docker exec reykc rcon-cli spark profiler start` and check the report |
| Bedrock players can't join | Geyser or Floodgate version mismatch | Update both mods, restart, check logs |
| Out of memory errors | Not enough RAM | See Part 3 |

---

## Part 3: Upgrading the server's dedicated RAM

### How RAM settings fit together

There are three layers, and each must be larger than the one inside it:

```
Windows PC RAM  >  WSL 2 limit (.wslconfig)  >  mem_limit (container)  >  MEMORY (Java heap)
```

- **`MEMORY`**: the Java heap the Minecraft server uses (for example `4G`).
- **`mem_limit`**: the hard cap on the whole container. Set it about **1 to 2 GB above `MEMORY`**, since Java also uses memory outside the heap.
- **`.wslconfig`**: the cap on everything Docker Desktop runs. Set it above `mem_limit`.
- **Windows itself** needs about **4 GB or more** left over, plus whatever else you run.

### Step 1: Check what you have

```powershell
systeminfo | findstr /C:"Total Physical Memory"
docker stats reykc --no-stream
```

If memory use sits near the cap, or the logs show `OutOfMemoryError`, raise it.

### Step 2: Raise the WSL 2 limit

Edit (or create) `C:\Users\<you>\.wslconfig`:

```
[wsl2]
memory=10GB
processors=6
```

Apply it:

```powershell
wsl --shutdown
```

Then restart Docker Desktop.

### Step 3: Update docker-compose.yml

Example for an 8 GB server heap:

```yaml
    environment:
      MEMORY: "8G"
      USE_AIKAR_FLAGS: "true"
    mem_limit: 10g
```

### Step 4: Apply and verify

```powershell
cd C:\minecraft
docker compose up -d --force-recreate
docker logs -f reykc
docker stats reykc --no-stream
```

Run `docker exec reykc rcon-cli spark healthreport` (or watch `docker stats`) while players are online to see real usage.

### Don't over-allocate

Giving Java far more RAM than it needs does not help, and it can cause longer garbage-collection pauses (lag spikes). Increase in steps, and watch the usage with spark. Using `USE_AIKAR_FLAGS: "true"` helps keep pauses short.

### RAM recommendations by player base

These are starting points for a Fabric server with Geyser/Floodgate. Real needs depend on view distance, how much of the world has been explored, and how heavy the mods are.

| Players online at once | Light mods / performance mods only (`MEMORY`) | Medium mods (20-60 mods) | Heavy modpack (100+ mods) | Suggested `mem_limit` (light / medium / heavy) |
|---|---|---|---|---|
| 1-5 | 3-4 GB | 4-6 GB | 6-8 GB | 5 / 7 / 9-10 GB |
| 6-10 | 4-5 GB | 6-8 GB | 8-10 GB | 6 / 9 / 11-12 GB |
| 11-20 | 5-6 GB | 8-10 GB | 10-12 GB | 7 / 11-12 / 13-14 GB |
| 21-40 | 6-8 GB | 10-12 GB | 12-16 GB | 9-10 / 13-14 / 17-18 GB |
| 40+ | 8-12 GB | 12-16 GB | 16 GB+ | 13-14 / 17-18 / 18+ GB |

Add roughly **0.5 GB** to the heap if many Bedrock players connect through Geyser. Lowering `VIEW_DISTANCE` (for example 8) and `SIMULATION_DISTANCE` (for example 6) reduces both RAM and CPU use.

### What PC RAM do you need?

| Server heap (`MEMORY`) | Minimum total PC RAM | Comfortable total PC RAM |
|---|---|---|
| 4 GB | 12 GB | 16 GB |
| 6 GB | 16 GB | 16-24 GB |
| 8 GB | 16 GB | 24-32 GB |
| 12 GB | 24 GB | 32 GB |
| 16 GB | 32 GB | 32-64 GB |

This assumes the PC is also running Windows, Docker Desktop, and playit. If you play or run other heavy programs on the same PC, move up one row.

### Signs you need more (or less) RAM

| Sign | Meaning |
|---|---|
| `OutOfMemoryError` in the logs, crash loops | Increase `MEMORY` and `mem_limit` |
| `docker stats` stays near the `mem_limit` | Increase `mem_limit` or lower view distance |
| Container is killed but the Java heap isn't full | `mem_limit` too close to `MEMORY`, so raise `mem_limit` |
| Regular lag spikes with plenty of RAM free | Over-allocated heap, or CPU limit. Try lowering `MEMORY`, and check `spark tps` |
| Windows or other apps freeze | WSL limit too high. Lower `.wslconfig` memory |

---

## Quick command reference

```powershell
docker ps                                          # status
docker logs -f reykc                               # live logs
docker exec -i reykc rcon-cli                      # server console
docker exec reykc rcon-cli list                    # players online
docker exec reykc rcon-cli spark tps               # performance
docker exec reykc rcon-cli whitelist add JavaName
docker exec reykc rcon-cli fwhitelist add BedrockName
docker restart reykc                               # restart (no compose changes)
cd C:\minecraft; docker compose up -d --force-recreate   # apply compose changes
docker stats reykc --no-stream                     # RAM and CPU usage
```

Never run `docker compose down -v` unless you have a backup. The `-v` flag deletes volumes.

---

## Checklist

- [ ] Backup made and copied off the PC
- [ ] World folders converted (Nether and End moved into `world`)
- [ ] `plugins` moved to `plugins-paper-old`
- [ ] `docker-compose.yml` set to `TYPE: FABRIC` with a pinned `VERSION`
- [ ] Fabric API, Geyser, Floodgate, spark added
- [ ] Server starts `healthy`, logs show no mod errors
- [ ] Java and Bedrock test joins both work through playit
- [ ] Mod list chosen with Bedrock limits in mind
- [ ] RAM layers set (`MEMORY` < `mem_limit` < `.wslconfig` < PC RAM)
- [ ] Backups scheduled
