# Panduan Demo Challenge Modul 3
## Perintah, lokasi kerja, dan naskah bicara

Panduan ini memakai VS Code, terminal PowerShell di Windows, Codex CLI, dan Gemini CLI. Jalankan sendiri proses live coding saat rekaman. Contoh audit di bawah merupakan bahan latihan; ketik dan jelaskan perubahannya langsung. Struktur proyek task-tracker belum diberikan, sehingga nama file dan perintah aplikasi harus mengikuti proyek yang sebenarnya.

## 0. Sebelum rekaman

- Siapkan VS Code, Node.js versi LTS yang memenuhi Node.js 20+, dan proyek task-tracker dari modul sebelumnya.
- Periksa versi Windows: dokumentasi Gemini CLI mencantumkan Windows 11 24H2+ sebagai lingkungan yang didukung. Pada Windows lama, periksa kompatibilitas sebelum hari demo.
- Sediakan akses akun yang bisa dipakai untuk Codex dan Gemini CLI. Pemasangan CLI tidak otomatis menyediakan akses layanan/model.
- Matikan bantuan penulisan kode AI di workspace audit-ts selama bagian manual.
- Awali isi video dengan audit TypeScript. Tampilkan instalasi dan penggunaan harness setelah bagian audit selesai.
- Buka VS Code > Terminal > New Terminal. Pilih PowerShell melalui panah di sebelah tombol +.
- Perintah terminal dimasukkan di terminal biasa. Prompt agent dimasukkan setelah antarmuka codex atau gemini terbuka. Kode TypeScript dan isi AGENTS.md ditulis di editor.
- Simpan tiap perubahan dengan Ctrl+S sebelum menjalankan pemeriksaan.
- Buka terminal kedua lewat tombol + untuk npm/npx ketika terminal pertama sedang menjalankan agent. Periksa lokasinya dengan pwd.

Bicara:
“Pada bagian pertama, saya akan mengaudit TypeScript secara manual. Saya memakai folder terpisah supaya percobaan ini tidak mengubah proyek task-tracker.”

## 1. Audit TypeScript manual

### 1A. Buat folder terpisah

Di terminal PowerShell:

```powershell
cd $env:USERPROFILE
mkdir demo-modul3
cd demo-modul3
mkdir audit-ts
cd audit-ts
npm init -y
npm install typescript @types/node tsx --save-dev
npx tsc --init
mkdir src
code .
```

Jika folder induk sudah ada, cukup masuk dengan cd; jangan buat ulang. Jika code tidak dikenali, buka folder lewat VS Code > File > Open Folder. Folder audit-ts harus menjadi saudara atau terpisah dari task-tracker, bukan berada di dalam src task-tracker.

tsx dipasang sebagai development dependency agar npx tsx tidak meminta pemasangan mendadak saat demo.

Buka tsconfig.json. Pastikan properti ini ada di dalam compilerOptions:

```json
"strict": true,
"types": ["node"]
```

Jika sudah ada "types": [], ubah nilainya, jangan buat properti types kedua. Pengaturan types membantu TypeScript mengenali API Node seperti console. Tidak perlu mengganti keseluruhan konfigurasi.

Di Explorer VS Code, klik kanan src > New File > beri nama audit.ts.

### 1B. Ketik contoh berbahaya, 6 baris

Di src/audit.ts:

```typescript
function formatTitle(title: any): string {
  return title.toUpperCase();
}

const taskTitle = 123;
console.log(formatTitle(taskTitle));
```

Simpan. Di terminal yang lokasinya audit-ts, jalankan:

```powershell
npx tsc --noEmit
npx tsx src/audit.ts
```

Pemeriksaan tipe dapat lolos, tetapi eksekusi menghasilkan error dengan inti pesan:

```text
TypeError: title.toUpperCase is not a function
```

Bicara:
“Saya menggunakan any pada parameter title. Akibatnya, TypeScript mengizinkan pemanggilan toUpperCase meskipun nilai yang masuk berupa angka. Pemeriksaan tipe lolos, tetapi program gagal saat dijalankan karena angka tidak memiliki method toUpperCase. Jadi, strict tidak melarang any yang saya tulis secara eksplisit.”

