# CHALLENGE MODUL 3 — PANDUAN DARI NOL + CARA RESET SEBELUM REKAMAN

**Untuk Windows, VS Code, dan Git Bash.** Panduan ini memakai folder **`modul3`** (bukan `modul13`). Kamu boleh mencoba tiap tugas dulu, mengembalikan kondisinya, lalu merekam tugas itu dari awal **tanpa harus mengulang tugas lain**.

## Cara membaca panduan

- **📍 GIT BASH** = ketik perintah di jendela Git Bash atau terminal Git Bash di VS Code, lalu tekan **Enter**. Jangan mengetik tanda `$`.
- **📍 VS CODE (EDITOR)** = klik/buat file di panel kiri VS Code, lalu ketik kodenya di area editor. Simpan dengan **Ctrl+S**.
- **📍 CODEX / CLAUDE (CHAT AGENT)** = ketik instruksi setelah perintah `codex` atau `claude` membuka antarmuka agent. **Bukan** perintah terminal biasa.
- **🎤 NARASI VIDEO** = contoh penjelasan singkat yang boleh kamu ucapkan saat merekam. Sesuaikan dengan hasil nyata di laptop.
- **🔁 RESET** = lakukan **setelah latihan dan sebelum mulai rekam nomor tersebut**. Backup jangan ikut dihapus.

> **Aturan penting:** Video **wajib dimulai dengan Tugas 1 (coding TypeScript manual)**, tanpa harness/AI. Baca dan latih dulu menggunakan panduan ini. Saat merekam Tugas 1, tutup AI dan **ketik kode sendiri** (jangan copy–paste).

---

# 0. PERSIAPAN — MULAI DARI NOL

### 0.1 Buka VS Code dan terminal

1. Buka **Visual Studio Code**.
2. Pilih **Terminal → New Terminal**.
3. Jika terminalnya PowerShell (`PS C:\...>`), kamu tetap bisa membuka aplikasi **Git Bash** dari Start Menu; atau di terminal VS Code pilih panah kecil di sebelah `+` → **Select Default Profile → Git Bash**, lalu buka terminal baru.
4. Seluruh perintah bertanda **GIT BASH** diketik di terminal Git Bash.

### 0.2 Cek aplikasi

📍 **GIT BASH — boleh dari folder mana saja:**

```bash
node -v
npm -v
git --version
```

**Hasil:** Masing-masing menampilkan nomor versi. Kalau `node`/`npm` tidak ditemukan, instal **Node.js LTS** dari https://nodejs.org/ dan buka ulang terminal. Kalau `git` tidak ditemukan, instal Git for Windows dari https://git-scm.com/downloads.

### 0.3 Buat folder `modul3`

📍 **GIT BASH — ketik satu per satu:**

```bash
cd ~
mkdir -p modul3
cd modul3
pwd
```

**Artinya:** `cd ~` menuju folder pengguna Windows, `mkdir -p modul3` membuat folder (kalau sudah ada tidak masalah), `cd modul3` masuk ke sana, dan `pwd` menunjukkan lokasi aktif. Hasilnya kurang lebih `/c/Users/NAMAKAMU/modul3`.

📍 **VS CODE:** **File → Open Folder → pilih folder `modul3`** yang tadi dibuat. Lalu buka terminal baru. Jangan buat `modul13` lagi.

🎤 **NARASI:** “Saya menyiapkan folder modul3 dan mengecek Node.js, npm, serta Git sebelum membuat proyek latihan.”

---

# 1. AUDIT TYPESCRIPT MANUAL — WAJIB PERTAMA DI VIDEO

**Tujuan:** Tunjukkan risiko `any`, munculkan runtime error, perbaiki dengan **Union Type + Type Guard**, lalu jalankan `npx tsc --noEmit` sampai tidak ada error.

### 1.1 Buat folder latihan TypeScript

📍 **GIT BASH — pastikan sedang di `modul3` (`pwd`), bukan di task-tracker:**

```bash
mkdir audit-ts
cd audit-ts
npm init -y
npm install typescript @types/node --save-dev
npx tsc --init
npm install tsx --save-dev
mkdir src
```

**Penjelasan:** `audit-ts` adalah proyek latihan terpisah. `npm init -y` membuat `package.json`; `npm install` memasang alat; `tsc --init` membuat `tsconfig.json`; dan `src` adalah folder kode.

