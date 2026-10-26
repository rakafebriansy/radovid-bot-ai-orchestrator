# Knowledge Base & Implementation Q&A Repository (LEARN.md)

> **[PRINSIP UTAMA]** Repositori terpusat untuk mendokumentasikan tanya-jawab teknis, keputusan arsitektur, dan logika basis kode yang ditanyakan pengguna secara eksplisit. Berbeda dari `RETROSPECTIVE.md` yang mencatat anomali galat, dokumen ini khusus merekam pemahaman konseptual implementasi.

---

## 1. Protokol Operasional & Syarat Aktivasi

> **[FATAL - ATURAN ANTI-AGRESIVITAS PENCATATAN]**
> Berkas `LEARN.md` HANYA BOLEH DIBACA DAN DIISI jika pengguna secara eksplisit menggunakan format prompt **FASE 4: Implementation Q&A Prompt** pada `README.md`.
> AI Agent DILARANG KERAS menyentuh, membaca, atau memodifikasi berkas ini pada instruksi umum, penugasan tiket/koding (Fase 1, 2, 3), perbaikan bug, maupun dialog kasual!

### 1.1 Syarat Aktivasi Mutlak (Template FASE 4 Only)
1. **[ATURAN]** `LEARN.md` tidak berlaku untuk penugasan tiket koding, inisialisasi arsitektur, perbaikan bug, atau percakapan umum.
2. **[WAJIB]** Pembacaan dan pencatatan entri hanya aktif jika prompt pengguna secara nyata menggunakan struktur **FASE 4: Implementation Q&A Prompt** dari `README.md` (memiliki blok Cakupan/Node, pertanyaan bernomor eksplisit, dan klausul instruksi baku).

### 1.2 Penanganan Pertanyaan Kasual (Single-Output Chat)
Jika pengguna mengajukan pertanyaan teknis atau arsitektur di luar template resmi FASE 4:
1. **[WAJIB]** Berikan jawaban lengkap secara eksklusif pada jendela percakapan terminal (*chat sidebar*).
2. **[TERLARANG]** Membuka, membuat entri baru, atau memodifikasi `LEARN.md`.
3. **[TERLARANG]** Berinisiatif mandiri mendokumentasikan dialog non-FASE 4 ke dalam repositori ini.

### 1.3 Pembatasan Lingkup & Validasi Berakar Kode (Full Grounding)
1. **Hanya Pertanyaan Eksplisit:** AI Agent DILARANG membuat rangkuman atau analisis otomatis di luar daftar pertanyaan yang diajukan pengguna.
2. **Berakar pada Basis Kode:** Seluruh jawaban wajib merujuk pada kode sumber nyata, konfigurasi, dan dokumen orchestrator (`global-docs/`, `global-guidelines/`, `nodes/[nama-node]/`).
3. **Bukti & Rasional:** Sertakan jalur berkas valid beserta nomor baris spesifik. Jelaskan alasan (*why*) di balik keputusan desain, bukan sekadar mendeskripsikan ulang kode (*what*).

### 1.4 Mekanisme Output Ganda Bersyarat (Dual-Output)
1. Saat prompt FASE 4 terpicu, AI Agent wajib menyajikan jawaban terstruktur langsung pada *chat sidebar*.
2. Pada giliran eksekusi yang sama, AI Agent wajib merekam sesi Q&A tersebut ke Bagian 4 berkas ini menggunakan Format Baku dan memperbarui Indeks Kategori.
3. Menulis ke berkas ini tanpa pemicu resmi FASE 4 merupakan **PELANGGARAN FATAL**.

---

## 2. Taksonomi & Sistem Kategorisasi

### 2.1 Format ID Entri
Gunakan format penomoran terstandarisasi: `[QA-YYYYMMDD-XX]`
- `YYYYMMDD`: Tahun, Bulan, Tanggal pencatatan (contoh: `20260913`).
- `XX`: Nomor urut pencatatan pada hari terkait (contoh: `01`, `02`).

### 2.2 Sepuluh Kategori Utama
Pilih satu kategori yang paling dominan mewakili topik pembahasan:

| Kategori Utama | Cakupan / Domain |
|---|---|
| **`Architecture & Pattern`** | Struktur folder, modularitas, alur sistem, pola desain, pemisahan tanggung jawab. |
| **`State Management & Data Flow`** | Alur data, global store (Zustand/Redux), server state (React Query), aliran props/event. |
| **`Database & Data Modeling`** | Skema tabel/koleksi, ORM (Prisma/TypeORM/Mongoose), migrasi, indexing, optimasi kueri. |
| **`API & Network Integration`** | REST endpoints, GraphQL, WebSocket, gRPC, format request/response, penanganan galat jaringan. |
| **`UI/UX & Design System`** | Komponen visual, Tailwind/CSS, konsistensi tema, responsivitas, aksesibilitas (a11y). |
| **`Security & Authentication`** | JWT, sesi, OAuth, RBAC/Permissions, hashing, sanitasi input, proteksi CORS/CSRF. |
| **`Testing & Quality Assurance`** | Strategi unit test, integration test, E2E, mock data, cakupan kode, asersi logika. |
| **`Build, Tooling & DevOps`** | Konfigurasi bundler (Vite/Webpack), Docker, CI/CD, script npm, environment variables. |
| **`Business Logic & Domain Rules`** | Logika perhitungan, validasi transaksi, aturan proses bisnis spesifik aplikasi. |
| **`Orchestrator & Workflow`** | Aturan template, manajemen tiket, SOP guidelines, mekanisme orkestrasi multi-node. |

### 2.3 Konvensi Penandaan (Tagging)
Gunakan format `#kebab-case` untuk penanda spesifik. Contoh:
- `#jwt-auth` `#zustand` `#prisma-relations` `#optimistic-update` `#tailwind-v4` `#rbac-middleware`

---

## 3. Format Entri Baku (Boilerplate Template)

Salin struktur Markdown berikut saat mencatat entri tanya-jawab baru:

```markdown
### [QA-YYYYMMDD-XX] <Judul Representatif Terkait Topik Bahasan>
- **Tanggal**: YYYY-MM-DD HH:mm
- **Scope / Target Node**: `[Global / Nama Node / Path Codebase]`
- **Kategori**: `[Pilih salah satu dari 10 Kategori Utama]`
- **Tags**: `#tag1 #tag2 #tag3`
- **File Referensi**:
  - `path/to/relevant-file-1.ext` (L10-L45)
  - `path/to/relevant-file-2.ext`

#### Pertanyaan Pengguna
1. **[Tulis ulang pertanyaan 1 secara presisi]**
2. **[Tulis ulang pertanyaan 2 secara presisi]**

#### Jawaban & Penjelasan Implementasi

##### 1. [Judul Poin Jawaban 1]
- **Ringkasan Inti**: [Penjelasan singkat 1-2 kalimat]
- **Detail Implementasi & Logika**:
  [Penjelasan komprehensif alur kerja kode]
- **Rujukan Kode Sumber**:
  ```[language]
  // Cuplikan kode atau referensi fungsi/kelas yang relevan
  ```
- **Rasional & Keputusan Teknis**:
  [Alasan pemilihan pendekatan, pertimbangan trade-off, kesesuaian pedoman]

##### 2. [Judul Poin Jawaban 2]
- **Ringkasan Inti**: [Penjelasan singkat 1-2 kalimat]
- **Detail Implementasi & Logika**:
  [Penjelasan komprehensif alur kerja kode]
- **Rujukan Kode Sumber**:
  ```[language]
  // Cuplikan kode atau referensi fungsi/kelas yang relevan
  ```
- **Rasional & Keputusan Teknis**:
  [Alasan pemilihan pendekatan, pertimbangan trade-off, kesesuaian pedoman]

---
```

---

## 4. Indeks Kategori & Arsip Log Tanya-Jawab

### 4.1 Indeks Kategori & Daftar Isi
Perbarui tautan indeks berikut setiap kali menambahkan entri baru (urutkan descending / terbaru di atas):

- **Architecture & Pattern**
  - *(Belum ada entri)*
- **State Management & Data Flow**
  - *(Belum ada entri)*
- **Database & Data Modeling**
  - *(Belum ada entri)*
- **API & Network Integration**
  - *(Belum ada entri)*
- **UI/UX & Design System**
  - *(Belum ada entri)*
- **Security & Authentication**
  - *(Belum ada entri)*
- **Testing & Quality Assurance**
  - *(Belum ada entri)*
- **Build, Tooling & DevOps**
  - *(Belum ada entri)*
- **Business Logic & Domain Rules**
  - *(Belum ada entri)*
- **Orchestrator & Workflow**
  - *(Belum ada entri)*

### 4.2 Arsip Entri Tanya-Jawab
*(Entri tanya-jawab baru yang disetujui dari pemicu FASE 4 disisipkan di bawah baris ini secara berurutan)*
