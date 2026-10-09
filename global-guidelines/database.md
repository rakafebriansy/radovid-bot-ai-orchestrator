# Database & Datetime Storage Standard

Pedoman ini mengatur standar arsitektur penyimpanan data di basis data (*database*), khususnya perlakuan terhadap data tanggal dan waktu (*datetime*). Penerapan standar ini bertujuan mencegah ketidakpastian zona waktu (*timezone ambiguity*), masalah *Daylight Saving Time* (DST), inkonsistensi data multi-kawasan (*multi-region*), dan kesalahan kalkulasi waktu lintas-layanan.

---

## Aturan Emas Penyimpanan Waktu (The Golden Rule)

1. **Database Selalu Netral Waktu (*Time-Neutral Storage*):** Basis data **DILARANG KERAS** menyimpan waktu lokal tanpa informasi zona waktu atau mengandalkan zona waktu lokal peladen (*server local timezone*) yang ambigu.
2. **Penyimpanan Selalu Berbasis UTC atau Epoch:** Data waktu yang masuk ke database harus selalu dinormalisasi ke format **UTC** atau **Epoch Time**.
3. **Konversi Waktu adalah Tanggung Jawab Lapisan Presentasi:** Konversi waktu dari UTC/Epoch ke zona waktu lokal pengguna (*user's local timezone*) dilakukan pada sisi *frontend*, *mobile client*, atau lapisan penyajian (*presentation/serialization layer*), **bukan** disimpan secara mentah di database.

---

## Pilihan Opsi Penyimpanan Datetime

Saat merancang arsitektur basis data, Anda **WAJIB** memilih salah satu dari dua opsi standar berikut secara konsisten:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                    OPSI PENYIMPANAN DATETIME                            │
├────────────────────────────────────┬────────────────────────────────────┤
│  OPSI 1: Epoch Time Milliseconds   │  OPSI 2: Timestamp with TZ (UTC)   │
│  - Format: Integer (64-bit)        │  - Format: ISO-8601 / TIMESTAMPTZ  │
│  - Nilai: Milidetik sejak Unix     │  - Nilai: Waktu terkonversi ke UTC │
│  - Fokus: Kecepatan & Komputasi    │  - Fokus: Agregasi & Readability   │
└────────────────────────────────────┴────────────────────────────────────┘
```

---

### OPSI 1: Epoch Time Milliseconds (Unix Millis / BigInt)

Opsi ini menyimpan waktu sebagai bilangan bulat (*integer*) yang merepresentasikan jumlah milidetik (*milliseconds*) yang telah berlalu sejak Unix Epoch (1 Januari 1970 pukul 00:00:00 UTC).

#### 1. Karakteristik & Tipe Data
*   **Tipe Kolom Database:** `BIGINT` (PostgreSQL, MySQL, MariaDB, SQLite), `INTEGER (64-bit)`, `NUMBER` (Oracle), atau `Int64` (ClickHouse).
*   **Format Data:** Bilangan bulat non-negatif (contoh: `1788777885000`).
*   **Unit Waktu:** Milidetik (*milliseconds*). Hindari menggunakan detik (*seconds*) kecuali jika berinteraksi dengan protokol warisan (*legacy*) tertentu untuk menjamin presisi tinggi.

#### 2. Alur Eksekusi (Workflow)
*   **Proses Simpan (Insert / Update):**
    1. Ambil waktu saat ini (*current time*).
    2. Konversi waktu tersebut ke nilai milidetik Epoch UTC.
    3. Simpan nilai integer ke kolom basis data (contoh: kolom `created_at` bertipe `BIGINT`).
*   **Proses Baca (Query / Display):**
    1. Ambil nilai integer milidetik dari database.
    2. Teruskan nilai integer ke klien atau format ke format tanggal dengan zona waktu lokal pengguna sebelum disajikan pada antarmuka.

#### 3. Skenario Penggunaan yang Disarankan
*   Sistem analitik data besar, *telemetry*, dan *event streaming* berkecepatan tinggi.
*   Pencatatan metrik performa (*performance metrics*) dan *audit logs* berskala besar.
*   Aplikasi *gaming*, *real-time state synchronization*, dan perangkat IoT.
*   Arsitektur di mana kueri perbandingan numerik murni (`WHERE created_at > 1788777885000`) lebih dominan daripada kueri agregasi tanggal berbasis kalender.

#### 4. Kelebihan & Batasan
*   **Kelebihan:** Ukuran penyimpanan ringkas, pengindeksan (*indexing*) sangat cepat, bebas dari ambiguitas parsing string antar bahasa pemrograman atau pustaka database.
*   **Batasan:** Nilai sulit dibaca langsung oleh manusia saat melakukan kueri manual (*raw SQL inspection*) tanpa fungsi pembantu, dan membutuhkan fungsi konversi tambahan saat melakukan kueri agregasi kalender (seperti *group by day/month*).

---

### OPSI 2: Timestamp with Timezone UTC (TIMESTAMPTZ / ISO-8601 UTC)

Opsi ini mengonversi waktu saat ini dari zona waktu pengguna atau sistem ke zona waktu **UTC (+00:00 / 'Z')** terlebih dahulu, baru kemudian disimpan ke dalam basis data sebagai tipe data *timestamp* berbasis zona waktu.

#### 1. Karakteristik & Tipe Data
*   **Tipe Kolom Database:** `TIMESTAMPTZ` / `TIMESTAMP WITH TIME ZONE` (PostgreSQL), `DATETIME` / `TIMESTAMP` dengan normalisasi UTC eksplisit (MySQL, MariaDB, SQLite), atau tipe `Date` / String ISO-8601 (MongoDB, DynamoDB, Firestore).
*   **Format Data:** Format ISO-8601 UTC standar (contoh: `2026-09-07T10:44:45.000Z` atau `2026-09-07 10:44:45+00`).

#### 2. Alur Eksekusi (Workflow)
*   **Proses Simpan (Insert / Update):**
    1. Tangkap waktu saat ini dari masukan pengguna atau server.
    2. Konversi waktu tersebut secara eksplisit ke zona waktu UTC (offset `+00:00`).
    3. Masukkan nilai waktu UTC tersebut ke kolom *timestamp* di database.
*   **Proses Baca (Query / Display):**
    1. Ambil nilai *timestamp* UTC dari database.
    2. Konversi *timestamp* UTC tersebut ke zona waktu lokal pengguna (*user's timezone*, misal: `Asia/Jakarta`, `America/New_York`) saat data dirender di antarmuka atau dikirimkan melalui respons API.

#### 3. Skenario Penggunaan yang Disarankan
*   Aplikasi bisnis umum: ERP, CRM, *E-Commerce*, dan *Billing System*.
*   Sistem perbankan, akuntansi, dan transaksi keuangan yang membutuhkan pelaporan berbasis kalender dan jam operasional.
*   Aplikasi yang sering memanfaatkan fungsi agregasi waktu bawaan SQL (seperti `DATE_TRUNC('month', created_at)`, `GROUP BY EXTRACT(YEAR FROM created_at)`).
*   Sistem yang membutuhkan kemudahan inspeksi data langsung oleh tim teknis melalui *database GUI tool*.

#### 4. Kelebihan & Batasan
*   **Kelebihan:** Sangat mudah dibaca saat kueri SQL manual (*human-readable*), didukung penuh oleh fungsi tanggal bawaan SQL engine, dan merupakan standar internasional ISO.
*   **Batasan:** Bergantung pada ketepatan konfigurasi zona waktu koneksi database dan *driver* ORM agar tidak terjadi konversi implisit yang keliru.

---

## Matriks Perbandingan

| Parameter | Opsi 1: Epoch Time Millis | Opsi 2: Timestamp with Timezone UTC |
| :--- | :--- | :--- |
| **Tipe Data Database** | `BIGINT` / `INTEGER (64-bit)` | `TIMESTAMPTZ` / `TIMESTAMP WITH TIME ZONE` |
| **Format Nilai Mentah** | `1788777885000` | `2026-09-07T10:44:45.000Z` |
| **Zona Waktu Dasar** | Selalu UTC (Epoch 1970) | Selalu UTC (Offset +00:00) |
| **Efisiensi Indexing** | Sangat Tinggi (Perbandingan Numerik) | Tinggi (Operasi Tipe Data Waktu) |
| **Keterbacaan di SQL Tool** | Rendah (Butuh fungsi konversi) | Tinggi (*Human-Readable* langsung) |
| **Kueri Agregasi Kalender** | Memerlukan konversi epoch ke date | Alami menggunakan fungsi bawaan SQL |
| **Kompatibilitas Multi-DB** | Universal (Semua DB mendukung BigInt) | Perlu penyesuaian sintaks tipe data antar DB |
| **Kasus Penggunaan Utama** | IoT, Gaming, Analytics, Log Stream | Web Apps, ERP, E-Commerce, Financial Apps |

---

## Aturan Penerapan Proyek (Enforcement Rules)

1. **Konsistensi Tingkat Proyek (*Project-Wide Uniformity*):**
   * Pilihan antara **Opsi 1** atau **Opsi 2** wajib ditentukan pada fase perancangan sistem dan dicatat secara eksplisit pada dokumen `nodes/[nama-node]/docs/system-design.md` di sub-bab **Skema Data (Data Modeling)**.
   * **DILARANG KERAS** mencampuradukkan kedua opsi di dalam satu modul atau entitas data yang saling berelasi.
2. **Konvensi Penamaan Kolom Standar:**
   * Gunakan penamaan bahasa Inggris berformat *snake_case* secara konsisten:
     * `created_at` : Waktu penciptaan rekaman data.
     * `updated_at` : Waktu pembaruan terakhir rekaman data.
     * `deleted_at` : Waktu penghapusan lunak (*soft delete*).
     * `expires_at` : Waktu kedaluwarsa sesi atau token.
     * `published_at`: Waktu rilis konten.
3. **Pemberian Nilai Bawaan (Default Values):**
   * Jika menggunakan **Opsi 1 (Epoch Millis)**: Berikan nilai waktu milidetik melalui aplikasi atau fungsi basis data yang menghitung milidetik saat ini (misal: `(EXTRACT(EPOCH FROM NOW()) * 1000)::BIGINT`).
   * Jika menggunakan **Opsi 2 (Timestamp UTC)**: Gunakan fungsi bawaan UTC yang valid (misal: `CURRENT_TIMESTAMP AT TIME ZONE 'UTC'`, `NOW() AT TIME ZONE 'utc'`).

---

## Larangan Eksekusi DDL Langsung di Database (Migration-First Policy)

> 🔴 **ZERO DIRECT DDL EXECUTION:**
> AI Agent **DILARANG KERAS** mengeksekusi perintah Data Definition Language (DDL) seperti `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `TRUNCATE`, `RENAME COLUMN`, atau `ADD CONSTRAINT` secara langsung pada basis data aktif (*live/running database*) melalui terminal interaktif, skrip ad-hoc, atau kueri mentah mandiri.

### 1. Wajib Melalui Berkas Migrasi Terstruktur (Migration-First Policy)
1. **Version-Controlled Migrations:** Seluruh perubahan skema, struktur tabel, penambahan/penghapusan kolom, relasi *foreign key*, dan indeks **WAJIB** didefinisikan ke dalam berkas skrip migrasi resmi (*migration files*) yang tersimpan di dalam repositori kode (contoh: Prisma migrations, TypeORM/Knex migrations, Alembic, Flyway, Golang-migrate, atau file SQL migrasi berpenomoran urut di direktori proyek).
2. **Keterlacakan & Reversibilitas:** Perubahan skema harus dapat di-review (*diff inspection*), memiliki prosedur *rollback* (*down migration*), dan dapat direproduksi secara deterministik di lingkungan CI/CD, staging, maupun produksi.
3. **Dilarang Ad-hoc Console Mutations:** Mengubah tipe data, menambahkan kolom, atau mengubah tabel secara diam-diam melalui *database CLI/GUI tool* tanpa berkas migrasi adalah **PELANGGARAN FATAL**.

### 2. Protokol Izin Mutasi Database (State Mutation Gate)
- Eksekusi migrasi DDL, *seed*, dan manipulasi DML massal tergolong **Operasi Modifikasi State**.
- AI **DILARANG LANGSUNG MEMINTA PERMISSION** untuk mengeksekusi perintah yang mengubah state database ini tanpa terlebih dahulu menyajikan rencana implementasi (*Implementation Plan*) / preview perubahan dan mendapatkan persetujuan rencana dari pengguna.
- Kueri yang bersifat **Read-Only** (seperti `SELECT` non-locking, `EXPLAIN`, inspeksi skema baca) diperbolehkan langsung meminta permission/dieksekusi untuk pengumpulan konteks.