📍 **VS CODE (EDITOR):** Di Explorer kiri, buka `audit-ts/tsconfig.json`. Cari `"strict"` dan pastikan ada:

```json
"strict": true
```

Lalu klik kanan folder `src` → **New File** → beri nama **`audit.ts`**.

🎤 **NARASI:** “Saya membuat proyek audit-ts terpisah dari task-tracker dan mengaktifkan strict mode agar pemeriksaan TypeScript lebih ketat.”

### 1.2 Ketik kode bermasalah (MANUAL)

📍 **VS CODE (EDITOR) → `modul3/audit-ts/src/audit.ts`**. **Ketik sendiri**, jangan paste saat rekaman:

```typescript
function tampilkanNama(nama: any) {
  console.log("Nama: " + nama.toUpperCase());
}

const data1 = "azmi";
const data2 = 123;

tampilkanNama(data1);
tampilkanNama(data2);
```

**Penjelasan:** `any` membiarkan berbagai jenis data masuk. `toUpperCase()` hanya berlaku untuk string, tetapi kita juga mengirim angka `123`.

📍 **GIT BASH — harus berada di `modul3/audit-ts`:**

```bash
npx tsx src/audit.ts
```

**Hasil yang diharapkan:** Muncul `Nama: AZMI`, kemudian `TypeError: nama.toUpperCase is not a function`. Ini **runtime error** (bukan error instalasi).

🎤 **NARASI:** “Saat parameter memakai any, TypeScript tidak mencegah angka masuk ke operasi string. Hasilnya, program gagal ketika mencoba menjalankan toUpperCase pada angka.”

### 1.3 Perbaiki dengan Union Type dan Type Guard (MANUAL)

📍 **VS CODE (EDITOR) → file `src/audit.ts` yang sama**. Ganti isinya, **ketik sendiri**:

```typescript
function tampilkanNama(nama: string | number) {
  if (typeof nama === "string") {
    console.log("Nama: " + nama.toUpperCase());
  } else {
    console.log("Nama: " + nama.toString());
  }
}

tampilkanNama("azmi");
tampilkanNama(123);
```

**Penjelasan:** `string | number` adalah **Union Type** (hanya menerima dua jenis data). `typeof` adalah **Type Guard** yang memilih operasi yang aman sesuai tipenya.

📍 **GIT BASH — masih di `audit-ts`:**

```bash
npx tsx src/audit.ts
npx tsc --noEmit
```

**Hasil yang diharapkan:** `Nama: AZMI` dan `Nama: 123`. Perintah `tsc --noEmit` biasanya **tidak mencetak apa pun jika berhasil**: terminal kembali ke `$` tanpa pesan error.

🎤 **NARASI:** “Saya mengganti any dengan Union Type string atau number dan menggunakan Type Guard agar tiap data ditangani dengan tepat. Setelah diuji, program berjalan dan pemeriksaan tsc tidak melaporkan error.”

### 🔁 RESET TUGAS 1 (setelah latihan, sebelum rekaman)

📍 **FILE EXPLORER WINDOWS, bukan agent:**

1. **Tutup file yang sedang diedit**, lalu kembali ke folder `modul3` di File Explorer.
2. **Hapus hanya folder `audit-ts`** hasil latihan (jangan hapus `modul3` atau folder lain).
3. Saat rekaman dimulai, ulangi **Tugas 1 dari langkah 1.1**. Dengan begitu pembuatan folder, pengetikan manual, error, dan perbaikan benar-benar terlihat.
4. Instalasi Node.js, Git, dan VS Code **tidak perlu diulang**.

> Kalau yang ingin kamu ulang **hanya kesalahan kode** (bukan instalasi), tidak perlu hapus folder: kosongkan `src/audit.ts`, lalu mulai lagi dari **1.2**. Tetapi untuk rekaman proses lengkap, gunakan reset folder seperti di atas.

---

# 2. INSTAL & KENALI HARNESS — CODEX CLI

**Tujuan:** Jalankan coding agent pada **proyek task-tracker yang sudah dibuat pada modul sebelumnya**, lalu minta agent menjelaskan strukturnya dan file instruksi yang dibaca.

### 2.1 Siapkan proyek task-tracker yang lama

