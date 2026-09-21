# Knowledge Base & Implementation Q&A Repository (`LEARN.md`)

Dokumen ini adalah repositori terpusat (*Single Source of Truth*) untuk menyimpan seluruh tanya-jawab (*Question & Answer*) seputar implementasi teknis, keputusan arsitektural, dan logika *codebase* yang ditanyakan oleh pengguna (*User*).

Berbeda dengan log retrospektif error (`RETROSPECTIVE.md`), dokumen ini khusus mendokumentasikan pemahaman konseptual dan teknis yang diperoleh dari proses tanya-jawab eksplisit antara pengguna dan AI Agent.

---

## 🛑 Protokol Operasional AI Agent (Wajib Dipatuhi)

> [!CAUTION]
> **ATURAN UTAMA ANTI-AGRESIVITAS PENCATATAN:**
> File `LEARN.md` **HANYA BOLEH DIBACA DAN DIISI** jika dan hanya jika pengguna secara eksplisit menggunakan format prompt **FASE 4: Implementation Q&A Prompt** yang ada di `README.md`.
> AI Agent **DILARANG KERAS** menyentuh, mencatat, atau memodifikasi file ini pada instruksi biasa, eksekusi tugas/koding (Fase 1, 2, 3), perbaikan bug, ataupun pertanyaan kasual!

Setiap kali AI Agent menerima instruksi tanya-jawab implementasi (*Implementation Q&A Prompt*), agen **WAJIB MUTLAK** mematuhi 5 hukum operasional berikut:

### 1. Syarat Pemicu Mutlak (Strict Activation Trigger — Template README.md Only)
- File `LEARN.md` **TIDAK BERLAKU** dan **DILARANG DIBUKA/DIISI** untuk instruksi biasa (*regular prompt*), tugas koding/tiket (Fase 3), inisialisasi/scaling (Fase 1 & 2), perbaikan bug, atau percakapan kasual.
- AI Agent **HANYA** diizinkan membuka, membaca, dan mencatatkan entri ke dalam file ini jika prompt pengguna secara nyata menggunakan struktur format **FASE 4: Implementation Q&A Prompt** dari `README.md` (memiliki blok *Cakupan / Node*, daftar pertanyaan eksplisit bernomor, dan klausul instruksi baku).
- **Penanganan Pertanyaan Kasual / Bebas:** Jika pengguna mengajukan pertanyaan teknis, alur kode, atau arsitektur di luar template FASE 4 `README.md`:
  - AI Agent **HANYA BOLEH** menjawab secara langsung di jendela percakapan (*chat sidebar*).
  - AI Agent **DILARANG KERAS** membuat entri atau memodifikasi `LEARN.md`.
  - AI Agent **DILARANG** berinisiatif mandiri untuk mendokumentasikan percakapan tersebut ke dalam file ini.

### 2. Hanya Menjawab Pertanyaan Eksplisit (Strictly Explicit Q&A Only)
- AI Agent **DILARANG KERAS** membuat penjelasan, rangkuman, atau inisiatif analisis secara otomatis tanpa pertanyaan eksplisit yang diajukan oleh pengguna.
- Hanya jelaskan poin-poin pertanyaan yang secara nyata tertulis pada daftar pertanyaan pengguna (contoh: *1. Pertanyaan A, 2. Pertanyaan B*).

### 3. Berakar pada Codebase & Orchestrator (No Hallucinations / Full Grounding)
- Seluruh jawaban **WAJIB** berakar pada analisis kode sumber (*source code*) nyata, konfigurasi, dan dokumen orchestrator (`global-docs/`, `global-guidelines/`, `nodes/[nama-node]/`).
- Sertakan path file yang valid (dan nomor baris jika relevan) sebagai bukti konkret implementasi.
- Jelaskan **alasan/rasional (*why*)** di balik keputusan arsitektur, bukan hanya sekadar mendeskripsikan ulang baris kode (*what*).