### 1C. Tampilkan proses perbaikan

Pertama, ubah hanya any menjadi string | number:

```typescript
function formatTitle(title: string | number): string {
  return title.toUpperCase();
}
```

Jangan hapus pemanggilan fungsi di bawahnya. Simpan, lalu jalankan:

```powershell
npx tsc --noEmit
```

Compiler sekarang menolak pemanggilan toUpperCase pada nilai yang mungkin berupa number. Ini tahap sementara yang sengaja ditampilkan, bukan hasil akhir.

Bicara:
“Setelah saya membatasi parameter menjadi string atau number, compiler mulai menunjukkan operasi yang belum aman. Saya perlu membedakan cara menangani kedua tipe tersebut.”

Lalu ubah seluruh isi src/audit.ts menjadi:

```typescript
function formatTitle(title: string | number): string {
  if (typeof title === "string") {
    return title.toUpperCase();
  }
  return String(title);
}

console.log(formatTitle(123));
console.log(formatTitle("belajar typescript"));
```

Simpan, kemudian:

```powershell
npx tsx src/audit.ts
npx tsc --noEmit
$LASTEXITCODE
```

Output aplikasi:

```text
123
BELAJAR TYPESCRIPT
```

tsc biasanya tidak menampilkan apa pun saat berhasil. $LASTEXITCODE sesudah tsc seharusnya 0.

Bicara:
“Saya memakai union type karena fungsi ini memang menerima string atau number. Pemeriksaan typeof menjadi type guard: pada cabang string, saya boleh menggunakan toUpperCase. Untuk angka, saya mengubahnya menjadi string secara eksplisit. Kedua contoh sekarang berjalan, dan pemeriksaan TypeScript selesai tanpa error.”

Catatan konsep: tsx menjalankan kode, sedangkan tsc --noEmit memeriksa tipe tanpa menghasilkan file JavaScript. Keduanya perlu diperlihatkan. Lolos tsc tidak membuktikan seluruh perilaku program bebas bug.

## 2. Instalasi dan pengenalan harness

### 2A. Instal setelah bagian manual

Di terminal biasa:

```powershell
npm install -g @openai/codex
codex --version
npm install -g @google/gemini-cli
gemini --version
```

Catat versi yang benar-benar muncul. Jika PowerShell memblokir npm.ps1, gunakan npm.cmd; jika npx.ps1 terblokir, gunakan npx.cmd. Launcher npm lain seperti gemini juga dapat dipanggil sebagai gemini.cmd bila tersedia. Tidak perlu mengubah execution policy.

Bicara:
“Sekarang saya beralih ke harness. Harness adalah aplikasi yang menghubungkan model AI dengan file proyek dan alat seperti terminal, sehingga agent dapat membaca kode, mengeditnya, dan menjalankan pemeriksaan.”

### 2B. Buka task-tracker yang sudah ada

Di VS Code pilih File > Open Folder > folder task-tracker. Pilih folder yang berisi package.json. Jangan menjalankan npm init di proyek yang sudah ada.

Buka terminal baru dan jalankan:

```powershell
pwd
npm install
npm run
npx tsc --noEmit
```

npm run menampilkan script yang tersedia. Gunakan package manager proyek jika ternyata proyek memakai pnpm/yarn; contoh panduan ini mengasumsikan npm. Selesaikan error bawaan proyek terlebih dahulu agar perubahan berikutnya memiliki titik awal yang bersih.

Jalankan:

```powershell
codex
```

Ikuti opsi Sign in with ChatGPT atau metode login yang tersedia untuk akunmu. Pilih izin yang sesuai folder kerja melalui tampilan harness. Tampilkan proses agent saat membaca file, menjalankan perintah, dan mengajukan perubahan.

Ketik prompt berikut di Codex:

```text
Jelaskan struktur proyek ini tanpa mengubah file.
Temukan definisi Task, lokasi pembuatan Task, penyimpanan data,
pemrosesan argumen CLI, serta script menjalankan aplikasi.
Sebutkan file instruksi yang aktif atau benar-benar kamu baca beserta path-nya.
Bedakan file instruksi agent dari file sumber dan dokumentasi biasa.
Jika tidak bisa memastikan suatu sumber instruksi, jelaskan keterbatasannya.
Tuliskan perintah konkret untuk menjalankan fitur CLI proyek ini.
```

Catat hasilnya:

| Hal | Hasil nyata |
|---|---|
| File definisi Task | ... |
| File pembuat Task | ... |
| File parser/perintah CLI | ... |
| Lokasi penyimpanan data | ... |
| Script menjalankan aplikasi | ... |
| File instruksi agent dan path | ... |
| Versi CLI dan model aktif | ... |

Bicara:
“Saya menjalankan harness dari root task-tracker agar agent bekerja pada proyek yang benar. Saya meminta pemetaan struktur terlebih dahulu supaya bisa menilai apakah agent memahami alur aplikasi dan file instruksi yang digunakannya.”

## 3. Perbandingan dengan dan tanpa AGENTS.md

Lakukan dua percobaan memakai Codex, model yang sama, dan prompt yang sama. Kedua percobaan hanya meminta rencana, sehingga kode sumber tetap sama.

Jika AGENTS.md proyek belum ada, lanjutkan langsung. Jika sudah ada, lakukan percobaan di salinan proyek; pindahkan sementara file instruksi proyek keluar dari salinan sebelum percobaan “tanpa”, lalu kembalikan untuk percobaan “dengan”. Jangan menghapus instruksi asli. Catat instruksi global atau file instruksi lain yang tetap aktif; “tanpa” di sini berarti tanpa AGENTS.md proyek yang sedang diuji.

### 3A. Tanpa AGENTS.md proyek

Mulai sesi Codex baru. Ketik prompt ini dan simpan hasilnya:

```text
Buat rencana menambahkan priority pada Task dengan nilai low, medium, atau high.
Jangan ubah file dulu. Sebutkan file yang perlu diubah,
keputusan implementasi, dan cara memverifikasinya.
```

Simpan jawaban di catatan di luar root proyek, atau ambil tangkapan layar.

### 3B. Buat AGENTS.md di root proyek

Di Explorer VS Code, buat AGENTS.md sejajar dengan package.json. Isinya:

```markdown
# Instruksi proyek task-tracker

- Gunakan bahasa Indonesia.
- Mulai respons perencanaan dengan urutan: Rencana, File terdampak, Verifikasi.
- Ikuti struktur dan gaya kode proyek yang sudah ada.
- Pertahankan strict TypeScript.
- Jangan gunakan any, @ts-ignore, atau type assertion untuk menutupi error.
- Hindari dependency baru jika fitur dapat dibuat dengan kode yang sudah tersedia.
- Priority Task wajib bernilai low, medium, atau high.
- Gunakan medium sebagai default untuk task baru tanpa input priority.
- Jika ada data lama tanpa priority, tangani pada batas pembacaan data.
- Jangan mengarang nama file atau hasil pengujian.
- Setelah implementasi, jalankan npx tsc --noEmit dan uji perilaku yang diubah.
- Pada checkpoint demo yang diminta pengguna, berhenti agar hasil bisa direkam.
```

Aturan default medium adalah keputusan untuk demo ini, bukan syarat tambahan dari soal.

Tutup sesi Codex setelah respons selesai, misalnya dengan Ctrl+C mengikuti petunjuk CLI, lalu jalankan codex lagi. Tidak cukup hanya mengganti teks prompt pada percakapan lama karena konteks percakapan dapat memengaruhi hasil.

Kirim prompt yang SAMA PERSIS dengan percobaan sebelumnya. Setelah respons, tanyakan file instruksi aktif untuk membantu pencatatan.

Bandingkan:

| Aspek | Tanpa AGENTS.md proyek | Dengan AGENTS.md proyek |
|---|---|---|
| Format rencana | Isi hasil | Isi hasil |
| Default priority | Disebut atau tidak | Sesuai medium atau tidak |
| Data lama | Dibahas atau tidak | Dibahas atau tidak |
| Type safety | Isi hasil | Isi hasil |
| Verifikasi | Isi hasil | Isi hasil |

