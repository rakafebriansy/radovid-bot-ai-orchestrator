# Database & Datetime Storage Standard

Pedoman ini mengatur standar arsitektur penyimpanan data di basis data, khususnya data tanggal dan waktu (datetime), guna mencegah ambiguitas zona waktu, isu Daylight Saving Time (DST), inkonsistensi multi-region, dan kesalahan kalkulasi waktu lintas-layanan.

---

## 1. Standar Penyimpanan Tanggal & Waktu (Datetime Storage Architecture)

### 1.1 Aturan Emas Penyimpanan Waktu (The Golden Rule)
1. **Time-Neutral Storage:** Database dilarang menyimpan waktu lokal tanpa informasi zona waktu atau mengandalkan zona waktu lokal server yang ambigu.
2. **Normalisasi UTC atau Epoch:** Seluruh data waktu yang disimpan ke database wajib dinormalisasi ke format UTC atau Epoch Time.
3. **Konversi di Lapisan Presentasi:** Konversi waktu dari UTC/Epoch ke zona waktu lokal pengguna dilakukan di sisi frontend, mobile client, atau presentation/serialization layer, bukan disimpan mentah di database.

### 1.2 Pilihan Format Penyimpanan Datetime
Saat merancang arsitektur basis data, wajib memilih salah satu dari dua opsi standar berikut secara konsisten:

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

#### 1.2.1 Opsi 1: Epoch Time Milliseconds (Unix Millis / BigInt)
Menyimpan waktu sebagai bilangan bulat integer yang merepresentasikan jumlah milidetik sejak Unix Epoch (1 Januari 1970 00:00:00 UTC).

- **Karakteristik & Tipe Data:**
  - **Tipe Kolom Database:** `BIGINT` (PostgreSQL, MySQL, MariaDB, SQLite), `INTEGER (64-bit)`, `NUMBER` (Oracle), atau `Int64` (ClickHouse).
  - **Format Data:** Integer non-negatif (contoh: `1788777885000`).
  - **Unit Waktu:** Milidetik (milliseconds). Hindari menggunakan detik kecuali saat berinteraksi dengan protokol legacy tertentu.
- **Alur Eksekusi (Workflow):**
  - **Proses Simpan (Insert / Update):** Ambil waktu saat ini -> konversi ke milidetik Epoch UTC -> simpan nilai integer ke kolom `BIGINT` (misal: kolom `created_at`).
  - **Proses Baca (Query / Display):** Ambil nilai integer dari database -> teruskan ke client atau format ke zona waktu lokal pengguna pada antarmuka.
- **Skenario Penggunaan yang Disarankan:**
  - Sistem analitik data besar, telemetry, dan event streaming berkecepatan tinggi.
  - Audit logs dan performance metrics berskala besar.
  - Gaming, sinkronisasi state real-time, dan perangkat IoT.
  - Arsitektur dengan dominasi kueri perbandingan numerik (`WHERE created_at > 1788777885000`).
- **Kelebihan & Batasan:**
  - **Kelebihan:** Ukuran penyimpanan ringkas, pengindeksan (indexing) sangat cepat, bebas ambiguitas parsing string antar bahasa/driver.
  - **Batasan:** Sulit dibaca langsung oleh manusia pada kueri manual tanpa fungsi pembantu, dan membutuhkan konversi tambahan saat kueri agregasi kalender (group by day/month).

#### 1.2.2 Opsi 2: Timestamp with Timezone UTC (TIMESTAMPTZ / ISO-8601 UTC)
Mengonversi waktu dari masukan pengguna/server ke zona waktu UTC (+00:00 / 'Z') sebelum disimpan ke kolom timestamp berbasis zona waktu.

- **Karakteristik & Tipe Data:**
  - **Tipe Kolom Database:** `TIMESTAMPTZ` / `TIMESTAMP WITH TIME ZONE` (PostgreSQL), `DATETIME` / `TIMESTAMP` dengan normalisasi UTC eksplisit (MySQL, MariaDB, SQLite), atau tipe `Date` / String ISO-8601 (MongoDB, DynamoDB, Firestore).
  - **Format Data:** ISO-8601 UTC standar (contoh: `2026-09-07T10:44:45.000Z` atau `2026-09-07 10:44:45+00`).
- **Alur Eksekusi (Workflow):**
  - **Proses Simpan (Insert / Update):** Tangkap waktu saat ini -> konversi eksplisit ke zona waktu UTC (offset `+00:00`) -> simpan ke kolom timestamp database.
  - **Proses Baca (Query / Display):** Ambil timestamp UTC dari database -> konversi ke zona waktu lokal pengguna (misal: `Asia/Jakarta`, `America/New_York`) saat render UI atau serialisasi respons API.
- **Skenario Penggunaan yang Disarankan:**
  - Aplikasi bisnis umum: ERP, CRM, E-Commerce, dan Billing System.
  - Sistem perbankan, akuntansi, dan transaksi keuangan berbasis kalender/jam operasional.
  - Aplikasi yang sering memanfaatkan fungsi agregasi SQL bawaan (`DATE_TRUNC('month', created_at)`, `GROUP BY EXTRACT(YEAR FROM created_at)`).
  - Sistem yang membutuhkan kemudahan inspeksi data langsung via GUI database tool.
