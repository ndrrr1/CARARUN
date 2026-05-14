# README - Cara Run Terbaru

Jalankan semua command dari folder utama repository, yaitu folder yang berisi:

```text
soal_1  soal_2  soal_3
```

Contoh:

```bash
cd ~/"Modul 4"
```

File tambahan yang disiapkan di `~/Downloads`:

```text
amba_files.zip
notes.csv.enc
server
```

---


# 1. Requirement Awal

## A. Update package list

```bash
sudo apt update
```

Penjelasan: memperbarui daftar package sebelum install dependency.

## B. Install dependency FUSE dan compiler

```bash
sudo apt install -y gcc pkg-config libfuse3-dev fuse3
```

Penjelasan: menginstall compiler C dan library FUSE3 yang dibutuhkan untuk Soal 1 dan Soal 2.

## C. Install Docker, Samba client, dan tools tambahan

```bash
sudo apt install -y docker.io smbclient cifs-utils tree zip unzip curl xxd
```

Penjelasan: menginstall Docker untuk container, `smbclient` untuk testing Samba, serta tools seperti `unzip`, `tree`, dan `xxd`.

## D. Aktifkan Docker

```bash
sudo systemctl enable --now docker
```

Penjelasan: memastikan service Docker aktif.

## E. Aktifkan module FUSE

```bash
sudo modprobe fuse
```

Penjelasan: memastikan module FUSE aktif di kernel.

## F. Aktifkan konfigurasi `allow_other`

```bash
sudo sed -i 's/^#user_allow_other/user_allow_other/' /etc/fuse.conf
```

Penjelasan: mengaktifkan konfigurasi `user_allow_other` pada FUSE.

```bash
grep -q '^user_allow_other' /etc/fuse.conf || echo 'user_allow_other' | sudo tee -a /etc/fuse.conf
```

Penjelasan: memastikan konfigurasi `user_allow_other` benar-benar ada.

---

# 2. Cara Run Soal 1

Soal 1 menjalankan FUSE untuk membaca file dari `amba_files.zip` dan membuat file virtual `tujuan.txt`.

## A. Siapkan bahan dari ZIP

Masuk ke folder Soal 1:

```bash
cd soal_1
```

Penjelasan: masuk ke folder pengerjaan Soal 1.

Lepaskan mount lama jika masih aktif:

```bash
fusermount3 -u mnt 2>/dev/null || true
```

Penjelasan: melepas mount FUSE lama agar tidak bentrok.

Bersihkan runtime lama:

```bash
rm -rf amba_files mnt kenz_rescue fuse.log amba_files.zip
```

Penjelasan: menghapus folder/file hasil run sebelumnya.

Copy bahan dari Downloads:

```bash
cp ~/Downloads/amba_files.zip .
```

Penjelasan: menyalin file bahan `amba_files.zip` ke folder Soal 1.

Buat mount point:

```bash
mkdir -p mnt
```

Penjelasan: membuat folder `mnt` untuk tempat hasil mount FUSE.

Unzip bahan:

```bash
unzip amba_files.zip
```

Penjelasan: mengekstrak file `amba_files.zip` sehingga terbentuk folder `amba_files`.

Cek isi bahan:

```bash
ls amba_files
```

Penjelasan: memastikan file `1.txt` sampai `7.txt` berhasil diekstrak.

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt
```

Cek salah satu isi file:

```bash
cat amba_files/1.txt
```

Penjelasan: memastikan isi file bahan terbaca dan memiliki baris koordinat.

## B. Compile program

```bash
gcc kenz_rescue.c $(pkg-config fuse3 --cflags --libs) -o kenz_rescue
```

Penjelasan: compile program `kenz_rescue.c` menjadi executable `kenz_rescue`.

## C. Jalankan FUSE

```bash
./kenz_rescue amba_files mnt -f > fuse.log 2>&1 &
```

Penjelasan: menjalankan FUSE dengan sumber `amba_files` dan mount point `mnt`.

```bash
sleep 2
```

Penjelasan: memberi waktu agar proses mount selesai.

## D. Cek hasil Soal 1

```bash
ls mnt
```

Penjelasan: melihat isi mount point. Harus muncul file `tujuan.txt`.

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt  tujuan.txt
```

