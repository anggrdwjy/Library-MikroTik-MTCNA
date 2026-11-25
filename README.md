# MTCNA - Implementation and Operation Guide

## Modul 1 - Introduction - Implementasi Awal Router

### 1.1 Tujuan Implementasi

Pada akhir modul ini, router telah dapat diakses secara aman, memiliki identitas yang jelas, telah dicadangkan, dan telah dapat memberikan akses internet dasar ke LAN. Modul ini merupakan fondasi untuk seluruh modul berikutnya.

### 1.2 Langkah Implementasi

1. Lakukan akses pertama melalui WinBox MAC, kemudian periksa default configuration. Apabila router merupakan perangkat bekas, lakukan reset tanpa default config agar konfigurasi bersih.
2. Atur identitas, buat pengguna administrator baru, nonaktifkan pengguna administrator default setelah pengujian, dan atur layanan yang tidak diperlukan.
3. Atur WAN DHCP-Client, alamat IP LAN, DNS, dan NAT masquerade.
4. Lakukan pencadangan dan export konfigurasi, kemudian simpan di luar router.
5. Periksa versi RouterOS dan lakukan pemutakhiran apabila diperlukan, kemudian perbarui firmware RouterBOOT.

### 1.3 Contoh Konfigurasi Lengkap

```routeros
# Identitas dan waktu
/system identity set name=PUSAT-RB01
/system clock set time-zone-name=Asia/Jakarta

# Buat pengguna administrator baru dan nonaktifkan admin default setelah pengujian
/user add name=admin-op group=full password=Am4nSekali2026! comment="Operator utama"
/user disable admin

# Amankan layanan - hanya izinkan layanan yang digunakan
/ip service set telnet disabled=yes
/ip service set ftp disabled=yes
/ip service set www disabled=yes
/ip service set api disabled=yes
/ip service set ssh port=2222
/ip service set winbox port=8291
/ip service set api-ssl disabled=yes

# WAN - DHCP Client
/ip dhcp-client add interface=ether1 use-peer-dns=yes add-default-route=yes comment="WAN ke ISP"

# LAN - Alamat IP
/ip address add address=192.168.88.1/24 interface=bridge-lan comment="Gateway LAN Pusat"

# DNS
/ip dns set servers=8.8.8.8,1.1.1.1 allow-remote-requests=yes

# NAT dasar - LAN ke Internet
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT - LAN ke Internet"
```

### 1.4 Pencadangan, Export, dan Restore

Backup (.backup) menyimpan seluruh konfigurasi termasuk kata sandi dan sertifikat, dan hanya dapat dikembalikan ke perangkat yang sama atau model yang sama. Export (.rsc) merupakan berkas teks yang berisi perintah CLI, dapat dibaca, disunting, dan digunakan untuk dokumentasi atau migrasi.

```routeros
# Pencadangan biner - simpan sebelum perubahan besar
/system backup save name=RB01-sebelum-firewall-2026-11-24 dont-encrypt=no password=BackupAm4n!

# Export teks - untuk dokumentasi dan audit
/export file=RB01-config-2026-11-24

# Lihat berkas
/file print

# Restore backup (perlu kehati-hatian karena akan menimpa konfigurasi)
/system backup load name=RB01-sebelum-firewall-2026-11-24.backup

# Restore export - salin isi berkas .rsc ke Terminal
/import file-name=RB01-config-2026-11-24.rsc
```

### 1.5 Pemutakhiran RouterOS dan Firmware

```routeros
# Periksa versi dan update channel
/system package update check-for-updates
/system package update set channel=stable
/system package update download
/system package update install

# Perbarui firmware RouterBOOT setelah reboot
/system routerboard print
/system routerboard upgrade
/system reboot
```

### 1.6 Studi Kasus: Kantor Baru dengan 15 Pengguna

**Situasi:** Sebuah kantor baru di Surabaya baru saja memasang layanan ISP. Router RB hAP ac2 masih baru. Pemilik kantor meminta agar layanan internet dapat segera beroperasi pada hari yang sama, namun tetap aman dan dapat dicadangkan.

**Implementasi:** Teknisi melakukan reset router tanpa default config, kemudian membuat bridge-lan untuk ether2 sampai dengan ether5 dan wlan1. Teknisi mengatur alamat IP 192.168.88.1/24, DHCP-Client pada ether1, NAT masquerade, dan DNS. Teknisi membuat pengguna admin-op serta menonaktifkan telnet dan ftp. Setelah internet beroperasi, teknisi segera melakukan backup dan export, kemudian menyimpannya pada Google Drive kantor dan pada laptop teknisi.

**Hasil:** Sebanyak 15 PC langsung memperoleh akses internet melalui kabel dan WiFi. Pemilik kantor memiliki berkas backup dan export sebagai bukti serah terima. Apabila router mengalami kerusakan, teknisi dapat memulihkannya ke router pengganti dalam waktu 15 menit dengan melakukan import berkas .rsc dan penyesuaian MAC.

### 1.7 Verifikasi dan Operasi

- Verifikasi: Jalankan perintah `/ping 8.8.8.8` dari router, kemudian lakukan ping dari PC client. Periksa `/ip dhcp-client print`, status harus `bound`.
- Verifikasi: Periksa `/ip firewall nat print`, pastikan masquerade tersedia dan hit count bertambah pada saat client melakukan penelusuran.
- Operasi: Lakukan pencadangan mingguan secara otomatis dengan scheduler atau secara manual setiap terdapat perubahan besar. Simpan dua salinan, yaitu pada router dan di luar router.

> **Catatan Operasional Modul 1:** Silakan menuliskan komentar pada setiap aturan dan interface. Pada saat Anda membuat supout atau pada saat teknisi lain mengambil alih, komentar merupakan dokumentasi yang dapat menghemat waktu secara signifikan.

### 1.8 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| Struktur Paket | Paket terpisah, contoh system, wireless, ppp, dhcp | Paket utama terintegrasi ditambah paket opsional, contoh wifi, container, rose-storage | Pada v7, pemutakhiran lebih sederhana karena jumlah paket lebih sedikit |
| Perintah Pembaruan | /system package update | /system/package/update, sintaks lama tetap didukung | Periksa channel stable sebelum melakukan pemutakhiran |
| RouterBOOT | /system routerboard upgrade | Sama, tetapi v7 melakukan pemeriksaan kompatibilitas perangkat keras secara lebih ketat | Lakukan pencadangan sebelum memutakhirkan firmware |
| Konfigurasi Default | Default config lama dengan firewall dasar yang minimal | Default config baru dengan firewall dasar yang lebih ketat pada sebagian model | Periksa aturan default setelah reset sebelum menghubungkan ke internet |
| Upgrade Lintas Versi | Tidak relevan | Upgrade dari v6 ke v7 memerlukan pencadangan penuh, pemeriksaan paket wireless, dan pengujian laboratorium | Jangan melakukan upgrade langsung pada jaringan produksi tanpa uji coba |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - Konfigurasi Awal
/system identity set name=PUSAT-RB01
/ip service set telnet disabled=yes
/ip service set ftp disabled=yes
/ip dhcp-client add interface=ether1 use-peer-dns=yes add-default-route=yes
/ip address add address=192.168.88.1/24 interface=bridge-lan
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
/system backup save name=RB01-v6-sebelum-upgrade
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - Konfigurasi Awal
/system/identity set name=PUSAT-RB01
/ip/service set telnet disabled=yes
/ip/service set ftp disabled=yes
/ip/dhcp-client add interface=ether1 use-peer-dns=yes add-default-route=yes
/ip/address add address=192.168.88.1/24 interface=bridge-lan
/ip/firewall/nat add chain=srcnat out-interface=ether1 action=masquerade
/system/backup save name=RB01-v7-setelah-upgrade
# Periksa paket pada v7
/system/package print
```

> **Catatan:** Pada RouterOS v7, sintaks dengan garis miring, contoh /ip/address, dan sintaks tanpa garis miring, contoh /ip address, keduanya dapat digunakan. Panduan ini menggunakan sintaks tanpa garis miring agar konsisten dengan ujian MTCNA, sedangkan contoh v7 menampilkan sintaks dengan garis miring untuk menunjukkan format baru yang umum digunakan pada dokumentasi v7.


## Modul 2 - DHCP - Manajemen IP Otomatis

### 2.1 Tujuan Implementasi

Client tidak perlu mengatur alamat IP secara manual. Router membagikan alamat IP, gateway, DNS, dan lease time secara konsisten. Anda juga dapat mengendalikan perangkat mana yang memperoleh alamat IP dan berapa lama alamat IP tersebut dipinjam.

### 2.2 Konsep yang Perlu Dipahami

- **DHCP Server:** Membagikan alamat IP pada LAN. Terdiri atas Pool, Network, dan Server.
- **DHCP Client:** Meminta alamat IP dari ISP atau dari router atas. Umumnya diterapkan pada ether1 WAN.
- **Leases:** Daftar alamat IP yang sedang dipinjam oleh client. Dapat dibuat statis untuk perangkat penting seperti printer atau server.
- **ARP:** Tabel yang memetakan alamat IP ke MAC. Digunakan untuk melihat perangkat yang terhubung dan untuk mendeteksi konflik alamat IP.

### 2.3 Contoh Konfigurasi

```routeros
# 1. Pool IP untuk LAN
/ip pool add name=lan-pool ranges=192.168.88.10-192.168.88.100 comment="Pool LAN Pusat"

