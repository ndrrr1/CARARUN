# README - Cara Run Lengkap Sesuai Soal Praktikum 4

Dokumen ini berisi cara menjalankan program secara runtut dan disesuaikan dengan alur soal, terutama untuk Soal 2 dan Soal 3 yang membutuhkan pembuktian lewat terminal/testing terpisah.

Jalankan semua command dari folder utama repository, yaitu folder yang berisi:

```text
soal_1  soal_2  soal_3
```

Contoh masuk ke folder repository:

```bash
cd ~/"Modul 4"
```

Jika nama folder berbeda, sesuaikan dengan lokasi folder repository masing-masing.

File tambahan yang harus sudah ada di `~/Downloads`:

```text
amba_files.zip
notes.csv.enc
server
```

Cek file tambahan:

```bash
ls ~/Downloads/amba_files.zip
ls ~/Downloads/notes.csv.enc
ls ~/Downloads/server
```

Penjelasan: tiga file tersebut digunakan sebagai bahan input tambahan. `amba_files.zip` digunakan untuk Soal 1, sedangkan `notes.csv.enc` dan `server` digunakan untuk Soal 2.

---

# 1. Requirement Awal

## A. Update package list

```bash
sudo apt update
```

Penjelasan: memperbarui daftar package agar dependency dapat diinstall dengan benar.

## B. Install dependency FUSE dan compiler

```bash
sudo apt install -y gcc pkg-config libfuse3-dev fuse3
```

Penjelasan: menginstall compiler C dan library FUSE3 yang dibutuhkan untuk Soal 1 dan Soal 2.

## C. Install Docker, Samba client, dan tools tambahan

```bash
sudo apt install -y docker.io smbclient cifs-utils tree zip unzip curl xxd
```

Penjelasan: menginstall Docker untuk container, `smbclient` untuk pengujian Samba, serta tools tambahan seperti `tree`, `unzip`, dan `xxd`.

## D. Aktifkan Docker

```bash
sudo systemctl enable --now docker
```

Penjelasan: memastikan service Docker aktif agar container dapat dijalankan.

## E. Aktifkan module FUSE

```bash
sudo modprobe fuse
```

Penjelasan: memastikan module FUSE aktif di kernel Linux.

## F. Aktifkan konfigurasi `allow_other`

```bash
sudo sed -i 's/^#user_allow_other/user_allow_other/' /etc/fuse.conf
```

Penjelasan: mengaktifkan opsi `user_allow_other` pada konfigurasi FUSE.

```bash
grep -q '^user_allow_other' /etc/fuse.conf || echo 'user_allow_other' | sudo tee -a /etc/fuse.conf
```

Penjelasan: memastikan baris `user_allow_other` benar-benar ada pada file konfigurasi FUSE.

## G. Cek Docker Compose

```bash
sudo docker compose version
```

Penjelasan: memastikan perintah `docker compose` tersedia untuk menjalankan Soal 3.

Jika command di atas error, jalankan:

```bash
sudo apt install -y docker-compose-plugin || sudo apt install -y docker-compose
```

---

# 2. Cara Run Soal 1

Soal 1 menjalankan FUSE untuk membaca file dari `amba_files.zip`, lalu membuat file virtual bernama `tujuan.txt`.

## A. Siapkan bahan Soal 1

Masuk ke folder Soal 1:

```bash
cd soal_1
```

Penjelasan: masuk ke folder pengerjaan Soal 1.

Lepaskan mount lama jika masih aktif:

```bash
fusermount3 -u mnt 2>/dev/null || true
```

Penjelasan: melepas mount FUSE lama agar tidak terjadi error saat mount ulang.

Bersihkan hasil run lama:

```bash
rm -rf amba_files mnt kenz_rescue fuse.log amba_files.zip
```

Penjelasan: menghapus folder hasil unzip, mount point lama, executable lama, log lama, dan copy ZIP lama.

Copy bahan dari Downloads:

