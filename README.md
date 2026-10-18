
## 1. IP, gateway, forwarding, NAT dan konektivitas

### Console jichi - command cek

```bash
date -Is
hostname
ip -4 addr show
ip -4 route
sysctl net.ipv4.ip_forward
iptables -t nat -S POSTROUTING
iptables -S FORWARD
ping -c 3 -W 2 1.1.1.1
```

**Hasil yang diharapkan:**

- eth0 memiliki IPv4 hasil DHCP NAT dan default route melalui eth0.
- eth1 sampai eth4 memiliki `192.217.1.1/24`, `.2.1/24`, `.3.1/24`, `.4.1/24`.
- `net.ipv4.ip_forward = 1`.
- Aturan MASQUERADE untuk sumber `192.217.0.0/16` keluar lewat `eth0`.
- Ping ke `1.1.1.1` mendapat balasan.

**Penjelasan saat demo:** “Jichi berfungsi sebagai gateway empat subnet. IPv4 forwarding meneruskan paket antarinterface, sedangkan MASQUERADE menerjemahkan alamat sumber internal saat trafik keluar ke NAT.”

### Console seluruh host Debian selain jichi - command cek

Ulangi pada utahime, shoko, gojo, tengen, geto, maki, panda, nanami, todo, kirara, yaga dan inumaki.

```bash
hostname
ip -4 addr show dev eth0
ip -4 route
gw=$(ip -4 route show default | awk 'NR==1 {print $3}')
if [ -n "$gw" ]; then ping -c 2 -W 2 "$gw"; else echo 'FAIL: gateway kosong'; fi
ping -c 2 -W 2 1.1.1.1
```

**Tambahan di utahime, untuk routing antarsegmen:**

```bash
ping -c 2 -W 2 192.217.4.10
```

**Hasil yang diharapkan:** IP sesuai tabel README konfigurasi; host Network 1 memakai gateway `.1.1`, host Network 4 memakai `.4.1`; gateway dan IP eksternal dapat dijangkau. Ping utahime ke tengen membuktikan konektivitas Network 1 ke Network 4.

## 2. DHCP Server tengen dan subnet lokal tanpa pool

### Console tengen - command cek

```bash
hostname
ip -4 addr show dev eth0
ip -4 route
dhcpd -t -4 -cf /etc/dhcp/dhcpd.conf
echo "Exit validasi DHCP: $?"
cat /etc/default/isc-dhcp-server
sed -n '/# NOMOR 2:/,/# NOMOR 4: Network/p' /etc/dhcp/dhcpd.conf
bash /root/m3-service.sh status
tail -n 20 /root/m3-dhcpd.log
```

**Hasil yang diharapkan:**

- Tengen memiliki `.4.10/24`, gateway `.4.1`.
- `Exit validasi DHCP: 0`.
- Subnet `192.217.4.0/24` dideklarasikan tanpa `range` atau pool dinamis.
- Daemon `dhcpd` aktif, port UDP 67 terbuka, interface server eth0.
- Start terbaru tidak menampilkan error `No subnet declaration`.

Banner copyright yang dicetak `dhcpd -t` merupakan output normal. Perhatikan exit validasi dan error setelah banner.

**Penjelasan saat demo:** “Server berada di Network 4, sehingga subnet lokalnya harus tetap dideklarasikan walaupun tidak membagikan IP dinamis di segmen tersebut. Scope untuk Network 2 dan 3 melayani permintaan yang diteruskan relay.”

## 3. Relay melayani client Network 2 dan 3

### Console jichi - command cek

```bash
cat /etc/default/isc-dhcp-relay
bash /root/m3-service.sh status
tail -n 20 /root/m3-relay.log
ping -c 2 -W 2 192.217.4.10
```

**Hasil yang diharapkan:** server tujuan `.4.10`; proses relay memiliki downstream `-id eth2 -id eth3` dan upstream `-iu eth4`.

### Console Itadori dan Ino - command cek

Ulangi pada kedua console.

```sh
sh /root/m3-client.sh check
ping -c 2 -W 2 192.217.4.10
ping -c 2 -W 2 1.1.1.1
```

