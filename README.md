# README — Panduan Rekaman Asistensi Modul 3

**Urutan wajib:** TypeScript manual → instal harness → AGENTS.md → priority → perbandingan dua harness. Wajib on cam dan tampilkan proses, termasuk error serta perbaikannya.

## Mulai dari mana?

Gunakan **VS Code**. Siapkan Node.js 20+, npm, dan proyek `task-tracker` dari praktikum. Buat folder `modul3`, taruh `task-tracker` di dalamnya, lalu buka `modul3` lewat **File → Open Folder**.

| Yang dikerjakan | Tempatnya |
|---|---|
| Perintah npm, npx, opencode, gemini | Terminal VS Code, bukan di file kode |
| Menulis TypeScript atau AGENTS.md | Editor VS Code melalui panel Explorer |
| Mengirim prompt AI | Chat OpenCode/Gemini setelah aplikasinya terbuka di terminal |
| Membaca penjelasan | Ucapkan sambil menampilkan proses yang sesuai |

**Membuka terminal:** klik kanan folder di Explorer VS Code → **Open in Integrated Terminal**. Untuk folder utama `modul3`, gunakan **Terminal → New Terminal**. Ketik perintah satu per satu, lalu Enter. Jika terminal sedang menampilkan chat harness, buka terminal kedua untuk menjalankan perintah biasa.

Narasi sudah siap dibaca. Pilih variasi yang sesuai hasil percobaan. Angka dan path file tetap dicatat dari hasil nyata.

**Pembuka:**

> Pada video ini, saya mengerjakan Challenge Asistensi Modul 3. Saya mulai dengan audit TypeScript secara manual tanpa AI, kemudian melanjutkan penggunaan harness pada proyek task-tracker.

## 1. Audit TypeScript manual

**Tempat:** terminal di folder `modul3`. Matikan extension AI seperti Copilot/Gemini dan tutup chat AI selama bagian manual.

```bash
mkdir audit-ts
cd audit-ts
npm init -y
npm install --save-dev typescript @types/node tsx
npx tsc --init
mkdir src
```

**Di editor:** buka `audit-ts/tsconfig.json`, pastikan `"strict": true`. Buat `audit-ts/src/audit.ts`, lalu tulis sendiri kode **5–15 baris** menggunakan `any` **atau** `as`. `as const` tidak dihitung. Kode dan perbaikannya wajib kamu susun sendiri sesuai aturan tugas.

**Di terminal `audit-ts`:**

```bash
npx tsx src/audit.ts
```

Tunjukkan error atau hasil aneh. Perbaiki langsung menggunakan **union type, type guard, atau generics**, lalu jalankan:

```bash
npx tsx src/audit.ts
npx tsc --noEmit
```

`tsx` menjalankan program; `tsc --noEmit` memeriksa tipe tanpa menghasilkan JavaScript. Pastikan program benar dan terminal kembali tanpa error tipe.

**Narasi awal — pilih sesuai kode:**

- **Any:** “Any membuat pemeriksaan tipe pada nilai ini dilewati. Operasi yang tidak sesuai dengan nilai sebenarnya bisa lolos pemeriksaan tipe dan baru bermasalah ketika dijalankan.”
- **As:** “As meminta compiler memperlakukan nilai sebagai tipe tertentu, tetapi tidak mengubah atau memvalidasi nilainya saat program berjalan. Pernyataan tipe yang keliru tetap bisa menyebabkan masalah.”

**Narasi perbaikan — pilih teknikmu:**

- **Union type:** “Saya menggunakan union type supaya kemungkinan tipe nilainya dinyatakan dengan jelas. Sebelum menjalankan operasi khusus, saya memastikan tipe yang sedang digunakan.”
- **Type guard:** “Saya memeriksa tipe sebelum nilai digunakan. Dengan begitu, operasi hanya dijalankan ketika tipe nilainya sesuai.”
- **Generics:** “Saya menggunakan generics agar hubungan tipe antara masukan dan keluaran tetap terjaga. Informasi tipe tidak hilang seperti ketika menggunakan any.”

**Setelah berhasil:**

> Hasil program sekarang sesuai dan pemeriksaan tsc selesai tanpa error. Kode ini sudah lolos pemeriksaan tipe, tetapi perilaku program tetap perlu diuji.

## 2. Instal dan kenali harness

**Tempat:** klik kanan `task-tracker` → **Open in Integrated Terminal**. Bagian manual harus sudah selesai.

```bash
npm install -g opencode-ai
npm install
npx tsc --noEmit
opencode
```

Pastikan pengecekan tipe awal bersih. Setelah OpenCode terbuka, ketik `/connect` di **chat OpenCode** dan hubungkan provider/model. Sembunyikan API key saat merekam.

Jika sudah ada `AGENTS.md`, gunakan salinan proyek tanpa file itu untuk percobaan awal; simpan versi aslinya. Jangan jalankan `/init` dahulu.

