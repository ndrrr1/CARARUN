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

## 6. Challenge 5 — eksperimen branch Neon

Ini langkah pada akun Neon milikmu. Paket tidak membuat branch cloud tanpa akses akun. Lakukan tahap ini setelah folder 04 dan sebelum bonus/cascade jika rekaman harus mengikuti nomor soal.

1. Pada parent yang sedang dipakai, catat jumlah User/Post dan kolom Comment di SQL Editor:

```sql
SELECT COUNT(*) AS total_user FROM "User";
SELECT COUNT(*) AS total_post FROM "Post";
SELECT column_name FROM information_schema.columns
WHERE table_schema='public' AND table_name='Comment'
ORDER BY ordinal_position;
```

2. Buat branch baru melalui **Branches → New branch**, pilih parent tersebut dan salinan **Current data**. Beri nama misalnya `demo-modul-4`.
3. Hentikan server dengan Ctrl+C. Ubah kedua URL `.env` ke pooled/direct branch baru. Isi `MAIN_DATABASE_URL` dengan URL direct parent jika ingin menjalankan verifikasi.
4. Karena branch menyalin tabel yang sudah ada, migrasi awal bisa sudah applied. Untuk memperlihatkan migrasi baru, tambahkan field opsional berikut di dalam model Comment pada `prisma/schema.prisma`:

```prisma
demoNote String?
```

5. Jalankan:

```powershell
npm run db:migrate -- --name demo_branch_note
npm run db:generate
npm run demo
```

6. Peluncur membuat sesi baru pada branch tersebut. Impor ulang `postman/POSTMAN-DEMO-SIAP.json` yang baru dibuat dan pilih collection sesi terbaru seperti nama pada terminal.
7. Bandingkan kolom Comment pada parent dan branch. `demoNote` hanya muncul di branch percobaan; catatan jumlah/data parent tetap sama selama tidak ada proses lain menulis ke parent.
8. Pada terminal tambahan, jalankan `npm run neon:verify` jika MAIN_DATABASE_URL sudah diisi. Skrip menulis probe hanya di branch percobaan, membaca parent melalui koneksi read-only, memastikan probe tidak ada di parent, lalu membersihkan probe. Bacakan hasil aktual `probeVisibleOnMain: false` dan mainBefore/mainAfter yang sama.

> Saya membuat branch Neon sebagai lingkungan eksperimen terpisah. Aplikasi diarahkan ke branch tersebut dan satu perubahan schema diterapkan melalui migrasi. Kolom baru muncul di branch percobaan, sementara parent tetap sesuai catatan awal. Verifikasi juga menunjukkan penulisan pada branch percobaan tidak muncul di parent.

Lanjutkan sisa rekaman memakai branch percobaan, karena Prisma Client telah mengikuti schema baru. Simpan pengisian koneksi di luar rekaman.
