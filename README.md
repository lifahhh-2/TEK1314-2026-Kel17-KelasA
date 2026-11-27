# TEK1314-2026-Kel17-KelasA
Repository Mata Kuliah Keamanan Siber Kelas A – Kelompok 17 | TEK1314 2026
**Kelompok:** Kelompok 17 – Kelas A
- **Anggota:**
  - Nurkholifah (`J0404241055`) – Lead
  - Muhammad Zeinal Haq (`J0404241108`) – Blue Team
  - Muhammad Rifqi Fadhilah (`J0404241013`) – Red Team

# PBL Keamanan Siber - Kelompok 17

## Skenario Proyek
Proyek PBL Kelompok 17 mengangkat skenario **Simulasi Pengujian Keamanan Server Web/IoT Menggunakan Ubuntu Server CLI sebagai Target**.

Target Server berperan sebagai server web yang sekaligus menjadi backend layanan IoT (contoh: dashboard monitoring sensor atau kontrol perangkat IoT sederhana). Ubuntu Server CLI dipilih karena ringan secara resource namun tetap merepresentasikan konfigurasi server produksi yang umum digunakan untuk layanan web maupun backend IoT.

Lingkungan proyek terdiri dari tiga node utama dalam satu segmen jaringan:
- **Kali Linux** — Attacker Node (Red Team)
- **Ubuntu Server CLI (Server Web/IoT)** — Target Server
- **Security Onion** — Monitoring Node (Blue Team)

## Topologi Jaringan
Ketiga node terhubung ke satu switch (Cisco 2960-24TT) dalam satu broadcast domain:

| Node | Role | Interface Switch |
|---|---|---|
| Target Server (Ubuntu Server CLI) | Target Node | Fa0/1 |
| Security Onion | Monitoring Node | Fa0/2 |
| Kali Linux | Attacker Node | Fa0/3 |

## Network
- Network: `192.168.17.0/24`
- Target Server (Ubuntu Server CLI): `192.168.17.5`
- Attacker Node (Kali Linux): `192.168.17.100`
- Monitoring Node (Security Onion): `192.168.17.200`

Detail alokasi IP selengkapnya: [`docs/design/ip_plan.md`](docs/design/ip_plan.md).

## Port yang Menjadi Fokus Pengujian
| Port | Service | Keterangan |
|------|---------|------------|
|  22  |   SSH   | Akses remote administrasi server |
|  80  |   HTTP  | Layanan web utama |
| 443  |  HTTPS  | Layanan web terenkripsi |
| 1883 |   MQTT  | Komunikasi data IoT (broker) |

## Peran Red Team dan Blue Team
**Red Team** menguji keamanan service web dan IoT pada Target Server (konfigurasi web server, autentikasi SSH, keamanan broker MQTT).
**Blue Team** memantau aktivitas jaringan melalui Security Onion untuk mendeteksi aktivitas pengujian Red Team.

## Struktur Design
- [`docs/design/topology.png`](docs/design/topology.png) — diagram topologi jaringan
- [`docs/design/ip_plan.md`](docs/design/ip_plan.md) — perencanaan IP Address dan OS setiap node

## Ringkasan Akhir Proyek (Fase 1–3)