### Console tengen - command cek log kedua client

```bash
grep -E 'DHCP(DISCOVER|OFFER|REQUEST|ACK).*02:12:03:(02|03):00:01' /root/m3-dhcpd.log | tail -n 16
```

**Hasil yang diharapkan:**

| Client | IP | Gateway | Jalur pada log server |
|---|---|---|---|
| Itadori | `.2.10-.2.35` | `192.217.2.1` | via `192.217.2.1` |
| Ino | `.3.10-.3.89` | `192.217.3.1` | via `192.217.3.1` |

`serverid` dari ACK harus `192.217.4.10` pada keduanya. Waktu ACK pada client/log membantu menghubungkan hasil dengan transaksi yang didemokan.

**Penjelasan saat demo:** “Broadcast DHCP dari kedua subnet diterima jichi pada interface client, kemudian diteruskan ke tengen. Server menggunakan informasi relay untuk memilih subnet dan pool yang sesuai.”

## 4. Pool 1 dan Pool 2 berdasarkan class MAC

### Console tengen - command cek

```bash
sed -n '/# NOMOR 4: class/,/# NOMOR 5: class/p' /etc/dhcp/dhcpd.conf
sed -n '/# NOMOR 4: Network/,/# NOMOR 5: Network/p' /etc/dhcp/dhcpd.conf
grep -E 'DHCPACK.*02:12:03:02:00:0[1-4]' /root/m3-dhcpd.log | tail -n 8
```

### Console Itadori, Fushiguro, Zenin dan Okkotsu - command cek

```sh
sh /root/m3-client.sh check
```

**Hasil yang diharapkan:**

| Client | MAC eth0 | IP yang diperbolehkan |
|---|---|---|
| Itadori | `02:12:03:02:00:01` | 192.217.2.10-192.217.2.35 |
| Fushiguro | `02:12:03:02:00:02` | 192.217.2.10-192.217.2.35 |
| Zenin | `02:12:03:02:00:03` | 192.217.2.70-192.217.2.95 |
| Okkotsu | `02:12:03:02:00:04` | 192.217.2.70-192.217.2.95 |

Konfigurasi harus memperlihatkan `allow members of` pada setiap pool dan tidak memuat range `.2.36-.2.69`. IP client boleh berbeda-beda di dalam range yang benar.

**Penjelasan saat demo:** “Semua client Divisi A berada pada subnet yang sama. Class berdasarkan MAC membatasi Itadori/Fushiguro ke Pool 1 dan Zenin/Okkotsu ke Pool 2. Pembuatan dua range saja belum membatasi identitas client; pembatasannya ditentukan oleh allow members of.”

## 5. Short lease, long lease, DNS dan domain-search

### Console tengen - command cek

```bash
head -n 9 /etc/dhcp/dhcpd.conf
sed -n '/# NOMOR 5: class/,/# NOMOR 2:/p' /etc/dhcp/dhcpd.conf
sed -n '/# NOMOR 5: Network/,/# NOMOR 6:/p' /etc/dhcp/dhcpd.conf
```

### Console Ino, Kusakabe dan Miwa - command cek

```sh
sh /root/m3-client.sh check
```

**Hasil yang diharapkan:**

| Client | Pool | Field `lease` dari ACK |
|---|---|---|
| Ino | `.3.10-.3.89` | `1800` |
| Kusakabe | `.3.10-.3.89` | `1800` |
| Miwa | `.3.100-.3.199` | `43200` |

Pada ketiganya harus terlihat:

```text
dns=192.217.1.2 192.217.1.3
search=k12.com
serverid=192.217.4.10
```

Resolver juga harus memuat `search k12.com` dan kedua nameserver. Field pada `DHCPACK TERAKHIR` adalah data yang dicatat hook dari DHCP.

**Penjelasan saat demo:** “Ino/Kusakabe menerima lease 30 menit dan Miwa 12 jam sesuai class masing-masing. DNS dan domain-search diberikan melalui DHCP options. Durasi dibuktikan dari ACK, bukan dari lamanya client dapat melakukan ping.”

