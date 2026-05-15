# README - Cara Run Persis Alur Soal Praktikum 4

README ini dibuat agar cara run dan hasil yang ditampilkan mengikuti contoh pada soal.

Jalankan dari folder utama repository yang berisi:

```text
soal_1  soal_2  soal_3
```

Contoh:

```bash
cd ~/"Modul 4"
```

File bahan tambahan yang dipakai saat run disimpan di `~/Downloads`:

```text
amba_files.zip
notes.csv.enc
```

File `server` untuk Soal 2 sudah berada di dalam repository:

```text
soal_2/server
```

Cek dulu dari root repository:

```bash
ls ~/Downloads/amba_files.zip
ls ~/Downloads/notes.csv.enc
ls soal_2/server
```

---

# Requirement Awal

```bash
sudo apt update
```

```bash
sudo apt install -y gcc pkg-config libfuse3-dev fuse3
```

```bash
sudo apt install -y docker.io smbclient cifs-utils tree zip unzip curl xxd
```

```bash
sudo systemctl enable --now docker
```

```bash
sudo modprobe fuse
```

```bash
sudo sed -i 's/^#user_allow_other/user_allow_other/' /etc/fuse.conf
```

```bash
grep -q '^user_allow_other' /etc/fuse.conf || echo 'user_allow_other' | sudo tee -a /etc/fuse.conf
```

Penjelasan: requirement ini menyiapkan compiler C, FUSE3, Docker, Samba client, dan tools testing.

---

# Soal 1 - Save Asisten Kenz

Target Soal 1:
- unzip `amba_files.zip`;
- setelah unzip, file ZIP tidak boleh tersisa di working directory;
- hasil unzip berupa folder `amba_files/` berisi `1.txt` sampai `7.txt`;
- program `kenz_rescue.c` menerima argumen `<source_directory>` dan `<mount_directory>`;
- file `1.txt` sampai `7.txt` muncul di mount point dan isinya sama persis dengan source;
- file virtual `tujuan.txt` hanya muncul di mount point, bukan di `amba_files/`;
- isi `tujuan.txt` dibuat on-the-fly dari fragmen `KOORD:`.

## A. Persiapan Soal 1

```bash
cd soal_1
```

```bash
fusermount3 -u mnt 2>/dev/null || fusermount -u mnt 2>/dev/null || true
```

```bash
rm -rf amba_files mnt kenz_rescue fuse.log amba_files.zip
```

```bash
cp ~/Downloads/amba_files.zip .
```

```bash
unzip amba_files.zip
```

```bash
rm -f amba_files.zip
```

```bash
mkdir -p mnt
```

```bash
ls
```

Output yang diharapkan tidak lagi menampilkan `amba_files.zip`.

```text
amba_files  kenz_rescue.c  mnt
```

```bash
ls amba_files
```

