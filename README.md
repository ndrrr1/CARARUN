### Challenge 1 — Comment, relasi, migrasi, CRUD

Buka `prisma/schema.prisma`, tunjukkan model Comment dan relasinya ke Post/User. `postId` adalah post tujuan, `authorId` adalah penulis komentar. Kedua relasi memakai `onDelete: Cascade`.

Tunjukkan file `prisma/migrations/20261028010100_add_comment/migration.sql`. Migrasi mengubah struktur database; generate membuat Prisma Client mengikuti schema. Peluncur menerapkan file migrasi yang sudah ada. Untuk menunjukkan alur pengembangan Prisma 7, perintahnya adalah `npm run db:migrate`, lalu `npm run db:generate`; jika tidak ada perubahan schema, hasilnya sudah sinkron.

Di folder 01, POST komentar benar-benar membuat record baru. Peluncur telah mengalokasikan ID kosong lewat sequence database, sehingga GET/PATCH/DELETE berikutnya sudah memiliki ID angka yang sama. Endpoint menerima `id` opsional untuk alur demo ini; bila `id` dihilangkan, Prisma tetap memakai autoincrement. Tidak ada ketergantungan pada script Postman untuk menyimpan ID.

> Saya menambahkan model Comment yang terhubung ke user dan post. CRUD memungkinkan komentar dibuat, dibaca, diubah, dan dihapus. Setelah komentar dihapus, membaca ID yang sama menghasilkan 404. Cascade akan saya buktikan pada bagian penutup.

### Challenge 2 — post satu user, pagination, published

Buka `src/controllers/userController.ts`, fungsi `posts`. Tunjukkan filter authorId, published, skip/take, dan count dengan where yang sama.

Pada awal sesi, user penulis demo punya dua post terbit dan satu draft. Folder 02 memakai ID user yang sudah disiapkan. Halaman 1 dan 2 memakai limit satu. Response menunjukkan data dan metadata: currentPage, limit, totalItems, totalPages. Setelah bulk, jumlah post bisa bertambah.

> Endpoint ini hanya mengambil post milik satu user. Pagination membagi daftar ke beberapa halaman. Limit satu menampilkan satu item per halaman. Filter published true menampilkan post terbit, sementara false menampilkan draft. Total data dihitung memakai filter yang sama.

### Challenge 3 — statistik per hari

Buka fungsi `daily` pada `src/controllers/postController.ts`, tunjukkan raw query `date_trunc('day', "createdAt")`, GROUP BY, dan `COUNT(*)::int`. Kirim request folder 03.

> Fungsi date_trunc mengelompokkan waktu pembuatan berdasarkan tanggal. COUNT menghitung jumlah post pada tiap tanggal. Response mengembalikan day dan total sebagai statistik harian. Tanggal pada implementasi ini memakai UTC.

Statistik mencakup semua post yang ada, termasuk data lain di database. Bacakan angka aktual yang terlihat. Jika semua post dibuat pada hari UTC yang sama, satu baris hasil sudah sesuai.

### Challenge 4 — bulk create

Buka fungsi `bulk`, tunjukkan `db.post.createMany({ data: posts })`. Folder 04 berisi body array dengan dua post dan authorId angka yang sudah diisi otomatis. Response sukses menampilkan count 2. Request berikutnya memperlihatkan daftar post setelah penambahan.

> Bulk create menerima beberapa post dalam satu request. Array divalidasi dan penulisnya diperiksa, lalu data disimpan memakai createMany. Nilai count dua menunjukkan dua record baru berhasil disimpan. Array kosong ditolak dengan status 400 dan pesan yang jelas.

### Bonus — validasi input

Buka `src/validation/schemas.ts`, tunjukkan format email, password minimal delapan karakter, boolean, dan validasi array. Jalankan folder 05.

> Validasi memeriksa data sebelum disimpan. Pada contoh ini email salah dan password terlalu pendek menghasilkan 400 beserta errors per field. Pengguna API dapat mengetahui input yang perlu diperbaiki. Boolean berbentuk string dan nilai filter di luar true atau false juga ditolak.