## 6. Fixed-address Kugisaki

### Console tengen - command cek

```bash
sed -n '/# NOMOR 6:/,$p' /etc/dhcp/dhcpd.conf
grep 'range ' /etc/dhcp/dhcpd.conf
grep -E 'DHCP(ACK|OFFER).*02:12:03:03:00:04' /root/m3-dhcpd.log | tail -n 8
```

### Console Kugisaki - cek hasil saat ini

```sh
sh /root/m3-client.sh check
```

### Pemicu demo - dua transaksi di Kugisaki

Gunakan saat diminta membuktikan IP yang sama pada dua permintaan. `fresh` memulai transaksi DHCP baru.

```sh
sh /root/m3-client.sh fresh
sh /root/m3-client.sh check
sh /root/m3-client.sh fresh
sh /root/m3-client.sh check
```

**Hasil yang diharapkan:** kedua transaksi mendapat `192.217.3.99/24`, MAC `02:12:03:03:00:04`, gateway `.3.1`, serverid `.4.10`. Alamat `.3.99` berada di luar kedua pool Network 3.

**Penjelasan saat demo:** “Kugisaki tetap menggunakan DHCP, dengan alamat yang ditentukan oleh reservasi MAC pada server. Setiap transaksi mendapatkan .3.99. Fixed-address tidak dicatat sebagai binding dinamis biasa, sehingga buktinya menggunakan deklarasi host, ACK dan IP client.”

## 7. Audit seluruh client dan database lease

### Console delapan client - command cek

Ulangi pada Itadori, Fushiguro, Zenin, Okkotsu, Ino, Kusakabe, Miwa dan Kugisaki.

```sh
sh /root/m3-client.sh audit
echo "Exit audit client: $?"
```

### Console tengen - command cek

```bash
ls -l /var/lib/dhcp/dhcpd.leases
python3 /root/m3-audit-leases.py
echo "Exit audit server: $?"
grep -E 'DHCPACK.*02:12:03:03:00:04' /root/m3-dhcpd.log | tail -n 2
```

**Hasil yang diharapkan:**

- Client: `HASIL: PASS NAMA_CLIENT` dan exit `0`.
- Mask `255.255.255.0`, gateway `.2.1`/`.3.1`, broadcast `.2.255`/`.3.255`.
- DNS `.1.2 .1.3`, domain-search `k12.com`, IP aktif sesuai ACK.
- Ping gateway, tengen dan `1.1.1.1` berhasil.
- Server: tujuh binding dinamis sesuai pool/durasi, tanpa binding aktif ganda; Kugisaki memakai bukti ACK terpisah.

**Penjelasan saat demo:** “Pemeriksaan mencocokkan options dari ACK dengan IP, route dan resolver yang aktif di client. Audit server membaca deklarasi lease terbaru per IP. Riwayat lease lama atau expired tidak dihitung sebagai bentrok aktif.”

Jika muncul MAC unknown, periksa apakah lease itu sisa identitas client lama yang masih aktif. Periksa waktu expiry dan identitas client untuk menyelesaikannya. Status FAIL perlu ditangani sebelum screenshot dipakai sebagai bukti berhasil.

## 8. DORA dan field giaddr

### Pemicu demo - jalankan bergantian pada tengen dan Ino

**Pertama, console tengen:**

```bash
bash /root/m3-capture.sh start
```

**Kedua, console Ino saat capture masih aktif:**

```sh
sh /root/m3-client.sh fresh
sh /root/m3-client.sh check
```

**Ketiga, kembali ke tengen setelah Ino mendapat ACK:**

```bash
bash /root/m3-capture.sh stop
```

### Console tengen - command cek ulang capture tersimpan

```bash
tcpdump -nn -vvv -r /root/bukti-m3/08-dora.pcap
```

**Hasil yang diharapkan:** pilih paket dengan MAC Ino dan `xid` yang sama.

