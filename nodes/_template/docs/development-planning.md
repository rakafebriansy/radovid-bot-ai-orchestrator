# Development Planning: Ticket & Plan Backlog Guide

> **[PRINSIP UTAMA]** Dokumen peta jalan strategis (*roadmap*) dan gudang antrean pasangan tiket & rencana (*ticket & plan backlog*) yang menjembatani dokumen makro (PRD dan System Design) dengan tugas mikro (tiket eksekusi dan rencana implementasi).

---

## 1. Aturan Sinkronisasi Pasangan Tiket & Rencana

### 1.1 Kontrol Pembuatan Berdasarkan Mode
1. **MODE 1 (Strict Backlog First):** AI Agent DILARANG KERAS membuat berkas pasangan tiket di `../tickets/` dan rencana di `../plans/` jika tugas tersebut belum tercatat sebagai target backlog di dokumen ini.
2. **MODE 2 (Retrospective per Prompt):** Aturan backlog dibatalkan; AI Agent diwajibkan mencatat pasangan tiket dan rencana secara retrospektif setelah menyelesaikan setiap prompt dari developer.

### 1.2 Transisi Status Backlog
1. **[WAJIB]** Saat item dari backlog ini dipindahkan wujudnya menjadi berkas fisik di `../tickets/TICKET-[EPOCH_TIMESTAMP]-[judul].md` dan `../plans/PLAN-[EPOCH_TIMESTAMP]-[judul].md`, ubah status pada dokumen ini menjadi `[CREATED]` atau coret item tersebut.
2. **[ATURAN RELATIVE PATH]** Seluruh tautan berkas di dalam dokumen ini WAJIB menggunakan relative path (contoh: `../tickets/`, `../plans/`, `../../global-docs/prd.md`). Dilarang menggunakan absolute path.

### 1.3 Kebijakan UI Slicing & Prototyping
1. **Konfirmasi Desain:** Untuk tugas perancangan antarmuka (*UI Slicing*), AI Agent wajib menanyakan kepada pengguna apakah desain sudah tersedia atau perlu dirancang via HTML di `../prototypes/`.
2. **Pemisahan Pasangan Tugas:** Jika menggunakan mekanisme prototipe, pasangan tiket & rencana perancangan prototipe WAJIB dibuat terpisah dari pasangan tiket & rencana implementasi produksi ke dalam codebase.

---

## 2. Struktur Rencana Pengembangan (Milestones)

Setiap fase wajib diuraikan dengan mencantumkan tujuan utama (*objective*) dan daftar antrean pasangan tiket & rencana (menggunakan ID format Epoch timestamp untuk mencegah bentrok multi-agent):

### Fase 1: [Nama Fase, misal: Inisialisasi & Autentikasi]
- **Tujuan:** [Jelaskan target pencapaian di fase ini]
- **Daftar Pasangan Tiket & Rencana (Backlog):**
  - `[ ]` **TICKET-[EPOCH_TIMESTAMP] & PLAN-[EPOCH_TIMESTAMP]:** [Judul/Deskripsi Singkat Fitur]
  - `[ ]` **TICKET-[EPOCH_TIMESTAMP] & PLAN-[EPOCH_TIMESTAMP]:** [Judul/Deskripsi Singkat Fitur]

### Fase 2: [Nama Fase]
- **Tujuan:** [Jelaskan target pencapaian di fase ini]
- **Daftar Pasangan Tiket & Rencana (Backlog):**
  - `[ ]` **TICKET-[EPOCH_TIMESTAMP] & PLAN-[EPOCH_TIMESTAMP]:** [Judul/Deskripsi Singkat Fitur]

---

## 3. Peta Jalan Proyek: [GANTI_DENGAN_NAMA_PROYEK]

> **Instruksi AI Agent:** Hapus placeholder judul di atas dan ganti dengan nama proyek atau node aktual. Susun peta jalan pengembangan dan rincian backlog pasangan tiket & rencana spesifik di bawah garis pemisah ini.

---

*(Peta jalan pengembangan dan rincian backlog pasangan tiket & rencana spesifik untuk proyek/node ini dicatatkan di sini)*