# 2. Network - gateway, DNS, dan informasi jaringan
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.1,8.8.8.8 domain=kantor.local comment="Network LAN"

# 3. DHCP Server pada bridge-lan
/ip dhcp-server add name=lan-dhcp interface=bridge-lan address-pool=lan-pool lease-time=1d authoritative=yes disabled=no comment="DHCP LAN Pusat"

# 4. DHCP Client pada WAN
/ip dhcp-client add interface=ether1 use-peer-dns=yes add-default-route=yes comment="WAN DHCP dari ISP"

# 5. Buat lease statis untuk printer dan server
/ip dhcp-server lease add address=192.168.88.10 mac-address=4C:CC:6A:AA:BB:CC server=lan-dhcp comment="Printer Kantor"
/ip dhcp-server lease add address=192.168.88.11 mac-address=4C:CC:6A:DD:EE:FF server=lan-dhcp comment="Server File"

# 6. Periksa ARP
/ip arp print
/ip dhcp-server lease print
```

### 2.4 Studi Kasus: Kantor dengan 50 Pengguna dan Perangkat Tetap

**Situasi:** Sebuah kantor memiliki 50 laptop, 3 printer, 1 server file, dan 5 CCTV. Sebelumnya, alamat IP sering mengalami konflik karena diatur secara manual.

**Implementasi:** Teknisi membuat pool 192.168.88.10-192.168.88.100 untuk laptop dengan lease-time 1 hari. Printer dan server dibuat lease statis di luar pool dinamis agar alamatnya tidak berubah. CCTV juga diberikan lease statis agar mudah dimonitor. Tabel ARP digunakan untuk audit mingguan. Apabila terdapat MAC yang tidak dikenal, teknisi melakukan pengecekan kepada pengguna.

**Hasil:** Konflik alamat IP tidak lagi terjadi. Printer tidak pernah berganti alamat IP sehingga driver tidak perlu diatur ulang. Apabila terdapat karyawan baru, karyawan tersebut cukup menghubungkan kabel dan langsung memperoleh alamat IP.

### 2.5 Verifikasi dan Troubleshooting

- Verifikasi: Dari PC client, jalankan `ipconfig /all`. Hasilnya harus menunjukkan gateway 192.168.88.1 dan DNS yang benar.
- Periksa leases: `/ip dhcp-server lease print` - status `bound` berarti client telah memperoleh alamat IP.
- Permasalahan umum: Client tidak memperoleh alamat IP. Periksa apakah DHCP server terpasang pada interface yang benar (bridge-lan, bukan hanya ether2). Periksa pula apakah terdapat DHCP server lain yang mengalami konflik.
- Permasalahan umum: Konflik alamat IP. Periksa ARP dan leases, kemudian cari MAC ganda. Nonaktifkan DHCP server liar pada modem ISP apabila terdapat.

```routeros
# Perintah diagnosis DHCP
/ip dhcp-server print
/ip dhcp-server lease print where status=bound
/ip arp print where interface=bridge-lan
/log print where topics~dhcp
```

### 2.6 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| DHCP Server IPv4 | /ip dhcp-server, /ip pool, /ip dhcp-server network | Sama, dengan peningkatan pada option sets dan skrip lease | Konfigurasi dasar IPv4 tidak berubah secara signifikan |
| DHCPv6 | /ipv6 dhcp-server dengan fitur terbatas | /ipv6/dhcp-server dengan manajemen pool dan binding yang lebih baik | Apabila Anda menggunakan IPv6, lakukan pengujian pada v7 terlebih dahulu |
| Lease Script | Didukung secara terbatas | Didukung dengan eksekusi yang lebih stabil dan variabel yang lebih lengkap | Gunakan lease script untuk notifikasi atau pencatatan otomatis |
| ARP | /ip arp dengan mode reply-only, enabled, proxy-arp | Sama, dengan integrasi yang lebih baik pada bridge VLAN filtering | Periksa tabel ARP setelah migrasi bridge |
| Performa | Memadai untuk jaringan kecil hingga menengah | Lebih efisien pada perangkat dengan CPU multicore | Tidak diperlukan perubahan desain untuk migrasi DHCP dasar |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - DHCP Server
/ip pool add name=lan-pool ranges=192.168.88.10-192.168.88.100
/ip dhcp-server network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.1,8.8.8.8
/ip dhcp-server add name=lan-dhcp interface=bridge-lan address-pool=lan-pool lease-time=1d
/ip dhcp-server lease add address=192.168.88.10 mac-address=4C:CC:6A:AA:BB:CC server=lan-dhcp comment="Printer Kantor"
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - DHCP Server
/ip/pool add name=lan-pool ranges=192.168.88.10-192.168.88.100
/ip/dhcp-server/network add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=192.168.88.1,8.8.8.8
/ip/dhcp-server add name=lan-dhcp interface=bridge-lan address-pool=lan-pool lease-time=1d
/ip/dhcp-server/lease add address=192.168.88.10 mac-address=4C:CC:6A:AA:BB:CC server=lan-dhcp comment="Printer Kantor"
# Fitur tambahan v7 - Option Sets untuk DHCP Option 43 atau 66 apabila diperlukan
/ip/dhcp-server/option/add name=option-tftp code=66 value="192.168.88.11"
```

> **Catatan:** Migrasi DHCP dari v6 ke v7 umumnya dapat dilakukan dengan melakukan export dan import tanpa perubahan besar. Periksa kembali lease statis setelah migrasi karena alamat MAC harus sesuai.


## Modul 3 - Bridging - Menggabungkan Interface

### 3.1 Tujuan Implementasi

Bridge membuat beberapa interface fisik (ethernet dan wireless) menjadi satu segmen jaringan yang sama, seolah-olah terhubung ke satu switch. Cara ini merupakan cara yang paling sederhana untuk membuat LAN yang rapi pada MikroTik.

### 3.2 Contoh Konfigurasi

```routeros
# Buat bridge utama
/interface bridge add name=bridge-lan comment="Bridge LAN Pusat" admin-mac=auto-mac=no auto-mac=yes

# Masukkan port ethernet
/interface bridge port add bridge=bridge-lan interface=ether2 comment="PC Admin"
/interface bridge port add bridge=bridge-lan interface=ether3 comment="PPPoE / Server"
/interface bridge port add bridge=bridge-lan interface=ether4 comment="AP Kantor"
/interface bridge port add bridge=bridge-lan interface=ether5 comment="Cadangan"

# Masukkan wireless ke bridge agar WiFi satu segmen dengan kabel
/interface bridge port add bridge=bridge-lan interface=wlan1 comment="WiFi Kantor"

# Pindahkan alamat IP LAN dari ether2 ke bridge (penting)
/ip address set [find address=192.168.88.1/24] interface=bridge-lan

# Periksa status bridge
/interface bridge print
/interface bridge port print
/interface bridge host print
```

### 3.3 Studi Kasus: Toko dengan Kabel dan WiFi dalam Satu Jaringan

**Situasi:** Sebuah toko retail memiliki kasir kabel, gudang WiFi, dan printer WiFi. Sebelumnya, WiFi berada pada subnet yang berbeda dengan kasir sehingga kasir tidak dapat mencetak ke printer WiFi.

**Implementasi:** Teknisi membuat bridge-lan, memasukkan ether2 untuk kasir dan wlan1 untuk WiFi ke bridge yang sama. Alamat IP 192.168.88.1/24 dipindahkan ke bridge. DHCP server juga dipindahkan ke bridge-lan. Setelah itu, kasir, printer, dan HP gudang seluruhnya berada pada 192.168.88.0/24 dan dapat saling melakukan ping.

**Hasil:** Printer dapat diakses dari kasir tanpa NAT tambahan. Manajemen menjadi satu dan tidak memerlukan routing antar subnet.

### 3.4 Verifikasi dan Tips Operasi

