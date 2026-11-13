# Demo Pembuktian Modul 4 — MLBB Shop

Ikuti nomor **Spesifikasi Fitur 1–7** pada modul. Request bernomor merujuk collection Postman; body-nya sudah terisi. Request tambahan dibuat di tab baru.

## Persiapan

**Alamat demo:** `http://localhost:3001`.

**Terminal VS Code, folder `mlbb-shop` yang berisi `package.json`:**

1. Jika `JALANKAN-NEON.cmd` masih menjalankan server, hentikan dengan **Ctrl+C**.
2. Jalankan `npm.cmd run demo:reset:neon` untuk mengembalikan data awal.
3. Jalankan lagi **JALANKAN-NEON.cmd**. Tunggu server Neon siap dan periksa alamatnya `http://localhost:3001`.

**Postman:** import `postman/MLBB-Shop-28-Testcase.postman_collection.json`. Pastikan semua request collection menuju `http://localhost:3001`. Jika URL masih memakai port lain, unduh collection dari `http://localhost:3001/postman/collection.json`, lalu impor ulang.

Saat memasuki live coding di bagian akhir, ikuti langkah perpindahan server ke `dev:neon` yang tersedia di bawah.

**Bukti awal — tunjukkan sebelum mengirim request:**

| Ketentuan modul | Tunjukkan di mana | Bukti yang terlihat |
| --- | --- | --- |
| Stack wajib | VS Code: `package.json`, `src/lib/database.ts` | Express 5, TypeScript, Prisma 7, adapter Neon |
| Schema | VS Code: `prisma/schema.prisma` | Hero, Skin, Player, PlayerHero; empat enum; unique, default saldo, dan cascade |
| Dataset dan seed | VS Code: `src/lib/seed-data.ts`; terminal baru: `npm.cmd run db:studio` | TRUNCATE + createMany; di Studio ada 133 Hero, 892 Skin, 3 Player, 4 PlayerHero |

## 1. Middleware

**Jalankan di Postman; lihat juga terminal server.**

| Buktikan | Jalankan | Hasil yang ditunjukkan |
| --- | --- | --- |
| Server dan koneksi Neon | Request **01**: `GET /health` | 200; `status: true`, database `connected`, mode `neon` |
| Logger | Lihat terminal setelah request 01 | Ada method, URL, dan status: `GET /health 200` |
| Error handler | Request tambahan: `GET /api/heroes/999` | 404; JSON berisi `status: false` dan `message` |
| notFound | Request tambahan: `GET /api/ngawur` | 404; format response gagal yang sama |

**Penjelasan singkat:** setiap request dicatat dan setiap error memakai format yang ditentukan modul. Tunjukkan `HttpError`, `notFound`, dan `errorHandler` di `src/middleware/errors.ts`.

## 2. Hero

**Jalankan di Postman, urut seperti tabel.**

| Buktikan | Jalankan | Hasil yang ditunjukkan |
| --- | --- | --- |
| Pagination default | Tambahan: `GET /api/heroes` | 10 data; metadata currentPage 1, limit 10, totalItems 133, totalPages 14 |
| Pagination halaman 2 | Tambahan: `GET /api/heroes?page=2&limit=5` | 5 data, id 6–10; currentPage 2, totalItems 133, totalPages 27 |
| Filter role | Request **02** | Semua MARKSMAN; tampil 5 data, totalItems 20 |
| Search | Request **03**, lalu tambahan `GET /api/heroes?search=LA` | Keduanya totalItems 10; membuktikan search tidak peka kapital |
| Detail beserta skin | Request **04** | Layla, 11 skin pada `data.skins` |
| Hero tidak ditemukan | Request **05** | 404 |
| Tambah hero | Request **06** | 201; Collie id 134 |
| Nama duplikat | Request **07** | 409 |
| Field wajib kurang | Request **08** | 400 |

**Tunjukkan kode:** `listHeroes`, `heroDetail`, dan `createHero` di `src/controllers/catalog.controller.ts`: filter, skip/take, `include`, serta validasi sebelum create.

