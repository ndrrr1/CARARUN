

## 1. Jelaskan apa itu VLSM · 00:00–00:50

**Di layar**, buka README ini dan tunjuk dua baris contoh, WiFi Utara dan link Pusat–Utara.

> VLSM adalah Variable Length Subnet Mask, yaitu cara membagi satu blok IP menjadi subnet dengan ukuran berbeda sesuai kebutuhan. WiFi Utara perlu 11.000 host, sedangkan link antar-router cukup dua alamat interface. Kami memakai `/18` untuk WiFi dan `/30` untuk link. Untuk Kelompok 14, aturan soal memberi oktet kedua 8 kali 14, yaitu 112, sehingga blok induknya `10.112.0.0/16`.

| Contoh | Kebutuhan | Network / prefix | Mask | Gateway atau IP ujung link |
| --- | --- | --- | --- | --- |
| WiFi Publik Utara | 11.000 host + gateway | `10.112.0.0/18` | `255.255.192.0` | `10.112.0.1` |
| Pegawai Balai Kota | 1.800 host + gateway | `10.112.224.0/21` | `255.255.248.0` | `10.112.224.1` |
| Sensor dan SCADA Pelabuhan | 5.000 host + gateway | `10.112.160.0/19` | `255.255.224.0` | `10.112.160.1` |
| Link Pusat–Utara | 2 interface | `10.112.241.32/30` | `255.255.255.252` | Pusat `10.112.241.33`, Utara `10.112.241.34` |

Kalau penguji menunjuk angka `/18`, jawab bahwa `/19` menyediakan 8.190 host valid, sedangkan WiFi Utara perlu 11.001 termasuk gateway. `/18` menyediakan 16.382 host valid.

## 2. Tampilkan topologi di Packet Tracer · 00:50–01:40

1. Di Packet Tracer pilih **File → Open** dan buka `K14_VLSM(1).pkt`. Pilih tab **Logical** di atas area gambar dan **Realtime** di kanan bawah. Lakukan ini sebelum demo.
2. Saat giliran topologi, kembali ke kanvas Logical. Gunakan kontrol zoom sampai router pusat dan cabangnya masuk layar. Tunjuk **R-PUSAT**, lalu arahkan kursor sepanjang **PC-Balai → SW-P_Pegawai-Balai → R-PUSAT → R-UTARA → R-PELABUHAN → SW-Pel_Sensor&SCADA → PC Sensor Pel**.
3. Untuk nama perangkat dan port, buka **Options → Preferences → Interface** lalu aktifkan **Show Device Name Labels** dan, bila perlu, **Always Show Port Labels in Logical Workspace**. Kalau tampilan terlalu padat, gunakan pilihan label ketika mouse berada di atas kabel. Siapkan preferensi ini sebelum timer mulai.
4. Kalau ingin prefix terlihat di kanvas, pilih **Place Note** pada toolbar, klik ruang kosong di dekat perangkat atau kabel, lalu ketik labelnya. Buat label `Balai 10.112.224.0/21`, `Pusat–Utara 10.112.241.32/30`, dan `Pelabuhan 10.112.160.0/19` sebelum demo. Klik **Select** untuk kembali menunjuk perangkat. Saat tampil, tunjuk ketiga label sambil mengikuti jalurnya.

> Di tampilan ini, PC-Balai masuk ke R-PUSAT lewat switch LAN Balai. Sambungan R-PUSAT ke R-UTARA memakai subnet `/30`. Dari Utara, paket diteruskan ke R-PELABUHAN dan masuk ke LAN Sensor Pelabuhan `/19`.

## 3. Interface dan OSPF · 01:40–03:00

Dari topologi, klik **R-PUSAT → CLI**. Jalankan **satu baris setiap kali**.

```text
show ip interface brief
show ip ospf neighbor
show ip route
```

| Command | Kenapa dijalankan | Hasil aktual dari output kamu |
| --- | --- | --- |
| `show ip interface brief` | Memastikan port dan IP yang dipakai aktif | `Fa1/0` dengan `10.112.224.1` dan `Fa0/1` dengan `10.112.241.33` tampil `up/up`. |
| `show ip ospf neighbor` | Memastikan router bertetangga untuk bertukar informasi rute | Lima tetangga `FULL/DR`. Untuk demo, tunjuk Utara `14.0.0.4` dengan Address `10.112.241.34`. |
| `show ip route` | Memastikan prefix tujuan sudah masuk tabel routing | `O 10.112.160.0/19 [110/3] via 10.112.241.34` dan `C 10.112.224.0/21`. |