Bicara:
“Permintaan dan kode awal saya buat sama. Yang saya ubah adalah konteks proyek melalui AGENTS.md. Saya membandingkan format rencana, keputusan default priority, penanganan data lama, dan rencana verifikasinya. Dari hasil yang tampil, perbedaan yang saya temukan adalah ...”

Lengkapi kalimat terakhir berdasarkan bukti. Jika hasil hampir sama, katakan demikian; tidak perlu memaksakan perbedaan.

## 4. Tambahkan priority dan biarkan compiler memandu perbaikan

### 4A. Ubah hanya definisi Task

Gunakan Ctrl+Shift+F untuk mencari interface Task atau type Task. Ikuti file hasil pemetaan harness.

Tambahkan hanya baris ini di dalam tipe yang sudah ada:

```typescript
priority: "low" | "medium" | "high";
```

Pertahankan seluruh field lama. Jangan menambahkan tanda ? karena priority diminta sebagai field wajib. Jangan dulu memperbaiki object pembuat Task.

Simpan, lalu di terminal biasa pada task-tracker:

```powershell
npx tsc --noEmit
```

Rekam seluruh daftar error yang benar-benar muncul. Pesannya dapat berbunyi bahwa property priority hilang pada object tertentu; jumlah dan lokasi tergantung proyek.

Bicara:
“Saya menambahkan priority sebagai field wajib dengan tiga pilihan nilai. Saya belum memperbaiki tempat pembuatan Task. Sekarang compiler menunjukkan bagian bertipe yang belum memenuhi definisi baru.”

Jika tidak ada error, jangan mengklaim seluruh alur sudah benar. Periksa apakah file masuk cakupan tsconfig, apakah tipe Task benar-benar dipakai, atau apakah any/type assertion/JSON.parse melewati pemeriksaan. Compiler tidak memvalidasi isi file JSON pada runtime secara otomatis.

### 4B. Minta agent memperbaiki error

Di Codex:

```text
Saya sudah menambahkan field wajib priority: "low" | "medium" | "high" pada Task.
Jalankan npx tsc --noEmit dan jelaskan seluruh error yang ditemukan.
Perbaiki pemakaian Task yang terdampak, termasuk pembuatan object dan fixture
yang relevan. Gunakan medium sebagai default ketika priority tidak diberikan.
Jika ada penyimpanan data lama, tangani data tanpa priority saat dibaca dan
validasi nilai eksternal, bukan sekadar memaksa tipenya.
Jangan menjadikan priority optional, menonaktifkan strict, atau menutupi
masalah dengan any, @ts-ignore, atau type assertion.
Tunjukkan perubahan file dan jalankan npx tsc --noEmit kembali hingga lulus.
Verifikasi perilaku terkait priority. Jangan kerjakan filter list --status dulu.
```

Tampilkan operasi agent dan perubahan file langsung. Gunakan panel Source Control bila proyek memakai Git, atau buka file yang diubah agent.

Setelah agent selesai, periksa sendiri:

```powershell
npx tsc --noEmit
$LASTEXITCODE
```

Jalankan aplikasi dengan perintah yang sesuai package.json. Buat/lihat task untuk memastikan default priority benar. Jika proyek membaca JSON, periksa pula data lama tanpa priority.

Bicara:
“Agent memperbaiki lokasi yang terdampak perubahan tipe. Saya mengecek hasilnya lewat compiler dan perilaku aplikasi. Field priority tetap wajib, dan nilai default ditangani di tempat pembuatan atau pembacaan data yang sesuai.”

## 5. Bandingkan dua harness pada list --status done

### 5A. Buat titik awal yang sama

Selesaikan priority terlebih dahulu. Pastikan fitur filter belum dikerjakan pada titik awal ini.

Sebelum menyalin, siapkan data contoh memakai command proyek yang sudah ada:
- minimal satu task dengan status done;
- minimal satu task dengan status lain yang valid.

Nama command add/update/done mengikuti proyek. Jika belum diketahui, minta Codex menjelaskan command yang sudah ada, tanpa mengimplementasikan filter.