| Field tcpdump | Nilai/arti |
|---|---|
| DHCP-Message option 53 | Discover -> Offer -> Request -> ACK |
| Client-Ethernet-Address | `02:12:03:03:00:01` |
| xid | Sama dalam satu transaksi |
| Server-ID option 54 | `192.217.4.10` |
| Your-IP / yiaddr | Alamat Ino di `.3.10-.3.89` |
| Lease-Time option 51 | `1800` detik |
| Gateway-IP / giaddr | `192.217.3.1` |
| Default-Gateway option 3 | `192.217.3.1` |
| DNS option 6 | `.1.2`, `.1.3` |
| Domain-Search option 119 | `k12.com` |

**Penjelasan saat demo:** “Discover mencari layanan, Offer menawarkan alamat, Request memilih tawaran, dan ACK mengonfirmasi lease. Giaddr .3.1 menunjukkan interface jichi yang menerima request Ino. Nilai ini membantu server memilih subnet Network 3.”

Jika client yang dipilih Itadori, giaddr menjadi `.2.1`. Pada segmen tengen, MAC client berada dalam payload DHCP, sedangkan frame Ethernet berasal dari relay. Capture berdasarkan port 67/68 tetap menangkap transaksi tersebut.

## 9. Relay mati, DHCP baru gagal, kemudian pulih

### A. Pemicu gangguan - console jichi

```bash
date -Is
bash /root/m3-service.sh stop
bash /root/m3-service.sh status
```

**Hasil yang diharapkan:** `dhcrelay TIDAK AKTIF`. Exit status nonzero dari pemeriksaan layanan saat mati memang sesuai skenario.

### B. Pemicu permintaan DHCP baru - console Itadori

```sh
if sh /root/m3-client.sh fresh; then
    echo 'FAIL simulasi: client masih memperoleh DHCP; periksa relay/server lain.'
else
    echo 'SESUAI: DHCP baru gagal saat relay dihentikan.'
fi
```

### C. Cek hasil gangguan - console Itadori

```sh
ip -4 addr show dev eth0
ip -4 route
cat /root/m3-udhcpc.log
```

**Hasil yang diharapkan:** Discover dikirim berulang, tidak memperoleh lease/ACK, tidak ada IPv4 global dan default route hasil DHCP pada eth0.

### D. Cek server tetap hidup - console tengen

```bash
bash /root/m3-service.sh status
```

### E. Pemulihan relay - console jichi

```bash
bash /root/m3-service.sh start
bash /root/m3-service.sh status
```

### F. Pemicu permintaan ulang dan cek pemulihan - console Itadori

```sh
sh /root/m3-client.sh fresh
sh /root/m3-client.sh audit
```

### G. Cek ACK pemulihan - console tengen

```bash
grep -E 'DHCP(DISCOVER|OFFER|REQUEST|ACK).*02:12:03:02:00:01' /root/m3-dhcpd.log | tail -n 8
```

**Hasil yang diharapkan:** Itadori kembali mendapat Pool 1 `.2.10-.2.35`, gateway `.2.1`, options benar dan `HASIL: PASS itadori`. IP dinamis boleh berbeda dari sebelum gangguan.

**Penjelasan saat demo:** “Fresh melepas IP dan memulai Discover baru, sehingga pengujian bergantung pada relay. Ketika relay dihentikan, broadcast client tidak diteruskan ke tengen. Setelah relay aktif, transaksi DHCP berhasil lagi. Client lain dengan lease lama tidak harus langsung kehilangan konektivitas.”

## 10. Autostart sesudah restart node

### Sebelum restart - console jichi dan tengen

Ulangi pada kedua node. Persiapan autostart dan pengaturan Start command harus sudah selesai di README konfigurasi.

Start command pada **jichi dan tengen** harus sama:

```text
/bin/bash /etc/network/modul3/console-start.sh
```

Baris ini diperiksa melalui pengaturan GNS3. Command di bawah hanya memeriksa file/status dan tidak menjalankan ulang boot.