- Verifikasi: `/interface bridge host print` harus menunjukkan MAC client dari ether dan wlan pada bridge yang sama.
- Kesalahan umum: Alamat IP masih menempel pada ether2, bukan pada bridge. Akibatnya, client pada ether lain tidak memperoleh gateway. Pindahkan alamat IP ke bridge.
- Tips: Jangan memasukkan ether1 WAN ke bridge-lan. WAN harus tetap terpisah agar NAT dan firewall bekerja dengan benar.

> **Bridge dan DHCP:** Apabila DHCP server masih menempel pada ether2 setelah bridge dibuat, client pada ether3 tidak akan memperoleh alamat IP. Selalu arahkan DHCP server, alamat IP, dan firewall ke nama bridge, bukan ke port fisik.


### 3.5 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| Bridge Dasar | /interface bridge, /interface bridge port | Sama, dengan peningkatan hardware offload | Konfigurasi bridge dasar tetap sama |
| VLAN Filtering | Didukung, tetapi konfigurasi pada perangkat tertentu kurang stabil | Didukung secara lebih stabil dengan bridge VLAN table yang lebih jelas | Pada v7, gunakan bridge VLAN filtering untuk jaringan dengan banyak VLAN |
| Hardware Offload | Terbatas pada sebagian chip switch | Lebih luas dan lebih konsisten pada perangkat CRS dan RB baru | Periksa kolom H pada bridge port untuk memastikan offload aktif |
| STP / RSTP | STP, RSTP | STP, RSTP, MSTP yang lebih baik | Untuk jaringan dengan loop, aktifkan RSTP pada kedua versi |
| Fast Forward | Tersedia pada sebagian model | Tersedia dan lebih terprediksi | Nonaktifkan fast-forward apabila Anda menggunakan firewall pada bridge |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - Bridge
/interface bridge add name=bridge-lan
/interface bridge port add bridge=bridge-lan interface=ether2
/interface bridge port add bridge=bridge-lan interface=ether3
/interface bridge port add bridge=bridge-lan interface=wlan1
/ip address add address=192.168.88.1/24 interface=bridge-lan
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - Bridge dengan VLAN Filtering dan Hardware Offload
/interface/bridge add name=bridge-lan vlan-filtering=no comment="Aktifkan vlan-filtering=yes apabila menggunakan VLAN"
/interface/bridge/port add bridge=bridge-lan interface=ether2 hw=yes
/interface/bridge/port add bridge=bridge-lan interface=ether3 hw=yes
/interface/bridge/port add bridge=bridge-lan interface=wlan1
/ip/address add address=192.168.88.1/24 interface=bridge-lan
# Contoh VLAN pada v7 - lebih jelas dan terstruktur
/interface/bridge/vlan add bridge=bridge-lan tagged=bridge-lan,ether2 vlan-ids=10
```

> **Catatan:** Apabila Anda melakukan migrasi bridge dengan VLAN, lakukan pengujian pada laboratorium terlebih dahulu. Kesalahan pada bridge VLAN filtering dapat menyebabkan seluruh trafik VLAN terputus.


## Modul 4 - Routing - Menghubungkan Antar Jaringan

### 4.1 Tujuan Implementasi

Routing menentukan jalur paket dari satu network ke network lain. Dengan static routing, Anda dapat menghubungkan kantor pusat dan cabang, atau menghubungkan jaringan PPPoE ke LAN, tanpa harus mengandalkan routing dinamis.

### 4.2 Konsep Dasar

- **Route Flags:** D = Dynamic, A = Active, C = Connected, S = Static. Route yang aktif akan digunakan.
- **Default Route:** Jalur 0.0.0.0/0 yang digunakan apabila tidak terdapat route yang lebih spesifik. Umumnya mengarah ke ISP.
- **Static Route:** Route yang Anda buat secara manual untuk network tujuan tertentu.

### 4.3 Contoh Konfigurasi

```routeros
# Router Pusat - 192.168.88.0/24 dan link 192.168.10.0/30
# Link ke cabang melalui ether5 atau melalui tunnel, IP link 192.168.10.1/30
/ip address add address=192.168.10.1/30 interface=ether5 comment="Link ke Cabang"

# Route ke LAN Cabang 192.168.20.0/24 melalui gateway link
/ip route add dst-address=192.168.20.0/24 gateway=192.168.10.2 comment="Route ke Cabang" distance=1

# Default route ke ISP (umumnya telah diperoleh dari DHCP Client, tetapi dapat diatur manual)
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1 comment="Default ke ISP" distance=1

# Router Cabang - mirror
# /ip address add address=192.168.10.2/30 interface=ether1
# /ip address add address=192.168.20.1/24 interface=bridge-lan
# /ip route add dst-address=192.168.88.0/24 gateway=192.168.10.1 comment="Route ke Pusat"
# /ip route add dst-address=0.0.0.0/0 gateway=192.168.10.1 comment="Default via Pusat"

# Periksa routing
/ip route print
/ip route print where dst-address=192.168.20.0/24
/ping 192.168.20.1 src-address=192.168.88.1
/traceroute 192.168.20.10
```

### 4.4 Studi Kasus: Pusat dan Cabang Terhubung untuk Akses Server

**Situasi:** Kantor pusat di Surabaya memiliki server file 192.168.88.11. Cabang di Sidoarjo 192.168.20.0/24 memerlukan akses ke server tersebut setiap hari. Keduanya dihubungkan melalui link wireless PTP 192.168.10.0/30.

**Implementasi:** Pada router pusat ditambahkan route ke 192.168.20.0/24 melalui 192.168.10.2. Pada router cabang ditambahkan route balik ke 192.168.88.0/24 melalui 192.168.10.1. Firewall forward pada kedua sisi mengizinkan trafik antar LAN. NAT tidak dilakukan untuk trafik antar cabang agar alamat IP asli tetap terlihat.

**Hasil:** PC cabang dapat melakukan ping dan mengakses file server pusat dengan latensi 2-4 ms. Apabila link terputus, route menjadi tidak aktif (tidak terdapat flag A) dan log menunjukkan link down sehingga mudah didiagnosis.

### 4.5 Verifikasi dan Troubleshooting

- Verifikasi: `/ip route print` - route harus memiliki flag A dan S. Apabila tidak terdapat flag A, gateway tidak dapat di-ping.
- Verifikasi: `/tool traceroute` dari cabang ke server pusat harus melalui 192.168.10.1, bukan melalui internet.
- Permasalahan umum: Dapat melakukan ping ke gateway, tetapi tidak dapat melakukan ping ke LAN seberang. Periksa firewall forward dan pastikan route balik telah tersedia pada kedua sisi. Routing harus dua arah.
- Permasalahan umum: Route tertukar dengan default route. Pastikan distance static route lebih kecil atau lebih spesifik daripada default.


### 4.6 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| Mesin Routing | Mesin routing lama, terpisah untuk setiap protokol | Mesin routing baru yang ditulis ulang, terintegrasi dan lebih cepat | Migrasi routing dinamis memerlukan pengujian menyeluruh |
| Static Route | /ip route add dst-address gateway | /ip/route/add dst-address gateway routing-table=main, sintaks lama tetap didukung | Untuk static route dasar, tidak terdapat perubahan yang signifikan |
| Routing Table / VRF | VRF terbatas melalui routing-mark | /routing/table, /routing/rule, dan VRF yang lebih terstruktur | Gunakan routing table terpisah apabila Anda mengelola banyak VRF |
| Route Filter | /routing filter dengan sintaks lama | /routing/filter/rule dengan sintaks baru yang lebih ekspresif | Filter lama tidak dapat langsung disalin ke v7 tanpa penyesuaian |
| OSPF / BGP | OSPFv2, BGP dengan fitur terbatas | OSPFv2, OSPFv3, BGP dengan dukungan yang lebih lengkap, termasuk BFD dan ECMP yang lebih baik | Apabila Anda menggunakan OSPF atau BGP, rencanakan migrasi secara bertahap |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - Static Route
/ip address add address=192.168.10.1/30 interface=ether5 comment="Link ke Cabang"
/ip route add dst-address=192.168.20.0/24 gateway=192.168.10.2 comment="Route ke Cabang"
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1 comment="Default ke ISP"
/ip route print
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - Static Route dengan Routing Table
/ip/address add address=192.168.10.1/30 interface=ether5 comment="Link ke Cabang"
/ip/route/add dst-address=192.168.20.0/24 gateway=192.168.10.2 routing-table=main comment="Route ke Cabang"
/ip/route/add dst-address=0.0.0.0/0 gateway=192.168.1.1 routing-table=main comment="Default ke ISP"
# Contoh VRF pada v7 - hanya pada v7
/routing/table/add name=vrf-cabang fib
/routing/rule/add src-address=192.168.20.0/24 table=vrf-cabang action=lookup
/ip/route/print
```

