# Deklarasi Tech Stack & Replikasi Target Node (Kelompok 17)

Tabel ini memetakan teknologi pada sistem asli proyek IoT **EduRover** ke lingkungan simulasi **Target Node (VM Ubuntu)** pada jaringan lab terisolasi `192.168.17.0/24`.

| Komponen | Teknologi pada Proyek IoT Asli | Direplikasi sebagai (Target Node) |
| :--- | :--- | :--- |
| **Sistem Operasi Backend** | Linux pada laptop server lokal; aplikasi Laravel/PHP | Ubuntu Server 22.04 LTS (VM VirtualBox, IP `192.168.17.5`) + Nginx Host |
| **Database / Penyimpanan** | MySQL atau MariaDB pada server lokal | MySQL 8.0 dalam Docker Container (`appnet`, port 3306 terisolasi total) |
| **Protokol Sensor** | HTTP REST melalui Wi-Fi lokal; ESP32-S3 mengirim data via header `X-API-KEY` | HTTP REST API (`/api/v1/checkpoint`) & Mosquitto Host (Port 1883) |
| **Dashboard / Monitoring** | Aplikasi web Laravel; leaderboard realtime via Laravel Reverb/WebSocket | Web Container (`nginx:alpine` di loopback `127.0.0.1:8080`) diakses via browser lab |
