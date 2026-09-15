# TEK1314-2026-Kel17-KelasA
Repository Mata Kuliah Keamanan Siber Kelas A – Kelompok 17 | TEK1314 2026
- Muhammad Rifqi Fadhilah (J0404241013)
- Nurkholifah (J0404241055)
- Muhammad Zeinal Haq (J0404241108)

# PBL Keamanan Siber - Kelompok 17

## Skenario Proyek
Proyek PBL Kelompok 17 mengangkat skenario **Simulasi Pengujian Keamanan Server Web/IoT Menggunakan Ubuntu Server CLI sebagai Target**.

Target Server berperan sebagai server web yang sekaligus menjadi backend layanan IoT (contoh: dashboard monitoring sensor atau kontrol perangkat IoT sederhana). Ubuntu Server CLI dipilih karena ringan secara resource namun tetap merepresentasikan konfigurasi server produksi yang umum digunakan untuk layanan web maupun backend IoT.

Lingkungan proyek terdiri dari tiga node utama dalam satu segmen jaringan:
- **Kali Linux** — Attacker Node (Red Team)
- **Ubuntu Server CLI (Server Web/IoT)** — Target Server
- **Security Onion** — Monitoring Node (Blue Team)

## Topologi Jaringan
Ketiga node terhubung ke satu switch (Cisco 2960-24TT) dalam satu broadcast domain:

| Node | Role | Interface Switch |
|---|---|---|
| Target Server (Ubuntu Server CLI) | Target Node | Fa0/1 |
| Security Onion | Monitoring Node | Fa0/2 |
| Kali Linux | Attacker Node | Fa0/3 |

## Network
- Network: `192.168.17.0/24`
- Target Server (Ubuntu Server CLI): `192.168.17.5`
- Attacker Node (Kali Linux): `192.168.17.100`
- Monitoring Node (Security Onion): `192.168.17.200`

Detail alokasi IP selengkapnya: [`docs/design/ip_plan.md`](docs/design/ip_plan.md).

## Port yang Menjadi Fokus Pengujian
| Port | Service | Keterangan |
|------|---------|------------|
|  22  |   SSH   | Akses remote administrasi server |
|  80  |   HTTP  | Layanan web utama |
| 443  |  HTTPS  | Layanan web terenkripsi |
| 1883 |   MQTT  | Komunikasi data IoT (broker) |

## Peran Red Team dan Blue Team
**Red Team** menguji keamanan service web dan IoT pada Target Server (konfigurasi web server, autentikasi SSH, keamanan broker MQTT).
**Blue Team** memantau aktivitas jaringan melalui Security Onion untuk mendeteksi aktivitas pengujian Red Team.

## Struktur Design
- [`docs/design/topology.png`](docs/design/topology.png) — diagram topologi jaringan
- [`docs/design/ip_plan.md`](docs/design/ip_plan.md) — perencanaan IP Address dan OS setiap node