> **Catatan:** Untuk static route sederhana, konfigurasi v6 dan v7 hampir identik. Perbedaan utama muncul pada saat Anda menggunakan OSPF, BGP, VRF, atau route filter. Dalam hal tersebut, Anda wajib melakukan pengujian pada laboratorium sebelum migrasi.


## Modul 5 - Wireless - AP, Station, dan Link Nirkabel

### 5.1 Tujuan Implementasi

Modul ini bertujuan untuk membangun jaringan wireless yang stabil dan aman, mulai dari Access Point untuk kantor, Station untuk client, hingga link point-to-point antar gedung.

### 5.2 Konsep Penting

- **Band dan Frekuensi:** 2.4 GHz memiliki jangkauan yang lebih jauh, tetapi lebih ramai. 5 GHz lebih bersih dan memiliki throughput yang lebih tinggi. Pilihlah channel yang paling sepi berdasarkan hasil scan.
- **Chains dan Data-Rates:** Chains menentukan MIMO. Jangan memaksakan data-rate tinggi apabila sinyal lemah. Biarkan pada mode otomatis atau atur secara manual dengan nilai yang realistis.
- **Tx Power dan Country:** Ikutilah regulasi negara yang berlaku. Tx power yang terlalu tinggi dapat menyebabkan interferensi dan justru menurunkan kualitas.

### 5.3 Contoh Konfigurasi AP Kantor

```routeros
# Security Profile WPA2-PSK
/interface wireless security-profiles add name=kantor-sec mode=dynamic-keys authentication-types=wpa2-psk wpa2-pre-shared-key=KantorAman2026! comment="Security Kantor"

# AP 2.4 GHz untuk kantor
/interface wireless set wlan1 mode=ap-bridge band=2ghz-b/g/n channel-width=20/40mhz-Ce frequency=2437 ssid=KANTOR-PUSAT security-profile=kantor-sec disabled=no comment="AP Kantor"

# Masukkan wlan1 ke bridge agar satu segmen dengan kabel
/interface bridge port add bridge=bridge-lan interface=wlan1

# Access List - batasi MAC yang boleh terhubung (opsional, untuk perangkat tetap)
/interface wireless access-list add mac-address=AA:BB:CC:11:22:33 interface=wlan1 comment="Laptop Manager" authentication=yes forwarding=yes

# Pengaturan default
/interface wireless set wlan1 default-authenticate=yes default-forwarding=yes
```

### 5.4 Contoh Konfigurasi Station dan Link PTP

```routeros
# Sisi Station - pada Cabang atau Client PTP
# Pada router station, wlan1 sebagai station-bridge agar bridge pada kedua sisi menyatu
/interface wireless set wlan1 mode=station-bridge band=5ghz-a/n/ac channel-width=20/40/80mhz-XXXX frequency=auto ssid=LINK-PUSAT-CABANG security-profile=ptp-sec disabled=no

# Pada sisi AP PTP (Pusat)
/interface wireless set wlan1 mode=ap-bridge band=5ghz-a/n/ac channel-width=20/40/80mhz-XXXX frequency=5180 ssid=LINK-PUSAT-CABANG security-profile=ptp-sec disabled=no

# Security khusus PTP - kata sandi berbeda dengan AP kantor
/interface wireless security-profiles add name=ptp-sec mode=dynamic-keys authentication-types=wpa2-psk wpa2-pre-shared-key=PTPAman2026! comment="Security PTP"

# Monitoring
/interface wireless registration-table print
/interface wireless snooper snoop wlan1
/interface wireless monitor wlan1
```

### 5.5 Studi Kasus: Link PTP 2 KM dan Hotspot Kantor

**Situasi:** Gudang yang berjarak 2 KM dari kantor pusat tidak memiliki kabel fiber. Gudang tersebut memerlukan internet 30 Mbps dan akses ke server pusat. Di dalam kantor juga diperlukan WiFi untuk 20 karyawan.

**Implementasi:** Teknisi memasang 2 unit SXT 5 GHz. Satu unit sebagai AP-Bridge pada kantor dengan SSID LINK-PUSAT-CABANG, dan satu unit sebagai Station-Bridge pada gudang. Frekuensi dipilih setelah melakukan scan Snooper, yaitu frekuensi yang paling sepi pada 5180. Keamanan menggunakan WPA2-PSK. Pada gudang, ether1 dari SXT masuk ke bridge-lan gudang sehingga PC gudang berada dalam satu segmen dengan LAN gudang. Pada kantor, AP 2.4 GHz terpisah untuk karyawan dengan SSID KANTOR-PUSAT.

**Hasil:** Link PTP stabil dengan signal -55 dBm, CCQ 95 persen, dan throughput 40 Mbps. Karyawan gudang dapat mengakses server pusat melalui static route. WiFi kantor tidak terganggu karena menggunakan frekuensi yang berbeda dengan PTP.

### 5.6 Verifikasi dan Operasi

- Verifikasi: `/interface wireless registration-table print` - periksa signal strength, Tx/Rx rate, dan uptime.
- Verifikasi: `/interface wireless monitor` - periksa CCQ di atas 80 persen untuk link yang sehat.
- Tips: Lakukan scan berkala dengan Snooper apabila terdapat keluhan lambat. Channel yang ramai dapat menurunkan throughput secara signifikan.
- Tips: Jangan menyamakan SSID PTP dengan SSID kantor agar client tidak salah terhubung.

> **Keamanan Wireless:** Selalu gunakan WPA2-PSK minimal, jangan menggunakan WEP atau mode terbuka. Gantilah kata sandi default, nonaktifkan WPS apabila tidak digunakan, dan pisahkan kata sandi AP kantor dengan kata sandi PTP.


### 5.7 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| Paket Wireless | Paket wireless, /interface wireless | Paket wifi atau wifiwave2, /interface wifi untuk perangkat AX | Perangkat lama tetap menggunakan wireless, perangkat AX menggunakan wifi |
| Perintah Konfigurasi | /interface wireless set, /interface wireless security-profiles | /interface/wifi set, /interface/wifi/security, /interface/wifi/channel, /interface/wifi/datapath | Konfigurasi tidak dapat disalin langsung, perlu penyesuaian sintaks |
| CAPsMAN | CAPsMAN v1, /caps-man | CAPsMAN v2, /interface/wifi/capsman | CAPsMAN v2 hanya tersedia pada v7 dan lebih terstruktur |
| Standar WiFi | Hingga 802.11ac (WiFi 5) pada sebagian besar perangkat | Hingga 802.11ax (WiFi 6) pada perangkat AX, dengan dukungan WPA3 | Gunakan v7 apabila Anda memerlukan WiFi 6 atau WPA3 |
| Keamanan | WPA, WPA2-PSK | WPA2-PSK, WPA3-PSK, OWE, dan SAE | Untuk keamanan tertinggi pada v7, gunakan WPA3 apabila client mendukung |
| Monitoring | /interface wireless registration-table, snooper | /interface/wifi/registration-table, /interface/wifi/monitor | Nama perintah berubah, tetapi konsep monitoring tetap sama |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - AP dengan Wireless Package
/interface wireless security-profiles add name=kantor-sec mode=dynamic-keys authentication-types=wpa2-psk wpa2-pre-shared-key=KantorAman2026!
/interface wireless set wlan1 mode=ap-bridge band=2ghz-b/g/n channel-width=20/40mhz-Ce frequency=2437 ssid=KANTOR-PUSAT security-profile=kantor-sec disabled=no
/interface bridge port add bridge=bridge-lan interface=wlan1
/interface wireless registration-table print
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - AP dengan Wifi Package (Perangkat AX, contoh hAP ax3)
/interface/wifi/security/add name=kantor-sec authentication-types=wpa2-psk,wpa3-psk passphrase=KantorAman2026! comment="Security Kantor v7"
/interface/wifi/channel/add name=ch-24 frequency=2437 width=20/40mhz
/interface/wifi/datapath/add name=dp-lan bridge=bridge-lan
/interface/wifi/configuration/add name=cfg-kantor ssid=KANTOR-PUSAT security=kantor-sec channel=ch-24 datapath=dp-lan
/interface/wifi/set wifi1 configuration=cfg-kantor disabled=no
/interface/wifi/registration-table print
# Untuk perangkat lama pada v7 yang masih menggunakan wireless, perintah v6 tetap berlaku
# /interface/wireless print
```

> **Catatan:** Perbedaan ini merupakan perbedaan yang paling signifikan antara v6 dan v7. Apabila Anda memigrasikan jaringan wireless, Anda harus menentukan terlebih dahulu apakah perangkat Anda menggunakan paket wireless atau wifi, kemudian menyusun ulang konfigurasi sesuai dengan paket tersebut.


## Modul 6 - Firewall - Proteksi, NAT, dan FastTrack

### 6.1 Tujuan Implementasi

Firewall merupakan modul yang paling penting untuk operasi. Tujuannya adalah melindungi router dari serangan, melindungi client dari trafik yang tidak diinginkan, melakukan NAT agar LAN dapat mengakses internet, dan melakukan port forwarding untuk server yang perlu diakses dari luar.

### 6.2 Konsep Dasar

- **Connection Tracking dan States:** Established = koneksi yang telah terbentuk, Related = koneksi turunan yang sah, New = koneksi baru, Invalid = paket yang rusak atau tidak sah.
- **Chains:** Input = trafik ke router itu sendiri, Forward = trafik yang melewati router (client ke internet atau antar LAN), Output = trafik dari router ke luar.
- **Urutan Aturan Penting:** Aturan dibaca dari atas ke bawah. Aturan accept established harus berada di atas, aturan drop harus berada di bawah. Aturan yang salah urutan dapat memblokir seluruh trafik.

### 6.3 Contoh Konfigurasi Firewall Filter

```routeros
# === CHAIN INPUT - Lindungi Router ===
# Terima koneksi yang telah terbentuk
/ip firewall filter add chain=input connection-state=established,related action=accept comment="Input - Accept Established"

