Purple Team Loop Log – Kelompok 17 (Kelas A)

Proyek: SOC Engagement – Mengamankan Infrastruktur Backend EduRover / RoboRace

Mata Kuliah: TEK1314 – Keamanan Siber (D4 Teknologi Rekayasa Komputer SV IPB)

Repositori: github.com/lifahhh-2/TEK1314-2026-Kel17-KelasA

1. Informasi Tim & Infrastruktur

Tim

Lead: Nurkholifah (J0404241055)

Blue Team: Muhammad Zeinal Haq (J0404241108)

Red Team: Muhammad Rifqi Fadhilah (J0404241013)

Infrastruktur Lab (192.168.17.0/24)

Target Node: 192.168.17.5 (Ubuntu Server 22.04 LTS, Nginx, Docker appnet: Laravel + MySQL 8.0, Elastic Agent)

Attacker Node: 192.168.17.100 (Kali Linux)

Monitoring Node (Mgmt): 192.168.17.200 (Security Onion)

Monitoring Node (Sniffer): Promiscuous / SPAN (No IP)

2. Pemetaan Bukti & Penamaan File

Folder docs/redteam/evidence/

Before Hardening:

host_header_injection (Item #1 – PoC X-Forwarded-Host: evil.com)

laravel_debug_aktif1 s.d. laravel_debug_aktif6 (Item #2 – PoC /api/jsonws/index.jsp dengan stack trace Laravel)

openredirect_hostreflection_via_requestline (Item #3 – PoC --request-target ke 100.100.100.200)

After Hardening (Re-Test curl):

hostheaderinjection_evidence (Item #1 – Re-test X-Forwarded-Host: evil.com mengarah ke https://192.168.17.5)

laravel_debug_evidence (Item #2 – Re-test /api/jsonws/index.jsp mengembalikan {"message":"Resource not found."})

Folder docs/blueteam/evidence/

Before Hardening:

deteksi_host_header_injection (Item #1 – Log roborace_access.log dengan xfh="evil.com")

wireshark_host_header_injection (Item #1 – Capture Wireshark eth0 saat serangan before)

Bukti_Log Nginx – 6x Request (Item #2 – Log /var/log/nginx/access.log 6x GET /api/jsonws/index.jsp)

wireshark_laravel_debug (Item #2 – Capture Wireshark eth0 saat serangan before)

deteksi_openredirect_hostreflection (Item #3 – Log roborace_access.log dengan host="100.100.100.200")

wireshark_openredirect_hostreflection_via_requestline (Item #3 – Capture Wireshark eth0 saat serangan before)

Hardening & After Hardening:

hardening_hostheaderinjection_tahap1 s.d. hardening_hostheaderinjection_tahap7 (Item #1 – Konfigurasi bootstrap/app.php, rebuild Docker, konfigurasi Nginx, dan reload)

hostheaderinjection_evidence2 (Item #1 – Log roborace_access.log saat re-test after)

wireshark_hardening_hostheaderinjection_evidence & wireshark_hardening_hostheaderinjection_evidence2 (Item #1 – Capture Wireshark eth0 saat re-test after)

hardening_laravel_debug_tahap1 s.d. hardening_laravel_debug_tahap6 (Item #2 – Perubahan .env, pengecekan bootstrap/app.php, rebuild Docker, artisan config:cache, dan artisan about)

laravel_debug_evidence2 (Item #2 – Log roborace_access.log saat re-test after)

wireshark_hardening_laravel_debug_evidence (Item #2 – Capture Wireshark eth0 saat re-test after)

3. Siklus Purple Team Loop – Skenario 3 (Item #1–#3)

Item #1: Host Header Injection (X-Forwarded-Host)

A. Before Hardening

Serangan (Red Team): Mengirimkan permintaan curl -sk -H "X-Forwarded-Host: evil.com" https://192.168.17.5 sehingga aset dan tautan navigasi halaman ter-inject mengarah ke evil.com (host_header_injection).

Deteksi & Analisis (Blue Team):

Log kustom /var/log/nginx/roborace_access.log mencatat request GET / HTTP/1.1 (200) dari 192.168.17.100 dengan host="192.168.17.5" dan xfh="evil.com" (deteksi_host_header_injection).

Capture Wireshark pada eth0 menunjukkan lalu lintas TCP / TLSv1.3 Application Data terenkripsi antara 192.168.17.100 dan 192.168.17.5 (wireshark_host_header_injection).

B. Mitigasi & After Hardening

Mitigasi (Blue Team):

Membatasi trustProxies di bootstrap/app.php ke ['127.0.0.1', '172.16.0.0/12'] dengan header HEADER_X_FORWARDED_FOR | HEADER_X_FORWARDED_PROTO, lalu rebuild container app dan menjalankan artisan optimize:clear serta artisan config:cache (hardening_hostheaderinjection_tahap1 s.d. hardening_hostheaderinjection_tahap4).

Memperbarui /etc/nginx/sites-available/roborace agar X-Forwarded-For menggunakan $remote_addr, mengosongkan X-Forwarded-Host dan X-Forwarded-Port, lalu menguji konfigurasi dan me-reload Nginx (hardening_hostheaderinjection_tahap5 s.d. hardening_hostheaderinjection_tahap7).

Re-Test & Verifikasi (Red Team & Blue Team):

Pengujian ulang curl -sk -H "X-Forwarded-Host: evil.com" https://192.168.17.5 tidak lagi memunculkan evil.com dan aset kembali mengarah ke https://192.168.17.5 (hostheaderinjection_evidence).

Log roborace_access.log tetap merekam percobaan dari 192.168.17.100 dengan xfh="evil.com" (hostheaderinjection_evidence2), sementara capture Wireshark pada eth0 menunjukkan sesi TLSv1.3 terenkripsi (wireshark_hardening_hostheaderinjection_evidence, wireshark_hardening_hostheaderinjection_evidence2).

Item #2: Laravel Debug Mode Aktif

A. Before Hardening

Serangan (Red Team): Mengakses curl -sk -i https://192.168.17.5/api/jsonws/index.jsp yang menghasilkan 404 Not Found (JSON) namun membocorkan stack trace lengkap Laravel beserta path file internal server (laravel_debug_aktif1 s.d. laravel_debug_aktif6).

Deteksi & Analisis (Blue Team):

Log /var/log/nginx/access.log mencatat 6 kali percobaan GET /api/jsonws/index.jsp dari 192.168.17.100 dengan status 404 (Bukti_Log Nginx – 6x Request).

Capture Wireshark pada eth0 menampilkan paket TCP dan TLSv1.3 Application Data terenkripsi antara 192.168.17.100 dan 192.168.17.5 (wireshark_laravel_debug).

B. Mitigasi & After Hardening

Mitigasi (Blue Team):

Mengubah .env dari APP_ENV=local, APP_DEBUG=true, LOG_LEVEL=debug menjadi APP_ENV=production, APP_DEBUG=false, LOG_LEVEL=error (hardening_laravel_debug_tahap1, hardening_laravel_debug_tahap2).

Meninjau bootstrap/app.php, melakukan rebuild container app, menjalankan artisan optimize:clear dan artisan config:cache, serta memastikan status Environment: production dan Debug Mode: OFF melalui artisan about --only=environment (hardening_laravel_debug_tahap3 s.d. hardening_laravel_debug_tahap6).

Re-Test & Verifikasi (Red Team & Blue Team):

Pengujian ulang curl -sk -i https://192.168.17.5/api/jsonws/index.jsp mengembalikan 404 Not Found dengan respons JSON ringkas {"message":"Resource not found."} tanpa stack trace (laravel_debug_evidence).

Log roborace_access.log mencatat permintaan GET /api/jsonws/index.jsp berstatus 404 dari 192.168.17.100 (laravel_debug_evidence2), dan capture Wireshark pada eth0 menunjukkan lalu lintas TLSv1.3 terenkripsi (wireshark_hardening_laravel_debug_evidence).

Item #3: Open Redirect & Host Reflection via Request-Line

A. Before Hardening

Serangan (Red Team): Mengirimkan curl -sk -i --request-target "http://100.100.100.200/latest/meta-data/" https://192.168.17.5 yang menghasilkan respons 301 Moved Permanently ke http://100.100.100.200/latest/meta-data (openredirect_hostreflection_via_requestline).

Deteksi & Analisis (Blue Team):

Log /var/log/nginx/roborace_access.log mencatat GET http://100.100.100.200/latest/meta-data/ HTTP/1.1 (301) dari 192.168.17.100 dengan host="100.100.100.200" dan xfh="-" (deteksi_openredirect_hostreflection).

Capture Wireshark pada eth0 menunjukkan koneksi TCP dan TLSv1.3 Application Data terenkripsi antara 192.168.17.100 dan 192.168.17.5 (wireshark_openredirect_hostreflection_via_requestline).

B. Mitigasi & After Hardening

Mitigasi (Blue Team): [Menunggu konfigurasi/bukti hardening Item #3 di folder hardening / blueteam]

Re-Test & Verifikasi (Red Team & Blue Team): [Menunggu hasil re-test after hardening untuk Item #3]