**Kalimat saat output sesuai**

> IP interface Balai dan link ke Utara cocok dengan tabel. Tetangga Utara sudah `FULL`, dan rute OSPF ke LAN Pelabuhan memakai alamat R-UTARA sebagai next-hop.

Jika diminta menampilkan mask router, jalankan `show running-config` dan cari `interface FastEthernet1/0`. Baris IP yang dicari ialah `ip address 10.112.224.1 255.255.248.0`.

## 4. Ping dari Balai Kota · 03:00–04:50

**PC-Balai → Desktop → Command Prompt**. Jalankan satu per satu.

```text
ipconfig /all
ping 10.112.224.1
ping 10.112.160.2
```

| Cek | Patokan hasil |
| --- | --- |
| IP PC-Balai | IP `10.112.224.2`, mask `255.255.248.0`, gateway `10.112.224.1`. |
| Ping gateway | `Reply from 10.112.224.1` menunjukkan PC bisa menjangkau gateway lokal. |
| Ping Pelabuhan | `Reply from 10.112.160.2` menunjukkan paket mencapai PC Sensor Pel dan responsnya kembali. |

**Kalau masih dalam slot waktu**, jalankan `tracert 10.112.160.2` di PC-Balai. Jalur yang diharapkan melewati R-PUSAT, R-UTARA, lalu R-PELABUHAN. Tunjukkan nama router pada topologi sambil membaca hop yang tampil.

**Kalimat saat ping berhasil**

> PC-Balai mendapat balasan dari gateway lokal dan dari PC Sensor Pel. Jadi koneksi lintas subnet yang kami contohkan berhasil dua arah.

## 5. DHCP WiFi dan server simulasi · 04:50–06:30

**Smartphone-U → Config → Wireless0**. Tunjukkan IP lease yang tampil, mask `255.255.192.0`, dan SSID `Wifi-Utara`. Gateway-nya `10.112.0.1`.

**R-UTARA → CLI** pada prompt `#`. Cocokkan alamat dalam daftar dengan IP smartphone.

```text
show ip dhcp binding
```

**PC-Balai → Desktop → Command Prompt**. Uji akses ke Server-Internet pada topologi Packet Tracer.

```text
ping 8.8.8.8
```

**Kalimat saat hasil sesuai**

> IP smartphone tercatat sebagai lease pada R-UTARA. PC-Balai juga mendapat balasan dari Server-Internet melalui R-ISP di simulasi.

Kalau lease kosong saat demo, sebutkan hasilnya dan lanjutkan ke ping server agar tidak habis waktu di pengaturan WiFi.

## 6. Penutup · 06:30–07:00

> Kami membagi `10.112.0.0/16` menjadi 14 LAN dan enam link internal. Total blok yang dialokasikan 61.752 dari 65.536 alamat, yaitu 94,23% dari blok induk. Masih ada 3.784 alamat di luar blok yang sudah dialokasikan.

Tutup dengan satu kalimat tentang hasil yang tampil. Contoh bila semua uji utama berhasil `Ping Balai ke Pelabuhan dan server simulasi membalas, dan IP smartphone cocok dengan lease DHCP di R-UTARA.`

## Pegangan saat ada gangguan

| Gejala | Yang langsung dicek |
| --- | --- |
| Ping pertama timeout | Tunggu ARP terbentuk dan ulangi sekali. Jika tetap gagal, lanjut ke cek yang relevan. |
| Ping ke gateway Balai gagal | Cek IP/mask/gateway PC-Balai, `Fa1/0` R-PUSAT, dan `SW-P_Pegawai-Balai`. |
| Gateway membalas, Pelabuhan gagal | Pada R-PUSAT cek tetangga Utara `FULL` dan rute `10.112.160.0/19`. Jika rute ada, cek gateway `10.112.160.1` pada LAN Pelabuhan. |
| Smartphone belum mendapat IP | Cocokkan SSID dan autentikasi di Wireless0. Cek `show ip dhcp pool` dan `show ip dhcp binding` di R-UTARA. |
| Hanya ping server simulasi yang gagal | Cek default route R-PUSAT via `203.0.113.113` dan rute balik `10.112.0.0/16` di R-ISP via `203.0.113.114`. |

