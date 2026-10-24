# Blue Team Hardening Checklist (Target Node - Kelompok 17)

Daftar kontrol keamanan (*hardening*) minimum yang diterapkan pada Target Node (`192.168.17.5`). Screenshot bukti fisik disimpan di `docs/blueteam/evidence/`.

| Kategori | Kontrol Hardening | Target Service | Status | Bukti File di GitHub |
| :--- | :--- | :--- | :--- | :--- |
| **A. Access & Auth** | Nonaktifkan login password & Wajibkan SSH Key Auth (`PasswordAuthentication no`) | OpenSSH Server | **Belum** | [evidence/ssh_key_auth.jpeg](evidence/ssh_key_auth.jpeg) |
| **A. Access & Auth** | Nonaktifkan `PermitRootLogin` & pasang `Fail2ban` | OpenSSH Server | **Sudah** | [permitrootlogin.jpe](evidence/permitrootlogin.jpeg) |
| **B. Service & Surface** | Konfigurasi Firewall UFW (Whitelist port 22, 80, 443, 1883 dari `192.168.17.0/24`) | UFW Firewall | **Sudah** | [ufw_status.jpeg](evidence/ufw_status.jpeg) |
| **B. Service & Surface** | Sembunyikan identitas versi Nginx (`server_tokens off`)[cite: 4] | Nginx Web Server | **Belum** | [](evidence/nginx_tokens.png) |
| **B. Service & Surface** | Terapkan Nginx Rate Limiting pada endpoint API[cite: 4] | Nginx Web Server | **Belum** | [](evidence/rate_limit.png) |
| **C. Data & App** | Isolasi Total Port 3306 MySQL (Docker `appnet` Internal Only, No Published Port) | MySQL Docker | **Sudah** | [docker_compose.png](evidence/docker_compose.png) |
| **C. Data & App** | Enkripsi kredensial DB via file `.env` & set izin akses ketat (`chmod 600 .env`) | Backend / Environment | **Sudah** | [env_chmod.png](evidence/env_chmod.png) |
| **C. Data & App** | Force Redirect HTTP ke HTTPS (301) + HSTS Header (`max-age=31536000`) | Nginx Web Server | **Belum** | [](evidence/hsts_header.png) |
