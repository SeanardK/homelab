# Security Lab — Ethical Hacking Practice Environment

Satu network isolated berisi **attacker box** (`kali`, full Kali Linux
headless) dan beberapa **target rentan** buat latihan.

## ⚠️ Aturan pakai — baca dulu sebelum jalan

- **Cuma buat dipakai Anda sendiri, di jaringan lokal Anda sendiri.** Jangan
  pernah expose port-port ini ke internet (jangan tambahin ke Cloudflare
  tunnel, jangan port-forward router ke sini).
- Network `security-lab` sengaja **terisolasi**, tidak konek ke stack lain
  (mysql_prod, vaultwarden, dll) di server ini. Jangan sambungkan container
  lain ke network ini.
- Target-target ini penuh kerentanan **yang disengaja** — anggap sebagai
  "radioactive", cukup buat latihan, jangan taruh data asli di dalamnya.
- Semua aktivitas testing harus terbatas ke target-target di sini saja.
  Jangan arahkan tools (nmap, sqlmap, burp, metasploit, dst) ke domain atau
  IP siapa pun selain milik Anda sendiri tanpa izin eksplisit — itu ilegal
  di hampir semua yurisdiksi.

## Target yang tersedia

| Target | URL (dari LAN) | Cocok buat latihan |
|---|---|---|
| **OWASP Juice Shop** | http://<IP-server>:8100 | Web app modern — SQLi, XSS, broken auth, IDOR, broken access control, dll. Ada scoreboard & hint system bawaan (`/#/score-board`), bagus buat mulai. |
| **DVWA** | http://<IP-server>:8101 | Klasik PHP+MySQL — SQLi, command injection, file upload, CSRF, dll. Ada slider difficulty (low/medium/high/impossible) di menu "DVWA Security" setelah login. |
| **WebGoat** | http://<IP-server>:8102 | Lesson terstruktur dari OWASP, step-by-step. |
| **WebWolf** (companion WebGoat) | http://<IP-server>:8103 | Dipakai beberapa lesson WebGoat (file upload/phishing simulation). |

Ganti `<IP-server>` dengan IP LAN server ini (`192.168.100.56`, sama seperti
yang dipakai Jenkins).

## Setup awal per target

- **Juice Shop**: langsung jalan, tidak perlu login/setup.
- **DVWA**: buka URL → klik "Create / Reset Database" di halaman setup
  pertama kali → login default `admin` / `password`.
- **WebGoat**: buka URL → register akun sendiri (data lokal, tidak terkirim
  ke mana pun) → ikutin lesson di sidebar.

## Menjalankan

```bash
cd security-lab
docker compose up -d
```

Build pertama kali `kali` container **lama** (image `kali-linux-headless`
besar, bisa 30-60+ menit tergantung koneksi & disk). Target-target lain
(juice-shop, dvwa, webgoat) jauh lebih cepat.

## Kali container

Container `kali` (`kali-linux-headless` — semua tools Kali versi CLI,
tanpa GUI) jalan terus di background (`sleep infinity`), satu network sama
persis dengan target-target di atas. Masuk ke dalamnya:

```bash
docker exec -it kali bash
```

Dari dalam situ, target-target bisa diakses **langsung lewat nama
container** (tidak perlu IP host / port host sama sekali, karena satu
docker network):

```bash
nmap -sV juice-shop dvwa webgoat
curl http://dvwa/
sqlmap -u "http://dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="security=low; PHPSESSID=..."
```

- `juice-shop` → port 3000 (internal)
- `dvwa` → port 80 (internal)
- `webgoat` → port 8080 (internal, path `/WebGoat`)
- `webgoat` → port 9090 (internal, WebWolf, path `/WebWolf`)

Folder `./kali/workspace` di-mount ke `/root/workspace` di dalam container
— taruh hasil scan/loot/notes di situ supaya tidak hilang kalau container
di-recreate.

`msfconsole` (Metasploit) belum di-setup database-nya — jalankan
`msfdb init` sekali di dalam container kalau mau pakai fitur database-nya.

Target-target di atas **tetap** publish ke port host (lihat tabel "Target
yang tersedia") kalau Anda mau buka lewat browser biasa (mis. buat lihat
tampilan visual Juice Shop/DVWA), bukan lewat `kali`.
