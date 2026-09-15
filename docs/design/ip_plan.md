# IP Address Plan — Kelompok 17 (TEK1314 2026)

## Informasi Network
- Network Address: `192.168.17.0/24`
- Subnet Mask: `255.255.255.0`
- Broadcast Address: `192.168.17.255`
- Range IP Usable: `192.168.17.1` – `192.168.17.254`
- Topologi: Flat network (satu segmen L2) melalui satu switch Cisco 2960-24TT, tanpa router/L3 di antara node.

## Tabel Alokasi IP Address

| Hostname | Role | IP Address | OS Direncanakan | Interface Switch |
|---|---|---|---|---|
| Target Server | Target Node — Server Web/IoT | 192.168.17.5 | Ubuntu Server CLI | Fa0/1 |
| Security Onion | Monitoring Node (Blue Team) | 192.168.17.200 | Security Onion | Fa0/2 |
| Kali Linux | Attacker Node (Red Team) | 192.168.17.100 | Kali Linux | Fa0/3 |
| Switch 2960-24TT | Perangkat Distribusi | — | Cisco IOS | Fa0/1, Fa0/2, Fa0/3 |

## Skema Alokasi Range (Reserved untuk Ekspansi)

| Range IP | Peruntukan |
|---|---|
| 192.168.17.1 – 192.168.17.4 | Reserved — gateway/router |
| 192.168.17.5 | Target Server |
| 192.168.17.6 – 192.168.17.99 | Reserved — ekspansi Target Server |
| 192.168.17.100 | Attacker Node |
| 192.168.17.101 – 192.168.17.199 | Reserved — ekspansi Red Team |
| 192.168.17.200 | Monitoring Node |
| 192.168.17.201 – 192.168.17.254 | Reserved — ekspansi Blue Team |