- **Fase 1 – Preparation & Baseline Hardening:** Kontrol dasar sistem diterapkan (kunci SSH wajib & penonaktifan *password auth* / *root login*, *firewall* UFW *whitelist*, penyembunyian versi Nginx, pengamanan `.env`, dan *redirect* HTTP ke HTTPS). Pada `docs/blueteam/hardening_checklist.md`, kontrol *pending* (*"Terapkan Nginx Rate Limiting pada endpoint API"*) telah digantikan dengan **Pembatasan CORS Origin pada endpoint API** (`allowed_origins => ['https://192.168.17.5']` pada `config/cors.php`).
- **Fase 2 – Detection & Analysis (Skenario 1–3):**
  - **Skenario 1 (Reconnaissance – Nmap & Gobuster):** Tidak ada gangguan layanan; selain port web/SSH, terdeteksi port tambahan (`3306`, `8000`, `8081`) yang perlu diverifikasi isolasi Docker/UFW-nya, sedangkan akses berkas sensitif klasik pada *path* yang diuji mengembalikan `403`.
  - **Skenario 2 (SSH Brute-Force – Hydra):** Ditolak pada lapisan metode autentikasi sebelum percobaan *password* berjalan; teridentifikasi potensi *blind spot* deteksi jika mekanisme bergantung pada kegagalan *password* di `auth.log`.
  - **Skenario 3 (Web Vulnerability Scanning – Nikto, WhatWeb, cURL):** Mengonfirmasi 3 temuan utama pada lapisan Nginx–Laravel: **Host Header Injection** (`X-Forwarded-Host`), **Laravel Debug Mode Aktif** (`APP_DEBUG=true`), dan **Open Redirect & Host Reflection via Request-Line**.
- **Fase 3 – Containment, Eradication, Recovery & Re-Test (Purple Team Loop):**
  - **Item #1 (Host Header Injection):** `trustProxies` Laravel dibatasi ke `['127.0.0.1', '172.16.0.0/12']` dengan `X-Forwarded-For` dan `X-Forwarded-Proto`, serta Nginx mengosongkan `X-Forwarded-Host` & `X-Forwarded-Port` dan memakai `$remote_addr`. Hasil *re-test*: **digagalkan**. - **Item #2 (Laravel Debug Mode):** Diatur `APP_ENV=production`, `APP_DEBUG=false`, `LOG_LEVEL=error` pada `.env` serta respons JSON `404` generik (`{"message":"Resource not found."}`). Hasil *re-test*: **digagalkan** (*stack trace* tidak lagi muncul). - **Item #3 (Open Redirect & Host Reflection):** Nginx menolak `$host` selain `192.168.17.5` (`return 400;`) pada blok HTTP/HTTPS dan memakai *redirect* statis, disertai *hardening* tambahan (pembatasan CORS, *security headers* & HSTS, `SESSION_SECURE_COOKIE=true`, serta penyembunyian versi PHP/Apache). Hasil *re-test*: **digagalkan** (respons `400 Bad Request`).

## Tautan Seluruh Dokumen & Bukti (Fase 1–3)

- [README Proyek](README.md) – Ringkasan proyek, lingkungan lab, status *hardening*, dan navigasi repositori.
- [Attack Plan (Red Team)](docs/redteam/attack_plan.md) – Rencana skenario pengujian dan serangan Red Team.
- [Evidence Red Team](docs/redteam/evidence/) – Bukti *output* pemindaian awal, eksploitasi PoC, dan pengujian ulang `curl`.
- [Hardening Checklist (Blue Team)](docs/blueteam/hardening_checklist.md) – Daftar kontrol *hardening* beserta status pembaruan.
- [Evidence Blue Team](docs/blueteam/evidence/) – Bukti konfigurasi *hardening*, log Nginx (`access.log` & `roborace_access.log`), dan *capture* Wireshark (`eth0`).
- [Purple Team Log](docs/purple-team-log.md) – Catatan siklus *Before*, *Hardening*, dan *After / Re-Test* beserta tautan bukti Red Team & Blue Team untuk Item #1–#3.
- [Desain Arsitektur & IP Plan](docs/design/) – Topologi jaringan (`docs/design/topology.png`) dan rencana pengalamatan IP (`docs/design/ip_plan.md`).
- [Deklarasi Tech Stack & Presentasi Minggu 9](docs/presentation-minggu9/) – Pemetaan teknologi (`docs/presentation-minggu9/tech_stack_declaration.md`) dan materi presentasi (`docs/presentation-minggu9/slides.pdf`).

video demo https://drive.google.com/file/d/1gU133AH14SPOfyWyv7VgHenrXRQaOrfb/view?usp=sharing
