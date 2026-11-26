# Purple Team Loop Log – Kelompok 17 (Kelas A)

- **Mata Kuliah:** TEK1314 – Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB University)
- **Target Sistem:** Infrastruktur Backend EduRover / RoboRace (`192.168.17.5`)
- **Sumber Serangan (Attacker / Kali Linux):** `192.168.17.100`
- **Node Monitoring (Security Onion):** `192.168.17.1` / `eth0`
- **Fokus:** Skenario 3 – Web Vulnerability Scanning (Item #1, Item #2, dan Item #3)

---

## Ringkasan Alur Bukti (Before & After Hardening)

- **Bukti Red Team (`docs/redteam/evidence/`):** Output pengujian serangan / *re-test* `curl` dari sisi penyerang.
- **Bukti Blue Team (`docs/blueteam/evidence/`):** Log Nginx (`/var/log/nginx/access.log` dan `/var/log/nginx/roborace_access.log`), tangkapan paket Wireshark pada `eth0`, serta langkah konfigurasi *hardening* pada aplikasi Laravel dan Nginx.

---

## Item #1 – Host Header Injection (`X-Forwarded-Host`)

### A. Before Hardening
- **Aksi Red Team:** Mengirimkan *request* `curl -sk -H "X-Forwarded-Host: evil.com" https://192.168.17.5 | grep "evil.com"`[cite: 10].
- **Hasil Serangan:** URL aset dan tautan navigasi aplikasi (`/events`, `/challenges`, `/leaderboard`, `/login`, `/register`, dll.) terpantul mengarah ke `evil.com`[cite: 10].
- **Bukti Red Team (`docs/redteam/evidence/`):**
  - `host_header_injection.jpg` — *PoC Host Header Injection via X-Forwarded-Host: evil.com (Before Hardening)*[cite: 10, 36]
- **Bukti Blue Team (`docs/blueteam/evidence/`):**
  - `deteksi_host_header_injection.jpg` — *Log Kustom roborace_access.log Menangkap Field xfh="evil.com" (Before Hardening)* (`27/Nov/2026:02:31:15`–`02:31:39`, `192.168.17.100`, status `200`, `host="192.168.17.5"`, `xfh="evil.com"`)[cite: 9, 36]
  - `wireshark_host_header_injection.jpg` — *Capture Wireshark pada eth0 Saat PoC Item #1 (TLSv1.3 Terenkripsi)*[cite: 11, 36]

### B. After Hardening & Re-Test
- **Mitigasi Blue Team:**
  - Membatasi `trustProxies` pada `bootstrap/app.php` ke proxy tepercaya (`127.0.0.1`, `172.16.0.0/12`) dengan header `HEADER_X_FORWARDED_FOR | HEADER_X_FORWARDED_PROTO`, lalu *rebuild* kontainer serta menjalankan `artisan optimize:clear` dan `artisan config:cache`[cite: 17, 18, 19, 20].
  - Memperbarui `/etc/nginx/sites-available/roborace` agar `X-Forwarded-For` menggunakan `$remote_addr` dan mengosongkan `X-Forwarded-Host` serta `X-Forwarded-Port` (`""`), kemudian memverifikasi `nginx -t` dan memuat ulang Nginx[cite: 21, 22, 23].
- **Hasil Re-Test:** Saat diuji ulang dengan `X-Forwarded-Host: evil.com`, referensi `evil.com` tidak lagi muncul pada respons dan aset kembali mengarah ke `https://192.168.17.5/...`[cite: 15]. Upaya penyisipan `xfh="evil.com"` tetap tercatat di `roborace_access.log`, sementara paket jaringan di `eth0` tetap tampak sebagai sesi `TLSv1.3` terenkripsi[cite: 16, 24, 34].
- **Bukti Red Team (`docs/redteam/evidence/`):**
  - `hardening_hostheaderinjection_evidence.jpg` — *Re-Test Host Header Injection via curl dengan X-Forwarded-Host: evil.com (After Hardening)*[cite: 15, 35]
- **Bukti Blue Team (`docs/blueteam/evidence/`):**
  - `hardening_hostheaderinjection_tahap1.jpg` s.d. `hardening_hostheaderinjection_tahap7.jpg` — *Tahapan Hardening Laravel & Nginx untuk Item #1*[cite: 17, 18, 19, 20, 21, 22, 23, 35]
  - `hardening_hostheaderinjection_evidence2.jpg` — *Log roborace_access.log Saat Re-Test Item #1 (After Hardening)*[cite: 16, 35]
  - `wireshark_hardening_hostheaderinjection_evidence.jpg` — *Capture Wireshark pada eth0 Saat Re-Test Item #1 (After Hardening, 05:34:56)*[cite: 24, 35]
  - `wireshark_hardening_hostheaderinjection_evidence2.jpg` — *Capture Wireshark pada eth0 Saat Re-Test Item #1 (After Hardening, 05:35:10)*[cite: 34, 35]

---

## Item #2 – Laravel Debug Mode Aktif

### A. Before Hardening
- **Aksi Red Team:** Mengirimkan *request* `curl -sk -i https://192.168.17.5/api/jsonws/index.jsp` ke *endpoint* yang tidak ada[cite: 2].
- **Hasil Serangan:** Server mengembalikan status `404 Not Found` (JSON) beserta *stack trace* lengkap Laravel (termasuk *path* `/var/www/html/vendor/...` dan urutan *middleware*) karena *debug mode* masih aktif[cite: 2, 3, 4, 5, 6, 7].
- **Bukti Red Team (`docs/redteam/evidence/`):**
  - `laravel_debug_aktif1.jpg` s.d. `laravel_debug_aktif6.jpg` — *PoC Laravel Debug Mode Aktif – Stack Trace pada Respons 404 /api/jsonws/index.jsp (Before Hardening)*[cite: 2, 3, 4, 5, 6, 7, 36]
- **Bukti Blue Team (`docs/blueteam/evidence/`):**
  - `enampercobaan.jpg` — *Log Nginx (/var/log/nginx/access.log) – 6x Percobaan Akses /api/jsonws/index.jsp dari 192.168.17.100 (Before Hardening)*[cite: 1, 36]
  - `wireshark_laraveldebug.jpg` — *Capture Wireshark pada eth0 Saat PoC Item #2 (TLSv1.3 Terenkripsi)*[cite: 8, 36]

### B. After Hardening & Re-Test
- **Mitigasi Blue Team:**
  - Mengubah konfigurasi `.env` dari `APP_ENV=local`, `APP_DEBUG=true`, `LOG_LEVEL=debug` menjadi `APP_ENV=production`, `APP_DEBUG=false`, dan `LOG_LEVEL=error`[cite: 27, 28].
  - Memeriksa konfigurasi `bootstrap/app.php`, melakukan *rebuild* kontainer `app`, menjalankan `artisan optimize:clear` dan `artisan config:cache`, serta memastikan `artisan about --only=environment` menunjukkan `Environment: production` dan `Debug Mode: OFF`[cite: 29, 30, 31, 32].
- **Hasil Re-Test:** Permintaan `curl -sk -i https://192.168.17.5/api/jsonws/index.jsp` mengembalikan `404 Not Found` dengan *body* ringkas `{"message":"Resource not found."}` tanpa *stack trace*[cite: 25]. Akses tetap tercatat pada `roborace_access.log` (status `404`), dan lalu lintas pada `eth0` tetap terenkripsi `TLSv1.3`[cite: 26, 33].
- **Bukti Red Team (`docs/redteam/evidence/`):**
  - `hardening_laravel_debug_evidence.jpg` — *Re-Test Laravel Debug Mode Nonaktif – Respons 404 JSON Tanpa Stack Trace (After Hardening)*[cite: 25, 35]
- **Bukti Blue Team (`docs/blueteam/evidence/`):**
  - `hardening_laravel_debug_tahap1.jpg` s.d. `hardening_laravel_debug_tahap6.jpg` — *Tahapan Hardening & Verifikasi Environment Laravel untuk Item #2*[cite: 27, 28, 29, 30, 31, 32, 35]
  - `hardening_laravel_debug_evidence2.jpg` — *Log roborace_access.log Saat Re-Test Item #2 (After Hardening)*[cite: 26, 35]
  - `wireshark_hardening_laravel_debug_evidence.jpg` — *Capture Wireshark pada eth0 Saat Re-Test Item #2 (After Hardening)*[cite: 33, 35]

---

## Item #3 – Open Redirect & Host Reflection via Request-Line

### A. Before Hardening
- **Aksi Red Team:** Mengirimkan `curl -sk -i --request-target "http://100.100.100.200/latest/meta-data/" https://192.168.17.5`[cite: 13].
- **Hasil Serangan:** Server mengembalikan `301 Moved Permanently` dengan header `Location: http://100.100.100.200/latest/meta-data/` serta *body* HTML yang memantulkan target tujuan[cite: 13].
- **Bukti Red Team (`docs/redteam/evidence/`):**
  - `openredirect_hostreflection_via_requestline.jpg` — *PoC Open Redirect & Host Reflection via Request-Line ke 100.100.100.200 (Before Hardening)*[cite: 13, 36]
- **Bukti Blue Team (`docs/blueteam/evidence/`):**
  - `deteksi_openredirect_hostreflection.jpg` — *Log Kustom roborace_access.log Menangkap Perubahan Field host="100.100.100.200" (Before Hardening)*[cite: 12, 36]
  - `wireshark_openredirect_hostreflection_via_requestline.jpg` — *Capture Wireshark pada eth0 Saat PoC Item #3 (TLSv1.3 Terenkripsi)*[cite: 14, 36]

### B. After Hardening & Re-Test
- *Menunggu bukti konfigurasi mitigasi dan hasil uji ulang (re-test) pasca-hardening untuk Item #3.*