### Cascade dan penutup

Folder 06 menggunakan post/user khusus yang disiapkan terpisah, sehingga data pagination tetap tersedia. Pengujiannya membuktikan:

- Menghapus Post menghapus komentarnya.
- Menghapus User menghapus post miliknya.
- Menghapus User menghapus komentarnya pada post orang lain.
- Post milik user lain tetap bisa dibaca dengan status 200.

> Saya menghapus post dan membuktikan komentarnya ikut hilang. Kemudian saya menghapus user khusus demo; post dan komentarnya ikut terhapus, sedangkan post milik user lain tetap tersedia. Ini membuktikan relasi cascade bekerja sesuai model. Demonstrasi CRUD, pagination, statistik harian, bulk create, branch Neon, dan validasi selesai.

2. Tambahkan satu baris pada model Comment

Di VS Code, buka prisma/schema.prisma. Cari model Comment, lalu tambahkan:
demoNote String?

Posisinya boleh tepat setelah content. Model lengkapnya menjadi:
model Comment {
  id        Int      @id @default(autoincrement())
  content   String
  demoNote  String?
  postId    Int
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([authorId])
}

Artinya:
- demoNote: nama kolom baru.
- String: isinya berupa teks.
- ?: kolom boleh kosong, sehingga komentar yang sudah ada tetap valid.
Simpan dengan Ctrl+S.

3. Terapkan perubahan ke database branch baru

Di terminal proyek, jalankan:
npm run db:migrate -- --name demo_branch_note

Perintah ini membuat file migrasi dan menerapkan perubahan schema ke database yang dituju .env. Setelah berhasil, jalankan:
npm run db:generate

Perintah kedua memperbarui Prisma Client agar mengenali kolom baru. Pada Prisma 7, generate dijalankan terpisah. prisma.io

4. Tunjukkan buktinya di Neon

Buka SQL Editor, pilih branch demo-modul-4, lalu jalankan:
SELECT column_name
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name = 'Comment'
  AND column_name = 'demoNote';

Kemudian ganti pilihan branch di SQL Editor ke parent, lalu jalankan SQL yang sama.

Hasil yang diharapkan:
Tempat menjalankan SQL	Hasil
Branch demo-modul-4	Muncul baris demoNote
Parent/database awal	Tidak ada baris hasil


Perbedaan itu menjadi bukti bahwa migrasi hanya mengubah branch percobaan. Jumlah User/Post pada parent juga bisa dibandingkan dengan catatan sebelum eksperimen.

5. Lanjutkan demo aplikasi

Kembali ke terminal:
npm run demo

6. Peluncur membuat sesi baru pada branch tersebut. Impor ulang `postman/POSTMAN-DEMO-SIAP.json` yang baru dibuat dan pilih collection sesi terbaru seperti nama pada terminal.
7. Bandingkan kolom Comment pada parent dan branch. `demoNote` hanya muncul di branch percobaan; catatan jumlah/data parent tetap sama selama tidak ada proses lain menulis ke parent.
8. Pada terminal tambahan, jalankan `npm run neon:verify` jika MAIN_DATABASE_URL sudah diisi. Skrip menulis probe hanya di branch percobaan, membaca parent melalui koneksi read-only, memastikan probe tidak ada di parent, lalu membersihkan probe. Bacakan hasil aktual `probeVisibleOnMain: false` dan mainBefore/mainAfter yang sama.

> Saya membuat branch Neon sebagai lingkungan eksperimen terpisah. Aplikasi diarahkan ke branch tersebut dan satu perubahan schema diterapkan melalui migrasi. Kolom baru muncul di branch percobaan, sementara parent tetap sesuai catatan awal. Verifikasi juga menunjukkan penulisan pada branch percobaan tidak muncul di parent.

Lanjutkan sisa rekaman memakai branch percobaan, karena Prisma Client telah mengikuti schema baru. Simpan pengisian koneksi di luar rekaman.
