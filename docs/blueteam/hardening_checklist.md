# Blue Team Hardening Checklist (Target Node - Kelompok 17)

Daftar kontrol keamanan (*hardening*) minimum yang diterapkan pada Target Node (`192.168.17.5`). Screenshot bukti fisik disimpan di `docs/blueteam/evidence/`.

| Kategori | Kontrol Hardening | Target Service | Status | Bukti File di GitHub |
| :--- | :--- | :--- | :--- | :--- |
| **A. Access & Auth** | Nonaktifkan login password & Wajibkan SSH Key Auth (`PasswordAuthentication no`) | OpenSSH Server | **Sudah** | [SSH_key_auth.jpeg](evidence/SSH_key_auth.jpeg) |
| **A. Access & Auth** | Nonaktifkan `PermitRootLogin` & pasang `Fail2ban` | OpenSSH Server | **Sudah** | [Permitrootlogin.jpeg](evidence/Permitrootlogin.jpeg) |
| **B. Service & Surface** | Konfigurasi Firewall UFW (Whitelist port 22, 80, 443, 1883 dari `192.168.17.0/24`) | UFW Firewall | **Sudah** | [Ufw_status.jpeg](evidence/Ufw_status.jpeg) |
| **B. Service & Surface** | Sembunyikan identitas versi Nginx (`server_tokens off`)[cite: 4] | Nginx Web Server | **Sudah** | [Nginx_tokens.jpeg](evidence/Nginx_tokens.jpeg) |
| **B. Service & Surface** | Terapkan Nginx Rate Limiting pada endpoint API[cite: 4] | Nginx Web Server | **Belum** | [](evidence/rate_limit.png) |
| **C. Data & App** | Isolasi Total Port 3306 MySQL (Docker `appnet` Internal Only, No Published Port) | MySQL Docker | **Sudah** | [Isolasi_Total_Port3306_MySQL.jpeg](evidence/Isolasi_Total_Port3306_MySQL.jpeg) |
| **C. Data & App** | Enkripsi kredensial DB via file `.env` & set izin akses ketat (`chmod 600 .env`) | Backend / Environment | **Sudah** | [Env_chmod.jpeg](evidence/Env_chmod.jpeg) |
| **C. Data & App** | Force Redirect HTTP ke HTTPS (301) + HSTS Header (`max-age=31536000`) | Nginx Web Server | **Sudah** | [Redirect_HTTP_HTTPS_HSTS.jpeg](evidence/Redirect_HTTP_HTTPS_HSTS.jpeg) |
