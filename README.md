# Cara Menjalankan Program

Seluruh langkah dijalankan dari folder utama repository, yaitu folder yang berisi:

```text
soal_1  soal_2  soal_3
```

Jika ZIP didownload dari GitHub, biasanya nama folder hasil extract menjadi:

```text
SISOP-4-2026-IT-124-main
```

Masuk ke folder tersebut:

```bash
cd ~/Downloads/SISOP-4-2026-IT-124-main
```

Jika nama folder berbeda, sesuaikan dengan lokasi hasil extract.

Cek posisi folder:

```bash
ls
```

Output yang diharapkan:

```text
soal_1  soal_2  soal_3
```

---

## Requirement Awal

### A. Update package list

```bash
sudo apt update
```

### B. Install dependency FUSE dan compiler

```bash
sudo apt install -y gcc pkg-config libfuse3-dev fuse3
```

### C. Install dependency Docker, Samba, dan tools tambahan

```bash
sudo apt install -y docker.io smbclient cifs-utils tree zip unzip curl xxd
```

### D. Aktifkan Docker dan FUSE

```bash
sudo systemctl enable --now docker
```

```bash
sudo modprobe fuse
```

### E. Aktifkan konfigurasi `allow_other`

```bash
sudo sed -i 's/^#user_allow_other/user_allow_other/' /etc/fuse.conf
```

```bash
grep -q '^user_allow_other' /etc/fuse.conf || echo 'user_allow_other' | sudo tee -a /etc/fuse.conf
```

---

# Cara Run Soal 1

Soal 1 berfokus pada FUSE yang membaca file perjalanan Mas Amba dan membuat file virtual `tujuan.txt`.

## A. Menyiapkan folder dan file bahan

Masuk ke folder Soal 1:

```bash
cd soal_1
```

Lepaskan mount lama jika masih ada:

```bash
fusermount3 -u mnt 2>/dev/null || true
```

Bersihkan file runtime lama:

```bash
rm -rf amba_files
```

```bash
rm -rf mnt
```

```bash
rm -f kenz_rescue
```

```bash
rm -f fuse.log
```

Buat folder bahan dan mount point:

```bash
mkdir -p amba_files
```

```bash
mkdir -p mnt
```

Buat file `1.txt`:

```bash
cat > amba_files/1.txt <<'EOF'
=== HARI 1 ===

Hari pertama ekspedisi pertama. Saya berangkat dari Tembok Ratapan Keputih jam 5 pagi.
Tujuan saya: Petilasan Puncak Gunung Kawi, untuk meng-update firmware Pusaka Pesugihan v2.7 milik mendiang paman.

KOORD: -7.957

Sampai nanti, paman.
-- Amba
EOF
```

Buat file `2.txt`:

```bash
cat > amba_files/2.txt <<'EOF'
=== HARI 2 ===

Hari kedua. Sampai Pos 1 Gunung Kawi setelah delapan jam jalan kaki.
Kabut tipis, masih bisa lihat warung madura di pinggir jalur.
Saya mampir makan gajah baru 12 batang. Pemilik warung nanya saya mau kemana. Saya jawab 'mau naik anjing'. Dia langsung paham, tidak banyak tanya.

KOORD: 382728

Sampai nanti, paman.
-- Amba
EOF
```

Buat file `3.txt`:

```bash
cat > amba_files/3.txt <<'EOF'
=== HARI 3 ===

Hari ketiga. Pos 2. Sinyal HP mati total.
Saya baca lagi catatan paman: 'kalau sudah di sini, jangan menengok ke belakang sampai puncak'.
Saya patuh. Tidak menengok. Walaupun ada suara "HAI 4nt3k-4nt3k 453N9" di belakang saya seharian. Pelan, sabar, mengikuti ritme jalan saya.

KOORD: 443728, 

Sampai nanti, paman.
-- Amba
EOF
```

Buat file `4.txt`:

```bash
cat > amba_files/4.txt <<'EOF'
=== HARI 4 ===

Hari keempat. Pos 3 - Sumber Air Sungai Kembar.
Saya isi botol sambil baca mantra v2.7. Mantranya panjang sekali, kayak log kernel waktu boot Arch Windows.
Selesai mantra, satu daun beringin jatuh tepat ke tangan saya. Pertanda baik, kata paman dulu. kaki mulai terasa hangat (atau bisa jadi gweh ngompol).

KOORD: 112.469

Sampai nanti, paman.
-- Amba
EOF
```

Buat file `5.txt`:

```bash
cat > amba_files/5.txt <<'EOF'
=== HARI 5 ===

Hari kelima. Trek makin berat. Kabut tebal sekali, jarak pandang dua meter.
Saya hampir tersesat dua kali. Kompas tidak stabil di sekitar Pos 5, selalu menunjuk arah yang berbeda.
Tanah bergetar pelan, semacam denyut nadi. Lokasi sudah dekat.

KOORD: 8688227961, 

Sampai nanti, paman.
-- Amba
EOF
```

Buat file `6.txt`:

```bash
cat > amba_files/6.txt <<'EOF'
=== HARI 6 ===

Hari keenam. Pos 6 - Pondok Tua sebelum puncak.
Saya bermalam di sini. Tidak ada penjaga, hanya angin dan bunyi gamelan sayup-sayup dari arah puncak.
Besok subuh saya naik ke petilasan. Sesuai catatan paman, pusaka harus diaktifkan tepat tengah malam berikutnya.

KOORD: 23:

Sampai nanti, paman.
-- Amba
EOF
```

Buat file `7.txt`:

```bash
cat > amba_files/7.txt <<'EOF'
=== HARI 7 ===

Hari ketujuh. Petilasan Puncak Kawi.
Saya gelar pusaka di altar batu. Saat lilin pertama saya nyalakan, kabut tiba-tiba membentuk sosok pria bermuka familiar, sendirian, berdiri tepat di hadapan saya. Sang Pria Solo Penjaga Puncak!!!
Dia bicara pelan: "Didatengin tanpa janjian saya diem, didatengin tanpa suguhan saya juga diem, disalahin terus klo ada pendaki ngilang saya juga diem...TAPI KALI INI SAYA AKAN LAWAN!...firmware ini cuma bisa di-update kalau kau bawa tumbal. Bukan ayam bangkok, bukan kadal sunda" Melainkan Sebiji  Asisten praktikum yang psikopat. Yang doyan kasih soal sulit. Yang rilis soal jam 00.00 dan 03.00 untuk meneror mahasigma. Cari dia. Bawa dia. Tengah malam berikutnya, di sini.'
Saya turun gunung dengan rencana baru. Saya cari nama-nama asisten Sisop. Satu nama menonjol: Asisten Kenz.

KOORD: 59 WIB

Sampai nanti, paman.
-- Amba
EOF
```

Cek file bahan:

```bash
ls amba_files
```

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt
```

## B. Compile program

```bash
gcc kenz_rescue.c $(pkg-config fuse3 --cflags --libs) -o kenz_rescue
```

Jika tidak ada error, executable `kenz_rescue` berhasil dibuat.

## C. Jalankan FUSE

```bash
./kenz_rescue amba_files mnt -f > fuse.log 2>&1 &
```

Tunggu sebentar:

```bash
sleep 2
```

## D. Cek hasil sesuai soal

Cek isi mount point:

```bash
ls mnt
```

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt  tujuan.txt
```

Cek isi file virtual:

```bash
cat mnt/tujuan.txt
```

Output yang diharapkan:

```text
Tujuan Mas Amba: -7.957382728443728,112.4698688227961,23:59 WIB
```

Pastikan `tujuan.txt` tidak ada di folder asli:

```bash
ls amba_files
```

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt
```

Stop FUSE:

```bash
fusermount3 -u mnt
```

Kembali ke root repository:

```bash
cd ..
```

---

# Cara Run Soal 2

Soal 2 berfokus pada FUSE terenkripsi, file `.enc`, Docker server, dan client.

## A. Menyiapkan folder storage dan file terenkripsi

Masuk ke folder Soal 2:

```bash
cd soal_2
```

Lepaskan mount lama jika ada:

```bash
fusermount3 -u fuse_mount 2>/dev/null || true
```

Hapus container lama jika ada:

```bash
sudo docker rm -f db_app 2>/dev/null || true
```

Bersihkan runtime lama:

```bash
rm -f fuse
```

```bash
rm -f client
```

```bash
rm -f fuse.log
```

```bash
rm -rf encrypted_storage/*
```

```bash
rm -rf fuse_mount/*
```

Buat folder backend:

```bash
mkdir -p encrypted_storage/tests
```

Buat folder mount:

```bash
mkdir -p fuse_mount
```

Buat file `notes.csv.enc` dari base64:

```bash
cat > encrypted_storage/tests/notes.csv.enc.b64 <<'EOF'
FwMCHhkEWhgZAhMFfBcSGx8YWiIzJSIpJSM1NTMlJXw=
EOF
```

Decode menjadi file `.enc`:

```bash
base64 -d encrypted_storage/tests/notes.csv.enc.b64 > encrypted_storage/tests/notes.csv.enc
```

Hapus file base64 sementara:

```bash
rm -f encrypted_storage/tests/notes.csv.enc.b64
```

Cek file terenkripsi:

```bash
ls encrypted_storage/tests
```

Output yang diharapkan:

```text
notes.csv.enc
```

## B. Compile program FUSE dan client

Compile FUSE:

```bash
gcc fuse.c $(pkg-config fuse3 --cflags --libs) -o fuse
```

Compile client:

```bash
gcc client.c -o client
```

Beri permission executable untuk server:

```bash
chmod +x server
```

## C. Jalankan FUSE dan cek enkripsi/dekripsi

Jalankan FUSE:

```bash
./fuse encrypted_storage fuse_mount -o allow_other -f > fuse.log 2>&1 &
```

Tunggu proses mount:

```bash
sleep 2
```

Cek file yang muncul di mount point:

```bash
ls fuse_mount/tests
```

Output yang diharapkan:

```text
notes.csv
```

Cek isi file hasil decrypt:

```bash
cat fuse_mount/tests/notes.csv
```

Output yang diharapkan:

```text
author,notes
admin,TEST_SUCCESS
```

Buat file uji baru lewat mount point:

```bash
echo "halo database" > fuse_mount/test.txt
```

Baca file uji:

```bash
cat fuse_mount/test.txt
```

Output:

```text
halo database
```

Cek backend:

```bash
ls encrypted_storage
```

Output minimal:

```text
test.txt.enc
tests
```

Cek isi terenkripsi:

```bash
xxd encrypted_storage/test.txt.enc
```

Isi file seharusnya tidak terbaca sebagai teks normal.

## D. Jalankan Docker server dan client

Build Docker image:

```bash
sudo docker build -t soal-2-modul-4-sisop .
```

Jalankan container server:

```bash
sudo docker run -d --name db_app -p 9000:9000 -v "$(pwd)/fuse_mount:/app/db" soal-2-modul-4-sisop
```

Cek container:

```bash
sudo docker ps
```

Cek log server:

```bash
sudo docker logs db_app
```

Jalankan client:

```bash
./client 127.0.0.1 9000
```

Di dalam client, coba:

```text
HELP
```

Untuk keluar:

```text
EXIT
```

Stop container:

```bash
sudo docker rm -f db_app
```

Stop FUSE:

```bash
fusermount3 -u fuse_mount
```

Kembali ke root repository:

```bash
cd ..
```

---

# Cara Run Soal 3

Soal 3 berfokus pada Samba server, user/group, permission share, persistence data, dan logging.

## A. Menyiapkan folder volume dan permission

Masuk ke folder Soal 3:

```bash
cd soal_3
```

Buat folder `docs`:

```bash
mkdir -p data/docs
```

Buat folder `ebooks`:

```bash
mkdir -p data/ebooks
```

Buat folder `papers`:

```bash
mkdir -p data/papers
```

Buat folder `sourcecode`:

```bash
mkdir -p data/sourcecode
```

Buat folder log:

```bash
mkdir -p logs
```

Buat file log utama:

```bash
touch logs/libraryit.log
```

Beri permission executable:

```bash
chmod +x entrypoint.sh
```

Matikan Samba host agar port 445 tidak bentrok:

```bash
sudo systemctl stop smbd nmbd 2>/dev/null || true
```

## B. Build dan jalankan Docker Compose

Hapus container lama jika ada:

```bash
sudo docker rm -f libraryit-server 2>/dev/null || true
```

```bash
sudo docker rm -f libraryit-logger 2>/dev/null || true
```

Matikan compose lama jika ada:

```bash
sudo docker compose down --remove-orphans 2>/dev/null || true
```

Build image:

```bash
sudo docker compose build --no-cache
```

Jalankan container:

```bash
sudo docker compose up -d
```

Tunggu container siap:

```bash
sleep 5
```

Cek container:

```bash
sudo docker ps
```

Output yang diharapkan terdapat:

```text
libraryit-server
libraryit-logger
```

## C. Tes akses user dan permission share

Tes daftar share sebagai `member`:

```bash
smbclient -L //127.0.0.1 -U member --password='member123' -m SMB3
```

Output yang diharapkan:

```text
docs
ebooks
papers
IPC$
```

Share `sourcecode` tidak muncul untuk `member`.

Tes anonymous access:

```bash
smbclient -L //127.0.0.1 -N -m SMB3
```

Hasil yang diharapkan: anonymous access gagal.

Tes member membaca `docs`:

```bash
smbclient //127.0.0.1/docs -U member --password='member123' -m SMB3 -c "ls"
```

Tes contributor upload ke `ebooks`:

```bash
echo "test ebook" > ~/test_ebook.txt
```

```bash
smbclient //127.0.0.1/ebooks -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_ebook.txt test_ebook.txt; ls"
```

Tes contributor upload ke `papers`:

```bash
echo "test paper" > ~/test_paper.txt
```

```bash
smbclient //127.0.0.1/papers -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_paper.txt test_paper.txt; ls"
```

Tes librarian upload ke `docs`:

```bash
echo "dokumen librarian" > ~/test_docs.txt
```

```bash
smbclient //127.0.0.1/docs -U librarian --password='lib789' -m SMB3 -c "put $HOME/test_docs.txt test_docs.txt; ls"
```

Tes contributor tidak bisa menulis ke `docs`:

```bash
echo "dokumen contributor" > ~/contributor_docs.txt
```

```bash
smbclient //127.0.0.1/docs -U contributor --password='contrib456' -m SMB3 -c "put $HOME/contributor_docs.txt contributor_docs.txt; ls"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED
```

atau upload gagal.

Tes member tidak bisa akses `sourcecode`:

```bash
smbclient //127.0.0.1/sourcecode -U member --password='member123' -m SMB3 -c "ls"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED
```

Tes contributor upload ke `sourcecode`:

```bash
echo "print('hello sourcecode')" > ~/main.py
```

```bash
smbclient //127.0.0.1/sourcecode -U contributor --password='contrib456' -m SMB3 -c "put $HOME/main.py main.py; ls"
```

## D. Cek persistence dan logging

Cek persistence data:

```bash
sudo find data -type f
```

Output yang diharapkan:

```text
data/ebooks/test_ebook.txt
data/papers/test_paper.txt
data/docs/test_docs.txt
data/sourcecode/main.py
```

Cek log final:

```bash
cat logs/libraryit.log | tail -30
```

Cek log container logger:

```bash
sudo docker logs libraryit-logger --tail=30
```

Format log yang diharapkan:

```text
[YYYY-MM-DD HH:MM:SS] [INFO] [username] [AKSI] [file/share]
[YYYY-MM-DD HH:MM:SS] [WARNING] [username] [DENIED] [file/share]
```

Stop Soal 3:

```bash
sudo docker compose down
```

Kembali ke root repository:

```bash
cd ..
```
---

# Bersih-Bersih Semua Hasil Run

Command ini dijalankan dari folder utama repository, yaitu folder yang berisi `soal_1`, `soal_2`, dan `soal_3`.

```bash
bash -lc 'set +e; fusermount3 -u soal_1/mnt 2>/dev/null || true; fusermount3 -u soal_2/fuse_mount 2>/dev/null || true; sudo docker rm -f db_app libraryit-server libraryit-logger 2>/dev/null || true; (cd soal_3 && sudo docker compose down --remove-orphans 2>/dev/null || sudo docker-compose down --remove-orphans 2>/dev/null || true); rm -rf soal_1/amba_files soal_1/mnt soal_1/kenz_rescue soal_1/fuse.log; rm -f soal_2/fuse soal_2/client soal_2/fuse.log; sudo rm -rf soal_2/encrypted_storage/* soal_2/fuse_mount/*; sudo rm -rf soal_3/data/docs/* soal_3/data/ebooks/* soal_3/data/papers/* soal_3/data/sourcecode/* soal_3/logs/*; mkdir -p soal_2/encrypted_storage soal_2/fuse_mount soal_3/data/docs soal_3/data/ebooks soal_3/data/papers soal_3/data/sourcecode soal_3/logs; touch soal_3/logs/libraryit.log; sudo chown -R "$USER:$USER" soal_3/data soal_3/logs soal_2/encrypted_storage soal_2/fuse_mount 2>/dev/null || true; chmod 755 soal_2/encrypted_storage soal_2/fuse_mount soal_3/data soal_3/data/docs soal_3/data/ebooks soal_3/data/papers soal_3/data/sourcecode soal_3/logs; chmod 644 soal_3/logs/libraryit.log; echo "Cleanup selesai. Repository sudah bersih dari hasil runtime."'
```

Setelah itu cek struktur folder:

```bash
tree
```