```bash
cp ~/Downloads/amba_files.zip .
```

Penjelasan: menyalin file `amba_files.zip` ke folder Soal 1.

Buat mount point:

```bash
mkdir -p mnt
```

Penjelasan: membuat folder `mnt` sebagai tempat hasil mount FUSE.

Unzip bahan:

```bash
unzip amba_files.zip
```

Penjelasan: mengekstrak file `amba_files.zip` sehingga terbentuk folder `amba_files`.

Cek isi folder bahan:

```bash
ls amba_files
```

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt
```

Penjelasan: memastikan semua file bahan berhasil diekstrak.

Cek salah satu file:

```bash
cat amba_files/1.txt
```

Penjelasan: memastikan file bahan dapat dibaca dan memiliki format koordinat.

## B. Compile program Soal 1

```bash
gcc kenz_rescue.c $(pkg-config fuse3 --cflags --libs) -o kenz_rescue
```

Penjelasan: compile program `kenz_rescue.c` menjadi executable bernama `kenz_rescue`.

## C. Jalankan FUSE Soal 1

Jalankan FUSE:

```bash
./kenz_rescue amba_files mnt -f > fuse.log 2>&1 &
```

Penjelasan: menjalankan FUSE dengan folder sumber `amba_files` dan mount point `mnt`. Proses dijalankan di background agar terminal masih dapat digunakan untuk testing.

Tunggu proses mount:

```bash
sleep 2
```

Penjelasan: memberi waktu agar proses mount selesai.

## D. Cek hasil Soal 1

Cek isi mount point:

```bash
ls mnt
```

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt  tujuan.txt
```

Penjelasan: file `tujuan.txt` harus muncul di mount point sebagai file virtual.

Baca file tujuan:

```bash
cat mnt/tujuan.txt
```

Output yang diharapkan:

```text
Tujuan Mas Amba: -7.957382728443728,112.4698688227961,23:59 WIB
```

Penjelasan: output ini membuktikan program berhasil menggabungkan potongan koordinat dari file `1.txt` sampai `7.txt`.

Pastikan `tujuan.txt` tidak ada di folder asli:

```bash
ls amba_files
```

Penjelasan: `tujuan.txt` harus hanya muncul di mount point, bukan di folder asli.

Stop FUSE Soal 1:

```bash
fusermount3 -u mnt
```

Penjelasan: melepas mount FUSE setelah testing selesai.

Kembali ke root repository:

```bash
cd ..
```

---

# 3. Cara Run Soal 2

Soal 2 menjalankan mini database service dengan FUSE terenkripsi. Folder asli adalah `encrypted_storage`, sedangkan folder yang dibaca normal adalah `fuse_mount`. File di backend disimpan sebagai `.enc`, sedangkan di mount point tampil sebagai file normal.

Soal 2 dijalankan dengan 2 terminal agar sesuai alur testing FUSE:  
Terminal 1 digunakan untuk menjalankan FUSE secara foreground, sedangkan Terminal 2 digunakan untuk testing FUSE, Docker, dan client.

---

## A. Persiapan Soal 2

Jalankan bagian ini di salah satu terminal terlebih dahulu.

Masuk ke folder Soal 2:

```bash
cd soal_2
```

Penjelasan: masuk ke folder pengerjaan Soal 2.

Lepaskan mount lama jika masih aktif:

```bash
fusermount3 -u fuse_mount 2>/dev/null || true
```

Penjelasan: melepas mount FUSE lama agar tidak bentrok.

Hapus container lama jika masih ada:

```bash
sudo docker rm -f db_app 2>/dev/null || true
```

Penjelasan: menghapus container lama bernama `db_app`.

Bersihkan file hasil compile lama:

```bash
rm -f fuse client fuse.log
```

Penjelasan: menghapus executable lama dan log lama.

Bersihkan folder runtime lama:

```bash
rm -rf encrypted_storage/* fuse_mount/*
```

