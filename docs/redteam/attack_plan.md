# Red Team Attack Plan Fase 2 (Kelompok 17)

Rencana pengujian penetrasi dan simulasi serangan Red Team menyasar Target Node replika (`192.168.17.5`).

## Skenario 1: Reconnaissance & Enumeration
* **Kategori:** Reconnaissance
* **Tools Utama:** `nmap`, `gobuster`
* **Target Node:** Nginx Web Server (`192.168.17.5:80/443`)
* **Detail Eksekusi:** Melakukan pemindaian port terbuka (22, 80, 443, 1883) dan pemindaian direktori tersembunyi untuk menemukan kebocoran file sensitif (seperti `.env`).
* **Ekspektasi:** Terdeteksi oleh Suricata NIDS di Security Onion.

## Skenario 2: SSH Credential Brute-Force
* **Kategori:** Exploitation / Akses
* **Tools Utama:** `hydra`
* **Target Node:** OpenSSH Service (`192.168.17.5:22`)
* **Detail Eksekusi:** Menguji ketahanan autentikasi SSH pada konfigurasi Fase 1 menggunakan daftar kata sandi umum (*wordlist*).
* **Ekspektasi:** Memicu alert login gagal di Security Onion (`auth.log`) dan terblokir oleh `Fail2ban` pada Fase 2.

## Skenario 3: API Key Spoofing & Replay Attack
* **Kategori:** Web / API / Protokol IoT
* **Tools Utama:** `Burp Suite`, `cURL`
* **Target Node:** Laravel REST API (`192.168.17.5/api/v1/checkpoint`)
* **Detail Eksekusi:** Mengirimkan *payload* data *checkpoint* palsu tanpa header `X-API-KEY` valid untuk mencoba memanipulasi skor pada Live Leaderboard.
* **Ekspektasi:** Request ditolak dengan respon HTTP `401 Unauthorized` atau `403 Forbidden` oleh aplikasi backend.