```bash
hostname
cat /root/m3-role
ls -l /etc/network/modul3/console-start.sh
ls -l /etc/network/modul3/start.sh
ls -l /etc/network/modul3/root-files/m3-boot.sh
cat /etc/network/modul3/state/boot-packages.txt
find /etc/network/modul3/state/boot-debs -maxdepth 1 -name '*.deb' | wc -l
cat /etc/network/modul3/state/autostart-ready.txt
cat /etc/network/modul3/state/boot-status.txt
(
    cd /etc/network/modul3/state/boot-debs || exit 1
    sha256sum -c SHA256SUMS
)
```

**Hasil yang diharapkan:** wrapper dan script startup tersedia, nama peran sesuai, daftar paket ada, checksum cache `OK`, dan persiapan menghasilkan `AUTOSTART SIAP: NAMA_NODE`. **Catat nilai `id=...` sebelum restart.** Jumlah file cache lebih dari nol saja belum membuktikan kelengkapan dependensi; validasi paket/versi dan percobaan boot dilakukan oleh script konfigurasi.

### Pemicu demo - Stop/Start melalui GNS3

1. Cadangkan screenshot/output bukti yang diperlukan dari `/root` sebelum Stop/Start; direktori tersebut belum tentu persisten.
2. Stop **jichi dan tengen** melalui GNS3.
3. Start **jichi terlebih dahulu**, tunggu prompt Bash dan periksa status boot jichi. Jika jichi berhasil, Start **tengen** dan tunggu prompt Bash.
4. Periksa status berikut langsung sesudah startup. Pada tahap bukti ini cukup gunakan command cek; menjalankan ulang script boot atau daemon akan mengaburkan bukti autostart.

### Sesudah restart - console jichi

```bash
date -Is
hostname
cat /etc/network/modul3/state/boot-status.txt
tail -n 35 /etc/network/modul3/boot.log
ip -4 addr show
ip -4 route
sysctl net.ipv4.ip_forward
iptables -t nat -S POSTROUTING
bash /root/m3-service.sh status
ping -c 2 -W 2 192.217.4.10
ping -c 2 -W 2 1.1.1.1
```

### Sesudah restart - console tengen

```bash
date -Is
hostname
cat /etc/network/modul3/state/boot-status.txt
tail -n 35 /etc/network/modul3/boot.log
ip -4 addr show dev eth0
ip -4 route
dhcpd -t -4 -cf /etc/dhcp/dhcpd.conf
echo "Exit validasi DHCP: $?"
bash /root/m3-service.sh status
ls -l /var/lib/dhcp/dhcpd.leases
ping -c 2 -W 2 192.217.4.1
ping -c 2 -W 2 1.1.1.1
```

### Pemicu DHCP baru dan cek - seluruh delapan client

Permintaan baru membuktikan bahwa server dan relay hasil autostart dapat mengalokasikan alamat.

```sh
sh /root/m3-client.sh fresh
sh /root/m3-client.sh audit
```

### Audit akhir - console tengen

```bash
python3 /root/m3-audit-leases.py
echo "Exit audit server: $?"
grep -E 'DHCPACK.*02:12:03:03:00:04' /root/m3-dhcpd.log | tail -n 2
```

**Hasil yang diharapkan:**

- File `boot-status.txt` menunjukkan `BOOT OK: jichi` / `BOOT OK: tengen`, waktunya sesudah restart, dan **ID berbeda dari sebelum restart**.
- Relay dan server aktif otomatis; IP, gateway, forwarding dan NAT tetap benar.
- Semua client mendapat DHCP baru dan lulus audit.
- Pool 1/Pool 2 tetap sesuai; Ino/Kusakabe 1800 detik; Miwa 43200 detik; Kugisaki `.3.99`.

**Penjelasan saat demo:** “Startup memulihkan script dan paket yang diperlukan, kemudian menerapkan konfigurasi jaringan dan menjalankan layanan. Pemeriksaan setelah restart menunjukkan layanan aktif otomatis. Permintaan DHCP baru dari kedua subnet membuktikan kebijakan alokasi tetap berjalan.”

### Jika boot gagal - command diagnosis saja

Jalankan pada node yang gagal. Wrapper tetap membuka Bash agar output bisa diperiksa:

