# Purple Team Loop Log – Kelompok 17 (Kelas A)

- **Mata Kuliah:** TEK1314 – Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB University)
- **Target Sistem:** Infrastruktur Backend EduRover / RoboRace (`192.168.17.5`)
- **Sumber Serangan (Attacker / Kali Linux):** `192.168.17.100`
- **Node Monitoring (Security Onion):** `192.168.17.1` / `eth0`
- **Fokus:** Skenario 3 – Web Vulnerability Scanning (Item #1, Item #2, dan Item #3)

---

## Ringkasan Alur Bukti (Before & After Hardening)

- **Bukti Red Team (`docs/redteam/evidence/`):** Output pengujian serangan / *re-test* `curl` dari sisi penyerang.
- **Bukti Blue Team (`docs/blueteam/evidence/`):** Log Nginx (`/var/log/nginx/access.log` dan `/var/log/nginx/roborace_access.log`), tangkapan paket Wireshark pada `eth0`, serta langkah konfigurasi *hardening* pada aplikasi Laravel, PHP/Apache, dan Nginx.

---

## Item #1 – Host Header Injection (`X-Forwarded-Host`)

### A. Before Hardening
- **Aksi Red Team:** Mengirimkan *request* `curl -sk -H "X-Forwarded-Host: evil.com" https://192.168.17.5 | grep "evil.com"`.
- **Hasil Serangan:** URL aset dan tautan navigasi aplikasi (`/events`, `/challenges`, `/leaderboard`, `/login`, `/register`, dll.) terpantul mengarah ke `evil.com`.
- **Bukti Red Team:**
  - [host_header_injection.jpg](redteam/evidence/host_header_injection.jpg) — *PoC Host Header Injection via X-Forwarded-Host: evil.com (Before Hardening)*
- **Bukti Blue Team:**
  - [deteksi_host_header_injection.jpg](blueteam/evidence/deteksi_host_header_injection.jpg) — *Log Kustom roborace_access.log Menangkap Field xfh="evil.com" (Before Hardening)* (`27/Nov/2026:02:31:15`–`02:31:39`, `192.168.17.100`, status `200`, `host="192.168.17.5"`, `xfh="evil.com"`)
  - [wireshark_host_header_injection.jpg](blueteam/evidence/wireshark_host_header_injection.jpg) — *Capture Wireshark pada eth0 Saat PoC Item #1 (TLSv1.3 Terenkripsi)*

### B. After Hardening & Re-Test
- **Mitigasi Blue Team:**
  - Membatasi `trustProxies` pada `bootstrap/app.php` ke proxy tepercaya (`127.0.0.1`, `172.16.0.0/12`) dengan header `HEADER_X_FORWARDED_FOR | HEADER_X_FORWARDED_PROTO`, lalu *rebuild* kontainer serta menjalankan `artisan optimize:clear` dan `artisan config:cache`.
  - Memperbarui `/etc/nginx/sites-available/roborace` agar `X-Forwarded-For` menggunakan `$remote_addr` dan mengosongkan `X-Forwarded-Host` serta `X-Forwarded-Port` (`""`), kemudian memverifikasi `nginx -t` dan memuat ulang Nginx.
- **Hasil Re-Test:** Saat diuji ulang dengan `X-Forwarded-Host: evil.com`, referensi `evil.com` tidak lagi muncul pada respons dan aset kembali mengarah ke `https://192.168.17.5/...`. Upaya penyisipan `xfh="evil.com"` tetap tercatat di `roborace_access.log`, sementara paket jaringan di `eth0` tetap tampak sebagai sesi `TLSv1.3` terenkripsi.
- **Bukti Red Team:**
  - [hardening_hostheaderinjection_evidence.jpg](redteam/evidence/hardening_hostheaderinjection_evidence.jpg) — *Re-Test Host Header Injection via curl dengan X-Forwarded-Host: evil.com (After Hardening)*
- **Bukti Blue Team:**
  - Tahapan Hardening Laravel & Nginx (`tahap1` s.d. `tahap7`):
    [hardening_hostheaderinjection_tahap1.jpg](blueteam/evidence/hardening_hostheaderinjection_tahap1.jpg) ·
    [hardening_hostheaderinjection_tahap2.jpg](blueteam/evidence/hardening_hostheaderinjection_tahap2.jpg) ·
    [hardening_hostheaderinjection_tahap3.jpg](blueteam/evidence/hardening_hostheaderinjection_tahap3.jpg) ·
    [hardening_hostheaderinjection_tahap4.jpg](blueteam/evidence/hardening_hostheaderinjection_tahap4.jpg) ·
    [hardening_hostheaderinjection_tahap5.jpg](blueteam/evidence/hardening_hostheaderinjection_tahap5.jpg) ·
    [hardening_hostheaderinjection_tahap6.jpg](blueteam/evidence/hardening_hostheaderinjection_tahap6.jpg) ·
    [hardening_hostheaderinjection_tahap7.jpg](blueteam/evidence/hardening_hostheaderinjection_tahap7.jpg)
  - [hardening_hostheaderinjection_evidence2.jpg](blueteam/evidence/hardening_hostheaderinjection_evidence2.jpg) — *Log roborace_access.log Saat Re-Test Item #1 (After Hardening)*
  - [wireshark_hardening_hostheaderinjection_evidence.jpg](blueteam/evidence/wireshark_hardening_hostheaderinjection_evidence.jpg) — *Capture Wireshark pada eth0 Saat Re-Test Item #1 (After Hardening, 05:34:56)*
  - [wireshark_hardening_hostheaderinjection_evidence2.jpg](blueteam/evidence/wireshark_hardening_hostheaderinjection_evidence2.jpg) — *Capture Wireshark pada eth0 Saat Re-Test Item #1 (After Hardening, 05:35:10)*

---

## Item #2 – Laravel Debug Mode Aktif

### A. Before Hardening
- **Aksi Red Team:** Mengirimkan *request* `curl -sk -i https://192.168.17.5/api/jsonws/index.jsp` ke *endpoint* yang tidak ada.
- **Hasil Serangan:** Server mengembalikan status `404 Not Found` (JSON) beserta *stack trace* lengkap Laravel (termasuk *path* `/var/www/html/vendor/...` dan urutan *middleware*) karena *debug mode* masih aktif.
- **Bukti Red Team:**
  - PoC Laravel Debug Mode Aktif (`aktif1` s.d. `aktif6`):
    [laravel_debug_aktif1.jpg](redteam/evidence/laravel_debug_aktif1.jpg) ·
    [laravel_debug_aktif2.jpg](redteam/evidence/laravel_debug_aktif2.jpg) ·
    [laravel_debug_aktif3.jpg](redteam/evidence/laravel_debug_aktif3.jpg) ·
    [laravel_debug_aktif4.jpg](redteam/evidence/laravel_debug_aktif4.jpg) ·
    [laravel_debug_aktif5.jpg](redteam/evidence/laravel_debug_aktif5.jpg) ·
    [laravel_debug_aktif6.jpg](redteam/evidence/laravel_debug_aktif6.jpg)
- **Bukti Blue Team:**
  - [enampercobaan.jpg](blueteam/evidence/enampercobaan.jpg) — *Log Nginx (/var/log/nginx/access.log) – 6x Percobaan Akses /api/jsonws/index.jsp dari 192.168.17.100 (Before Hardening)*
  - [wireshark_laraveldebug.jpg](blueteam/evidence/wireshark_laraveldebug.jpg) — *Capture Wireshark pada eth0 Saat PoC Item #2 (TLSv1.3 Terenkripsi)*

### B. After Hardening & Re-Test
- **Mitigasi Blue Team:**
  - Mengubah konfigurasi `.env` dari `APP_ENV=local`, `APP_DEBUG=true`, `LOG_LEVEL=debug` menjadi `APP_ENV=production`, `APP_DEBUG=false`, dan `LOG_LEVEL=error`.
  - Memeriksa konfigurasi `bootstrap/app.php`, melakukan *rebuild* kontainer `app`, menjalankan `artisan optimize:clear` dan `artisan config:cache`, serta memastikan `artisan about --only=environment` menunjukkan `Environment: production` dan `Debug Mode: OFF`.
- **Hasil Re-Test:** Permintaan `curl -sk -i https://192.168.17.5/api/jsonws/index.jsp` mengembalikan `404 Not Found` dengan *body* ringkas `{"message":"Resource not found."}` tanpa *stack trace*. Akses tetap tercatat pada `roborace_access.log` (status `404`), dan lalu lintas pada `eth0` tetap terenkripsi `TLSv1.3`.
- **Bukti Red Team:**
  - [hardening_laravel_debug_evidence.jpg](redteam/evidence/hardening_laravel_debug_evidence.jpg) — *Re-Test Laravel Debug Mode Nonaktif – Respons 404 JSON Tanpa Stack Trace (After Hardening)*
- **Bukti Blue Team:**
  - Tahapan Hardening & Verifikasi Environment Laravel (`tahap1` s.d. `tahap6`):
    [hardening_laravel_debug_tahap1.jpg](blueteam/evidence/hardening_laravel_debug_tahap1.jpg) ·
    [hardening_laravel_debug_tahap2.jpg](blueteam/evidence/hardening_laravel_debug_tahap2.jpg) ·
    [hardening_laravel_debug_tahap3.jpg](blueteam/evidence/hardening_laravel_debug_tahap3.jpg) ·
    [hardening_laravel_debug_tahap4.jpg](blueteam/evidence/hardening_laravel_debug_tahap4.jpg) ·
    [hardening_laravel_debug_tahap5.jpg](blueteam/evidence/hardening_laravel_debug_tahap5.jpg) ·
    [hardening_laravel_debug_tahap6.jpg](blueteam/evidence/hardening_laravel_debug_tahap6.jpg)
  - [hardening_laravel_debug_evidence2.jpg](blueteam/evidence/hardening_laravel_debug_evidence2.jpg) — *Log roborace_access.log Saat Re-Test Item #2 (After Hardening)*
  - [wireshark_hardening_laravel_debug_evidence.jpg](blueteam/evidence/wireshark_hardening_laravel_debug_evidence.jpg) — *Capture Wireshark pada eth0 Saat Re-Test Item #2 (After Hardening)*

---

## Item #3 – Open Redirect & Host Reflection via Request-Line

### A. Before Hardening
- **Aksi Red Team:** Mengirimkan `curl -sk -i --request-target "http://100.100.100.200/latest/meta-data/" https://192.168.17.5`.
- **Hasil Serangan:** Server mengembalikan `301 Moved Permanently` dengan header `Location: http://100.100.100.200/latest/meta-data/` serta *body* HTML yang memantulkan target tujuan.
- **Bukti Red Team:**
  - [openredirect_hostreflection_via_requestline.jpg](redteam/evidence/openredirect_hostreflection_via_requestline.jpg) — *PoC Open Redirect & Host Reflection via Request-Line ke 100.100.100.200 (Before Hardening)*
- **Bukti Blue Team:**
  - [deteksi_openredirect_hostreflection.jpg](blueteam/evidence/deteksi_openredirect_hostreflection.jpg) — *Log Kustom roborace_access.log Menangkap Perubahan Field host="100.100.100.200" (Before Hardening)*
  - [wireshark_openredirect_hostreflection_via_requestline.jpg](blueteam/evidence/wireshark_openredirect_hostreflection_via_requestline.jpg) — *Capture Wireshark pada eth0 Saat PoC Item #3 (TLSv1.3 Terenkripsi)*

### B. After Hardening & Re-Test
- **Mitigasi Blue Team:**
  - Memperketat `/etc/nginx/sites-available/roborace` dengan pengecekan `if ($host != "192.168.17.5") { return 400; }`, menetapkan `proxy_set_header Host 192.168.17.5;`, menyembunyikan `X-Powered-By`, menambahkan *security headers* (`X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, `Content-Security-Policy`), serta memuat ulang Nginx setelah `nginx -t`.
  - Membatasi `allowed_origins` pada `config/cors.php` ke `https://192.168.17.5` dan menambahkan `SESSION_SECURE_COOKIE=true` pada `.env`.
  - Menerapkan *hardening* pada `Dockerfile` serta konfigurasi PHP (`expose_php=Off`, `display_errors=Off`, `display_startup_errors=Off`, `log_errors=On`) dan Apache (`ServerTokens Prod`, `ServerSignature Off`), diikuti *rebuild* kontainer serta `artisan optimize:clear` dan `artisan config:cache`.
- **Hasil Re-Test:** Saat diuji ulang dengan `--request-target "http://100.100.100.200/latest/meta-data/"`, server menolak permintaan dengan status **`400 Bad Request`** (tidak lagi melakukan *redirect* `301` ke `100.100.100.200`) dan menyertakan *header* keamanan tambahan. Log `roborace_access.log` mencatat status `400` pada `06:15:31`–`06:18:56`, dan paket jaringan pada `eth0` tetap tampak sebagai sesi `TLSv1.3` terenkripsi.
- **Bukti Red Team:**
  - [open_redirect&host_reflection_evidence.jpg](redteam/evidence/open_redirect&host_reflection_evidence.jpg) — *Re-Test Open Redirect & Host Reflection via curl – Respons 400 Bad Request (After Hardening)*
- **Bukti Blue Team:**
  - Konfigurasi Nginx, CORS, & Session (`tahap1` s.d. `tahap55`):
    [hardening_openredirect_tahap1.jpg](blueteam/evidence/hardening_openredirect_tahap1.jpg) ·
    [hardening_openredirect_tahap2.jpg](blueteam/evidence/hardening_openredirect_tahap2.jpg) ·
    [hardening_openredirect_tahap_2.jpg](blueteam/evidence/hardening_openredirect_tahap_2.jpg) ·
    [hardening_openredirect_tahap3.jpg](blueteam/evidence/hardening_openredirect_tahap3.jpg) ·
    [hardening_openredirect_tahap4.jpg](blueteam/evidence/hardening_openredirect_tahap4.jpg) ·
    [hardening_openredirect_tahap55.jpg](blueteam/evidence/hardening_openredirect_tahap55.jpg)
  - Hardening Dockerfile, PHP/Apache, Rebuild, & Verifikasi (`tahap6` s.d. `tahap10`):
    [hardening_openredirect_tahap_6.jpg](blueteam/evidence/hardening_openredirect_tahap_6.jpg) ·
    [hardening_openredirect_tahap6.jpg](blueteam/evidence/hardening_openredirect_tahap6.jpg) ·
    [hardening_openredirect_tahap7.jpg](blueteam/evidence/hardening_openredirect_tahap7.jpg) ·
    [hardening_openredirect_tahap7_2.jpg](blueteam/evidence/hardening_openredirect_tahap7_2.jpg) ·
    [hardening_openredirect_tahap8.jpg](blueteam/evidence/hardening_openredirect_tahap8.jpg) ·
    [hardening_openredirect_tahap9.jpg](blueteam/evidence/hardening_openredirect_tahap9.jpg) ·
    [hardening_openredirect_tahap10.jpg](blueteam/evidence/hardening_openredirect_tahap10.jpg)
  - [open_redirect&host_reflection_evidence_2.jpg](blueteam/evidence/open_redirect&host_reflection_evidence_2.jpg) — *Log roborace_access.log Menunjukkan Status 400 Saat Re-Test Item #3 (After Hardening)*
  - [wireshark_hardening_openredirect.jpg](blueteam/evidence/wireshark_hardening_openredirect.jpg) — *Capture Wireshark pada eth0 Saat Re-Test Item #3 (After Hardening, 06:18:56)*
````[cite: 38, 39, 40, 41, 42, 43]