Buat GEMINI.md di root task-tracker, berisi:

```markdown
@./AGENTS.md
```

Gemini CLI memakai GEMINI.md secara default; impor tersebut memberikan aturan proyek yang sama dengan yang digunakan Codex.

Tutup sesi agent, simpan semua file, lalu lewat File Explorer salin folder task-tracker menjadi dua folder saudara:
- task-tracker-codex
- task-tracker-gemini

Kedua salinan dibuat SEBELUM salah satu agent mengerjakan filter. Jangan menyalin hasil Codex untuk dijadikan titik awal Gemini. Sertakan kode, package.json, lockfile, AGENTS.md, GEMINI.md, konfigurasi, dan data contoh yang sama. node_modules boleh tidak disalin; instal ulang di masing-masing salinan.

Jika penyimpanan data memakai path absolut atau database bersama, pastikan pengujian list tidak mengubah dataset bersama; jangan mengaku datanya terisolasi hanya karena folder kode disalin.

Bicara:
“Saya membuat dua salinan dari kondisi awal yang sama. Keduanya menerima fitur, aturan proyek, data contoh, dan urutan prompt yang sama. Saya juga mencatat model aktif karena hasil dipengaruhi model, bukan hanya harness.”

### 5B. Jalankan Codex pada salinan pertama

VS Code > File > Open Folder > task-tracker-codex.
Terminal biasa:

```powershell
npm ci
npx tsc --noEmit
codex
```

Gunakan npm install jika proyek tidak memiliki package-lock.json. Jika npm ci mengeluhkan lockfile yang tidak cocok, perbaiki titik awal dan gunakan lockfile hasil perbaikan yang sama di kedua salinan.

Prompt perencanaan:

```text
Baca instruksi proyek yang aktif.
Buat rencana menambahkan filter list --status done.
Tanpa --status, list harus tetap menampilkan semua task.
Dengan --status done, hanya task berstatus done yang ditampilkan.
Status tidak valid atau --status tanpa nilai harus menghasilkan pesan yang jelas
dan exit code bukan nol. Hasil kosong harus ditangani dengan jelas.
Ikuti struktur CLI proyek, jangan mengubah data ketika menjalankan list,
dan jangan menambah dependency tanpa kebutuhan.
Sebutkan file terdampak dan cara menguji.
Berhenti setelah rencana; jangan mengedit file dulu.
```

Catat kualitas rencana. Kemudian kirim:

```text
Implementasikan rencana tersebut.
Setelah implementasi pertama selesai, jalankan npx tsc --noEmit.
Tampilkan output dan jumlah diagnostic TypeScript yang muncul.
Berhenti di checkpoint ini sebelum memperbaiki error hasil pemeriksaan,
supaya saya bisa mencatat kondisi implementasi pertama.
```

Di terminal kedua, verifikasi:

```powershell
npx tsc --noEmit
```

Catat error pemeriksaan pertama meskipun jumlahnya nol. Sesudah dicatat, kirim:

```text
Lanjutkan dari checkpoint. Perbaiki error yang ditemukan jika ada,
lalu jalankan npx tsc --noEmit dan uji semua kriteria fitur.
Tampilkan command, hasil, file yang diubah, dan sisa masalah.
```

Jika perlu arahan korektif tambahan di luar tiga prompt terjadwal ini, catat sebagai intervensi manual.

### 5C. Jalankan Gemini pada salinan kedua

VS Code > File > Open Folder > task-tracker-gemini.
Terminal biasa:

```powershell
npm ci
npx tsc --noEmit
gemini
```

Login melalui opsi Sign in with Google yang sesuai akunmu. Setelah antarmuka Gemini terbuka, periksa konteks:

```text
/memory show
```

Pastikan isi AGENTS.md dari impor GEMINI.md terlihat. Jika tidak, periksa letak file dan mulai sesi baru. Catat file instruksi global lain yang ikut termuat.

Kirim tiga prompt yang SAMA PERSIS dan dalam urutan yang sama seperti percobaan Codex: perencanaan, implementasi pertama dengan checkpoint, lalu penyelesaian. Catat hasilnya secara terpisah.