## Jika penguji minta tambahan

| Diminta | Lokasi dan command |
| --- | --- |
| Bukti HTTP | **PC-Balai → Desktop → Web Browser**. Buka `http://8.8.8.8` jika HTTP di Server-Internet aktif. |
| Bukti DNS | DNS di file yang diperiksa masih Off. Siapkan sebelum demo lewat **Server-Internet → Services → DNS**, pilih On dan tambahkan A record `www.smartcity.test` ke `8.8.8.8`. Uji dari **Command Prompt PC-Balai** dengan `nslookup www.smartcity.test`. |
| Bukti tunnel GRE | **R-PUSAT → CLI**. Jalankan `show interfaces Tunnel0` lalu `ping 192.168.14.2`. Tunjukkan respons endpoint sesuai hasil. Pada konfigurasi saat ini OSPF di Tunnel0 pasif. |

## Jawaban cepat

| Pertanyaan | Jawaban |
| --- | --- |
| Kenapa VLSM? | Tiap LAN punya kebutuhan host berbeda. Prefix dipilih sesuai kapasitas yang diperlukan. |
| Kenapa bloknya `/16`? | `/17` hanya punya 32.768 alamat, sedangkan alokasi internal perlu 61.752. `/16` menyediakan 65.536 alamat. |
| Network dan broadcast link Pusat–Utara? | Network `10.112.241.32`, broadcast `10.112.241.35`, dan dua IP interface berada di antaranya. |
| Apa beda VLSM dan OSPF? | VLSM membagi blok alamat. OSPF bertukar informasi rute menuju subnet yang sudah dibuat. |
| 94,23% itu apa? | Persentase blok induk yang masuk alokasi subnet, termasuk ruang cadangan host di tiap subnet. |
| Semua LAN sudah diuji? | Demo mengambil jalur Balai–Pelabuhan dan contoh WiFi Utara. Untuk hasil seluruh topologi, tunjukkan catatan pengujian sesuai checklist bagian 9 `README.md`. |

## Kenapa dan cara baca output

### A. Kenapa memilih ukuran subnet ini?

`/n` menyatakan jumlah bit network. IPv4 memiliki 32 bit, sehingga sisa `32 − n` bit dipakai untuk alamat di dalam subnet. Untuk subnet pada rancangan ini, dua alamat blok dipakai sebagai network dan broadcast. Perhitungan LAN menyediakan satu alamat valid tambahan untuk gateway.

| Subnet contoh | Kenapa prefix itu dipilih | Isi blok alamat |
| --- | --- | --- |
| WiFi Utara `10.112.0.0/18` | `11.000 + 1 + 2 = 11.003`. Blok 8.192 alamat kurang, jadi ambil 16.384 alamat (`2^14`), yaitu `/18`. | Network `10.112.0.0`, gateway `10.112.0.1`, host valid sampai `10.112.63.254`, broadcast `10.112.63.255`. |
| Balai `10.112.224.0/21` | `1.800 + 1 + 2 = 1.803`. Blok minimum 2.048 alamat (`2^11`), yaitu `/21`. | Network `10.112.224.0`, gateway `10.112.224.1`, PC `10.112.224.2`, broadcast `10.112.231.255`. |
| Sensor Pelabuhan `10.112.160.0/19` | `5.000 + 1 + 2 = 5.003`. Blok minimum 8.192 alamat (`2^13`), yaitu `/19`. | Network `10.112.160.0`, gateway `10.112.160.1`, sensor `10.112.160.2`, broadcast `10.112.191.255`. |
| Pusat–Utara `10.112.241.32/30` | Dua interface, satu network, dan satu broadcast memerlukan empat alamat (`2^2`), yaitu `/30`. | Network `.32`, R-PUSAT `.33`, R-UTARA `.34`, broadcast `.35`. |

**Kenapa mulai dari blok terbesar?** Network ID harus dimulai pada batas ukuran bloknya. Mengalokasikan blok besar dahulu memudahkan penempatan subnet kecil pada ruang tersisa dan menghindari ruang kosong yang terpecah. Hitungan akhirnya `61.752 ÷ 65.536 = 94,23%` dibulatkan. Blok yang sudah dialokasikan mencakup kapasitas cadangan di tiap subnet. Blok induk `/16` menyisakan 3.784 alamat yang belum dialokasikan.