```bash
cat mnt/tujuan.txt
```

Penjelasan: membaca file virtual hasil gabungan koordinat.

Output yang diharapkan:

```text
Tujuan Mas Amba: -7.957382728443728,112.4698688227961,23:59 WIB
```

Pastikan `tujuan.txt` tidak ada di folder asli:

```bash
ls amba_files
```

Penjelasan: membuktikan `tujuan.txt` hanya file virtual di mount point.

Stop FUSE:

```bash
fusermount3 -u mnt
```

Penjelasan: melepas mount FUSE setelah selesai testing.

Kembali ke root repository:

```bash
cd ..
```

---

# 3. Cara Run Soal 2

Soal 2 menjalankan FUSE terenkripsi, menampilkan file `.enc` sebagai file normal, lalu menjalankan server melalui Docker.

## A. Siapkan storage dan server

Masuk ke folder Soal 2:

```bash
cd soal_2
```

Penjelasan: masuk ke folder pengerjaan Soal 2.

Lepaskan mount lama:

```bash
fusermount3 -u fuse_mount 2>/dev/null || true
```

Penjelasan: melepas mount FUSE lama.

Hapus container lama:

```bash
sudo docker rm -f db_app 2>/dev/null || true
```

Penjelasan: menghapus container server lama agar tidak bentrok.

Bersihkan file runtime lama:

```bash
rm -f fuse client fuse.log
```

Penjelasan: menghapus executable dan log lama.

Bersihkan folder storage dan mount:

```bash
rm -rf encrypted_storage/* fuse_mount/*
```

Penjelasan: membersihkan hasil run sebelumnya.

Buat folder backend dan mount point:

```bash
mkdir -p encrypted_storage/tests
```

Penjelasan: membuat folder backend untuk file `.enc`.

```bash
mkdir -p fuse_mount
```

Penjelasan: membuat folder mount point FUSE.

Copy file terenkripsi dari Downloads:

```bash
cp ~/Downloads/notes.csv.enc encrypted_storage/tests/notes.csv.enc
```

Penjelasan: memasukkan file `.enc` sebagai bahan uji FUSE.

Copy server dari Downloads:

```bash
cp ~/Downloads/server ./server
```

Penjelasan: memasukkan binary server ke folder Soal 2.

Beri permission executable:

```bash
chmod +x server
```

Penjelasan: membuat file `server` dapat dijalankan.

Cek file bahan:

```bash
ls encrypted_storage/tests
```

Output yang diharapkan:

```text
notes.csv.enc
```

Cek server:

```bash
ls -l server
```

Penjelasan: memastikan file server tersedia dan executable.

## B. Compile FUSE dan client

```bash
gcc fuse.c $(pkg-config fuse3 --cflags --libs) -o fuse
```

Penjelasan: compile program FUSE Soal 2.

```bash
gcc client.c -o client
```

Penjelasan: compile program client.

Cek hasil compile:

```bash
ls
```

Output minimal yang diharapkan:

```text
client  fuse  server
```

## C. Jalankan FUSE dan cek decrypt

```bash
./fuse encrypted_storage fuse_mount -o allow_other -f > fuse.log 2>&1 &
```

Penjelasan: menjalankan FUSE dengan backend `encrypted_storage` dan mount point `fuse_mount`.

```bash
sleep 2
```

Penjelasan: menunggu proses mount selesai.

Cek tampilan file di mount point:

```bash
ls fuse_mount/tests
```

Output yang diharapkan:

```text
notes.csv
```

Penjelasan: file asli `notes.csv.enc` tampil sebagai `notes.csv`.

Cek isi decrypt:

```bash
cat fuse_mount/tests/notes.csv
```

Output yang diharapkan:

```text
author,notes
admin,TEST_SUCCESS
```

Penjelasan: membuktikan file `.enc` berhasil didekripsi saat dibaca melalui FUSE.

Tes tulis file baru:

```bash
echo "halo database" > fuse_mount/test.txt
```

Penjelasan: menulis file normal melalui mount point.

Baca file baru:

```bash
cat fuse_mount/test.txt
```

Output yang diharapkan:

```text
halo database
```

Cek file backend:

```bash
ls encrypted_storage
```

Output minimal:

```text
test.txt.enc  tests
```

Penjelasan: file yang ditulis dari mount point tersimpan sebagai `.enc`.

Cek isi file terenkripsi:

```bash
xxd encrypted_storage/test.txt.enc
```

Penjelasan: memastikan isi backend tidak terbaca sebagai teks normal.

## D. Jalankan Docker server dan client

Build Docker image:

```bash
sudo docker build -t soal-2-modul-4-sisop .
```

Penjelasan: membuat image Docker untuk server Soal 2.

Jalankan container server:

```bash
sudo docker run -d --name db_app -p 9000:9000 -v "$(pwd)/fuse_mount:/app/db" soal-2-modul-4-sisop
```

Penjelasan: menjalankan server pada port `9000` dan menghubungkan `fuse_mount` ke `/app/db`.

Cek container:

```bash
sudo docker ps
```

Penjelasan: memastikan container `db_app` berjalan.

Cek log server:

```bash
sudo docker logs db_app
```

Penjelasan: melihat output server.

Jalankan client:

```bash
./client 127.0.0.1 9000
```

Penjelasan: menghubungkan client ke server.

Di dalam client, ketik:

```text
HELP
```

Penjelasan: melihat daftar command server.

Keluar dari client:

```text
EXIT
```

Penjelasan: menutup client.

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

# 4. Cara Run Soal 3

Soal 3 menjalankan Samba server menggunakan Docker Compose, mengatur permission user, persistence data, dan logging.

## A. Siapkan folder volume

Masuk ke folder Soal 3:

```bash
cd soal_3
```

Penjelasan: masuk ke folder pengerjaan Soal 3.

Buat folder volume:

```bash
mkdir -p data/docs data/ebooks data/papers data/sourcecode logs
```

Penjelasan: membuat folder share dan folder log yang digunakan sebagai volume Docker.

Buat file log utama:

```bash
touch logs/libraryit.log
```

Penjelasan: membuat file log utama.

Beri permission executable pada entrypoint:

```bash
chmod +x entrypoint.sh
```

Penjelasan: agar script entrypoint dapat dijalankan oleh container.

Matikan Samba host:

```bash
sudo systemctl stop smbd nmbd 2>/dev/null || true
```

Penjelasan: mencegah port `445` bentrok dengan Samba di container.

## B. Build dan jalankan Docker Compose

Hapus container lama:

```bash
sudo docker rm -f libraryit-server libraryit-logger 2>/dev/null || true
```

Penjelasan: membersihkan container lama jika masih ada.

Matikan compose lama:

```bash
sudo docker compose down --remove-orphans 2>/dev/null || true
```

Penjelasan: menghentikan service lama dan membersihkan orphan container.

Build image:

```bash
sudo docker compose build --no-cache
```

Penjelasan: membangun ulang image Docker tanpa cache.

Jalankan container:

```bash
sudo docker compose up -d
```

Penjelasan: menjalankan Samba server dan logger di background.

Tunggu service siap:

```bash
sleep 5
```

Penjelasan: memberi waktu agar Samba selesai startup.

Cek container:

```bash
sudo docker ps
```

Output yang diharapkan:

```text
libraryit-server
libraryit-logger
```

## C. Tes user dan permission share

Tes daftar share sebagai member:

```bash
smbclient -L //127.0.0.1 -U member --password='member123' -m SMB3
```

Penjelasan: melihat share yang dapat diakses oleh user `member`.

Tes anonymous access:

```bash
smbclient -L //127.0.0.1 -N -m SMB3
```

Penjelasan: memastikan akses tanpa login ditolak.

Tes member baca docs:

```bash
smbclient //127.0.0.1/docs -U member --password='member123' -m SMB3 -c "ls"
```

Penjelasan: memastikan `member` dapat membaca share `docs`.

Tes contributor upload ke ebooks:

```bash
echo "test ebook" > ~/test_ebook.txt
```

```bash
smbclient //127.0.0.1/ebooks -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_ebook.txt test_ebook.txt; ls"
```

Penjelasan: memastikan `contributor` dapat menulis ke `ebooks`.