Penjelasan: membersihkan isi backend storage dan mount point dari hasil run sebelumnya.

Buat folder backend untuk file terenkripsi:

```bash
mkdir -p encrypted_storage/tests
```

Penjelasan: membuat folder backend tempat file `.enc` disimpan.

Buat folder mount point:

```bash
mkdir -p fuse_mount
```

Penjelasan: membuat folder yang akan digunakan sebagai mount point FUSE.

Copy file terenkripsi dari Downloads:

```bash
cp ~/Downloads/notes.csv.enc encrypted_storage/tests/notes.csv.enc
```

Penjelasan: memasukkan file `notes.csv.enc` ke backend agar dapat tampil sebagai `notes.csv` melalui FUSE.

Copy binary server dari Downloads:

```bash
cp ~/Downloads/server ./server
```

Penjelasan: memasukkan binary server ke folder Soal 2 agar dapat digunakan oleh Docker.

Beri permission executable pada server:

```bash
chmod +x server
```

Penjelasan: membuat file `server` dapat dijalankan.

Cek file terenkripsi:

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

Penjelasan: memastikan file `server` ada dan executable.

---

## B. Compile FUSE dan client

Compile FUSE:

```bash
gcc fuse.c $(pkg-config fuse3 --cflags --libs) -o fuse
```

Penjelasan: compile program FUSE Soal 2 menjadi executable `fuse`.

Compile client:

```bash
gcc client.c -o client
```

Penjelasan: compile program client untuk koneksi TCP ke server database.

Cek hasil compile:

```bash
ls
```

Output minimal yang diharapkan:

```text
client  fuse  server
```

---

## C. Terminal 1 - Jalankan FUSE Soal 2

Buka Terminal 1, lalu masuk ke folder Soal 2:

```bash
cd ~/"Modul 4"/soal_2
```

Jika nama folder repository berbeda, sesuaikan path-nya.

Jalankan FUSE secara foreground:

```bash
./fuse encrypted_storage fuse_mount -o allow_other -f
```

Penjelasan: command ini menjalankan FUSE secara foreground. Terminal ini harus tetap dibiarkan hidup selama testing Soal 2 berlangsung.

Jangan tutup Terminal 1 sebelum semua testing Soal 2 selesai.

---

## D. Terminal 2 - Testing FUSE Soal 2

Buka Terminal 2, lalu masuk ke folder Soal 2:

```bash
cd ~/"Modul 4"/soal_2
```

Jika nama folder repository berbeda, sesuaikan path-nya.

Cek file yang muncul di mount point:

```bash
ls fuse_mount/tests
```

Output yang diharapkan:

```text
notes.csv
```

Penjelasan: file asli bernama `notes.csv.enc`, tetapi di mount point tampil sebagai `notes.csv`.

Baca file hasil decrypt:

```bash
cat fuse_mount/tests/notes.csv
```

Output yang diharapkan:

```text
author,notes
admin,TEST_SUCCESS
```

Penjelasan: membuktikan bahwa file `.enc` berhasil didekripsi saat dibaca melalui FUSE.

Tes tulis file baru lewat mount point:

```bash
echo "halo database" > fuse_mount/test.txt
```

Penjelasan: menulis file normal ke mount point.

Baca file baru:

```bash
cat fuse_mount/test.txt
```

Output yang diharapkan:

```text
halo database
```

Cek backend storage:

```bash
ls encrypted_storage
```

Output minimal yang diharapkan:

```text
test.txt.enc  tests
```

Penjelasan: membuktikan file yang ditulis melalui mount point tersimpan sebagai file `.enc` di backend.

Cek isi terenkripsi:

```bash
xxd encrypted_storage/test.txt.enc
```

Penjelasan: isi file backend tidak boleh terbaca sebagai teks normal.

---

## E. Terminal 2 - Testing Operasi FUSE Lengkap

Buat folder baru di mount point:

```bash
mkdir fuse_mount/demo_dir
```

Penjelasan: menguji operasi `mkdir`.

Buat file baru di dalam folder:

```bash
echo "abcde" > fuse_mount/demo_dir/sample.txt
```

Penjelasan: menguji operasi `create` dan `write`.

Baca file:

```bash
cat fuse_mount/demo_dir/sample.txt
```

Output yang diharapkan:

```text
abcde
```

Penjelasan: menguji operasi `open` dan `read`.

Cek metadata file:

```bash
stat fuse_mount/demo_dir/sample.txt
```

Penjelasan: menguji operasi metadata seperti `getattr`.

Potong isi file menjadi 3 karakter:

```bash
truncate -s 3 fuse_mount/demo_dir/sample.txt
```

Penjelasan: menguji operasi `truncate`.

Baca ulang file:

```bash
cat fuse_mount/demo_dir/sample.txt
```

Output yang diharapkan:

```text
abc
```

Update timestamp file:

```bash
touch fuse_mount/demo_dir/sample.txt
```

Penjelasan: menguji operasi update waktu file atau `utimens`.

Hapus file:

```bash
rm fuse_mount/demo_dir/sample.txt
```

Penjelasan: menguji operasi `unlink`.

Hapus folder:

```bash
rmdir fuse_mount/demo_dir
```

Penjelasan: menguji operasi `rmdir`.

---

## F. Terminal 2 - Build dan Jalankan Docker Server

Build Docker image:

```bash
sudo docker build -t soal-2-modul-4-sisop .
```

Penjelasan: membuat Docker image untuk server database Soal 2.

Jalankan container `db_app`:

```bash
sudo docker run -d --name db_app -p 9000:9000 -v "$(pwd)/fuse_mount:/app/db" soal-2-modul-4-sisop
```

Penjelasan: menjalankan container server pada port `9000` dan melakukan bind mount `fuse_mount` ke `/app/db`.

Cek container:

```bash
sudo docker ps
```

Output yang diharapkan terdapat:

```text
db_app
```

Cek log server:

```bash
sudo docker logs db_app
```

Penjelasan: memastikan server berjalan dan listening pada port `9000`.

---

## G. Terminal 2 - Jalankan Client dan Test Command Database

Jalankan client:

```bash
./client 127.0.0.1 9000
```

Penjelasan: menghubungkan client ke server database melalui TCP connection pada port `9000`.

Di dalam client, jalankan command berikut satu per satu:

```text
HELP
```

Penjelasan: menampilkan daftar command yang tersedia.

```text
CREATE DATABASE demo
```

Penjelasan: membuat database bernama `demo`.

```text
CREATE TABLE demo users name password
```

Penjelasan: membuat table `users` di database `demo` dengan kolom `name` dan `password`.

```text
INSERT demo users admin 12345
```

Penjelasan: memasukkan data ke table `users`.

```text
LIST DATABASE
```

Penjelasan: menampilkan daftar database.

```text
LIST TABLE demo
```

Penjelasan: menampilkan daftar table dalam database `demo`.

```text
SELECT demo users
```

Penjelasan: menampilkan isi table `users`.

```text
UPDATE demo users admin root
```

Penjelasan: mengubah data lama `admin` menjadi `root`.

```text
SELECT demo users
```

Penjelasan: memastikan data sudah berubah.

```text
DELETE demo users root
```

Penjelasan: menghapus data dengan key `root`.

```text
SELECT demo users
```

Penjelasan: memastikan data sudah terhapus.

```text
DROP DATABASE demo
```

Penjelasan: menghapus database `demo`.

```text
EXIT
```

Penjelasan: keluar dari client.

---

## H. Stop Soal 2

Di Terminal 2, hentikan container server:

```bash
sudo docker rm -f db_app
```

Penjelasan: menghentikan dan menghapus container `db_app`.

Di Terminal 1, tekan:

```text
CTRL + C
```