**Kenapa gateway diakhiri `.1`?** Rancangan memakai alamat host valid pertama sebagai gateway. PC-Balai mengirim paket untuk subnet lain ke interface R-PUSAT `10.112.224.1`. Alamat host pertama bisa berbeda dari `.1` jika subnet dimulai di tengah oktet, seperti pada link `/30` ini.

### B. Kenapa `show ip interface brief` dulu?

Command ini menunjukkan IP dan status link sebelum kita menilai routing. Kalau link antar-router belum aktif, adjacency dan rute yang bergantung pada link itu dapat hilang. Berikut seluruh interface R-PUSAT dari output yang kamu kirim.

| Interface | IP di output | Terhubung ke | Status / Protocol |
| --- | --- | --- | --- |
| `FastEthernet0/0` | `203.0.113.114` | R-ISP | `up/up` |
| `FastEthernet0/1` | `10.112.241.33` | R-UTARA | `up/up` |
| `Ethernet0/0/0` | `10.112.241.45` | R-DC | `up/up` |
| `Ethernet0/1/0` | `10.112.241.53` | R-BARAT | `up/up` |
| `Ethernet0/2/0` | `10.112.241.41` | R-TIMUR | `up/up` |
| `Ethernet0/3/0` | `10.112.241.37` | R-SELATAN | `up/up` |
| `FastEthernet1/0` | `10.112.224.1` | Switch LAN Pegawai Balai | `up/up` |
| `FastEthernet1/1` | `10.112.241.1` | Switch LAN Server Balai | `up/up` |
| `Tunnel0` | `192.168.14.1` | Interface GRE ke Pelabuhan | `up/up` |
| `Vlan1` | `unassigned` | SVI bawaan tanpa IP pada rancangan ini | `administratively down/down` |

| Bagian output | Artinya |
| --- | --- |
| `Status` dan `Protocol` | Yang pertama menunjukkan keadaan interface, yang kedua keadaan line protocol. `up/up` berarti keduanya aktif. Cek ping secara terpisah untuk membuktikan komunikasi sampai tujuan. |
| `OK? YES` | IP tercatat valid pada interface. |
| `Method manual` | Alamat tercatat berasal dari konfigurasi manual. |
| `Method unset` | Informasi metode pemberian IP tidak ditetapkan pada tampilan ini. IP tetap terpasang pada port yang memiliki nilai di kolom `IP-Address`. Cek baris `ip address` di `show running-config` bila ditanya cara penetapannya. |
| `Vlan1 unassigned` dan `administratively down` | Pada file ini, konfigurasi Vlan1 memang berisi `no ip address` dan `shutdown`. Gateway LAN berjalan di interface fisik `Fa1/0` dan `Fa1/1`, sehingga status Vlan1 sesuai desain. |

**Kenapa mask tidak terlihat?** Output ringkas menampilkan alamat IP tanpa mask. Cari `interface FastEthernet1/0` di `show running-config` untuk melihat pasangan `10.112.224.1 255.255.248.0`. Baris `C 10.112.224.0/21` di tabel routing juga menunjukkan subnet yang terhubung ke port itu.

### C. Kenapa muncul lima `FULL/DR`?

OSPF bertukar informasi rute antar-router di area 0. `show ip ospf neighbor` memeriksa apakah pertukaran informasi antara dua router yang bertetangga sudah membentuk adjacency. Ini lima tetangga yang terlihat pada R-PUSAT.

| Router tetangga | Neighbor ID | Address di link | Interface lokal R-PUSAT | State |
| --- | --- | --- | --- | --- |
| R-DC | `14.0.0.2` | `10.112.241.46` | `Ethernet0/0/0` | `FULL/DR` |
| R-UTARA | `14.0.0.4` | `10.112.241.34` | `FastEthernet0/1` | `FULL/DR` |
| R-SELATAN | `14.0.0.6` | `10.112.241.38` | `Ethernet0/3/0` | `FULL/DR` |
| R-TIMUR | `14.0.0.5` | `10.112.241.42` | `Ethernet0/2/0` | `FULL/DR` |
| R-BARAT | `14.0.0.3` | `10.112.241.54` | `Ethernet0/1/0` | `FULL/DR` |