### 5D. Perintah uji fitur

Perintah aplikasi ditentukan oleh package.json, bukan oleh nama harness.

Jika script dev menjalankan CLI, misalnya "dev": "tsx src/index.ts", gunakan:

```powershell
npm run dev -- list
npm run dev -- list --status done
npm run dev -- list --status tidak-valid
$LASTEXITCODE
npm run dev -- list --status
$LASTEXITCODE
npx tsc --noEmit
```

Jika script start menjalankan CLI, pakai npm start -- list --status done.
Jika tanpa script dan file entry point-nya src/cli.ts, pakai npx tsx src/cli.ts list --status done.
Jangan memakai contoh npm run dev bila script dev sebenarnya menjalankan server frontend.

Periksa:
1. list tanpa opsi menampilkan semua task.
2. list --status done hanya menampilkan task done.
3. Status tidak valid ditolak dengan pesan jelas dan exit code bukan nol.
4. --status tanpa nilai ditolak dengan jelas.
5. Ketika tidak ada task done, hasil kosong disampaikan dengan benar.
6. Menjalankan list tidak mengubah data.
7. npx tsc --noEmit akhirnya lulus.

Untuk kasus kosong, gunakan fixture atau data uji terpisah yang didukung proyek. Jangan menghapus data utama demi pengujian.

Bicara:
“Saya mengecek hasil compiler dan fungsi filternya. Compiler yang bersih belum menjamin logika filter benar, jadi saya juga menjalankan command dengan status done, status tidak valid, dan tanpa opsi.”

## 6. Tabel penilaian

Isi berdasarkan demo, bukan perkiraan.

| Aspek | Codex CLI | Gemini CLI |
|---|---|---|
| Versi CLI | ... | ... |
| Model aktif / mode auto | ... | ... |
| Error tsc pada baseline | ... | ... |
| Kualitas rencana, skor 1–5 | ... | ... |
| Alasan skor rencana | ... | ... |
| Error tsc sesudah implementasi pertama | ... | ... |
| Error tsc pada hasil akhir | ... | ... |
| Intervensi manual di luar alur terjadwal | ... | ... |
| Rincian intervensi | ... | ... |
| Kenyamanan alur kerja, skor 1–5 | ... | ... |
| Alasan skor kenyamanan | ... | ... |
| Hasil uji perilaku filter | ... | ... |

Definisi sederhana:
- Error tsc dihitung sebagai jumlah diagnostic error, bukan jumlah baris output atau jumlah file.
- Satu intervensi manual adalah satu arahan korektif tambahan atau satu tindakan edit kode manual. Pilih definisi ini dan pakai konsisten.
- Prompt awal, tiga tahap terjadwal, dan klik persetujuan normal tidak dihitung sebagai intervensi korektif. Catat persetujuan terpisah bila memengaruhi kenyamanan.
- Rencana dinilai dari ketepatan file, parsing argumen, validasi status, kompatibilitas list lama, dan verifikasi.
- Kenyamanan dinilai dari kemudahan membaca rencana, meninjau diff, memberi persetujuan, dan melanjutkan dari error.
- Catat model dan versi; satu percobaan tidak membuktikan harness tertentu selalu unggul.

## 7. Naskah kesimpulan satu paragraf

Ganti semua bagian dalam kurung siku menggunakan hasil nyata:

“Pada percobaan fitur list --status done, Codex CLI menghasilkan rencana yang [temuan], sedangkan Gemini CLI [temuan]. Setelah implementasi pertama, pemeriksaan TypeScript menunjukkan [jumlah] error pada Codex dan [jumlah] error pada Gemini; hasil akhirnya masing-masing [jumlah] dan [jumlah] error. Saya memberikan [jumlah] intervensi tambahan pada Codex dan [jumlah] pada Gemini. Dari sisi alur kerja, saya lebih nyaman menggunakan [nama harness] karena [alasan konkret]. Kesimpulan ini berlaku untuk proyek, model, versi, dan percobaan yang saya gunakan.”

Jika keduanya sama-sama bagus, sampaikan hasil setara dengan alasan yang terlihat.


