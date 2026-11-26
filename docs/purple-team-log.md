# Purple Team Loop Log – Kelompok 17 (Kelas A)

- **Mata Kuliah:** TEK1314 – Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB University)
- **Target Sistem:** Infrastruktur Backend EduRover / RoboRace (`192.168.17.5`)
- **Sumber Serangan (Attacker / Kali Linux):** `192.168.17.100`
- **Node Monitoring (Security Onion):** `192.168.17.1` / `eth0`
- **Fokus:** Skenario 3 – Web Vulnerability Scanning (Item #1, Item #2, dan Item #3)

---

## Ringkasan Alur Bukti (Before & After Hardening)

- **Bukti Red Team (`redteam/evidence/`):** Output pengujian serangan / *re-test* `curl` dari sisi penyerang.
- **Bukti Blue Team (`blueteam/evidence/`):** Log Nginx (`/var/log/nginx/access.log` dan `/var/log/nginx/roborace_access.log`), tangkapan paket Wireshark pada `eth0`, serta langkah konfigurasi *hardening* pada aplikasi Laravel dan Nginx.

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
- *Menunggu bukti konfigurasi mitigasi dan hasil uji ulang (re-test) pasca-hardening untuk Item #3.*
````[cite: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39]