### 4. Output Ganda Bersyarat (Conditional Dual-Output: Chat Sidebar + LEARN.md)
- Mekanisme *Dual-Output* ini **HANYA DIAKTIFKAN** saat pengguna menggunakan template prompt FASE 4 dari `README.md`.
- AI Agent **WAJIB** mengetikkan jawaban lengkap, terstruktur, dan mudah dipahami langsung pada **chat sidebar** percakapan.
- Pada saat yang bersamaan, AI Agent **WAJIB** menyalin dan merekam sesi tanya-jawab tersebut ke bagian [Arsip Log Tanya-Jawab](#-arsip-log-tanya-jawab) di dalam file ini menggunakan [Format Entri Baku](#-format-entri-baku-boilerplate) serta memperbarui [Indeks Kategori](#-indeks-kategori--daftar-isi).
- Jika pengguna TIDAK menggunakan template resmi FASE 4, output **WAJIB TUNGGAL** (hanya di *chat sidebar*) dan modifikasi terhadap file ini adalah **PELANGGARAN FATAL**.

### 5. Disiplin Taksonomi & Kategorisasi
- AI Agent **WAJIB** mengklasifikasikan setiap entri Q&A ke dalam **Kategori Utama** dan menyematkan **Tags** yang relevan agar memudahkan pengguna dalam membaca dan mencari di kemudian hari.

---

## 🏷️ Taksonomi & Sistem Kategorisasi

Untuk menjaga keteraturan dan kemudahan pencarian (*searchability*), gunakan standar taksonomi berikut saat mencatat entri baru:

### 1. Format ID Entri
Gunakan format penomoran: `[QA-YYYYMMDD-XX]`
- `YYYYMMDD`: Tahun, Bulan, Tanggal pencatatan (contoh: `20260913`).
- `XX`: Nomor urut pada hari tersebut (contoh: `01`, `02`).

### 2. Daftar Kategori Utama (Pilih Salah Satu)
| Kategori Utama | Cakupan / Domain |
| :--- | :--- |
| **`Architecture & Pattern`** | Struktur folder, modularitas, alur sistem, design patterns, separation of concerns. |
| **`State Management & Data Flow`** | Alur data, global store (Zustand/Redux), server state (React Query), props/event flow. |
| **`Database & Data Modeling`** | Skema tabel/koleksi, ORM (Prisma/TypeORM/Mongoose), migrasi, indexing, query optimization. |
| **`API & Network Integration`** | REST endpoints, GraphQL, WebSocket, gRPC, format request/response, error handling network. |
| **`UI/UX & Design System`** | Komponen visual, Tailwind/CSS, konsistensi tema, responsivitas, aksesibilitas (a11y). |
| **`Security & Authentication`** | JWT, sesi, OAuth, RBAC/Permissions, hashing, sanitasi input, proteksi CORS/CSRF. |
| **`Testing & Quality Assurance`** | Strategi unit test, integration test, E2E, mock data, coverage, assertions. |
| **`Build, Tooling & DevOps`** | Konfigurasi bundler (Vite/Webpack), Docker, CI/CD, script npm, environment variables. |
| **`Business Logic & Domain Rules`** | Logika perhitungan, validasi transaksi, aturan proses bisnis spesifik aplikasi. |
| **`Orchestrator & Workflow`** | Aturan template, manajemen tiket, SOP guidelines, mekanisme multi-node. |

### 3. Konvensi Tagging
Gunakan format `#kebab-case` untuk tag spesifik. Contoh:
- `#jwt-auth` `#zustand` `#prisma-relations` `#optimistic-update` `#tailwind-v4` `#rbac-middleware`

---

## 📋 Format Entri Baku (Boilerplate)

AI Agent **WAJIB** menyalin struktur *markdown* berikut saat menambahkan rekaman tanya-jawab baru ke dalam file ini:

```markdown
### [QA-YYYYMMDD-XX] <Judul Singkat Representatif Terkait Topik Pertanyaan>
- **Tanggal**: YYYY-MM-DD HH:mm
- **Scope / Target Node**: `[Global / Nama Node / Path Codebase]`
- **Kategori**: `[Pilih salah satu dari Kategori Utama di atas]`
- **Tags**: `#tag1 #tag2 #tag3`
- **File Referensi**:
  - `path/to/relevant-file-1.ext` (L10-L45)
  - `path/to/relevant-file-2.ext`

#### ❓ Pertanyaan Pengguna
1. **[Tulis ulang pertanyaan 1 secara presisi]**
2. **[Tulis ulang pertanyaan 2 secara presisi]**

#### 💡 Jawaban & Penjelasan Implementasi

##### 1. [Judul Poin Jawaban 1]
- **Ringkasan Inti**: [Penjelasan singkat 1-2 kalimat]
- **Detail Implementasi & Logika**:
  [Penjelasan komprehensif alur kerja kode]
- **Rujukan Kode Sumber**:
  ```[language]
  // Cuplikan kode atau referensi fungsi/kelas yang relevan
  ```
- **Rasional & Keputusan Teknis**:
  [Mengapa pendekatan ini yang dipilih, pertimbangan trade-off, atau kesesuaian dengan pedoman]

##### 2. [Judul Poin Jawaban 2]
- **Ringkasan Inti**: [Penjelasan singkat 1-2 kalimat]
- **Detail Implementasi & Logika**:
  [Penjelasan komprehensif alur kerja kode]
- **Rujukan Kode Sumber**:
  ```[language]
  // Cuplikan kode atau referensi fungsi/kelas yang relevan
  ```
- **Rasional & Keputusan Teknis**:
  [Mengapa pendekatan ini yang dipilih, pertimbangan trade-off, atau kesesuaian dengan pedoman]

---
```

---

## 🗂️ Indeks Kategori & Daftar Isi

*AI Agent WAJIB memperbarui tautan indeks di bawah ini setiap kali menambahkan entri baru (urutkan dari yang terbaru / descending).*

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

---

## 📚 Arsip Log Tanya-Jawab

*Entri tanya-jawab baru akan disisipkan di bawah baris ini secara berurutan.*