Penjelasan: menghentikan proses FUSE yang berjalan secara foreground.

Kembali ke Terminal 2, lepaskan mount FUSE jika masih aktif:

```bash
fusermount3 -u fuse_mount 2>/dev/null || true
```

Penjelasan: memastikan mount FUSE sudah benar-benar dilepas.

Kembali ke root repository:

```bash
cd ..
```

---

# 4. Cara Run Soal 3

Soal 3 menjalankan Samba server dengan Docker Compose. Terdapat dua container utama, yaitu `libraryit-server` untuk Samba server dan `libraryit-logger` untuk logger. Testing dilakukan dengan dua terminal agar logger dapat dipantau secara real-time.

Folder koleksi yang digunakan:

```text
docs
ebooks
papers
sourcecode
```

User yang digunakan:

```text
member
contributor
librarian
```

Group yang digunakan:

```text
readonly
staff
```

---

## A. Persiapan Soal 3

Masuk ke folder Soal 3:

```bash
cd soal_3
```

Penjelasan: masuk ke folder pengerjaan Soal 3.

Buat folder volume:

```bash
mkdir -p data/docs data/ebooks data/papers data/sourcecode logs
```

Penjelasan: membuat folder koleksi dan folder log sebagai volume yang tersimpan di host.

Buat file log utama:

```bash
touch logs/libraryit.log
```

Penjelasan: membuat file log utama untuk mencatat aktivitas.

Beri permission executable pada entrypoint:

```bash
chmod +x entrypoint.sh
```

Penjelasan: membuat script `entrypoint.sh` dapat dijalankan oleh container.

Matikan Samba host agar port 445 tidak bentrok:

```bash
sudo systemctl stop smbd nmbd 2>/dev/null || true
```

Penjelasan: menghentikan Samba bawaan host agar port `445` dapat digunakan container.

---

## B. Build dan Jalankan Docker Compose

Hapus container lama jika masih ada:

```bash
sudo docker rm -f libraryit-server libraryit-logger 2>/dev/null || true
```

Penjelasan: membersihkan container lama agar tidak bentrok.

Matikan compose lama:

```bash
sudo docker compose down --remove-orphans 2>/dev/null || true
```

Penjelasan: menghentikan service lama dan membersihkan orphan container.

Build image tanpa cache:

```bash
sudo docker compose build --no-cache
```

Penjelasan: membangun ulang image agar konfigurasi terbaru digunakan.

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

---

## C. Cek User dan Group di Container

Cek user Samba:

```bash
sudo docker exec libraryit-server pdbedit -L
```

Output yang diharapkan memuat user:

```text
member
contributor
librarian
```

Penjelasan: memastikan user Samba otomatis dibuat oleh container.

Cek group `readonly`:

```bash
sudo docker exec libraryit-server getent group readonly
```

Penjelasan: memastikan group `readonly` ada.

Cek group `staff`:

```bash
sudo docker exec libraryit-server getent group staff
```

Penjelasan: memastikan group `staff` ada.

---

## D. Terminal 1 - Pantau Logger Real-Time

Buka Terminal 1, lalu masuk ke folder Soal 3:

```bash
cd ~/"Modul 4"/soal_3
```

Jika nama folder repository berbeda, sesuaikan path-nya.

Jalankan logger real-time:

```bash
sudo docker logs -f libraryit-logger
```

Penjelasan: terminal ini digunakan untuk memantau log aktivitas secara real-time. Biarkan terminal ini tetap hidup selama testing Samba dilakukan di Terminal 2.

---

## E. Terminal 2 - Testing Share dan Permission

Buka Terminal 2, lalu masuk ke folder Soal 3:

```bash
cd ~/"Modul 4"/soal_3
```

Jika nama folder repository berbeda, sesuaikan path-nya.

Tes daftar share sebagai `member`:

```bash
smbclient -L //127.0.0.1 -U member --password='member123' -m SMB3
```