📍 **FILE EXPLORER WINDOWS:** Cari folder **`task-tracker` lama** dari modul sebelumnya. **Copy**, kemudian **Paste** ke folder `modul3`, sehingga tersedia `modul3/task-tracker`. **Sebelum latihan Tugas 2, copy lagi folder ini dan beri nama `task-tracker-sebelum-2` sebagai backup.**

> Jangan membuat folder `task-tracker` kosong sebagai pengganti proyek lama. Kalau proyek lamanya tidak ada, cari atau siapkan proyek tersebut dulu sebelum melanjutkan tugas 2–5.

📍 **VS CODE:** **File → Open Folder → pilih `modul3/task-tracker`**.

📍 **GIT BASH — terminal di root `task-tracker`:**

```bash
pwd
ls
npm install
```

`pwd` harus berakhir `/modul3/task-tracker`. `ls` menunjukkan file proyek. `npm install` memasang dependensi berdasarkan `package.json`.

### 2.2 Instal dan jalankan Codex

📍 **GIT BASH — boleh dijalankan dari folder apa pun:**

```bash
npm install -g @openai/codex
codex --version
```

**Hasil:** versi Codex tampil. Instalasi ini bersifat global: **tidak perlu diulang setiap rekaman**.

📍 **GIT BASH — kembali/pastikan di root `modul3/task-tracker`:**

```bash
codex
```

Login jika diminta. Jangan merekam password atau kode login.

📍 **CHAT CODEX (bukan Git Bash biasa) — ketik prompt berikut:**

```text
Jelaskan struktur proyek task-tracker ini tanpa mengubah file.
Sebutkan fungsi aplikasi, lokasi tipe Task, file penting,
cara menjalankan aplikasi, dan perintah pemeriksaan TypeScript.
Sebutkan juga file instruksi agent yang benar-benar kamu baca.
Jika tidak ada, katakan tidak ada.
```

**Hasil:** Agent menjelaskan berdasarkan file proyekmu. **Catat nama file yang benar-benar ditemukan**, jangan mengarang lokasi file.

🎤 **NARASI:** “Setelah coding manual, saya menjalankan Codex CLI pada proyek task-tracker lama. Agent saya minta memetakan struktur dan menyebutkan file instruksi yang dibaca.”

### 🔁 RESET TUGAS 2

- Karena promptnya **hanya membaca**, normalnya **tidak ada kode untuk dikembalikan**.
- Setelah latihan, keluar dari Codex (gunakan perintah keluar yang ditampilkan aplikasi), lalu saat merekam jalankan `codex` lagi untuk **sesi baru**.
- Jika saat latihan agent **ternyata mengubah file**, tutup VS Code/agent lalu arsipkan folder kerja `task-tracker` dan **copy `task-tracker-sebelum-2` menjadi folder `task-tracker`** melalui File Explorer. Backup jangan ikut dihapus.
- **Tidak perlu uninstall/reinstall Codex**; cukup rekam `codex --version` sebagai bukti sudah terpasang.

---

# 3. UJI AGENTS.md — DENGAN VS TANPA INSTRUKSI

**Tujuan:** Kirim **prompt yang sama persis** sebelum dan setelah membuat `AGENTS.md`.

### 3.1 Kondisi TANPA AGENTS.md

📍 **FILE EXPLORER — sebelum latihan Tugas 3:** Copy folder kerja `task-tracker` (hasil Tugas 2), paste di `modul3`, lalu beri nama **`task-tracker-sebelum-3`**. Simpan tanpa diubah.

📍 **VS CODE:** Pastikan root `task-tracker` **belum memiliki `AGENTS.md`**. Kalau sudah ada sejak proyek lama, **jangan hapus sembarangan**: buat salinan untuk eksperimen dan simpan file asli sebagai backup.

📍 **GIT BASH — di `modul3/task-tracker`:**

```bash
codex
```

📍 **CHAT CODEX — prompt percobaan A:**

```text
Saya ingin menambahkan fitur priority pada task-tracker.
Jelaskan rencana perubahan, file yang perlu diperiksa,
dan cara menguji hasilnya. Jangan ubah file apa pun.
```

**Hasil:** Catat respons pertama. Keluar dari sesi Codex setelah selesai.

### 3.2 Buat AGENTS.md

📍 **VS CODE (EDITOR):** Klik kanan folder utama `task-tracker` → **New File** → `AGENTS.md`. Isikan:

```markdown
# Aturan Proyek Task Tracker

- Gunakan TypeScript dan hindari `any`.
- Pertahankan fitur yang sudah berjalan.
- Jelaskan rencana sebelum mengubah kode.
- Ubah hanya file yang relevan.
- Setelah mengubah kode, jalankan `npx tsc --noEmit`.
- Untuk priority, gunakan "low" | "medium" | "high".
```

### 3.3 Kondisi DENGAN AGENTS.md

📍 **GIT BASH — root `task-tracker`:**

```bash
codex
```

📍 **CHAT CODEX:** Kirim **prompt percobaan A yang sama persis**. Catat apakah jawaban kini mengikuti aturan file tersebut.

🎤 **NARASI:** “Saya membandingkan prompt yang identik sebelum dan sesudah membuat AGENTS.md. File ini memberi aturan agar agent lebih konsisten, misalnya menghindari any dan menjalankan pemeriksaan tipe. Saya membandingkan jawaban nyata, bukan mengasumsikan pasti lebih baik.”

### 🔁 RESET TUGAS 3

- **Cara paling aman untuk semua kondisi:** Tutup Codex dan VS Code, arsipkan/hapus **hanya folder kerja `task-tracker` hasil latihan**, lalu **copy folder `task-tracker-sebelum-3` menjadi `task-tracker`** di `modul3`. Buka lagi proyek ini dan rekam mulai **3.1**.
- Alternatif bila **hanya** file `AGENTS.md` yang berubah dan tadinya tidak ada: cukup hapus file baru tersebut. Jangan hapus aturan asli yang sudah ada sebelum latihan.
- **Setelah Tugas 3 berhasil DIREKAM, biarkan `AGENTS.md` tetap ada** untuk Tugas 4–5.

---

# 4. FITUR PRIORITY — TAMPILKAN ERROR, LALU MINTA AGENT MEMPERBAIKI

**Tujuan:** Tambahkan `priority: "low" | "medium" | "high"` pada tipe `Task`, tunjukkan error dari compiler (jika ada), lalu minta Codex memperbaiki semua bagian yang terdampak.

### 4.0 Buat backup SEBELUM latihan Tugas 4

📍 **FILE EXPLORER:** Tutup sesi Codex. Di folder `modul3`, **copy folder `task-tracker`**, lalu **paste dan beri nama `task-tracker-sebelum-4`**. Ini checkpoint yang harus tetap utuh. Pastikan `AGENTS.md` dari Tugas 3 sudah masuk dalam backup.

### 4.1 Cek kondisi sebelum perubahan

📍 **GIT BASH — di `modul3/task-tracker`:**

```bash
npx tsc --noEmit
```

Catat hasil awal. Kalau ada error lama, pisahkan dari error baru.

### 4.2 Tambahkan priority secara manual

📍 **VS CODE (EDITOR):** Tekan **Ctrl+Shift+F**, cari `interface Task` atau `type Task`. Buka file yang mendefinisikan `Task` dan **tambahkan properti ini** di dalam tipe yang sudah ada:

```typescript
priority: "low" | "medium" | "high";
```

**Jangan mengganti seluruh definisi Task** dengan contoh dari internet; cukup tambah properti. `priority` di sini **wajib**, jangan tambah tanda `?`.

📍 **GIT BASH — root `task-tracker`:**

```bash
npx tsc --noEmit
```

**Hasil:** Compiler bisa menampilkan bahwa objek Task belum memiliki `priority`. **Jumlah error tergantung proyekmu**; kalau tidak ada error, periksa bahwa tipe yang kamu ubah dipakai oleh kode proyek dan termasuk pemeriksaan `tsconfig.json`.

### 4.3 Minta Codex memperbaiki

📍 **GIT BASH — root `task-tracker`:**

```bash
codex
```

📍 **CHAT CODEX — prompt:**

```text
Saya menambahkan field priority: "low" | "medium" | "high"
pada tipe Task. Jalankan npx tsc --noEmit dan catat
seluruh error. Buat rencana singkat, lalu perbaiki bagian
pembuatan, pembaruan, penyimpanan, dan tampilan Task
jika relevan. Tangani data lama yang belum punya priority.
Jangan gunakan any dan ikuti AGENTS.md.
Setelah selesai, jalankan lagi npx tsc --noEmit,
uji fitur yang tersedia, dan sebutkan file yang diubah.
```

