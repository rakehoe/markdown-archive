# Minecraft Custom Domain Setup (Oracle Cloud Free + frp)

Goal: let players join the Minecraft server (Java + Bedrock) through one GoDaddy subdomain, without installing anything on their devices, while staying on free services.

**Status:** Java join through the custom subdomain is working. Bedrock RakNet and NetherNet are still to be tested.
**Decision:** `frpc` runs as a Docker container (added to the Minecraft compose file), with secrets kept in a git-ignored `.env` file.

---

## 1. Background: what was tried and why it changed

**Playit.gg free plan**
- Custom domains are not available on the free plan.
- A CNAME to the tunnel address (`xxxx.tun.ply.gg`) did not work, because playit routes by hostname and does not recognize your own domain.
- An SRV record (`_minecraft._tcp.play` → tunnel hostname, port 25565) resolved and pinged, but the join still failed with "Can't connect to server". The direct playit address worked.
- Conclusion: the free plan cannot serve your own domain. A real public IP with real ports is needed.

**Chosen solution:** a free Oracle Cloud VM running **frps** (frp server), with **frpc** (frp client) on the Windows PC next to the Minecraft Docker container. A plain DNS **A record** then points the subdomain at the VM.

```
Player -> play.yourdomain.com -> Oracle VM (frps) -> tunnel -> Windows PC (frpc) -> Minecraft container
```

Notes:
- "Oracle Cloud" (a website, no install) is not "Oracle VirtualBox" (a VM program). You do not need VirtualBox, and the Hyper-V Debian VM is not involved.
- Run `frpc` on Windows (or as a Docker container) beside the Minecraft container.

---

## 2. Create the Oracle Cloud account

1. Go to cloud.oracle.com and click **Sign up**.
2. Enter your details and choose a **home region** close to your players. It cannot be changed later.
3. Verify email and phone, and add a payment card (used for identity verification).
4. Wait for the account to finish provisioning, then sign in to the console.

---

## 3. Create the free instance

1. Menu → **Compute → Instances → Create instance**.
2. Name it (example: `mconfig`).
3. **Placement → Advanced options → Capacity type:** choose **On-demand capacity**. Do **not** use "Dedicated host" (paid).
4. **Image:** Change image → **Canonical Ubuntu** (22.04 or 24.04, Minimal is fine).
5. **Shape:** Change shape → Virtual machine → Specialty and previous generation → **VM.Standard.E2.1.Micro** (marked *Always Free-eligible*). This is AMD x86_64, so use the **linux_amd64** frp build later. If you use the ARM shape (A1.Flex), use the arm64 build.
6. **Security:** leave Shielded instance off, add no security attributes.
7. **Networking:** Create new VCN, Create new public subnet, and turn **Automatically assign public IPv4 address** ON.
8. **SSH keys:** Generate a key pair for me, then save the **private key** (and public key). The private key cannot be downloaded again.
9. **Storage:** leave defaults.
10. Check the review page shows no cost, then click **Create**. Wait for **Running**.

If you get "out of capacity", wait and retry.

---

## 4. Fix a missing public IP (if the Public IP column is blank)

If the instance was created without a public IP:

1. Open the instance → **Networking** tab → click the primary VNIC name.
2. Open the **IP administration** tab.
3. On the row with your private IP, click **⋮ → Edit**.
4. Set Public IP type to **Ephemeral public IP** → **Update**.
5. The Public IP now appears on the instance list. Copy it.

If the option is greyed out, the subnet is probably private.

---

## 5. Open the ports in Oracle's network

Instance → Networking tab → click the **subnet** link → **Default Security List** → **Add Ingress Rules**.

Use Source Type **CIDR**, Source CIDR `0.0.0.0/0`, and leave Source Port Range empty.

| Protocol | Destination port | Purpose |
|---|---|---|
| TCP | 7000 | frp control connection |
| TCP | 25565 | Java |
| UDP | 19132 | Bedrock RakNet |
| TCP | 19132 | Geyser signaling |
| UDP | 19133 | NetherNet media |

