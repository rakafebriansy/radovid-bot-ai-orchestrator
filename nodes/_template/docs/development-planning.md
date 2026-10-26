# Development Planning & Ticket Backlog Guide

> **[PRINSIP UTAMA]** Dokumen peta jalan strategis (*roadmap*) dan gudang antrean tiket (*ticket backlog*) yang menjembatani dokumen makro (PRD dan System Design) dengan tugas mikro (tiket eksekusi).

---

## 1. Aturan Sinkronisasi & Kebijakan Tiket

### 1.1 Kontrol Pembuatan Tiket Berdasarkan Mode
1. **MODE 1 (Strict Backlog First):** AI Agent DILARANG KERAS membuat berkas tiket baru di `tickets/` jika tiket tersebut belum tercatat sebagai target backlog di dokumen ini.
2. **MODE 2 (Retrospective per Prompt):** Aturan backlog dibatalkan; AI Agent diwajibkan membuat tiket retrospektif setelah menyelesaikan setiap prompt dari developer.

### 1.2 Transisi Status Backlog
1. **[WAJIB]** Saat tiket dari backlog ini dipindahkan wujudnya menjadi berkas markdown di `tickets/TICKET-XX.md`, ubah status tiket pada dokumen ini menjadi `[CREATED]` atau coret item tersebut.

### 1.3 Kebijakan UI Slicing & Prototyping
1. **Konfirmasi Desain:** Untuk tiket perancangan antarmuka (*UI Slicing*), AI Agent wajib menanyakan kepada pengguna apakah desain sudah tersedia atau perlu dirancang via HTML di `prototypes/`.
2. **Pemisahan Tiket:** Jika menggunakan mekanisme prototipe, tiket perancangan prototipe WAJIB dibuat terpisah dari tiket implementasi produksi ke dalam codebase.

---

## 2. Struktur Rencana Pengembangan (Milestones)

Setiap fase wajib diuraikan dengan mencantumkan tujuan utama (*objective*) dan daftar antrean tiket:

### Fase 1: [Nama Fase, misal: Inisialisasi & Autentikasi]
- **Tujuan:** [Jelaskan target pencapaian di fase ini]
- **Daftar Tiket (Backlog):**
  - `[ ]` **TICKET-01:** [Judul/Deskripsi Singkat]
  - `[ ]` **TICKET-02:** [Judul/Deskripsi Singkat]

### Fase 2: [Nama Fase]
- **Tujuan:** [Jelaskan target pencapaian di fase ini]
- **Daftar Tiket (Backlog):**
  - `[ ]` **TICKET-03:** [Judul/Deskripsi Singkat]

---

## 3. Peta Jalan Proyek: [GANTI_DENGAN_NAMA_PROYEK]

> **Instruksi AI Agent:** Hapus placeholder judul di atas dan ganti dengan nama proyek atau node aktual. Susun peta jalan pengembangan dan rincian backlog tiket spesifik di bawah garis pemisah ini.

---

*(Peta jalan pengembangan dan rincian backlog tiket spesifik untuk proyek/node ini dicatatkan di sini)*