- **Kelebihan & Batasan:**
  - **Kelebihan:** Sangat mudah dibaca manusia (human-readable) pada kueri SQL manual, didukung native oleh fungsi tanggal SQL, standar internasional ISO.
  - **Batasan:** Bergantung pada ketepatan konfigurasi zona waktu koneksi database dan driver ORM agar terhindar dari konversi implisit yang keliru.

### 1.3 Matriks Perbandingan Format Waktu

| Parameter | Opsi 1: Epoch Time Millis | Opsi 2: Timestamp with Timezone UTC |
| :--- | :--- | :--- |
| **Tipe Data Database** | `BIGINT` / `INTEGER (64-bit)` | `TIMESTAMPTZ` / `TIMESTAMP WITH TIME ZONE` |
| **Format Nilai Mentah** | `1788777885000` | `2026-09-07T10:44:45.000Z` |
| **Zona Waktu Dasar** | Selalu UTC (Epoch 1970) | Selalu UTC (Offset +00:00) |
| **Efisiensi Indexing** | Sangat Tinggi (Perbandingan Numerik) | Tinggi (Operasi Tipe Data Waktu) |
| **Keterbacaan di SQL Tool** | Rendah (Butuh fungsi konversi) | Tinggi (Human-Readable langsung) |
| **Kueri Agregasi Kalender** | Memerlukan konversi epoch ke date | Alami menggunakan fungsi bawaan SQL |
| **Kompatibilitas Multi-DB** | Universal (Semua DB mendukung BigInt) | Perlu penyesuaian sintaks tipe data antar DB |
| **Kasus Penggunaan Utama** | IoT, Gaming, Analytics, Log Stream | Web Apps, ERP, E-Commerce, Financial Apps |

### 1.4 Aturan Penerapan Proyek (Enforcement Rules)
1. **Konsistensi Tingkat Proyek:**
   - Pilihan Opsi 1 atau Opsi 2 wajib ditentukan pada fase perancangan sistem dan dicatat di `nodes/[nama-node]/docs/system-design.md` pada sub-bab **Skema Data (Data Modeling)**.
   - Dilarang mencampuradukkan kedua opsi di dalam satu modul atau entitas data yang saling berelasi.
2. **Konvensi Penamaan Kolom Standar:**
   - Gunakan penamaan bahasa Inggris berformat `snake_case` secara konsisten:
     - `created_at`: Waktu penciptaan rekaman data.
     - `updated_at`: Waktu pembaruan terakhir rekaman data.
     - `deleted_at`: Waktu penghapusan lunak (soft delete).
     - `expires_at`: Waktu kedaluwarsa sesi atau token.
     - `published_at`: Waktu rilis konten.
3. **Pemberian Nilai Bawaan (Default Values):**
   - **Opsi 1 (Epoch Millis):** Berikan nilai milidetik via aplikasi atau fungsi database: `(EXTRACT(EPOCH FROM NOW()) * 1000)::BIGINT`.
   - **Opsi 2 (Timestamp UTC):** Gunakan fungsi bawaan UTC valid: `CURRENT_TIMESTAMP AT TIME ZONE 'UTC'` atau `NOW() AT TIME ZONE 'utc'`.

---

## 2. Integritas Skema & Eksekusi DDL (Migration-First Policy)

### 2.1 Larangan Eksekusi DDL Langsung di Database

> **ZERO DIRECT DDL EXECUTION:**
> AI Agent DILARANG KERAS mengeksekusi perintah DDL (`CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `TRUNCATE`, `RENAME COLUMN`, `ADD CONSTRAINT`) langsung pada basis data aktif (live/running database) melalui terminal interaktif, skrip ad-hoc, atau kueri mentah mandiri.

#### 2.1.1 Wajib Melalui Berkas Migrasi Terstruktur
1. **Version-Controlled Migrations:** Seluruh perubahan skema, struktur tabel, penambahan/penghapusan kolom, relasi foreign key, dan indeks WAJIB didefinisikan ke dalam berkas skrip migrasi resmi di repositori (contoh: Prisma migrations, TypeORM/Knex migrations, Alembic, Flyway, Golang-migrate, atau file SQL migrasi berpenomoran urut di direktori proyek).
2. **Keterlacakan & Reversibilitas:** Perubahan skema wajib dapat ditinjau (diff inspection), memiliki skrip rollback (down migration), dan dapat direproduksi secara deterministik di lingkungan CI/CD, staging, maupun produksi.
3. **Dilarang Mutasi Ad-hoc Console:** Mengubah tipe data, menambahkan kolom, atau mengubah tabel secara diam-diam via GUI/CLI database tanpa berkas migrasi adalah pelanggaran fatal.

#### 2.1.2 Protokol Izin Mutasi Database (State Mutation Gate)
1. **Klasifikasi Modifikasi State:** Eksekusi migrasi DDL, seed data, dan manipulasi DML massal tergolong Operasi Modifikasi State.
2. **Kewajiban Implementation Plan:** Dilarang langsung meminta permission atau mengeksekusi mutasi state database tanpa terlebih dahulu menyajikan Implementation Plan dan preview perubahan serta mendapatkan persetujuan rencana dari pengguna sesuai [safe-file-operations.md](./safe-file-operations.md).
3. **Operasi Read-Only:** Kueri pasif non-locking (`SELECT`, `EXPLAIN`, inspeksi skema baca) diizinkan langsung meminta permission/dieksekusi untuk pengumpulan konteks awal.
