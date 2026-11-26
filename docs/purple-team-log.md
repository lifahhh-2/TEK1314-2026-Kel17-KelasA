# Purple Team Loop Log – Kelompok 17 (Kelas A)

- **Proyek:** SOC Engagement – Mengamankan Infrastruktur Backend EduRover / RoboRace
- **Mata Kuliah:** TEK1314 – Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB)
- **Target & IP Plan:** `192.168.17.5` (Target / RoboRace) | `192.168.17.100` (Attacker / Kali) | `192.168.17.1` (Gateway/Jump)

---

## Ringkasan Siklus & Status Bukti

| Item | Kerentanan / Skenario | Status *Before* | Status *After* |
| :--- | :--- | :--- | :--- |
| **#1** | Host Header Injection (`X-Forwarded-Host`) | Lengkap | Lengkap (*Hardening* & *Re-test*) |
| **#2** | Laravel Debug Mode Aktif (`APP_DEBUG=true`) | Lengkap | Lengkap (*Hardening* & *Re-test*) |
| **#3** | Open Redirect & Host Reflection via Request-Line | Lengkap | Menunggu bukti *after* |

---

## Item #1 – Host Header Injection (`X-Forwarded-Host`)

### A. Before Hardening
- **Red Team (PoC):** Permintaan `curl -sk -H "X-Forwarded-Host: evil.com" https://192.168.17.5 | grep "evil.com"` memunculkan aset dan tautan navigasi yang mengarah ke `evil.com`.
  - ![PoC Host Header Injection via X-Forwarded-Host: evil.com (Before Hardening)](../redteam/evidence/host_header_injection.jpg)
- **Blue Team (Log Nginx):** `roborace_access.log` mencatat `GET / HTTP/1.1` (`200`) dari `192.168.17.100` dengan `host="192.168.17.5"` dan `xfh="evil.com"` pada `27/Nov/2026:02:31:15`–`02:31:39`.
  - ![Log Kustom roborace_access.log Menangkap Field xfh="evil.com" (Before Hardening)](../blueteam/evidence/deteksi_host_header_injection.jpg)
- **Blue Team (Wireshark):** Tangkapan di `eth0` (`02:31:39`) memperlihatkan sesi `TCP` dan `TLSv1.3 Application Data` terenkripsi antara `192.168.17.100` dan `192.168.17.5`.
  - ![Capture Wireshark pada eth0 Saat PoC Item #1 (TLSv1.3 Terenkripsi)](../blueteam/evidence/wireshark_host_header_injection.jpg)

### B. After Hardening & Re-Test
- **Blue Team (Konfigurasi & Penerapan):**
  - Kondisi awal `trustProxies(at: '*')` di `bootstrap/app.php`:
    - ![Kondisi Awal trustProxies di bootstrap/app.php](../blueteam/evidence/hardening_hostheaderinjection_tahap1.jpg)
  - Pembatasan `trustProxies` ke `['127.0.0.1', '172.16.0.0/12']` dan *header* `HEADER_X_FORWARDED_FOR | HEADER_X_FORWARDED_PROTO`:
    - ![Perubahan trustProxies di bootstrap/app.php](../blueteam/evidence/hardening_hostheaderinjection_tahap2.jpg)
  - *Rebuild* container `app` via Docker Compose:
    - ![Rebuild Container Laravel App untuk Item #1](../blueteam/evidence/hardening_hostheaderinjection_tahap3.jpg)
  - Cek sintaks PHP (`php -l`) dan *clear/cache* konfigurasi Laravel:
    - ![Cek Sintaks dan Cache Konfigurasi Laravel Item #1](../blueteam/evidence/hardening_hostheaderinjection_tahap4.jpg)
  - Konfigurasi awal `/etc/nginx/sites-available/roborace`:
    - ![Konfigurasi Awal Nginx roborace Sebelum Pengetatan Header](../blueteam/evidence/hardening_hostheaderinjection_tahap5.jpg)
  - Pengosongan `X-Forwarded-Host` dan `X-Forwarded-Port` serta penetapan `X-Forwarded-For` ke `$remote_addr` di Nginx:
    - ![Konfigurasi Nginx roborace Setelah Pengetatan Proxy Header](../blueteam/evidence/hardening_hostheaderinjection_tahap6.jpg)
  - Verifikasi *symlink* `sites-enabled`, `nginx -t`, dan `systemctl reload nginx`:
    - ![Uji Konfigurasi dan Reload Nginx Item #1](../blueteam/evidence/hardening_hostheaderinjection_tahap7.jpg)
- **Red Team (Re-Test):** Uji ulang `curl -sk -H "X-Forwarded-Host: evil.com" https://192.168.17.5` tidak lagi memunculkan `evil.com`, dan aset `app-*` kembali merujuk ke `https://192.168.17.5`.
  - ![Re-Test curl X-Forwarded-Host Setelah Hardening Item #1](../redteam/evidence/hardening_hostheaderinjection_evidence.jpg)
- **Blue Team (Log & Wireshark Saat Re-Test):**
  - Log `roborace_access.log` tetap mencatat upaya `xfh="evil.com"` (`05:33:20`–`05:35:10`), namun dampak *poisoning* URL/aset berhasil dicegah:
    - ![Log roborace_access.log Saat Re-Test Item #1](../blueteam/evidence/hardening_hostheaderinjection_evidence2.jpg)
  - Wireshark di `eth0` (`05:34:56` dan `05:35:10`) merekam lalu lintas `TCP` dan `TLSv1.3 Application Data` terenkripsi:
    - ![Capture Wireshark pada eth0 Saat Re-Test Item #1 (05:34:56)](../blueteam/evidence/wireshark_hardening_hostheaderinjection_evidence.jpg)
    - ![Capture Wireshark pada eth0 Saat Re-Test Item #1 (05:35:10)](../blueteam/evidence/wireshark_hardening_hostheaderinjection_evidence2.jpg)

---

## Item #2 –