**Prompt A — tempel di chat OpenCode:**

```text
Jelaskan struktur task-tracker dan fungsi file utamanya. Baca package.json
untuk menentukan perintah menjalankan CLI. Berikan rencana singkat menambah
priority pada Task, tanpa mengubah kode. Sebutkan path file instruksi yang
benar-benar kamu baca; jika tidak ada, nyatakan tidak ada.
```

Tampilkan hasil. Catat nama model, perintah CLI, dan path file instruksi; periksa bukti file atau informasi pemuatannya bila tersedia.

**Narasi:**

> Saya menggunakan OpenCode sebagai harness untuk membantu agent membaca proyek, mengedit file, dan menjalankan perintah. Saya meminta penjelasan struktur proyek lebih dulu supaya perubahan dilakukan pada bagian yang tepat. Saya juga mencatat file instruksi yang dibaca dan perintah menjalankan proyek berdasarkan package.json.

## 3. Bandingkan tanpa dan dengan AGENTS.md

**Tempat:** editor VS Code. Simpan jawaban Prompt A. Buat `task-tracker/AGENTS.md`, sejajar dengan `package.json`, berisi:

```markdown
# Instruksi Proyek
- Jawab dalam bahasa Indonesia.
- Susun jawaban menjadi: Rencana, File Terkait, dan Validasi.
- Ikuti struktur proyek; hindari any/as untuk menutupi error.
- Pertahankan perilaku perintah yang sudah ada.
- Setelah mengubah kode, jalankan npx tsc --noEmit.
```

Gabungkan dengan instruksi asli jika sebelumnya sudah ada. Mulai **sesi OpenCode baru** di folder yang sama: tutup terminal harness, buka terminal baru di `task-tracker`, lalu jalankan `opencode`. Pakai model dan kode yang sama, kemudian kirim **Prompt A persis sama**. Tampilkan kedua jawaban. Catat jika percobaan pertama masih memakai instruksi global/folder induk.

**Narasi pembuka:**

> AGENTS.md berisi arahan kerja untuk agent. Saya memberikan prompt yang sama pada dua sesi dengan kode dan model yang sama untuk membandingkan pengaruh tambahan instruksi ini.

**Jika hasil mengikuti format baru:**

> Setelah AGENTS.md ditambahkan, jawaban mengikuti pembagian Rencana, File Terkait, dan Validasi. Perbedaannya terlihat pada format dan arahan pemeriksaan kode. File ini membantu menyampaikan aturan proyek secara konsisten.

**Jika hasil hampir sama:**

> Pada percobaan ini, kedua jawaban masih cukup mirip. Saya belum melihat perubahan besar dari penambahan AGENTS.md. Namun, aturan proyek sekarang sudah tertulis dan dapat digunakan kembali oleh agent.

## 4. Tambahkan priority

**Tempat:** chat OpenCode pada proyek `task-tracker`.

**Prompt pertama:**

```text
Tambahkan hanya field wajib priority: "low" | "medium" | "high" pada
Task. Jangan jadikan opsional. Jangan perbaiki pemakaiannya dulu.
Jalankan npx tsc --noEmit, tampilkan seluruh error, lalu berhenti.
```

Tampilkan perubahan dan **semua error compiler**, lalu catat jumlahnya. Jika tidak ada error, periksa tipe Task yang diedit dan cakupan file pemeriksaan compiler.

**Narasi saat error muncul:**

> Priority wajib bernilai low, medium, atau high. Bagian kode yang belum memenuhi definisi Task sekarang ditandai oleh compiler. Error tersebut membantu menunjukkan bagian yang perlu diperbaiki.

**Prompt kedua — masih di chat OpenCode:**

```text
Perbaiki seluruh error akibat priority, termasuk pembuatan task, data,
penyimpanan, dan tampilan yang relevan. Default priority adalah medium,
termasuk untuk data lama. Pertahankan field wajib; jangan menutupi error
menggunakan any/as atau mematikan strict. Jalankan npx tsc --noEmit sampai
bersih. Tunjukkan perintah membuat dan menampilkan task dengan priority.
```

Rekam edit agent dan tampilkan diff. **Buka terminal kedua di `task-tracker`**, jalankan `npx tsc --noEmit`, lalu perintah pengujian yang ditemukan agent.

**Setelah berhasil:**

> Agent telah menyesuaikan kode yang terdampak penambahan priority. Pemeriksaan TypeScript sekarang selesai tanpa error. Saya juga mencoba membuat dan menampilkan task untuk memastikan priority bekerja saat program dijalankan.

## 5. Bandingkan OpenCode dan Gemini CLI

**Di File Explorer komputer:** setelah poin 4, salin `task-tracker` menjadi `task-tracker-opencode` dan `task-tracker-gemini` di dalam `modul3`. Kode, data awal, dan AGENTS.md harus identik; keduanya belum memiliki filter baru.