The ports match the Docker compose mapping:

```yaml
- "25565:25565"        # Java (TCP)
- "19132:19132/udp"    # Bedrock RakNet (UDP)
- "19132:19132/tcp"    # Geyser built-in signaling (TCP)
- "19133:19133/udp"    # NetherNet media (UDP)
```

---

## 6. SSH into the server

From Windows PowerShell:

```
ssh -i C:\path\to\your-private-key.key ubuntu@YOUR_PUBLIC_IP
```

Type `yes` at the fingerprint prompt. If Windows says the key permissions are too open, fix the key file's permissions (remove other users' access) and retry.

---

## 7. Open the ports in Ubuntu's firewall

Oracle's Ubuntu images block ports by default:

```
sudo iptables -I INPUT 1 -p tcp --dport 7000 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 25565 -j ACCEPT
sudo iptables -I INPUT 1 -p udp --dport 19132 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 19132 -j ACCEPT
sudo iptables -I INPUT 1 -p udp --dport 19133 -j ACCEPT
sudo apt update && sudo apt install -y iptables-persistent wget curl
sudo netfilter-persistent save
```

Answer **Yes** when asked to save the current rules.

---

## 8. Install and run frps on the server

1. On GitHub, open the `fatedier/frp` releases page and copy the link to **frp_x.y.z_linux_amd64.tar.gz**.
2. The Ubuntu Minimal image has no `nano`. Install it first with `sudo apt update && sudo apt install -y nano`, or create files with `cat > file << 'EOF'` instead.
3. On the server:

```
wget PASTE_LINK_HERE
tar -xzf frp_*_linux_amd64.tar.gz
cd frp_*_linux_amd64
nano frps.toml
```

4. Contents of `frps.toml` (choose your own long random token and keep it secret):

```
bindPort = 7000
auth.method = "token"
auth.token = "pick-a-long-random-string"
```

5. Test it: `./frps -c frps.toml`. It should report that it is listening on 7000. Stop it with Ctrl+C.

### Keep it running after reboots (systemd)

```
sudo mkdir -p /opt/frp
sudo cp frps frps.toml /opt/frp/
sudo nano /etc/systemd/system/frps.service
```

Contents:

```
[Unit]
Description=frp server
After=network.target

[Service]
Type=simple
ExecStart=/opt/frp/frps -c /opt/frp/frps.toml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Then:

```
sudo systemctl daemon-reload
sudo systemctl enable --now frps
sudo systemctl status frps
```

---

## 9. Run frpc as a Docker container on the Windows PC

`frpc` runs as a service in the same compose file as the Minecraft server, so it shares Docker's network and reaches the server by name.

### 9.1 Secrets in `.env` (never commit this)

Create `.env` next to the compose file (make sure it is not named `.env.txt`):

```
FRP_TOKEN=your-long-random-string
VPS_IP=YOUR_PUBLIC_IP
```

Rules:
- `VPS_IP` is the plain IP only: no `$`, no quotes, no spaces around `=`.
- Avoid `$` characters in the token, because Docker Compose can try to expand them (use `$$` for a literal one).
- Add `.env` to `.gitignore` so it never reaches GitHub. Optionally commit a `.env.example` with placeholder values.
- If the token was ever committed or pushed, treat it as leaked: generate a new token, update `frps.toml` on the VPS (`sudo systemctl restart frps`) and `.env` (`docker compose up -d frpc`). Removing it from the file does not remove it from git history.

### 9.2 `frpc.toml`

Same folder as the compose file. It contains no secrets, so it is safe to commit. `localIP` is the Minecraft container's name, not `127.0.0.1` (inside the frpc container, `127.0.0.1` would mean frpc itself).

```
serverAddr = "{{ .Envs.VPS_IP }}"
serverPort = 7000
auth.method = "token"
auth.token = "{{ .Envs.FRP_TOKEN }}"

[[proxies]]
name = "mc-java"
type = "tcp"
localIP = "MC_CONTAINER"
localPort = 25565
remotePort = 25565