**Hasil:** Rekam rencana agent, file yang diedit, error sebelum/sesudah, dan pemeriksaan akhir. Setujui perubahan hanya setelah membacanya.

📍 **GIT BASH — setelah keluar dari agent:**

```bash
npx tsc --noEmit
npm run
```

`npm run` **menampilkan daftar script yang tersedia**, bukan otomatis menjalankan aplikasi. Jalankan script proyek sesuai `package.json` (misalnya `npm run dev` **hanya kalau script `dev` ada**).

🎤 **NARASI:** “Saya menambahkan priority sebagai Union Type wajib. Compiler menunjukkan bagian kode yang perlu disesuaikan. Codex kemudian memperbaiki bagian terdampak, dan saya mengecek kembali TypeScript serta fungsi aplikasinya.”

### 🔁 RESET TUGAS 4

📍 **FILE EXPLORER — setelah latihan, SEBELUM rekam Tugas 4:**

1. Tutup sesi Codex dan VS Code yang masih membuka file di `task-tracker`.
2. Di dalam `modul3`, hapus atau pindahkan ke lokasi arsip **hanya folder kerja `task-tracker` yang berubah saat latihan**.
3. **Copy** folder `task-tracker-sebelum-4` → **Paste**, lalu beri nama salinannya **`task-tracker`**.
4. Buka `modul3/task-tracker` lagi di VS Code dan mulai rekam dari **4.1**.
5. **Jangan hapus** `task-tracker-sebelum-4`.

> Dengan cara ini, **Tugas 1–3 tidak perlu diulang**. Setelah Tugas 4 selesai direkam, lanjut ke Tugas 5 dari proyek yang sudah mempunyai priority.

---

# 5. BANDINGKAN DUA HARNESS — CODEX VS CLAUDE CODE

**Tujuan:** Minta dua agent membuat fitur yang sama: `list --status done`. Bandingkan rencana, error TypeScript, intervensi manual, dan kenyamanan.

### 5.0 Buat baseline yang SAMA

📍 **FILE EXPLORER — setelah Tugas 4 SELESAI:**

1. Copy `modul3/task-tracker` yang sudah memiliki priority.
2. Paste dan beri nama **`task-tracker-awal-5`**. **Jangan diedit** (ini backup/baseline).
3. Copy `task-tracker-awal-5` dua kali, lalu beri nama **`task-tracker-codex`** dan **`task-tracker-claude`**.
4. Pastikan keduanya berisi kode dan `AGENTS.md` yang sama.

### 5.1 Uji awal kedua folder

📍 **VS CODE:** **File → Open Folder → `modul3/task-tracker-codex`**.

📍 **GIT BASH — root `task-tracker-codex`:**

```bash
npm install
npx tsc --noEmit
codex
```

📍 **CHAT CODEX — gunakan prompt di bawah:**

```text
Baca AGENTS.md di root proyek.
Tambahkan fitur list --status done supaya hanya
menampilkan Task yang statusnya done.
Perintah list tanpa filter harus tetap menampilkan semua Task.
Tangani status tidak valid sesuai tipe status proyek.
Sebelum coding, jelaskan rencana dan file yang akan diubah.
Jangan gunakan any. Setelah implementasi pertama, jalankan
npx tsc --noEmit, laporkan jumlah error, kemudian perbaiki.
Uji fitur dan laporkan intervensi manual yang dibutuhkan.
```

**Catat:** rencana, jumlah error `tsc` setelah implementasi pertama, error akhir, apakah fitur bekerja, dan berapa kali kamu memberi arahan tambahan.

### 5.2 Instal dan jalankan Claude Code

📍 **TERMINAL POWERSHELL — bukan Git Bash**, dapat dibuka dari menu terminal VS Code:

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Hasil:** Tunggu instalasi selesai. Buka terminal baru, kemudian cek `claude --version`. Perlu login dan akun dengan akses Claude Code yang sesuai. Instalasi ini juga **tidak perlu diulang** setiap rekaman.

📍 **VS CODE:** **File → Open Folder → `modul3/task-tracker-claude`**.

📍 **GIT BASH — root `task-tracker-claude`:**

```bash
npm install
npx tsc --noEmit
claude
```

📍 **CHAT CLAUDE:** Kirim **prompt yang sama persis** seperti pada **5.1**. Pastikan Claude benar-benar membaca `AGENTS.md` sesuai permintaan; jangan menganggap file itu otomatis dibaca.