# Drop paket invalid
/ip firewall filter add chain=input connection-state=invalid action=drop comment="Input - Drop Invalid"

# Terima ICMP ping dari LAN (opsional, untuk diagnosis)
/ip firewall filter add chain=input protocol=icmp src-address=192.168.88.0/24 action=accept comment="Input - Allow Ping LAN"

# Terima WinBox dan SSH hanya dari IP Admin
/ip firewall filter add chain=input protocol=tcp dst-port=8291 src-address=192.168.88.20 action=accept comment="Input - Allow WinBox Admin"
/ip firewall filter add chain=input protocol=tcp dst-port=2222 src-address=192.168.88.20 action=accept comment="Input - Allow SSH Admin"

# Drop sisanya yang menuju router
/ip firewall filter add chain=input action=drop comment="Input - Drop All Lainnya"

# === CHAIN FORWARD - Lindungi Client ===
# FastTrack untuk trafik yang telah established (letakkan di atas)
/ip firewall filter add chain=forward connection-state=established,related action=fasttrack-connection comment="Forward - FastTrack"

# Terima established dan related (untuk yang tidak di-FastTrack)
/ip firewall filter add chain=forward connection-state=established,related action=accept comment="Forward - Accept Established"

# Drop invalid
/ip firewall filter add chain=forward connection-state=invalid action=drop comment="Forward - Drop Invalid"

# Izinkan trafik dari LAN menuju Cabang
/ip firewall filter add chain=forward src-address=192.168.88.0/24 dst-address=192.168.20.0/24 action=accept comment="Forward - Allow ke Cabang"
```

### 6.4 NAT, Address-List, dan Port Forwarding

```routeros
# Source NAT - LAN ke Internet
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT - LAN ke Internet"

# Source NAT spesifik - apabila memiliki IP publik statis, gunakan src-nat
# /ip firewall nat add chain=srcnat out-interface=ether1 src-address=192.168.88.0/24 action=src-nat to-addresses=103.10.20.5

# Destination NAT - Port Forwarding ke Server Web Lokal 192.168.88.10 port 80
/ip firewall nat add chain=dstnat dst-port=80 protocol=tcp in-interface=ether1 action=dst-nat to-addresses=192.168.88.10 to-ports=80 comment="NAT - Forward Web Server"

# Redirect - Arahkan DNS client ke DNS router
/ip firewall nat add chain=dstnat dst-port=53 protocol=udp src-address=192.168.88.0/24 action=redirect to-ports=53 comment="NAT - Redirect DNS"

# Address-List - Daftar blokir
/ip firewall address-list add list=BLOKIR-SITUS address=203.0.113.10 comment="Situs diblokir"
/ip firewall filter add chain=forward dst-address-list=BLOKIR-SITUS action=drop comment="Forward - Drop ke BLOKIR-SITUS"

# Address-List - Daftar IP Admin
/ip firewall address-list add list=ADMIN address=192.168.88.20 comment="IP Admin"
```

### 6.5 Studi Kasus: Mengamankan Router ISP Kecil dan Membuka Akses CCTV

**Situasi:** Sebuah ISP kecil memiliki 30 client PPPoE. Router sering mengalami serangan brute force WinBox dari internet. Di sisi lain, pemilik toko meminta agar CCTV pada 192.168.88.15 dapat dilihat dari HP pada saat berada di luar.

**Implementasi:** Teknisi membuat filter input yang hanya mengizinkan WinBox dari alamat IP administrator dan dari alamat IP kantor, selebihnya drop. Pada chain forward, teknisi mengaktifkan FastTrack agar throughput tidak menurun pada saat trafik tinggi. Teknisi membuat DST-NAT untuk CCTV, yaitu port 8000 dari internet di-forward ke 192.168.88.15 port 8000, tetapi hanya untuk alamat IP yang dikenal, selebihnya drop. Teknisi membuat Address-List untuk alamat IP yang mencoba brute force lebih dari 5 kali, kemudian drop secara otomatis.

**Hasil:** Serangan brute force menurun secara signifikan karena port WinBox tidak lagi terbuka ke internet. CCTV dapat diakses oleh pemilik dari HP tanpa membuka akses kepada semua orang. Throughput tetap stabil karena FastTrack.

### 6.6 Verifikasi dan Operasi

- Verifikasi: `/ip firewall filter print stats` - perhatikan packet dan byte count. Aturan yang tidak pernah hit berarti kemungkinan salah urutan atau tidak diperlukan.
- Verifikasi: Cobalah mengakses WinBox dari alamat IP bukan administrator, akses harus gagal. Cobalah dari alamat IP administrator, akses harus berhasil.
- Verifikasi NAT: Dari client, penelusuran harus berjalan. Dari luar, akses ke alamat IP publik port 80 harus sampai ke server lokal.
- Operasi: Periksa log firewall secara mingguan. Apabila terdapat alamat IP yang berulang kali mencoba login, masukkan ke Address-List blokir.

> **FastTrack dan Queue:** FastTrack akan melewati firewall filter, mangle, dan queue untuk koneksi yang di-FastTrack. Apabila Anda menggunakan Simple Queue yang memerlukan mangle, jangan FastTrack trafik yang di-queue, atau letakkan aturan FastTrack setelah aturan yang mengecualikan trafik tersebut.


### 6.7 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| Firewall Filter | /ip firewall filter, chain input, forward, output | Sama, dengan peningkatan performa connection tracking | Aturan dasar dapat dimigrasikan langsung |
| Connection Tracking | Terbatas pada CPU single-core untuk sebagian proses | Lebih efisien pada CPU multicore dan perangkat dengan hardware offload | Periksa beban CPU setelah migrasi |
| FastTrack | Didukung, /ip firewall filter action=fasttrack-connection | Didukung, dengan dukungan hardware offload yang lebih luas | FastTrack tetap melewati queue dan mangle |
| NAT | /ip firewall nat, masquerade, src-nat, dst-nat | Sama, ditambah NAT dengan hairpin yang lebih stabil | Konfigurasi NAT dasar tidak berubah |
| Mangle dan Marking | /ip firewall mangle | Sama, dengan penambahan matcher baru | Periksa kembali mangle apabila Anda menggunakan policy routing |
| IPv6 Firewall | /ipv6 firewall filter, nat terbatas | /ipv6/firewall/filter dengan fitur yang lebih lengkap | Apabila Anda menggunakan IPv6, lakukan audit aturan pada v7 |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - Firewall Dasar
/ip firewall filter add chain=input connection-state=established,related action=accept comment="Input - Accept Established"
/ip firewall filter add chain=input connection-state=invalid action=drop comment="Input - Drop Invalid"
/ip firewall filter add chain=input protocol=tcp dst-port=8291 src-address=192.168.88.20 action=accept comment="Input - Allow WinBox Admin"
/ip firewall filter add chain=input action=drop comment="Input - Drop All"
/ip firewall filter add chain=forward connection-state=established,related action=fasttrack-connection comment="Forward - FastTrack"
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT - LAN ke Internet"
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - Firewall Dasar (sintaks setara)
/ip/firewall/filter/add chain=input connection-state=established,related action=accept comment="Input - Accept Established"
/ip/firewall/filter/add chain=input connection-state=invalid action=drop comment="Input - Drop Invalid"
/ip/firewall/filter/add chain=input protocol=tcp dst-port=8291 src-address=192.168.88.20 action=accept comment="Input - Allow WinBox Admin"
/ip/firewall/filter/add chain=input action=drop comment="Input - Drop All"
/ip/firewall/filter/add chain=forward connection-state=established,related action=fasttrack-connection comment="Forward - FastTrack"
/ip/firewall/nat/add chain=srcnat out-interface=ether1 action=masquerade comment="NAT - LAN ke Internet"
# Tambahan pada v7 - Periksa connection tracking
/ip/firewall/connection-tracking/print
```