## 3. Skin

**Jalankan di Postman.**

| Buktikan | Jalankan | Hasil yang ditunjukkan |
| --- | --- | --- |
| Filter hero | Request **09** | totalItems 11; skin Layla, termasuk `hero.name` |
| Filter rarity | Request **10** | totalItems 98; semua GRAND |
| Detail beserta hero | Request **11** | Silverbolt Raiko, hero Selena |
| Pagination | Tambahan: `GET /api/skins?page=2&limit=5` | 5 skin; currentPage 2, limit 5, totalItems 892 |
| Skin tidak ditemukan | Tambahan: `GET /api/skins/99999` | 404 |

**Tunjukkan kode:** `listSkins` dan `skinDetail` di controller yang sama; relasi hero diambil dengan `include`.

## 4. Player

**Jalankan request 12–13 sekarang. Detail/update/delete dibuktikan pada urutan collection 22–27.**

| Buktikan | Jalankan di Postman | Hasil yang ditunjukkan |
| --- | --- | --- |
| Buat player dan saldo default | Request **12** | 201; JunglerMalam id 4, BP 10000, Diamond 0 |
| Nickname unik | Request **13** | 409; ArekITS sudah dipakai |
| Nickname wajib diisi | Tambahan: `POST /api/players`, body `{"nickname":""}` | 400 |
| Detail dan koleksi | Nanti request **22** | Player 3 beserta Freya, Layla, currency, dan pricePaid |
| Update nickname | Nanti request **23–24** | MageITS berhasil 200; nickname ArekITS ditolak 409 |
| Hapus player | Nanti request **26–27**, setelah statistik | DELETE 200; GET player yang dihapus 404 |

**Tunjukkan kode:** default BP/Diamond dan `onDelete: Cascade` pada schema; operasi player di `src/controllers/player.controller.ts`.

## 5. Top Up dan Match

**Jalankan di Postman.**

| Buktikan | Jalankan | Hasil yang ditunjukkan |
| --- | --- | --- |
| Top up positif | Request **14** | 200; Diamond player 3 menjadi 706 |
| Amount tidak valid | Request **15**, lalu tambahan `POST /api/players/3/topup`, body `{"amount":1.5}` | Keduanya 400; saldo tidak bertambah |
| Hadiah menang | Request **16** | 200; bpEarned 300, BP player 3 menjadi 10300 |

**Tunjukkan kode:** `addBalance` di `src/services/player.service.ts` memakai `increment`; controller memilih WIN = 300 dan LOSE = 100. LOSE dijalankan setelah request 28 agar angka checkpoint tetap sesuai.

## 6. Beli Hero

**Jalankan di Postman.**

| Buktikan | Jalankan | Hasil yang ditunjukkan |
| --- | --- | --- |
| Harga null tidak bisa dipakai | Request **17** | Freya dengan BP ditolak 400 |
| Pembelian Diamond | Request **18** | 201; harga 599, sisa Diamond 107 |
| Pembelian BP | Request **19** | 201; harga 2000, sisa BP 8300 |
| Tidak boleh membeli hero yang dimiliki | Request **20** | 409 |
| Saldo kurang | Request **21** | 400 |
| Pembelian tersimpan | Request **22** | Freya dan Layla muncul; BP 8300, Diamond 107; currency dan pricePaid benar |
| Lanjut bukti update Player | Request **23–24** | Nickname menjadi MageITS; duplikat ditolak 409 |

**Tunjukkan kode:** `buyHero` di `src/services/player.service.ts`. Urutannya input → player → hero → harga → kepemilikan → saldo. `$transaction([ ... ])` berisi decrement saldo dan create PlayerHero, sehingga keduanya berhasil atau gagal bersama.

## 7. Statistik

**Jalankan di Postman: request 25 — `GET /api/statistics`.**