Output yang diharapkan memuat:

```text
docs
ebooks
papers
IPC$
```

Penjelasan: user `member` dapat melihat share yang diperbolehkan.

Tes anonymous access:

```bash
smbclient -L //127.0.0.1 -N -m SMB3
```

Output yang diharapkan: akses anonymous gagal.

Penjelasan: memastikan akses tanpa login tidak diperbolehkan.

Tes `member` baca `docs`:

```bash
smbclient //127.0.0.1/docs -U member --password='member123' -m SMB3 -c "ls"
```

Penjelasan: memastikan `member` dapat membaca share `docs`.

Tes `member` tidak bisa menulis ke `docs`:

```bash
echo "test member" > ~/member_test.txt
```

```bash
smbclient //127.0.0.1/docs -U member --password='member123' -m SMB3 -c "put $HOME/member_test.txt member_test.txt"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED
```

Penjelasan: membuktikan `member` bersifat read-only.

Tes `contributor` upload ke `ebooks`:

```bash
echo "test ebook" > ~/test_ebook.txt
```

```bash
smbclient //127.0.0.1/ebooks -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_ebook.txt test_ebook.txt; ls"
```

Penjelasan: membuktikan `contributor` dapat menulis ke share `ebooks`.

Tes `contributor` upload ke `papers`:

```bash
echo "test paper" > ~/test_paper.txt
```

```bash
smbclient //127.0.0.1/papers -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_paper.txt test_paper.txt; ls"
```

Penjelasan: membuktikan `contributor` dapat menulis ke share `papers`.

Tes `librarian` upload ke `docs`:

```bash
echo "dokumen librarian" > ~/test_docs.txt
```

```bash
smbclient //127.0.0.1/docs -U librarian --password='lib789' -m SMB3 -c "put $HOME/test_docs.txt test_docs.txt; ls"
```

Penjelasan: membuktikan hanya `librarian` yang dapat menulis ke share `docs`.

Tes `contributor` tidak bisa upload ke `docs`:

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

Penjelasan: membuktikan `contributor` tidak memiliki hak tulis pada share `docs`.

Tes `member` tidak bisa akses `sourcecode`:

```bash
smbclient //127.0.0.1/sourcecode -U member --password='member123' -m SMB3 -c "ls"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED
```

Penjelasan: membuktikan `member` tidak dapat mengakses share `sourcecode`.

Tes `contributor` upload ke `sourcecode`:

```bash
echo "int main() { return 0; }" > ~/test_sourcecode.c
```

```bash
smbclient //127.0.0.1/sourcecode -U contributor --password='contrib456' -m SMB3 -c "put $HOME/test_sourcecode.c test_sourcecode.c; ls"
```

Penjelasan: membuktikan `contributor` dapat menulis file kode ke share `sourcecode`.

---

## F. Terminal 2 - Cek Persistence Data

Cek file yang tersimpan di host:

```bash
sudo find data -type f
```

Output yang diharapkan memuat:

```text
data/ebooks/test_ebook.txt
data/papers/test_paper.txt
data/docs/test_docs.txt
data/sourcecode/test_sourcecode.c
```

Penjelasan: membuktikan data tersimpan di folder host `data`, bukan hanya di dalam container.

Cek permission folder penting:

```bash
sudo ls -ld data/docs data/sourcecode
```

Penjelasan: melihat permission folder `docs` dan `sourcecode` dari sisi host.

Tes host tidak dapat menulis langsung ke `docs` tanpa aturan Samba:

```bash
touch data/docs/test_dari_host.txt
```

Output yang diharapkan:

```text
Permission denied
```

Penjelasan: membuktikan modifikasi data seharusnya dilakukan melalui akses Samba, bukan langsung dari host biasa.

Jika file tetap berhasil dibuat karena permission host berbeda, hapus file uji tersebut:

```bash
rm -f data/docs/test_dari_host.txt
```

---