> **Catatan:** Migrasi firewall dari v6 ke v7 umumnya berjalan dengan lancar. Yang perlu diperhatikan adalah urutan aturan dan interaksi FastTrack dengan queue. Lakukan pengujian akses WinBox dari IP administrator dan dari IP bukan administrator setelah migrasi.


## Modul 7 - QoS - Pembagian Bandwidth yang Adil

### 7.1 Tujuan Implementasi

Tanpa QoS, satu pengguna yang melakukan unduhan besar dapat menyebabkan seluruh pengguna lain mengalami lambat. QoS memastikan setiap pengguna memperoleh jatah yang adil dan layanan penting tetap memperoleh prioritas.

### 7.2 Konsep Dasar

- **Simple Queue:** Queue per pengguna atau per segmen. Pengaturan meliputi target, max-limit, limit-at, dan bursting.
- **Max-Limit:** Batas maksimal bandwidth yang boleh digunakan. **Limit-At:** Jaminan minimal bandwidth.
- **Bursting:** Memberikan kecepatan ekstra sementara apabila jaringan sedang sepi, agar pengguna merasakan kecepatan yang baik, tetapi tidak merugikan pengguna lain.
- **PCQ:** Per Connection Queue. Satu queue untuk seluruh network, tetapi dibagi rata per pengguna secara otomatis. Sangat efisien untuk ISP kecil.

### 7.3 Contoh Konfigurasi Simple Queue dan PCQ

```routeros
# Simple Queue - Per Pengguna
# Pengguna VIP 192.168.88.20 - 10 Mbps download, 5 Mbps upload, burst 15 Mbps
/queue simple add name=VIP-Manager target=192.168.88.20/32 max-limit=5M/10M limit-at=2M/5M burst-limit=8M/15M burst-threshold=3M/8M burst-time=10s/10s comment="Queue VIP"

# Pengguna biasa 192.168.88.30 - 2 Mbps download, 1 Mbps upload
/queue simple add name=User-30 target=192.168.88.30/32 max-limit=1M/2M limit-at=512k/1M comment="Queue Pengguna Biasa"

# === PCQ - Satu Queue untuk Semua LAN ===
# Buat type PCQ Upload dan Download
/queue type add name=pcq-upload kind=pcq pcq-rate=1M pcq-classifier=src-address pcq-limit=50 comment="PCQ Upload 1M per pengguna"
/queue type add name=pcq-download kind=pcq pcq-rate=3M pcq-classifier=dst-address pcq-limit=50 comment="PCQ Download 3M per pengguna"

# Terapkan ke seluruh LAN - total 50 Mbps, dibagi rata per pengguna melalui PCQ
/queue simple add name=PCQ-LAN target=192.168.88.0/24 max-limit=25M/50M queue=pcq-upload/pcq-download comment="PCQ untuk LAN"

# Periksa queue
/queue simple print stats
/queue simple monitor PCQ-LAN
```

### 7.4 Studi Kasus: Warnet dan Kantor dengan 40 Pengguna

**Situasi:** Sebuah warnet dengan 20 PC dan sebuah kantor dengan 20 laptop berbagi internet 50 Mbps. Pada sore hari, pada saat seluruh pengguna mengunduh game, kantor tidak dapat melakukan video call. Pemilik meminta agar kantor diprioritaskan.

**Implementasi:** Teknisi membuat PCQ untuk warnet, yaitu tiap PC memperoleh 3 Mbps download dan 1 Mbps upload. Untuk kantor, teknisi membuat Simple Queue per departemen dengan limit-at yang lebih tinggi agar pada saat jaringan sibuk, kantor tetap memperoleh jaminan 5 Mbps. Bursting diaktifkan untuk kantor agar pada saat jaringan sepi, kantor dapat memperoleh 15 Mbps untuk mengunduh berkas besar.

**Hasil:** Video call kantor tidak lagi terputus-putus. Warnet tetap berjalan dengan adil dan tidak terdapat satu PC yang menghabiskan bandwidth. Pemilik dapat melihat statistik queue sebagai bukti kepada pelanggan.

### 7.5 Verifikasi dan Operasi

- Verifikasi: Lakukan speedtest dari PC yang di-queue. Hasilnya harus mendekati max-limit yang diatur.
- Verifikasi: `/queue simple print stats` - perhatikan rate dan packet. Apabila rate selalu 0, queue kemungkinan salah target atau tertukar antara upload dan download.
- Catatan: Urutan upload/download pada RouterOS adalah upload/download. Contoh 1M/3M berarti 1M upload dan 3M download. Jangan tertukar.
- Catatan: Jangan membuat queue yang total max-limit-nya melebihi bandwidth ISP secara berlebihan tanpa limit-at karena akan terjadi perebutan dan tidak adil.


### 7.6 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| Simple Queue | /queue simple dengan PCQ, FIFO, SFQ | Sama, ditambah jenis queue baru | Konfigurasi Simple Queue dasar dapat dimigrasikan langsung |
| PCQ | PCQ dengan pcq-rate dan pcq-classifier | Sama, dengan performa yang lebih baik pada beban tinggi | PCQ tetap menjadi pilihan utama untuk pembagian adil |
| Queue Tree | /queue tree dengan parent dan packet-mark | Sama, tetapi dengan integrasi yang lebih baik pada mangle v7 | Periksa kembali packet-mark setelah migrasi |
| Jenis Queue Baru | FIFO, SFQ, RED, PCQ | Ditambah CAKE dan FQ-CoDel untuk mengatasi bufferbloat | Gunakan CAKE pada v7 apabila Anda mengalami latensi tinggi pada saat trafik padat |
| Monitoring Queue | /queue simple print stats | Sama, dengan tampilan statistik yang lebih detail | Periksa rate dan queued bytes secara berkala |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - PCQ
/queue type add name=pcq-upload kind=pcq pcq-rate=1M pcq-classifier=src-address
/queue type add name=pcq-download kind=pcq pcq-rate=3M pcq-classifier=dst-address
/queue simple add name=PCQ-LAN target=192.168.88.0/24 max-limit=25M/50M queue=pcq-upload/pcq-download comment="PCQ untuk LAN"
/queue simple print stats
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - PCQ (sama) dan CAKE (baru)
/queue/type/add name=pcq-upload kind=pcq pcq-rate=1M pcq-classifier=src-address
/queue/type/add name=pcq-download kind=pcq pcq-rate=3M pcq-classifier=dst-address
/queue/simple/add name=PCQ-LAN target=192.168.88.0/24 max-limit=25M/50M queue=pcq-upload/pcq-download comment="PCQ untuk LAN"
# Contoh CAKE pada v7 - untuk antarmuka WAN agar latensi tetap rendah
/queue/type/add name=cake-wan kind=cake cake-bandwidth=50M cake-diffserv=diffserv4
/queue/simple/add name=CAKE-WAN target=ether1 max-limit=50M/50M queue=cake-wan/cake-wan comment="CAKE untuk WAN"
/queue/simple/print stats
```

> **Catatan:** Apabila Anda tidak mengalami bufferbloat, PCQ saja sudah memadai. Gunakan CAKE pada v7 hanya apabila Anda telah mengukur latensi dan memastikan bahwa bufferbloat merupakan penyebab utama.


## Modul 8 - Tunnels - PPPoE, PPTP, dan SSTP

### 8.1 Tujuan Implementasi

Tunnel digunakan untuk dua hal, yaitu memberikan layanan internet berbasis login seperti ISP (PPPoE), dan menghubungkan jaringan jarak jauh secara aman melalui internet (VPN).

### 8.2 Konsep Dasar

- **PPPoE:** Client login dengan username dan kata sandi untuk memperoleh alamat IP dan akses internet. Sangat umum digunakan oleh ISP rumahan.
- **PPP Profile:** Template yang menentukan alamat IP lokal, DNS, rate-limit, dan pengaturan lain untuk pengguna PPP.
- **IP Pool:** Rentang alamat IP yang dibagikan kepada client PPPoE atau VPN.
- **PPTP / SSTP:** Protokol VPN untuk menghubungkan cabang atau pekerja jarak jauh ke kantor pusat melalui tunnel terenkripsi.

### 8.3 Contoh Konfigurasi PPPoE Server

```routeros
# 1. IP Pool untuk PPPoE
/ip pool add name=pppoe-pool ranges=10.10.10.10-10.10.10.100 comment="Pool PPPoE Client"