| Field sesuai modul | Hasil yang ditunjukkan |
| --- | --- |
| `totalPlayers` | 4 |
| `totalHeroesOwned` | 6 |
| `totalDiamond` | 3074 |
| `mostOwnedHero` | Layla, heroId 18, owners 2 |
| `heroesPerRole` | FIGHTER 37, MAGE 26, ASSASSIN 20, MARKSMAN 20, TANK 18, SUPPORT 13 |

**Tunjukkan kode:** `src/controllers/statistics.controller.ts`: `count`, `aggregate` dengan `_sum`, serta `groupBy`. Total hero 134 karena Collie sudah dibuat.

**Selesaikan collection:** request **26** → 200, **27** → 404, **28** → 404.

**Setelah request 28, buktikan dua ketentuan berikut di Postman:**

| Ketentuan | Jalankan | Hasil |
| --- | --- | --- |
| Hadiah kalah | `POST /api/players/3/matches`, body `{"result":"LOSE"}` | bpEarned 100; BP menjadi 8400 |
| Cascade kepemilikan | `DELETE /api/players/1`, lalu di Prisma Studio cek PlayerHero dengan playerId 1 | Player terhapus; dua kepemilikannya ikut hilang |

## Terakhir: Live Coding Test Case 2 — Filter Lane

**Catatan monitor 1 — bagian privat kelompok.** Kerjakan setelah request 28. Gunakan kode awal tanpa filter lane saat memulai rekaman.

### 1. Pindahkan server ke mode live coding

1. Pada jendela **JALANKAN-NEON.cmd**, tekan **Ctrl+C** sampai server berhenti. Jika muncul pertanyaan menghentikan batch, jawab **Y**.
2. Buka terminal VS Code di folder `mlbb-shop` yang berisi `package.json`.
3. Jalankan `$env:PORT = "3001"`, lalu jalankan `npm.cmd run dev:neon` pada terminal yang sama.
4. Tunggu server siap di `http://localhost:3001`. Biarkan terminal ini terbuka sampai demo selesai.

**Hasil:** koneksi Neon dan data tetap sama; server sekarang otomatis memuat perubahan setelah kode disimpan. Lanjutkan dari data demo yang sudah ada.

### 2. Tunjukkan kondisi awal — Postman

Kirim `GET http://localhost:3001/api/heroes?lane=GOLD&limit=5`.

**Sebelum perubahan:** lane masih diabaikan, data bercampur lane dan totalItems 134 setelah Collie dibuat.

### 3. Tempel kode — VS Code

**File:** `src/controllers/catalog.controller.ts`.

**Ucapkan:** “Saya menambahkan query lane sesuai enum. Filter lane digabung dengan search dan role. Query daftar dan count memakai where yang sama, sehingga metadata menghitung hasil filter sebelum pagination.”

**Klik isi file → Ctrl+A → tempel seluruh blok berikut → Ctrl+S.** Ini isi lengkap file, termasuk semua method Hero dan Skin.