- **`Neighbor ID` dan `Address` berbeda** karena Router ID adalah identitas OSPF yang dikonfigurasi. `Address` adalah IP interface tetangga pada link. R-UTARA teridentifikasi sebagai `14.0.0.4` dan menerima kiriman di link melalui `10.112.241.34`.
- **`FULL`** berarti adjacency sudah lengkap dan database link-state sudah disinkronkan dengan tetangga itu. **`DR`** berarti router tetangga menjadi Designated Router pada segmen Ethernet tersebut. Pemilihan DR berlaku per segmen, jadi lima link berbeda dapat menampilkan lima DR berbeda.
- **`Pri 1`** menunjukkan prioritas OSPF tetangga untuk pemilihan DR/BDR. **`Dead Time 00:00:33`** menunjukkan sisa waktu sebelum tetangga dianggap putus jika Hello berikutnya tidak diterima. Angka ini berubah dan diperbarui ketika Hello diterima.
- **R-PELABUHAN** bertetangga secara fisik dengan R-UTARA. Karena itu R-PUSAT memperoleh rute Pelabuhan melalui Utara. Untuk melihat tetangga Pelabuhan langsung, jalankan `show ip ospf neighbor` di R-UTARA dan cari Router ID `14.0.0.7`. **R-ISP** memakai routing statis. **Tunnel0** disetel pasif pada OSPF.

### D. Kenapa tabel routing punya `O`, `C`, `L`, dan `S*`?

Di R-PUSAT, `show ip route` menunjukkan prefix yang dapat dituju oleh router beserta pilihan jalan keluarnya. Gunakan baris yang kamu dapat ini saat menjelaskan.

| Baris dari output | Cara membacanya |
| --- | --- |
| `O 10.112.160.0/19 [110/3] via 10.112.241.34, 01:14:28, FastEthernet0/1` | `O` berarti rute dipelajari melalui OSPF. `110` adalah administrative distance OSPF, `3` adalah cost jalur. Next-hop adalah R-UTARA `10.112.241.34` lewat `Fa0/1`. Angka `01:14:28` menunjukkan umur entri rute pada saat diperiksa. |
| `O 10.112.0.0/18 [110/2] via 10.112.241.34` | LAN WiFi Utara juga dicapai melalui R-UTARA. Cost 2 berbeda dari cost Pelabuhan 3. OSPF menjumlahkan cost interface pada jalur. Detail cost per-interface bisa dilihat dengan `show ip ospf interface` pada router terkait. |
| `C 10.112.224.0/21 is directly connected, FastEthernet1/0` | `C` berarti seluruh LAN Balai terhubung langsung ke `Fa1/0`. Router dapat mencapai host satu subnet tersebut melalui interface itu. |
| `L 10.112.224.1/32 is directly connected, FastEthernet1/0` | `L` menandai tujuan yang tepat sama dengan IP milik R-PUSAT. `/32` mencakup satu IPv4, sehingga router menerima dan memproses paket yang ditujukan ke IP itu. |
| `C 192.168.14.0/30 is directly connected, Tunnel0` | Subnet GRE terpasang lokal melalui Tunnel0. Untuk menguji apakah endpoint seberang membalas, ping `192.168.14.2` dari R-PUSAT. |
| `S* 0.0.0.0/0 [1/0] via 203.0.113.113` | `S` berarti statis, `*` menandai kandidat default, dan `[1/0]` menunjukkan administrative distance 1 serta metric 0. Tujuan tanpa rute yang lebih spesifik diteruskan ke R-ISP `203.0.113.113`. |

**Kenapa tertulis `10.0.0.0/8 is variably subnetted, 27 subnets, 10 masks`?** Packet Tracer menampilkan entri 10.x di bawah judul kelompok tersebut. Pada output ini ada **20 prefix internal** dari 14 LAN dan enam link. Router juga menampilkan **tujuh entri `L /32`** untuk IP interface 10.x miliknya, sehingga totalnya 27 entri. Sepuluh panjang prefix yang muncul ialah `/18`, `/19`, `/21`, `/22`, `/23`, `/24`, `/25`, `/27`, `/30`, dan `/32`. Blok induk yang dialokasikan tetap `10.112.0.0/16`.