**Catat:** aspek penilaian yang sama dengan Codex.

### 5.3 Uji aplikasi dan bandingkan

`list --status done` **bukan perintah bawaan Git Bash**. Cara menjalankannya mengikuti desain `task-tracker` milikmu. Lihat `package.json`, atau minta masing-masing agent menunjukkan perintah pengujian yang tepat.

**Contoh perilaku yang diharapkan:** Kalau ada 3 Task, dengan status `done`, `todo`, `done`, maka `list` menampilkan semuanya, sedangkan `list --status done` hanya menampilkan 2 Task berstatus `done`.

| Penilaian | Codex CLI | Claude Code |
|---|---|---|
| Rencana jelas? | Isi hasil nyata | Isi hasil nyata |
| Error `tsc` setelah implementasi awal | ... | ... |
| Error `tsc` akhir | ... | ... |
| Berapa intervensi manual? | ... | ... |
| Fitur berhasil diuji? | ... | ... |
| Alur kerja lebih nyaman? | ... | ... |

🎤 **NARASI:** “Saya menguji fitur yang sama di dua salinan proyek dengan kondisi awal sama. Saya membandingkan kejelasan rencana, hasil tsc, intervensi manual, dan kenyamanan menggunakan Codex serta Claude.”

### 🔁 RESET TUGAS 5

📍 **FILE EXPLORER — setelah latihan, SEBELUM rekam Tugas 5:**

1. Tutup kedua agent.
2. Hapus atau arsipkan **hanya folder latihan `task-tracker-codex` dan `task-tracker-claude`**.
3. **Copy dua kali dari `task-tracker-awal-5`**, lalu beri nama **`task-tracker-codex`** dan **`task-tracker-claude`** lagi.
4. Jalankan ulang pengujian dari **5.1** saat merekam.
5. **Tugas 1–4 tidak perlu diulang**. Codex dan Claude juga tidak perlu diinstal ulang.

### 5.4 Kesimpulan (wajib satu paragraf)

> “Berdasarkan percobaan fitur `list --status done` pada dua proyek dengan kondisi awal yang sama, Codex CLI menunjukkan [hasil nyata] dan Claude Code menunjukkan [hasil nyata]. Pada aspek rencana, [perbandingan]. Jumlah error TypeScript setelah implementasi pertama masing-masing [angka] dan [angka], sedangkan jumlah intervensi manual [perbandingan]. Dari segi kenyamanan dan hasil pengujian, [harness yang lebih sesuai] lebih cocok untuk proyek ini karena [alasan nyata].”

---

# RINGKASAN RESET PER NOMOR

| Mau ulang nomor | Apa yang di-reset? | Bagian lain perlu diulang? |
|---|---|---|
| **1** | Hapus folder latihan `audit-ts`, lalu buat lagi saat rekaman | **Tidak** |
| **2** | Sesi Codex baru; bila kode berubah, pulihkan dari `task-tracker-sebelum-2` | **Tidak** |
| **3** | Pulihkan `task-tracker` dari `task-tracker-sebelum-3` | **Tidak** |
| **4** | Pulihkan `task-tracker` dari `task-tracker-sebelum-4` | **Tidak** |
| **5** | Buat ulang dua folder harness dari `task-tracker-awal-5` | **Tidak** |

**Aturan aman:** Jangan menghapus folder backup (`task-tracker-sebelum-2`, `task-tracker-sebelum-3`, `task-tracker-sebelum-4`, dan `task-tracker-awal-5`). Sebelum menghapus folder kerja, lihat **nama dan alamat lengkapnya**. Jangan hapus proyek lama yang berada di luar `modul3`.

## Urutan rekaman video

**1 → 2 → 3 → 4 → 5**, sesuai instruksi challenge. Boleh rekam **satu nomor per video**, lalu gabungkan sesuai ketentuan pengumpulan. Awali video final dengan **coding TypeScript manual**, bukan pengenalan AI.

## Referensi instalasi resmi

- Node.js: https://nodejs.org/
- VS Code: https://code.visualstudio.com/
- Git for Windows: https://git-scm.com/downloads
- Codex CLI: https://github.com/openai/codex
- Claude Code: https://support.claude.com/en/articles/14552382-your-first-day-in-claude-code