```bash
cat /etc/network/modul3/state/boot-status.txt
tail -n 80 /etc/network/modul3/boot.log
cat /etc/network/modul3/state/boot-packages.txt
ls -l /etc/network/modul3/console-start.sh
```

`BOOT GAGAL` atau `BOOT MULAI` yang belum selesai berarti pengujian belum lulus. `BOOT OK` lama di riwayat `boot.log` tidak menggantikan status percobaan terbaru. Perbaikan untuk error APT `Pathname to install is not absolute` ada di **README_KONFIGURASI.md nomor 10A-10B**; gunakan script revisi utuh, lalu ulangi pengujian Stop/Start. Saat perbaikan, bedakan output pemulihan manual dari bukti autostart setelah restart.

## Checklist screenshot per nomor

| Soal | Console | Bukti utama |
|---|---|---|
| 1 | jichi + host infrastruktur | IP/route, forwarding, NAT, ping |
| 2 | tengen | Validasi exit 0, subnet lokal tanpa pool, daemon UDP 67 |
| 3 | jichi, tengen, Itadori, Ino | Relay, ACK dua subnet, gateway |
| 4 | tengen + empat client Network 2 | Class/pool dan IP sesuai identitas |
| 5 | tengen, Ino, Kusakabe, Miwa | 1800/43200 detik, DNS/search |
| 6 | tengen + Kugisaki | MAC, reservasi dan dua transaksi `.3.99` |
| 7 | tengen + delapan client | Audit client/server PASS dan ACK Kugisaki |
| 8 | tengen + Ino | DORA satu xid, server ID, yiaddr, lease, giaddr |
| 9 | jichi + Itadori + tengen | Relay mati/gagal DHCP, server hidup, pemulihan |
| 10 | jichi + tengen + delapan client | BOOT OK baru, daemon otomatis, DHCP baru dan audit |

## Cakupan pemeriksaan panduan

Pemeriksaan lokal pada revisi ini:

| Pemeriksaan | Cakupan |
|---|---|
| Sintaks dan konsistensi | Seluruh 42 blok Bash/sh kedua README, tujuh script yang disalin identik dengan sumber, dan ShellCheck tanpa warning. |
| DHCP | Parser ISC DHCP 4.4.3-P1 menerima konfigurasi final; kandidat salah tidak mengganti konfigurasi aktif dan restart gagal mengembalikan konfigurasi sebelumnya. |
| Client dan lease | Hook ACK, delapan kebijakan client, gateway/DNS/search/lease salah, binding terbaru, expired, dan duplikasi diuji dengan mock/fixture. |
| Boot jichi/tengen | Pemulihan `/root`, cache lama, dpkg offline tanpa APT, ID boot baru, cache rusak, kegagalan pemasangan/layanan, serta console sesudah boot gagal diuji dengan fixture. |
| Instalasi dpkg nyata | Dua paket kecil beserta dependensinya dipasang ke root terisolasi tanpa jaringan; tidak memasang paket ke sistem lab. |

Pengujian lokal tersebut tidak menjalankan proyek GNS3 pengguna atau membuktikan trafik DHCP nyata. Kriteria nomor 1-10 tetap harus dibuktikan melalui command dan capture di lab.

## Membaca hasil dan error

- Ping `1.1.1.1` menguji akses eksternal berdasarkan IP. Resolusi domain internet membutuhkan DNS BIND dari soal 11-12.
- `echo $?` harus langsung mengikuti command yang ingin diperiksa; `0` berarti command tersebut selesai sukses.
- Hasil audit FAIL menyebut syarat yang belum sesuai. Tampilkan hasil nyata dan selesaikan penyebabnya sebelum menyatakan soal berhasil.
- Jika console terlalu panjang, ambil beberapa screenshot berurutan. Untuk nomor 8, pertahankan MAC dan xid yang sama pada paket yang dijelaskan.
- Command di file ini belum dijalankan pada proyek GNS3 pengguna. Penjelasan dan nilai target merupakan panduan demonstrasi, bukan output eksekusi yang direkayasa.