# 2. PPP Profile
/ppp profile add name=pppoe-profile local-address=10.10.10.1 remote-address=pppoe-pool dns-server=10.10.10.1,8.8.8.8 rate-limit=2M/5M comment="Profile PPPoE 5M" only-one=no

# 3. Secret - Username dan Kata Sandi Client
/ppp secret add name=client-budi password=BudiAman123 service=pppoe profile=pppoe-profile comment="Client Budi - Paket 5M" disabled=no
/ppp secret add name=client-sari password=SariAman123 service=pppoe profile=pppoe-profile comment="Client Sari - Paket 5M"

# 4. PPPoE Server pada ether3
/interface pppoe-server server add service-name=ISP-SURABAYA interface=ether3 default-profile=pppoe-profile authentication=pap,chap,mschap2 disabled=no comment="PPPoE Server ISP"

# 5. NAT untuk PPPoE - agar client PPPoE dapat mengakses internet
/ip firewall nat add chain=srcnat src-address=10.10.10.0/24 out-interface=ether1 action=masquerade comment="NAT PPPoE ke Internet"

# 6. Periksa status
/ppp active print
/interface pppoe-server print
```

### 8.4 Contoh Konfigurasi PPPoE Client dan VPN

```routeros
# PPPoE Client - pada Router Client / CPE
/interface pppoe-client add name=pppoe-isp interface=ether1 user=client-budi password=BudiAman123 add-default-route=yes use-peer-dns=yes disabled=no comment="PPPoE Client ke ISP"

# PPTP Server - untuk VPN Cabang
/interface pptp-server server set enabled=yes default-profile=default-encryption authentication=mschap2
/ppp secret add name=vpn-cabang password=VpnAman123 service=pptp profile=default-encryption local-address=192.168.99.1 remote-address=192.168.99.10 comment="VPN Cabang"

# PPTP Client - pada Router Cabang
/interface pptp-client add name=pptp-pusat connect-to=103.10.20.5 user=vpn-cabang password=VpnAman123 profile=default-encryption add-default-route=no disabled=no comment="PPTP ke Pusat"

# SSTP Client - alternatif yang lebih aman, melalui port 443
/interface sstp-client add name=sstp-pusat connect-to=103.10.20.5 user=vpn-cabang password=VpnAman123 profile=default-encryption add-default-route=no disabled=no comment="SSTP ke Pusat"

# Setelah tunnel aktif, buat route melalui tunnel
# /ip route add dst-address=192.168.20.0/24 gateway=pptp-pusat comment="Route via VPN"
```

### 8.5 Studi Kasus: ISP Rumahan dengan 30 Client dan VPN Cabang

**Situasi:** Sebuah ISP rumahan pada sebuah perumahan memiliki 30 client. Sebelumnya, ISP tersebut menggunakan IP statis sehingga sering terdapat client yang belum membayar, tetapi tetap dapat mengakses internet. Cabang di kota lain memerlukan akses ke server billing di pusat.

**Implementasi:** Teknisi memasang PPPoE Server pada ether3. Setiap client dibuatkan secret dengan kata sandi masing-masing dan rate-limit 5 Mbps. Apabila client belum membayar, secret dinonaktifkan sehingga client tidak dapat login. Untuk cabang, dibuat PPTP tunnel ke pusat. Setelah tunnel aktif, dibuat static route melalui tunnel agar cabang dapat mengakses server billing 192.168.88.11 tanpa melalui internet terbuka.

**Hasil:** Penagihan menjadi mudah karena client yang belum membayar otomatis terputus. Client tidak dapat berbagi alamat IP karena login terikat pada username. Cabang dapat mengakses billing dengan aman. Administrator dapat melihat siapa yang sedang online melalui `/ppp active print`.

### 8.6 Verifikasi dan Operasi

- Verifikasi PPPoE: `/ppp active print` harus menunjukkan client yang login, beserta uptime dan alamat IP.
- Verifikasi VPN: `/interface pptp-client print` status harus `connected`, dan `/ping` ke alamat IP tunnel harus berhasil.
- Catatan: Selalu atur authentication ke pap,chap,mschap2 dan gunakan kata sandi yang kuat. Jangan menggunakan kata sandi yang sama untuk seluruh client.
- Catatan: Untuk PPPoE, jangan lupa NAT untuk subnet PPPoE. Apabila tidak terdapat NAT, client dapat login, tetapi tidak dapat melakukan penelusuran.


### 8.7 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| PPPoE | /interface pppoe-server, /ppp secret, /ppp profile | Sama, dengan peningkatan stabilitas dan logging | Konfigurasi PPPoE dasar tidak berubah |
| PPTP | Didukung secara penuh | Didukung, tetapi dianggap usang dan kurang aman | Hindari PPTP untuk jaringan baru, gunakan SSTP atau WireGuard |
| SSTP | Didukung | Didukung dengan peningkatan performa TLS | SSTP tetap menjadi pilihan apabila client hanya mendukung SSTP |
| OpenVPN | Didukung, dengan fitur terbatas | Didukung dengan peningkatan, termasuk UDP dan sertifikat yang lebih baik | Periksa kembali sertifikat setelah migrasi |
| WireGuard | Tidak tersedia | Tersedia secara native sejak v7.1, /interface wireguard | WireGuard merupakan pilihan utama untuk VPN baru pada v7 karena cepat dan sederhana |
| L2TP / IPsec | Didukung | Didukung dengan peningkatan IPsec dan IKEv2 | Untuk site-to-site yang aman, pertimbangkan IPsec atau WireGuard |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - PPPoE dan PPTP
/ip pool add name=pppoe-pool ranges=10.10.10.10-10.10.10.100
/ppp profile add name=pppoe-profile local-address=10.10.10.1 remote-address=pppoe-pool dns-server=10.10.10.1,8.8.8.8 rate-limit=2M/5M
/ppp secret add name=client-budi password=BudiAman123 service=pppoe profile=pppoe-profile
/interface pppoe-server server add service-name=ISP-SURABAYA interface=ether3 default-profile=pppoe-profile authentication=pap,chap,mschap2
# PPTP pada v6
/interface pptp-server server set enabled=yes default-profile=default-encryption authentication=mschap2
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - PPPoE (sama) dan WireGuard (baru)
/ip/pool/add name=pppoe-pool ranges=10.10.10.10-10.10.10.100
/ppp/profile/add name=pppoe-profile local-address=10.10.10.1 remote-address=pppoe-pool dns-server=10.10.10.1,8.8.8.8 rate-limit=2M/5M
/ppp/secret/add name=client-budi password=BudiAman123 service=pppoe profile=pppoe-profile
/interface/pppoe-server/server/add service-name=ISP-SURABAYA interface=ether3 default-profile=pppoe-profile authentication=pap,chap,mschap2
# WireGuard pada v7 - pengganti PPTP untuk VPN baru
/interface/wireguard/add name=wg-cabang listen-port=13231 private-key="PRIVATE-KEY-PUSAT" comment="WireGuard Pusat"
/interface/wireguard/peers/add interface=wg-cabang public-key="PUBLIC-KEY-CABANG" allowed-address=192.168.20.0/24 endpoint-address=103.10.20.6 endpoint-port=13231
/ip/address/add address=192.168.99.1/30 interface=wg-cabang
/ip/route/add dst-address=192.168.20.0/24 gateway=wg-cabang
```

> **Catatan:** Apabila Anda membangun VPN baru pada RouterOS v7, gunakan WireGuard sebagai pilihan utama. Gunakan SSTP atau IPsec hanya apabila perangkat lawan tidak mendukung WireGuard. PPTP tidak disarankan untuk jaringan baru karena alasan keamanan.

## Modul 9 - Misc - Tools, Monitoring, dan Troubleshooting

### 9.1 Tujuan Implementasi

Router yang baik bukan hanya router yang dapat dikonfigurasi, tetapi juga router yang dapat dipantau dan diperbaiki dengan cepat pada saat terdapat permasalahan. Modul ini merupakan SOP harian teknisi.

### 9.2 Tools yang Wajib Dikuasai

```routeros
# Ping dan Traceroute - Periksa konektivitas
/ping 8.8.8.8 count=10
/ping 192.168.88.20 src-address=192.168.88.1
/tool traceroute 192.168.20.10

# Torch - Lihat trafik real-time per IP dan port
/tool torch interface=bridge-lan src-address=192.168.88.0/24

# Monitor Traffic - Lihat throughput interface
/interface monitor-traffic ether1,bridge-lan

# Profiler - Periksa beban CPU dan cari proses yang berat
/tool profile

# Netwatch - Pantau host dan kirim notifikasi apabila down
/tool netwatch add host=8.8.8.8 interval=30s up-script="/log info \"Internet UP\"" down-script="/log warning \"Internet DOWN\""

# E-mail - Kirim backup atau log melalui email
/tool e-mail set address=mail.kantor.com from=router@kantor.com user=router password=EmailPass server=mail.kantor.com port=587 start-tls=yes
/tool e-mail send to=admin@kantor.com subject="Backup RB01" file=RB01-config-2026-11-24.rsc

# SNMP - Untuk monitoring dengan The Dude / Zabbix
/snmp set enabled=yes contact="Admin - 0812xxxx" location="Surabaya - Pusat" trap-version=2

# Graphing - Grafik trafik pada WebFig
/tool graphing interface add interface=ether1
/tool graphing queue add simple-queue=PCQ-LAN
```

