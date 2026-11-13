# ai-agents

Stack eksplorasi berbagai AI agent framework, jalan pakai model **Claude**
(Anthropic API). Satu `docker-compose.yml`, satu agent = satu (atau
beberapa) service di dalamnya — ditambah seiring waktu.

## agent-sandbox — sandbox manual + Hermes Agent (auto-start)

Runtime Python 3.11 + Node 20 + git/curl/uv siap pakai, buat eksplor agent
apapun secara manual. Container ini **juga jalanin Hermes gateway otomatis**
tiap start (`init: true` + `restart: unless-stopped`), jadi selamat dari
reboot server dan dari nutup terminal.

```bash
cd ai-agents
docker compose up -d agent-sandbox
docker compose exec agent-sandbox bash     # masuk manual
docker compose logs -f agent-sandbox       # log Hermes gateway
```

**Shared folder:** `./sandbox/workspace` (host) ↔ `/workspace` (container).
Project taruh di `/workspace/programming` (`/root/programming` di container
adalah symlink ke situ). File yang dibikin Hermes (root) bakal root-owned di
host; kalau IDE gak bisa edit: `docker exec ai-agents-sandbox chown -R 1000:1000 /workspace/programming`.

**Image & persistensi:** service pakai `ai-agents-sandbox:hermes` — snapshot
(`docker commit`) dari container yang udah ke-install Hermes, karena
Hermes ke-install lewat installer resminya (bukan dari `sandbox/Dockerfile`,
yang cuma resep image dasarnya). **Jangan `docker compose build`** ke tag
ini. Config Hermes (`/root/.hermes`: `.env`, auth, sesi, DB) ada di named
volume `ai_agents_hermes_home` — aman dari `compose down`/recreate, tapi
`down -v` bakal ngehapus.

**Backup** — folder `./backups` (host) ↔ `/backups` (container), di-gitignore
karena isinya secret (`.env` + `auth.json`). File yang dibikin dari dalam
container itu root-owned, jadi di-`chown` biar kebaca dari host:

```bash
docker exec ai-agents-sandbox sh -c 'f=/backups/hermes-home-$(date +%Y%m%d-%H%M%S).tgz && tar czf $f -C /root .hermes && chown 1000:1000 $f && chmod 600 $f'
# restore: docker exec ai-agents-sandbox tar xzf /backups/<file>.tgz -C /root
# snapshot image lama tersimpan sebagai ai-agents-sandbox:hermes-20260918
```

Install Hermes ulang / update? Lakukan di container, lalu `docker commit
ai-agents-sandbox ai-agents-sandbox:hermes` biar ikut ke-snapshot.

## Agent berikutnya

Ditambah sebagai service baru di `docker-compose.yml` seiring waktu. Tiap
agent boleh punya kebutuhan network/volume berbeda — jangan dipaksa satu
pola seragam.