**Kenapa paket ke Pelabuhan lewat Utara?** Tujuan `10.112.160.2` cocok dengan `10.112.160.0/19`. Prefix `/19` lebih spesifik dari default `/0`, sehingga R-PUSAT meneruskan paket ke `10.112.241.34`. R-UTARA meneruskan ke R-PELABUHAN lewat `10.112.241.50`. R-PELABUHAN terhubung langsung ke LAN sensor.

### E. Kenapa urutan uji host seperti ini?

| Uji | Yang bisa disimpulkan bila sesuai |
| --- | --- |
| `ipconfig /all` pada PC-Balai | Host `10.112.224.2` memakai mask `/21` dan tahu gateway `10.112.224.1` untuk tujuan di luar subnet. Ini memeriksa konfigurasi PC. |
| `ping 10.112.224.1` | Balasan menguji jalur PC → switch Balai → gateway, termasuk pengenalan alamat MAC melalui ARP. |
| `ping 10.112.160.2` | Balasan menguji jalur pergi ke PC Sensor Pel dan jalur balik ICMP. Cocok untuk membuktikan contoh komunikasi antar-subnet. |
| `tracert 10.112.160.2` | TTL dinaikkan bertahap untuk mengungkap router yang dilalui. Cocokkan hop dengan jalur Pusat → Utara → Pelabuhan pada topologi. |
| `show ip dhcp binding` di R-UTARA | Lease menunjukkan router telah memberikan alamat kepada klien. Cocokkan lease aktual dengan IP di Smartphone-U. Pool `WIFI_UTARA` menyediakan mask `/18`, gateway `10.112.0.1`, dan DNS `8.8.8.8`. Rentang `10.112.0.1–10.112.0.9` dikecualikan agar tersedia untuk alamat tetap seperti gateway. |
| `ping 8.8.8.8` | Pada lab ini `8.8.8.8` adalah Server-Internet di belakang R-ISP. R-PUSAT punya default menuju `203.0.113.113`. R-ISP punya rute balik `10.112.0.0/16` via R-PUSAT `203.0.113.114`. Balasan menguji jalur ke server simulasi. |
| `nslookup` dan Web Browser | `nslookup` menguji pemetaan nama ke IP setelah layanan DNS dan record disiapkan. Browser menguji HTTP. Ping ke alamat IP menguji ICMP dan bisa berhasil saat DNS masih Off. |

### F. Bukti pengujian yang sudah ada

| Pemeriksaan | Bukti saat ini |
| --- | --- |
| Interface R-PUSAT | Delapan port fisik yang digunakan dan Tunnel0 `up/up` pada output kamu. |
| OSPF R-PUSAT | Lima tetangga `FULL/DR` pada output kamu. |
| Routing R-PUSAT | Rute Pelabuhan `10.112.160.0/19` via `10.112.241.34`, LAN Balai terhubung langsung, dan default statis via `203.0.113.113` terlihat di output kamu. |
| Ping gateway, Pelabuhan, dan server simulasi | Isi setelah melihat hasil ping di Command Prompt PC-Balai. |
| Lease Smartphone-U | Isi setelah mencocokkan IP smartphone dengan `show ip dhcp binding` di R-UTARA. |

## Rujukan

- Tampilan Logical, label perangkat, dan port. [Cisco Networking Academy — The Packet Tracer User Interface](https://contenthub.netacad.com/legacy/I2PT/1.1/en/course/files/2.1.1.3%20Video%20-%20The%20Packet%20Tracer%20User%20Interface.pdf)
- Letak kontrol workspace dan alat Place Note. [Packet Tracer Help — Interface Overview](https://tutorials.ptnetacad.net/help/default/interfaceOverview.htm)
- Arti kolom Neighbor ID, State, Dead Time, dan Address. [Cisco — Understand Show IP OSPF Neighbor Command Output](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/13688-16.html)
- Administrative distance dan metric. [Cisco — Understand Administrative Distance](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/15986-admin-distance.html)
- Perhitungan cost dan pemeriksaan OSPF. [Cisco — OSPF Configuration Guide](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr3-fwd/ospf/ospf-configuration-guide/ospf.html)
- Entri lokal `/32`. [Cisco — Local Host Routes Installed in the Routing Table](https://www.cisco.com/c/en/us/support/docs/ip/ip-routing/116264-technote-ios-00.html)