Output:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt
```

Penjelasan: bagian ini mengikuti soal karena ZIP harus diekstrak lalu dihapus dari working directory.

## B. Compile Soal 1

```bash
gcc kenz_rescue.c $(pkg-config fuse3 --cflags --libs) -o kenz_rescue
```

Penjelasan: menghasilkan executable `kenz_rescue`.

## C. Mount FUSE Soal 1

```bash
./kenz_rescue amba_files mnt
```

Penjelasan: menjalankan program dengan dua argumen, yaitu source directory `amba_files` dan mount directory `mnt`.

## D. Cek passthrough file

```bash
ls mnt
```

Output yang diharapkan:

```text
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  7.txt  tujuan.txt
```

```bash
cat mnt/1.txt
```

Penjelasan: output harus sama seperti `cat amba_files/1.txt`.

Cek semua file source sama dengan file mount:

```bash
for i in 1 2 3 4 5 6 7; do diff mnt/$i.txt amba_files/$i.txt && echo "$i.txt OK"; done
```

Output yang diharapkan:

```text
1.txt OK
2.txt OK
3.txt OK
4.txt OK
5.txt OK
6.txt OK
7.txt OK
```

## E. Cek file virtual `tujuan.txt`

```bash
ls mnt
```

```bash
ls amba_files
```

Penjelasan: `tujuan.txt` harus ada di `mnt`, tetapi tidak boleh ada di `amba_files`.

```bash
stat mnt/tujuan.txt
```

Penjelasan: ukuran `tujuan.txt` konsisten saat dicek dengan `stat`.

```bash
cat mnt/tujuan.txt
```

Output yang diharapkan:

```text
Tujuan Mas Amba: -7.957382728443728,112.4698688227961,23:59 WIB
```

```bash
wc -c mnt/tujuan.txt
```

Penjelasan: command ini mengecek ukuran file virtual `tujuan.txt`. Angka byte dapat berbeda tergantung apakah program menambahkan newline atau tidak, tetapi ukurannya harus konsisten saat dicek ulang.

```bash
ls amba_files/tujuan.txt 2>&1
```

Output yang diharapkan:

```text
ls: cannot access 'amba_files/tujuan.txt': No such file or directory
```

Penjelasan: bagian ini membuktikan `tujuan.txt` dibuat virtual/on-the-fly di mount point.

## F. Unmount Soal 1

```bash
fusermount3 -u mnt 2>/dev/null || fusermount -u mnt
```

```bash
cd ..
```

---

# Soal 2 - Pec MOO

Target Soal 2:
- mini database service berjalan pada TCP port `9000`;
- client dapat menjalankan command database;
- FUSE menghubungkan `encrypted_storage` sebagai direktori asli dan `fuse_mount` sebagai mount point;
- file/folder di `fuse_mount` terlihat normal;
- file/folder di `encrypted_storage` tersimpan terenkripsi dengan XOR key `0x76`;
- nama file di backend ditambah `.enc`;
- `notes.csv.enc` di `encrypted_storage/tests` harus tampil sebagai `notes.csv` di `fuse_mount/tests`;
- Docker image bernama `soal-2-modul-4-sisop:latest`;
- container bernama `db_app`;
- bind mount `fuse_mount` ke `/app/db`.

Soal 2 dijalankan dengan 2 terminal.

---

## A. Persiapan Soal 2

Jalankan di salah satu terminal.

```bash
cd soal_2
```

```bash
fusermount3 -u fuse_mount 2>/dev/null || fusermount -u fuse_mount 2>/dev/null || true
```

```bash
sudo docker rm -f db_app 2>/dev/null || true
```

```bash
rm -f fuse client fuse.log
```

```bash
rm -rf encrypted_storage/* fuse_mount/*
```

```bash
mkdir -p encrypted_storage/tests
```

```bash
mkdir -p fuse_mount
```

```bash
cp ~/Downloads/notes.csv.enc encrypted_storage/tests/notes.csv.enc
```

```bash
cp ~/Downloads/server ./server
```

```bash
chmod +x server
```

```bash
ls encrypted_storage/tests
```

Output:

```text
notes.csv.enc
```

```bash
ls -l server
```

Penjelasan: file release dari soal disiapkan untuk testing FUSE dan Docker server.

## B. Compile Soal 2

```bash
gcc fuse.c $(pkg-config fuse3 --cflags --libs) -o fuse
```

```bash
gcc client.c -o client
```

```bash
ls
```

Output minimal:

```text
client  fuse  server
```

---

## C. Terminal 1 - Jalankan FUSE Foreground

Buka Terminal 1.

```bash
cd ~/"Modul 4"/soal_2
```

Jika folder repo berbeda, sesuaikan path.

```bash
./fuse encrypted_storage fuse_mount -o allow_other -f
```

Penjelasan: FUSE berjalan di foreground. Terminal 1 jangan ditutup selama testing Soal 2.

---

## D. Terminal 2 - Testing FUSE seperti Soal

Buka Terminal 2.

```bash
cd ~/"Modul 4"/soal_2
```

Jika folder repo berbeda, sesuaikan path.

Cek mount aktif:

```bash
mount | grep fuse_mount
```

Buat file di `fuse_mount`:

```bash
echo "isinya ini harusnya" > fuse_mount/file1.txt
```

Buat folder di `fuse_mount`:

```bash
mkdir fuse_mount/halo
```

Buat file di dalam folder:

```bash
echo "isinya ini harusnya" > fuse_mount/halo/file2.txt
```

Cek isi mount:

```bash
tree fuse_mount
```

Contoh output:

```text
fuse_mount
├── file1.txt
├── halo
│   └── file2.txt
└── tests
    └── notes.csv
```

Cek backend terenkripsi:

```bash
tree encrypted_storage
```

Contoh output:

```text
encrypted_storage
├── file1.txt.enc
├── halo
│   └── file2.txt.enc
└── tests
    └── notes.csv.enc
```

Cek isi file backend tidak terbaca normal:

```bash
cat encrypted_storage/file1.txt.enc
```

Penjelasan: output harus berupa teks tidak terbaca normal karena terenkripsi XOR.

Cek isi file lewat mount terbaca normal:

```bash
cat fuse_mount/file1.txt
```

Output:

```text
isinya ini harusnya
```

```bash
cat fuse_mount/halo/file2.txt
```

Output:

```text
isinya ini harusnya
```

Cek file checker dari soal:

```bash
ls fuse_mount/tests
```

Output:

```text
notes.csv
```

```bash
cat fuse_mount/tests/notes.csv
```

Output yang diharapkan:

```text
author,notes
admin,TEST_SUCCESS
```

Penjelasan: bagian ini membuktikan file `.enc` di backend tampil tanpa `.enc` dan terbaca normal di mount point.

## E. Testing operasi FUSE tambahan

```bash
stat fuse_mount/file1.txt
```

```bash
truncate -s 5 fuse_mount/file1.txt
```

```bash
cat fuse_mount/file1.txt
```

```bash
touch fuse_mount/file1.txt
```

```bash
rm fuse_mount/halo/file2.txt
```

```bash
rmdir fuse_mount/halo
```

Penjelasan: command ini membuktikan operasi `getattr`, `truncate`, `utimens`, `unlink`, dan `rmdir`.

---

## F. Terminal 2 - Docker Image dan Container

Build image:

```bash
sudo docker build -t soal-2-modul-4-sisop:latest .
```

Cek image:

```bash
sudo docker images
```

Output yang diharapkan memuat:

```text
soal-2-modul-4-sisop   latest
```

Jalankan container:

```bash
sudo docker run -d --name db_app -p 9000:9000 -v "$(pwd)/fuse_mount:/app/db" soal-2-modul-4-sisop:latest
```

Cek container:

```bash
sudo docker ps
```

Output yang diharapkan memuat:

```text
db_app
0.0.0.0:9000->9000/tcp
```

Penjelasan: container `db_app` berjalan dan port `9000` terbuka.

---

## G. Terminal 2 - Client Database

Jalankan client:

```bash
./client 127.0.0.1 9000
```

Di dalam client, jalankan command berikut satu per satu:

```text
HELP
```

```text
CREATE DATABASE tests
```

```text
CREATE TABLE tests users email password
```

```text
INSERT tests users admin@mail.com rahasia
```

```text
LIST DATABASE
```

```text
LIST TABLE tests
```

```text
SELECT tests users
```

```text
UPDATE tests users admin@mail.com root@mail.com
```

```text
SELECT tests users
```

```text
DELETE tests users root@mail.com
```

```text
SELECT tests users
```

```text
DROP DATABASE tests
```

```text
EXIT
```

Penjelasan: alur ini mengikuti command database yang tersedia pada soal.

Cek file database tersimpan melalui mount:

```bash
tree fuse_mount
```

Cek backend tetap terenkripsi:

```bash
tree encrypted_storage
```

---

## H. Stop Soal 2

Di Terminal 2:

```bash
sudo docker rm -f db_app
```

Di Terminal 1 tekan:

```text
CTRL + C
```

Di Terminal 2:

```bash
fusermount3 -u fuse_mount 2>/dev/null || fusermount -u fuse_mount 2>/dev/null || true
```

```bash
cd ..
```

---

# Soal 3 - LibraryIT

Target Soal 3:
- service utama bernama `libraryit-server`;
- service logger bernama `libraryit-logger`;
- Samba berjalan di port host `1445`;
- folder koleksi berada di `/libraryit` dalam container dan `./data` di host;
- user otomatis: `member`, `contributor`, `librarian`;
- password:
  - `member123`
  - `contrib456`
  - `lib789`
- group otomatis:
  - `readonly`: member
  - `staff`: contributor dan librarian
- share:
  - `ebooks` dan `papers`: staff bisa baca/tulis, readonly hanya baca;
  - `sourcecode`: readonly tidak boleh melihat/mengakses; staff dapat melihat tetapi tidak dapat menulis;
  - `docs`: semua dapat membaca, hanya librarian yang boleh menulis;
- data persistent di host;
- `docs` tidak boleh dimodifikasi langsung dari host;
- `sourcecode` di host permission `750`;
- logger real-time format:
  `[YYYY-MM-DD HH:MM:SS] [LEVEL] [USERNAME] [AKSI] [NAMA FILE/SHARE]`.

Soal 3 dijalankan dengan 2 terminal.

---

## A. Persiapan Soal 3

```bash
cd soal_3
```

```bash
mkdir -p data/docs data/ebooks data/papers data/sourcecode logs
```

```bash
touch logs/libraryit.log
```

```bash
chmod +x entrypoint.sh
```

```bash
sudo systemctl stop smbd nmbd 2>/dev/null || true
```

Penjelasan: Samba host dimatikan agar tidak bentrok dengan port container.

## B. Build dan Run Docker Compose

```bash
sudo docker rm -f libraryit-server libraryit-logger 2>/dev/null || true
```

```bash
sudo docker compose down --remove-orphans 2>/dev/null || true
```

```bash
sudo docker compose build --no-cache
```

```bash
sudo docker compose up -d
```

```bash
sleep 5
```

```bash
sudo docker ps
```

Output harus memuat:

```text
libraryit-server
libraryit-logger
0.0.0.0:1445->445/tcp
```

---

## C. Cek User dan Group Otomatis

```bash
sudo docker exec -it libraryit-server pdbedit -L
```

Output harus memuat:

```text
member
contributor
librarian
```

```bash
sudo docker exec -it libraryit-server getent group staff readonly
```

Output harus memuat:

```text
staff:x:...:contributor,librarian
readonly:x:...:member
```

```bash
sudo docker exec -it libraryit-server ls /libraryit
```

Output:

```text
docs  ebooks  papers  sourcecode
```

Penjelasan: bagian ini membuktikan user, group, dan folder koleksi otomatis terbentuk saat container berjalan.

---

## D. Terminal 1 - Logger Real-Time

Buka Terminal 1.

```bash
cd ~/"Modul 4"/soal_3
```

Jika folder repo berbeda, sesuaikan path.

```bash
sudo docker logs -f libraryit-logger
```

Penjelasan: logger dipantau real-time seperti contoh soal.

---

## E. Terminal 2 - Testing Samba Share

Buka Terminal 2.

```bash
cd ~/"Modul 4"/soal_3
```

Jika folder repo berbeda, sesuaikan path.

Cek daftar share sebagai member:

```bash
smbclient -L //localhost -p 1445 -U member%member123
```

Output yang diharapkan memuat:

```text
ebooks
papers
docs
IPC$
```

`sourcecode` tidak boleh terlihat untuk `member`.

Tes member tidak bisa akses sourcecode:

```bash
smbclient //localhost/SourceCode -p 1445 -U member%member123
```

Output yang diharapkan:

```text
tree connect failed: NT_STATUS_ACCESS_DENIED
```

Tes anonymous access:

```bash
smbclient -L //localhost -p 1445 -N
```

Output yang diharapkan: akses gagal.

---

## F. Testing Hak Akses Tulis

Buat file test:

```bash
echo "test ebook" > test_ebook.txt
```

Contributor upload ke `ebooks`:

```bash
smbclient //localhost/ebooks -p 1445 -U contributor%contrib456 -c "put test_ebook.txt; ls"
```

Output yang diharapkan: upload berhasil.

Buat file test:

```bash
echo "test paper" > test_paper.txt
```

Contributor upload ke `papers`:

```bash
smbclient //localhost/papers -p 1445 -U contributor%contrib456 -c "put test_paper.txt; ls"
```

Output yang diharapkan: upload berhasil.

Buat file docs:

```bash
echo "dokumen librarian" > test_docs.txt
```

Librarian upload ke `docs`:

```bash
smbclient //localhost/docs -p 1445 -U librarian%lib789 -c "put test_docs.txt; ls"
```

Output yang diharapkan: upload berhasil.

Contributor tidak boleh upload ke `docs`:

```bash
echo "dokumen contributor" > test.txt
```

```bash
smbclient //localhost/docs -p 1445 -U contributor%contrib456 -c "put test.txt"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED opening remote file
```

Member tidak boleh menulis ke `docs`:

```bash
echo "dokumen member" > member_test.txt
```

```bash
smbclient //localhost/docs -p 1445 -U member%member123 -c "put member_test.txt"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED
```

Contributor tidak boleh menulis ke `sourcecode`:

```bash
echo "print('hello world')" > hello_world.py
```

```bash
smbclient //localhost/sourcecode -p 1445 -U contributor%contrib456 -c "put hello_world.py"
```

Output yang diharapkan:

```text
NT_STATUS_ACCESS_DENIED opening remote file
```

Penjelasan: bagian ini mengikuti contoh soal bahwa `sourcecode` tidak dapat ditulis oleh contributor.

---

## G. Cek Persistence dan Permission Host

```bash
sudo find data -type f
```

Output yang diharapkan memuat:

```text
data/ebooks/test_ebook.txt
data/papers/test_paper.txt
data/docs/test_docs.txt
```

Penjelasan: file tersimpan permanen di folder host `data`.

Cek permission folder host:

```bash
ls -la ./data/
```

```bash
ls -ld ./data/sourcecode
```

Output `sourcecode` harus menunjukkan permission `750`, misalnya:

```text
drwxr-x--- root staff ... ./data/sourcecode
```

Tes host tidak boleh langsung menulis ke docs:

```bash
touch ./data/docs/test_dari_host.txt
```

Output yang diharapkan:

```text
touch: cannot touch './data/docs/test_dari_host.txt': Permission denied
```

Penjelasan: `docs` hanya dimodifikasi lewat Samba, bukan langsung dari host.

---

## H. Cek Log

Di Terminal 2:

```bash
cat logs/libraryit.log | tail -30
```

```bash
sudo docker logs libraryit-logger --tail=30
```

Di Terminal 1, log real-time harus menampilkan aktivitas dengan format seperti:

```text
[2025-05-08 10:00:01] [INFO] [contributor] [CONNECT] [sourceCode]
[2025-05-08 10:01:22] [WARNING] [member] [DENIED] [SourceCode]
[2025-05-08 10:02:45] [INFO] [librarian] [WRITE] [test.txt]
```

Penjelasan: `INFO` untuk aktivitas normal dan `WARNING` untuk akses yang ditolak.

---

## I. Stop Soal 3

Di Terminal 1 tekan:

```text
CTRL + C
```

Di Terminal 2:

```bash
sudo docker compose down
```

```bash
cd ..
```

---

# Cleanup Semua Hasil Run

Jalankan dari folder utama repository.

```bash
bash -lc 'set +e; fusermount3 -u soal_1/mnt 2>/dev/null || fusermount -u soal_1/mnt 2>/dev/null || true; fusermount3 -u soal_2/fuse_mount 2>/dev/null || fusermount -u soal_2/fuse_mount 2>/dev/null || true; sudo docker rm -f db_app libraryit-server libraryit-logger 2>/dev/null || true; (cd soal_3 && sudo docker compose down --remove-orphans 2>/dev/null || true); rm -rf soal_1/amba_files soal_1/mnt soal_1/kenz_rescue soal_1/fuse.log soal_1/amba_files.zip; rm -f soal_2/fuse soal_2/client soal_2/fuse.log; sudo rm -rf soal_2/encrypted_storage/* soal_2/fuse_mount/*; sudo rm -rf soal_3/data/docs/* soal_3/data/ebooks/* soal_3/data/papers/* soal_3/data/sourcecode/* soal_3/logs/*; rm -f ~/main.py ~/test_sourcecode.c ~/test_ebook.txt ~/test_paper.txt ~/test_docs.txt ~/contributor_docs.txt ~/member_test.txt test_ebook.txt test_paper.txt test_docs.txt test.txt member_test.txt hello_world.py; mkdir -p soal_2/encrypted_storage soal_2/fuse_mount soal_3/data/docs soal_3/data/ebooks soal_3/data/papers soal_3/data/sourcecode soal_3/logs; touch soal_3/logs/libraryit.log; sudo chown -R "$USER:$USER" soal_2/encrypted_storage soal_2/fuse_mount 2>/dev/null || true; echo "Cleanup selesai."'
```

Cek struktur akhir:

```bash
tree
```

Cek tidak ada `main.py`:

```bash
find . -name "main.py" -print
```

Jika tidak ada output, berarti bersih.

Catatan kondisi tree awal sebelum demo:
- `soal_2/server` tetap ada.
- `soal_1/mnt` boleh tidak ada karena dibuat saat run.
- `soal_2/encrypted_storage/tests` boleh tidak ada karena dibuat saat run.
- file testing seperti `hello_world.py`, `test_ebook.txt`, dan sejenisnya tidak boleh tersisa.
