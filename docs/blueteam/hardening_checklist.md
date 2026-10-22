# Blue Team Hardening Checklist (Target Node - Kelompok 17)

Daftar kontrol keamanan (*hardening*) minimum yang diterapkan pada Target Node (`192.168.17.5`). Screenshot bukti fisik disimpan di `docs/blueteam/evidence/`.

| Kategori | Kontrol Hardening | Target Service | Status | Bukti File di GitHub |
| :--- | :--- | :--- | :--- | :--- |
| **A. Access & Auth** | Nonaktifkan login password & Wajibkan SSH Key Auth (`PasswordAuthentication no`) | OpenSSH Server | **Selesai** | [ssh_key_auth.png](evidence/ssh_key_auth.png) |
| **A. Access & Auth** | Nonaktifkan `PermitRootLogin` & pasang `Fail2ban` | OpenSSH Server | **Selesai** | [fail2ban_status.png](evidence/fail2ban_status.png) |
| **B. Service & Surface** | Konfigurasi Firewall UFW (Whitelist port 22, 80, 443, 1883 dari `192.168.17.0/24`) | UFW Firewall | **Selesai** | [ufw_status.png](evidence/ufw_status.png) |
| **B. Service & Surface** | Sembunyikan identitas versi Nginx (`server_tokens off`)[cite: 4] | Nginx Web Server | **Selesai** | [nginx_tokens.png](evidence/nginx_tokens.png) |
| **B. Service & Surface** | Terapkan Nginx Rate Limiting pada endpoint API[cite: 4] | Nginx Web Server | **Belum** | [rate_limit.png](evidence/rate_limit.png) |
| **C. Data & App** | Isolasi Total Port 3306 MySQL (Docker `appnet` Internal Only, No Published Port) | MySQL Docker | **Selesai** | [docker_compose.png](evidence/docker_compose.png) |
| **C. Data & App** | Enkripsi kredensial DB via file `.env` & set izin akses ketat (`chmod 600 .env`) | Backend / Environment | **Selesai** | [env_chmod.png](evidence/env_chmod.png) |
| **C. Data & App** | Force Redirect HTTP ke HTTPS (301) + HSTS Header (`max-age=31536000`) | Nginx Web Server | **Belum** | [hsts_header.png](evidence/hsts_header.png) |