[[proxies]]
name = "mc-bedrock-udp"
type = "udp"
localIP = "MC_CONTAINER"
localPort = 19132
remotePort = 19132

[[proxies]]
name = "mc-geyser-signal"
type = "tcp"
localIP = "MC_CONTAINER"
localPort = 19132
remotePort = 19132

[[proxies]]
name = "mc-nethernet-media"
type = "udp"
localIP = "MC_CONTAINER"
localPort = 19133
remotePort = 19133
```

Replace `MC_CONTAINER` with the Minecraft service name (or `container_name`) from the compose file.

### 9.3 Compose service

Add under `services:`, at the same indentation as the Minecraft service:

```yaml
  frpc:
    image: fatedier/frpc:latest
    container_name: frpc
    restart: unless-stopped
    env_file: .env
    command: ["-c", "/etc/frp/frpc.toml"]
    volumes:
      - ./frpc.toml:/etc/frp/frpc.toml:ro
```

Tips:
- Pin a version tag matching your `frps` version instead of `latest`, since mismatched frp versions can cause problems.
- If the Minecraft container is in a different compose project, either move `frpc` into that compose file or use `localIP = "host.docker.internal"` (Docker Desktop on Windows), which relies on the ports being published to the host.

### 9.4 Start and check

```
docker compose up -d frpc
docker logs -f frpc
```

Success looks like "login to the server success" followed by "start proxy success" for each proxy. After changing `.env` or `frpc.toml`, run `docker compose up -d --force-recreate frpc`.

---

## 10. GoDaddy DNS

1. Delete the old SRV and CNAME records for `play`.
2. Add an **A record**: Name `play`, Value = your Oracle public IP, default TTL.

With Java on the default port 25565, no SRV record is needed.

---

## 11. Testing (in order)

1. **Java:** join `play.yourdomain.com` with no port. **Working.**
2. **Bedrock RakNet:** add `play.yourdomain.com`, port `19132`.
3. **NetherNet:** test last.

Test from a device off your home network (a phone hotspot works).

Useful checks:
- `nslookup play.yourdomain.com` should return the Oracle IP.
- `ipconfig /flushdns` clears stale DNS on Windows.

---

## 12. Caveats and security

- **NetherNet is the uncertain part.** It is WebRTC-based, so the media connection depends on the address the server advertises. If Geyser advertises a home or Docker address instead of the VPS IP, connections can fail even with the ports forwarded. Look for a public IP / ICE address setting in your NetherNet setup.
- **Secrets:** the NetherNet registration JSON contains a private key and ticket secret. Do not share it. If it has been exposed, regenerate it (for example by deleting the file and restarting so it re-registers, if your setup supports that).
- Keep the frp token in `.env` (git-ignored), never in a committed file, and forward only the Minecraft ports.
- Traffic now goes player → VPS → home PC, so ping depends on the VPS region.
- Free-tier availability and terms can change, and idle instances may be reclaimed, so check Oracle's current policy.

---

## 13. Quick troubleshooting

| Symptom | Likely cause |
|---|---|
| Public IP column blank | Instance created without a public IP; follow section 4 |
| SSH times out | Port 22 rule missing, or no public IP |
| frpc cannot connect | Port 7000 not open in Oracle **and** Ubuntu, or token mismatch |
| Java joins fail but frpc looks fine | Port 25565 not open in both firewalls, or the Minecraft container is not publishing 25565 |
| Bedrock fails | UDP 19132 not open in both firewalls, or the proxy is missing |
| NetherNet fails | See the NetherNet caveat in section 12 |
| frpc log: `lookup $1.2.3.4: no such host` | Stray `$` (or quotes) in the `VPS_IP` value; fix `.env` and recreate the container |
| frpc log: invalid token / authorization failed | `FRP_TOKEN` in `.env` does not match `frps.toml` on the VPS |
| frpc keeps restarting | frp exits when login fails and Docker restarts it; fix the login error in the logs |
| `nano: command not found` on the VPS | Minimal Ubuntu image; `sudo apt install -y nano` |
