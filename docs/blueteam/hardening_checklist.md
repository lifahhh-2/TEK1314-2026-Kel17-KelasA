
# Blue Team Hardening Checklist (Target Node - Kelompok 17)

Daftar kontrol keamanan (*hardening*) minimum yang diterapkan pada Target Node (`192.168.17.5`). Screenshot bukti fisik disimpan di `docs/blueteam/evidence/`.

| Kategori | Kontrol Hardening | Target Service | Status | Bukti File di GitHub |
| :--- | :--- | :--- | :--- | :--- |
| **A. Access & Auth** | Nonaktifkan login password & Wajibkan SSH Key Auth (`PasswordAuthentication no`) | OpenSSH Server | **Selesai** | `docs/blueteam/evidence/ssh_key_auth.png` |
| **A. Access & Auth** | Nonaktifkan `PermitRootLogin` & pasang `Fail2ban` | OpenSSH Server | **Selesai** | `docs/blueteam/evidence/fail2ban_status.png` |
| **B. Service & Surface** | Konfigurasi Firewall UFW (Whitelist port 22, 80, 443, 1883 dari `192.168.17.0/24`) | UFW Firewall | **Selesai** | `docs/blueteam/evidence/ufw_status.png` |
| **B. Service & Surface** | Sembunyikan identitas versi Nginx (`server_tokens off`) | Nginx Web Server | **Selesai** | `docs/blueteam/evidence/nginx_tokens.png` |
| **B. Service & Surface** | Terapkan Nginx Rate Limiting pada endpoint API | Nginx Web Server | **Proses** | `docs/blueteam/evidence/rate_limit.png` |
| **C. Data & App** | Isolasi Total Port 3306 MySQL (Docker `appnet` Internal Only, No Published Port) | MySQL Docker | **Selesai** | `docs/blueteam/evidence/docker_compose.png` |
| **C. Data & App** | Enkripsi kredensial DB via file `.env` & set izin akses ketat (`chmod 600 .env`) | Backend / Environment | **Selesai** | `docs/blueteam/evidence/env_chmod.png` |
| **C. Data & App** | Force Redirect HTTP ke HTTPS (301) + HSTS Header (`max-age=31536000`) | Nginx Web Server | **Proses** | `docs/blueteam/evidence/hsts_header.png` |