## G. Terminal 2 - Cek Logging

Cek log final:

```bash
cat logs/libraryit.log | tail -30
```

Penjelasan: menampilkan 30 baris terakhir log aktivitas.

Cek log dari container logger:

```bash
sudo docker logs libraryit-logger --tail=30
```

Penjelasan: memastikan container logger mencatat aktivitas Samba.

Format log yang diharapkan:

```text
[YYYY-MM-DD HH:MM:SS] [INFO] [username] [ACTION] [target]
[YYYY-MM-DD HH:MM:SS] [WARNING] [username] [DENIED] [target]
```

Penjelasan: log harus mencatat aktivitas yang berhasil dan aktivitas yang ditolak.

---

## H. Stop Soal 3

Di Terminal 1, tekan:

```text
CTRL + C
```

Penjelasan: menghentikan mode follow log real-time.

Di Terminal 2, hentikan Docker Compose:

```bash
sudo docker compose down
```

Penjelasan: menghentikan container `libraryit-server` dan `libraryit-logger`.

Kembali ke root repository:

```bash
cd ..
```

---

# 5. Bersih-Bersih Semua Hasil Run

Command ini dijalankan dari folder utama repository, yaitu folder yang berisi `soal_1`, `soal_2`, dan `soal_3`.

```bash
bash -lc 'set +e; fusermount3 -u soal_1/mnt 2>/dev/null || true; fusermount3 -u soal_2/fuse_mount 2>/dev/null || true; sudo docker rm -f db_app libraryit-server libraryit-logger 2>/dev/null || true; (cd soal_3 && sudo docker compose down --remove-orphans 2>/dev/null || sudo docker-compose down --remove-orphans 2>/dev/null || true); rm -rf soal_1/amba_files soal_1/mnt soal_1/kenz_rescue soal_1/fuse.log soal_1/amba_files.zip; rm -f soal_2/fuse soal_2/client soal_2/fuse.log; sudo rm -rf soal_2/encrypted_storage/* soal_2/fuse_mount/*; sudo rm -rf soal_3/data/docs/* soal_3/data/ebooks/* soal_3/data/papers/* soal_3/data/sourcecode/* soal_3/logs/*; rm -f ~/main.py ~/test_sourcecode.c ~/test_ebook.txt ~/test_paper.txt ~/test_docs.txt ~/contributor_docs.txt ~/member_test.txt; mkdir -p soal_2/encrypted_storage soal_2/fuse_mount soal_3/data/docs soal_3/data/ebooks soal_3/data/papers soal_3/data/sourcecode soal_3/logs; touch soal_3/logs/libraryit.log; sudo chown -R "$USER:$USER" soal_3/data soal_3/logs soal_2/encrypted_storage soal_2/fuse_mount 2>/dev/null || true; chmod 755 soal_2/encrypted_storage soal_2/fuse_mount soal_3/data soal_3/data/docs soal_3/data/ebooks soal_3/data/papers soal_3/data/sourcecode soal_3/logs; chmod 644 soal_3/logs/libraryit.log; echo "Cleanup selesai. Repository sudah bersih dari hasil runtime."'
```

Penjelasan: command ini melepas mount FUSE, menghentikan container, menghapus file hasil compile, membersihkan file runtime, menghapus file testing, membuat ulang folder penting, dan memperbaiki permission folder.

Cek struktur akhir:

```bash
tree
```

---

# 6. Catatan Demo

Untuk Soal 2:

```text
Terminal 1 = menjalankan FUSE foreground
Terminal 2 = testing FUSE, Docker server, dan client
```

Untuk Soal 3:

```text
Terminal 1 = memantau logger real-time
Terminal 2 = testing Samba user, permission, persistence, dan log
```

File berikut tetap aman di `~/Downloads` karena cleanup tidak menghapus isi Downloads:

```text
~/Downloads/amba_files.zip
~/Downloads/notes.csv.enc
~/Downloads/server
```
