

## 1. Topologi dan VLSM · 00:00–01:40

**Di layar**, tunjuk PC-Balai dan R-PUSAT, lalu ikuti link menuju R-UTARA dan R-PELABUHAN sampai **PC Sensor Pel**.

> Kami dari Kelompok 14. Oktet kedua mengikuti aturan 8 kali 14, yaitu 112. Jadi kami memakai blok induk privat `10.112.0.0/16`. R-PUSAT menghubungkan jaringan Balai Kota dengan wilayah lain. Jalur fisik menuju Pelabuhan melewati R-UTARA.

| Contoh | Kebutuhan | Network / prefix | Mask | Gateway atau IP ujung link |
| --- | --- | --- | --- | --- |
| WiFi Publik Utara | 11.000 host + gateway | `10.112.0.0/18` | `255.255.192.0` | `10.112.0.1` |
| Pegawai Balai Kota | 1.800 host + gateway | `10.112.224.0/21` | `255.255.248.0` | `10.112.224.1` |
| Sensor dan SCADA Pelabuhan | 5.000 host + gateway | `10.112.160.0/19` | `255.255.224.0` | `10.112.160.1` |
| Link Pusat–Utara | 2 interface | `10.112.241.32/30` | `255.255.255.252` | Pusat `10.112.241.33`, Utara `10.112.241.34` |

> Kami mulai dari blok terbesar. WiFi Utara membutuhkan 11.001 alamat host termasuk gateway. Kapasitas `/19` hanya 8.190 host valid, sehingga kami pilih `/18` dengan 16.382 host valid. Link Pusat–Utara memakai `/30` karena hanya memerlukan dua alamat interface.

## 2. Interface dan OSPF · 01:40–03:00

**R-PUSAT → CLI** pada prompt `#`. Jalankan **satu baris setiap kali**.

```text
show ip interface brief
show ip ospf neighbor
show ip route
```

| Setelah command | Tunjuk bagian output ini |
| --- | --- |
| `show ip interface brief` | `FastEthernet1/0` dengan `10.112.224.1` dan `FastEthernet0/1` dengan `10.112.241.33`. Port yang dipakai diharapkan `up/up`. |
| `show ip ospf neighbor` | R-UTARA memiliki Neighbor ID `14.0.0.4`, Address `10.112.241.34`, dan status `FULL`. R-PUSAT diharapkan memiliki lima tetangga fisik. |
| `show ip route` | `O 10.112.160.0/19` via `10.112.241.34`. LAN lokal `10.112.224.0/21` berstatus `C`. |

**Kalimat saat output sesuai**

> IP interface Balai dan link ke Utara cocok dengan tabel. Tetangga Utara sudah `FULL`, dan rute OSPF ke LAN Pelabuhan memakai alamat R-UTARA sebagai next-hop.

Jika diminta menampilkan mask router, jalankan `show running-config` dan cari `interface FastEthernet1/0`. Baris IP yang dicari ialah `ip address 10.112.224.1 255.255.248.0`.

## 3. Ping dari Balai Kota · 03:00–04:50

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

## 4. DHCP WiFi dan server simulasi · 04:50–06:30

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

## 5. Penutup · 06:30–07:00

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