Tes contributor upload ke papers:

```bash
echo "test paper" > ~/test_paper.txt
```

```bash
smbclient //127.0.0.1/papers -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_paper.txt test_paper.txt; ls"
```

Penjelasan: memastikan `contributor` dapat menulis ke `papers`.

Tes librarian upload ke docs:

```bash
echo "dokumen librarian" > ~/test_docs.txt
```

```bash
smbclient //127.0.0.1/docs -U librarian --password='lib789' -m SMB3 -c "put $HOME/test_docs.txt test_docs.txt; ls"
```

Penjelasan: memastikan `librarian` dapat menulis ke `docs`.

Tes contributor tidak bisa upload ke docs:

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

Penjelasan: membuktikan `contributor` tidak memiliki hak tulis ke `docs`.

Tes member tidak bisa akses sourcecode:

```bash
smbclient //127.0.0.1/sourcecode -U member --password='member123' -m SMB3 -c "ls"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED
```

Penjelasan: membuktikan `member` tidak bisa mengakses share `sourcecode`.

Tes contributor upload ke sourcecode:

```bash
echo "int main() { return 0; }" > ~/test_sourcecode.c
```

```bash
smbclient //127.0.0.1/sourcecode -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_sourcecode.c test_sourcecode.c; ls"
```

Penjelasan: membuktikan `contributor` dapat menulis file kode ke share `sourcecode`.

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
data/sourcecode/test_sourcecode.c
```

Penjelasan: membuktikan file tersimpan di folder host `data`.

Cek log final:

```bash
cat logs/libraryit.log | tail -30
```

Penjelasan: menampilkan log aktivitas yang sudah diformat.

Cek log container logger:

```bash
sudo docker logs libraryit-logger --tail=30
```

Penjelasan: memastikan container logger berjalan.

Stop Soal 3:

```bash
sudo docker compose down
```

Penjelasan: menghentikan semua service Docker Compose.

Kembali ke root repository:

```bash
cd ..
```

---

# 5. Bersih-Bersih 1 Command

Jalankan dari folder utama repository.

```bash
bash -lc 'set +e; fusermount3 -u soal_1/mnt 2>/dev/null || true; fusermount3 -u soal_2/fuse_mount 2>/dev/null || true; sudo docker rm -f db_app libraryit-server libraryit-logger 2>/dev/null || true; (cd soal_3 && sudo docker compose down --remove-orphans 2>/dev/null || sudo docker-compose down --remove-orphans 2>/dev/null || true); rm -rf soal_1/amba_files soal_1/mnt soal_1/kenz_rescue soal_1/fuse.log soal_1/amba_files.zip; rm -f soal_2/fuse soal_2/client soal_2/fuse.log; sudo rm -rf soal_2/encrypted_storage/* soal_2/fuse_mount/*; sudo rm -rf soal_3/data/docs/* soal_3/data/ebooks/* soal_3/data/papers/* soal_3/data/sourcecode/* soal_3/logs/*; rm -f ~/main.py ~/test_sourcecode.c ~/test_ebook.txt ~/test_paper.txt ~/test_docs.txt ~/contributor_docs.txt; mkdir -p soal_2/encrypted_storage soal_2/fuse_mount soal_3/data/docs soal_3/data/ebooks soal_3/data/papers soal_3/data/sourcecode soal_3/logs; touch soal_3/logs/libraryit.log; sudo chown -R "$USER:$USER" soal_3/data soal_3/logs soal_2/encrypted_storage soal_2/fuse_mount 2>/dev/null || true; chmod 755 soal_2/encrypted_storage soal_2/fuse_mount soal_3/data soal_3/data/docs soal_3/data/ebooks soal_3/data/papers soal_3/data/sourcecode soal_3/logs; chmod 644 soal_3/logs/libraryit.log; echo "Cleanup selesai. Repository sudah bersih dari hasil runtime."'
```

Penjelasan: command ini melepas mount FUSE, menghentikan container, menghapus hasil compile, membersihkan file runtime, menghapus file testing, membuat ulang folder penting, dan memperbaiki permission folder.

Cek hasil akhir:

```bash
tree
```