**Terminal pada folder `task-tracker-opencode`:**

```bash
npm install
opencode
```

**Terminal lain pada folder `task-tracker-gemini`:**

```bash
npm install -g @google/gemini-cli
npm install
gemini
```

Ikuti autentikasi. Mulai sesi baru pada kedua harness dan catat model masing-masing.

**Prompt yang sama — tempel di chat masing-masing harness:**

```text
Baca AGENTS.md terlebih dahulu dan ikuti instruksinya. Tambahkan fitur
list --status done agar hanya menampilkan task dengan status done.
Tanpa --status, list harus tetap menampilkan semua task.
Tampilkan rencana singkat, lalu implementasikan. Jalankan npx tsc --noEmit
setelah implementasi pertama dan tampilkan hasil sebelum perbaikan.
Jika ada error, perbaiki lalu jalankan lagi sampai bersih.
Tunjukkan perintah pengujian berdasarkan CLI proyek ini.
```

**Narasi proses:**

> Saya memakai dua salinan proyek dengan kondisi awal dan prompt yang sama. Saya membandingkan kejelasan rencana, jumlah error TypeScript, koreksi manual yang diperlukan, serta kenyamanan penggunaannya. Fitur yang dibuat adalah filter agar list hanya menampilkan task berstatus done.

**Pengujian:** buka terminal biasa di masing-masing salinan. Siapkan task `done` dan berstatus lain. Jalankan CLI dengan argumen `list`, lalu `list --status done`, serta pemeriksaan tipe.

Contoh **hanya jika** script `dev` di proyek menjalankan CLI:

```bash
npm run dev -- list
npm run dev -- list --status done
npx tsc --noEmit
```

Jika script berbeda, gunakan perintah dari agent. `list` harus menampilkan semua task; filter hanya menampilkan task `done`.

**Isi tabel dari rekaman:**

| Aspek | OpenCode | Gemini CLI |
|---|---|---|
| Model | — | — |
| Rencana: kejelasan, ketepatan file, validasi | — | — |
| Error tsc pertama → terakhir | — | — |
| Jumlah koreksi prompt/edit manual | — | — |
| Kenyamanan alur kerja | — | — |

**Kesimpulan siap baca — pilih SATU yang sesuai tabel, setelah kedua fitur berhasil. Jika hasil berbeda, sesuaikan kalimatnya.**

**Jika OpenCode lebih jelas, error/intervensinya tidak lebih banyak, dan lebih nyaman:**

> Dalam percobaan ini, saya lebih nyaman menggunakan OpenCode karena rencananya lebih mudah diikuti, dengan jumlah error dan intervensi manual yang tidak lebih banyak daripada Gemini CLI. Kedua harness berhasil membuat filter list --status done dan lolos pemeriksaan TypeScript pada hasil akhirnya. OpenCode lebih sesuai untuk alur kerja saya pada percobaan ini, dengan model dan pengaturan yang saya gunakan.

**Jika Gemini CLI lebih jelas, error/intervensinya tidak lebih banyak, dan lebih nyaman:**

> Dalam percobaan ini, saya lebih nyaman menggunakan Gemini CLI karena rencananya lebih mudah diikuti, dengan jumlah error dan intervensi manual yang tidak lebih banyak daripada OpenCode. Kedua harness berhasil membuat filter list --status done dan lolos pemeriksaan TypeScript pada hasil akhirnya. Gemini CLI lebih sesuai untuk alur kerja saya pada percobaan ini, dengan model dan pengaturan yang saya gunakan.

**Jika seluruh aspek sebanding:**

> Dalam percobaan ini, kedua harness menghasilkan rencana yang dapat diikuti, dengan jumlah error dan intervensi manual yang relatif serupa. Filter list --status done berhasil dibuat dan hasil akhirnya lolos pemeriksaan TypeScript. Saya tidak menemukan perbedaan besar dalam kenyamanan alur kerjanya, sehingga belum ada satu harness yang jelas lebih unggul pada konfigurasi ini.

## Sebelum mengumpulkan

- [ ] Poin 1–5 berurutan; bagian manual tanpa AI.
- [ ] Wajah, proses mengetik/edit, error, perbaikan, dan hasil akhir terlihat.
- [ ] File instruksi tercatat; perbandingan AGENTS.md ditampilkan.
- [ ] Tabel dua harness terisi dan kesimpulan satu paragraf dibacakan.
- [ ] Upload ke YouTube: `NamaKelompok_Asistensi_Modul3_NamaAsisten`.
- [ ] Cek link lewat incognito, pastikan bisa dibuka.
- [ ] Isi data dan kirim link melalui [Form Pengumpulan](https://forms.gle/qpKhndCBiz3BCx8P8).

Referensi instalasi: [OpenCode](https://docs.opencode.ai/docs/) · [Gemini CLI](https://geminicli.com/docs/get-started/installation/).