```typescript
import type { Request, Response } from "express";
import type { Database } from "../lib/database.js";
import {
  heroSchema,
  roleSchema,
  laneSchema,
  raritySchema,
  pagination,
  queryText,
  queryInteger,
  pathId,
  metadata,
} from "../lib/validation.js";
import { HttpError } from "../middleware/errors.js";
import type { Prisma } from "../generated/prisma/client.js";

export function catalogController(db: Database) {
  return {
    async listHeroes(req: Request, res: Response) {
      const { page, limit, skip } = pagination(req);
      const search = queryText(req, "search")?.trim();
      const role = queryText(req, "role");
      const lane = queryText(req, "lane");
      const where: Prisma.HeroWhereInput = {
        ...(search ? { name: { contains: search, mode: "insensitive" } } : {}),
        ...(role !== undefined ? { role: roleSchema.parse(role) } : {}),
        ...(lane !== undefined ? { lane: laneSchema.parse(lane) } : {}),
      };
      const [data, total] = await Promise.all([
        db.hero.findMany({ where, skip, take: limit, orderBy: { id: "asc" } }),
        db.hero.count({ where }),
      ]);
      res.json({
        status: true,
        message: "Berhasil mengambil data hero",
        data,
        metadata: metadata(page, limit, total),
      });
    },
    async heroDetail(req: Request, res: Response) {
      const data = await db.hero.findUnique({
        where: { id: pathId(req, "hero_id") },
        include: { skins: { orderBy: { id: "asc" } } },
      });
      if (!data) throw new HttpError(404, "Hero tidak ditemukan");
      res.json({
        status: true,
        message: "Berhasil mengambil detail hero",
        data,
      });
    },
    async createHero(req: Request, res: Response) {
      const input = heroSchema.parse(req.body);
      if (await db.hero.findUnique({ where: { name: input.name } }))
        throw new HttpError(409, "Nama hero sudah digunakan");
      const data = await db.hero.create({ data: input });
      res
        .status(201)
        .json({ status: true, message: "Hero berhasil ditambahkan", data });
    },
    async listSkins(req: Request, res: Response) {
      const { page, limit, skip } = pagination(req);
      const rarity = queryText(req, "rarity");
      const heroId = queryText(req, "heroId");
      const where: Prisma.SkinWhereInput = {
        ...(rarity !== undefined ? { rarity: raritySchema.parse(rarity) } : {}),
        ...(heroId !== undefined
          ? { heroId: queryInteger(req, "heroId") }
          : {}),
      };
      const [data, total] = await Promise.all([
        db.skin.findMany({
          where,
          skip,
          take: limit,
          orderBy: { id: "asc" },
          include: { hero: { select: { name: true } } },
        }),
        db.skin.count({ where }),
      ]);
      res.json({
        status: true,
        message: "Berhasil mengambil data skin",
        data,
        metadata: metadata(page, limit, total),
      });
    },
    async skinDetail(req: Request, res: Response) {
      const data = await db.skin.findUnique({
        where: { id: pathId(req, "skin_id") },
        include: { hero: true },
      });
      if (!data) throw new HttpError(404, "Skin tidak ditemukan");
      res.json({
        status: true,
        message: "Berhasil mengambil detail skin",
        data,
      });
    },
  };
}
```

### 4. Simpan, tunggu server, lalu typecheck

Setelah **Ctrl+S**, tunggu server di terminal `dev:neon` selesai memuat ulang dan siap kembali. Buka **terminal kedua** di folder proyek, lalu jalankan `npm.cmd run typecheck`. Server pada terminal pertama tetap berjalan.

**Hasil:** tidak ada error TypeScript. Field lane dan validasinya sudah tersedia di proyek; tidak perlu migrasi.

### 5. Buktikan hasil — Postman

Semua request berikut memakai method **GET** pada port **3001**.

| URL | Hasil yang ditunjukkan |
| --- | --- |
| `http://localhost:3001/api/heroes?lane=GOLD&page=1&limit=5` | 5 data, semuanya GOLD; totalItems 19, totalPages 4 |
| `http://localhost:3001/api/heroes?lane=GOLD&role=MARKSMAN&search=la&page=1&limit=1` | Layla; currentPage 1, totalItems 2, totalPages 2 |
| `http://localhost:3001/api/heroes?lane=GOLD&role=MARKSMAN&search=la&page=2&limit=1` | Claude; currentPage 2, totalItems tetap 2, totalPages 2 |
| `http://localhost:3001/api/heroes?lane=INVALID` | 400; status false dan message validasi |

**Ucapkan:** “Hasilnya sudah sesuai lane. Saat digabung dengan role, search, dan pagination, halaman pertama menampilkan Layla dan halaman kedua Claude. TotalItems tetap 2 meskipun tiap halaman hanya berisi 1 hero.”

**Jika perlu menjelaskan perubahan:** ada tiga tambahan, yaitu import `laneSchema`, membaca query `lane`, dan memasukkan lane ke objek `where`. Nilai valid: EXP, GOLD, MID, JUNGLE, ROAM.

**Penutup:** tunjukkan hasil pengujian dan pembagian tugas anggota.