### 9.3 Logging, Supout, dan Cara Melapor ke Support

```routeros
# Logging - Aktifkan log yang diperlukan
/system logging add topics=firewall,info action=memory
/system logging add topics=pppoe,info action=memory
/system logging add topics=wireless,info action=memory
/log print
/log print where topics~pppoe

# Supout - Berkas diagnosis untuk support MikroTik
/system sup-output name=supout-RB01-2026-11-24
/file print where name~supout

# Autosupout - Otomatis kirim supout pada saat crash
/system watchdog set auto-send-supout=yes send-email-to=admin@kantor.com

# Periksa resource dan health
/system resource print
/system health print
/system package print
```

### 9.4 Studi Kasus: SOP Harian dan Insiden Internet Terputus

**Situasi:** Sebuah kantor mengeluhkan internet terputus pada pukul 09.00. Teknisi tidak berada di tempat, tetapi terdapat staf yang dapat membantu pengecekan awal.

**SOP yang Dijalankan:** Staf membuka WinBox, memeriksa `/interface print` - status ether1 running atau tidak. Staf memeriksa `/ip dhcp-client print` - status bound atau searching. Staf memeriksa `/log print` - terdapat pesan link down atau DHCP failed. Staf memeriksa `/tool traceroute 8.8.8.8` - berhenti pada hop yang mana. Hasilnya adalah ether1 running, DHCP searching, dan log menunjukkan ISP tidak memberikan alamat IP. Teknisi menghubungi ISP dan ternyata terdapat gangguan massal. Sambil menunggu, teknisi mengirim supout dan log ke support ISP sebagai bukti.

**Hasil:** Karena SOP jelas, diagnosis awal selesai dalam waktu 10 menit tanpa teknisi datang ke lokasi. Kantor tidak panik karena terdapat komunikasi yang jelas, yaitu permasalahan berada pada ISP, bukan pada router kantor. Setelah ISP pulih, DHCP langsung bound kembali dan internet kembali beroperasi.

### 9.5 Checklist Operasi Harian, Mingguan, dan Bulanan

| Frekuensi | Tugas | Perintah / Cara Periksa |
| --- | --- | --- |
| Harian | Periksa internet dan DHCP | `/ping 8.8.8.8`, `/ip dhcp-server lease print` |
| Harian | Periksa CPU dan trafik | `/system resource print`, `/interface monitor-traffic ether1` |
| Harian | Periksa log penting | `/log print where time~today` |
| Mingguan | Periksa firewall hit dan blokir | `/ip firewall filter print stats` |
| Mingguan | Periksa wireless registration | `/interface wireless registration-table print` |
| Mingguan | Periksa PPPoE active | `/ppp active print` |
| Bulanan | Backup dan export | `/system backup save`, `/export file=` |
| Bulanan | Periksa pemutakhiran RouterOS | `/system package update check-for-updates` |
| Bulanan | Audit pengguna dan layanan | `/user print`, `/ip service print` |
| Insidental | Buat supout pada saat error berat | `/system sup-output` |

> **Dokumentasi adalah bagian dari operasi:** Setiap perubahan harus dicatat, meliputi tanggal, nama pelaksana, hal yang diubah, dan alasan perubahan. Tempelkan komentar pada aturan yang diubah. Hal tersebut yang membedakan teknisi profesional dengan teknisi yang hanya dapat melakukan konfigurasi.


### 9.6 Perbandingan RouterOS v6 dan v7

| Aspek | RouterOS v6 | RouterOS v7 | Catatan Operasional |
| --- | --- | --- | --- |
| Tools Dasar | Ping, traceroute, torch, profiler, netwatch | Sama, dengan peningkatan pada sniffer dan profiler | Perintah dasar tidak berubah |
| The Dude | Server The Dude tersedia pada paket dude | Server The Dude tidak lagi tersedia pada RouterOS v7, hanya client dan agen | Apabila Anda menggunakan The Dude server, pertahankan perangkat v6 atau gunakan server terpisah |
| Container | Tidak tersedia | Tersedia, /container, untuk menjalankan aplikasi dalam kontainer | Container memerlukan penyimpanan tambahan dan perangkat yang mendukung |
| SNMP | SNMP v1, v2c, v3 | SNMP v1, v2c, v3 dengan peningkatan keamanan dan MIB | Periksa kembali community dan user SNMP setelah migrasi |
| Logging | /system logging dengan topik standar | Sama, ditambah topik baru untuk container, wireguard, dan wifi | Sesuaikan topik log dengan layanan yang Anda gunakan |
| Supout | /system sup-output | Sama, dengan format yang diperbarui untuk v7 | Selalu sertakan versi RouterOS pada saat mengirim supout |
| Resource dan Health | /system resource print, /system health print | Sama, dengan informasi CPU dan suhu yang lebih detail pada perangkat baru | Periksa health setelah upgrade untuk memastikan suhu normal |

#### Contoh Konfigurasi RouterOS v6

```routeros
# RouterOS v6 - Monitoring dan Tools
/tool netwatch add host=8.8.8.8 interval=30s up-script="/log info \"Internet UP\"" down-script="/log warning \"Internet DOWN\""
/snmp set enabled=yes contact="Admin" location="Surabaya - Pusat" trap-version=2
/tool graphing interface add interface=ether1
/system logging add topics=firewall,info action=memory
/system sup-output name=supout-RB01-v6
```

#### Contoh Konfigurasi RouterOS v7

```routeros
# RouterOS v7 - Monitoring dan Tools
/tool/netwatch/add host=8.8.8.8 interval=30s up-script="/log info \"Internet UP\"" down-script="/log warning \"Internet DOWN\""
/snmp/set enabled=yes contact="Admin" location="Surabaya - Pusat" trap-version=2
/tool/graphing/interface/add interface=ether1
/system/logging/add topics=firewall,info action=memory
/system/logging/add topics=wireguard,info action=memory
/system/sup-output name=supout-RB01-v7
# Contoh container pada v7 - hanya apabila diperlukan
/container/add remote-image=monitoring-agent:latest interface=veth1 root-dir=disk1/container-agent
```

> **Catatan:** Perbedaan yang paling berdampak pada operasi adalah ketiadaan The Dude server pada v7 dan ketersediaan container pada v7. Rencanakan monitoring Anda dengan mempertimbangkan kedua hal tersebut sebelum melakukan migrasi massal.


## Lampiran A - Troubleshooting Cepat

| Gejala | Kemungkinan Penyebab | Periksa Pertama |
| --- | --- | --- |
| Client tidak memperoleh IP | DHCP server salah interface, terdapat DHCP liar | `/ip dhcp-server print`, `/ip dhcp-server lease print` |
| Dapat ping gateway, tidak dapat internet | NAT belum tersedia, DNS salah, default route hilang | `/ip firewall nat print`, `/ip dns print`, `/ip route print` |
| Dapat internet, tidak dapat akses server cabang | Route balik belum tersedia, firewall forward block | `/ip route print` pada kedua sisi, `/ip firewall filter print` |
| WiFi terhubung, tetapi tidak memperoleh IP | WLAN tidak masuk bridge, DHCP tidak pada bridge | `/interface bridge port print` |
| PPPoE tidak dapat login | Secret salah, profile salah, layanan tidak aktif | `/ppp secret print`, `/log print where topics~pppoe` |
| Router tidak dapat diakses WinBox | Firewall input drop, layanan disabled | Akses melalui MAC WinBox, periksa `/ip service print` |
| Internet lambat padahal bandwidth besar | Queue salah, FastTrack vs Queue, channel wireless ramai | `/queue simple print stats`, `/interface wireless snooper` |


## Lampiran B - Cheat Sheet Perintah Penting

```routeros
# Cepat - Periksa Status
/interface print
/ip address print
/ip route print
/ip dhcp-server lease print
/ip firewall filter print stats
/queue simple print stats
/ppp active print
/system resource print
/log print

# Cepat - Backup dan Export
/system backup save name=backup-hari-ini
/export file=config-hari-ini
/file print

# Cepat - Diagnosis
/ping 8.8.8.8
/tool traceroute 8.8.8.8
/tool torch interface=bridge-lan
/interface monitor-traffic ether1
```
