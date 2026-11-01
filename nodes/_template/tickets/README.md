# Local Tickets Directory (`tickets/`)

> **[PRINSIP UTAMA]** Pusat manajemen tugas lokal (*offline-first task management*) yang berfungsi sebagai sumber kebenaran pengerjaan tugas dan disinkronkan ke GitHub Projects.

---

## 1. Tata Kelola & Kebijakan Tiket

### 1.1 Konvensi Penamaan Berkas & Pasangan Beriringan (Inseparable Pair)
Setiap penugasan baru WAJIB memiliki pasangan berkas yang dibuat beriringan menggunakan timestamp UNIX epoch (dalam detik) dan judul deskriptif (*kebab-case*) yang sama:
- **Berkas Tiket:** `TICKET-[EPOCH_TIMESTAMP]-[JUDUL-SINGKAT-KEBAB-CASE].md`
- **Berkas Rencana Teknis:** `../plans/PLAN-[EPOCH_TIMESTAMP]-[JUDUL-SINGKAT-KEBAB-CASE].md`
- **Contoh Valid:** `TICKET-1730421820-login-form.md` berpasangan dengan `PLAN-1730421820-login-form.md`.
- **Rasional Anti-Collision:** Menggantikan urutan angka statis (`TICKET-01`) dengan timestamp UNIX Epoch untuk mencegah tabrakan ID antar-agen pada ekosistem multi-agent.

### 1.2 Format Tiket vs Rencana Implementasi
**[PEMISAHAN FORMAT MUTLAK]**
1. Berkas tiket **DILARANG** diisi dengan detail rencana teknis berbelit-belit, diff kode panjang, atau pseudocode.
2. Format tiket diwajibkan **hanya berisi User Stories, Acceptance Criteria (DoD), daftar berkas terdampak, serta AI Execution Log pasca-eksekusi**.
3. Seluruh rincian arsitektur teknis, pemetaan ketergantungan (Graphify), dan preview perubahan wajib ditulis pada berkas rencana pasangannya di `../plans/PLAN-[EPOCH]-[judul].md`.

### 1.3 Sumber Pasangan Tiket & Rencana (Backlog Ground Truth)
1. **[TERLARANG]** Mengarang tugas baru di luar kendali pengguna.
2. **MODE 1:** Pasangan tiket & rencana wajib merujuk langsung pada backlog di `docs/development-planning.md`.
3. **MODE 2:** Pasangan tiket & rencana dibuat secara retrospektif setelah menyelesaikan prompt developer.

### 1.4 Kebijakan Pemisahan UI Prototyping & Implementasi
1. Untuk tugas *UI Slicing*, konfirmasikan apakah desain sudah tersedia atau perlu dirancang via prototipe HTML di `prototypes/`.
2. Jika menggunakan prototipe, pasangan tiket & rencana perancangan prototipe WAJIB DIPISAHKAN dari pasangan tiket & rencana implementasi produksi ke dalam codebase.

### 1.5 Protokol AI Execution Log & Output
Pada setiap tiket, AI Agent WAJIB mencatat jejak kerja teknis pada seksi khusus di bagian bawah:
1. Rangkuman modifikasi file dan landasan keputusan arsitektur.
2. **Bukti Eksekusi (Evidence - O-03):** Potongan perintah dan output terminal aktual (pengujian lulus, metrik suite, pratinjau visual jika UI). Dilarang mengklaim PASS berdasarkan asumsi!
3. **Verifikasi Delivery Gate:** Evaluasi kelulusan 4 Pilar (Higienitas Kode, Hard Gate, Must Probe Loop, dan Tata Kelola).
4. **Daftar Belum Diuji (Untested List - O-16):** Deklarasi transparan skenario yang belum diuji beserta alasannya.

---

## 2. Format Baku Pembuatan Tiket (Boilerplate Template)

Salin struktur Markdown berikut saat membuat tiket baru. *(Catatan: Untuk laporan bug, wajib gunakan `../../global-docs/templates/bug_report_template.md`).*

```markdown
---
id: TICKET-[EPOCH_TIMESTAMP]
plan_id: PLAN-[EPOCH_TIMESTAMP]
title: [Judul Singkat Tugas]
status: Todo # Siklus Valid: Todo -> In Progress -> Done
priority: High # Siklus Valid: Low, Medium, High
labels: [Frontend, Backend, Feature, Bug, dsb]
---

# Deskripsi
Sebagai [Peran Pengguna],
Saya ingin [Kebutuhan Fitur / Aksi],
Sehingga [Manfaat / Tujuan Bisnis].

*(Penjelasan ringkas latar belakang masalah teknis atau konteks fungsional).*

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-[EPOCH_TIMESTAMP]-[judul-singkat].md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] [Kriteria 1]
- [ ] [Kriteria 2]
- [ ] [Kriteria 3]

## Target Lingkup File (Affected Files)
- `path/ke/file.ext`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. ...
  2. ...
- **Ringkasan File Terpengaruh:**
  - `path/ke/file/...`
- **Catatan & Keputusan Arsitektural:**
  - ...

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `npm run test` / `pytest` / dll.
- **Output Terminal:**
  ```text
  [Salin ringkasan output tes terminal, misal: PASS path/to/test (X tests passed)]
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** PASS (Zero-comment, no fallback/mock, no magic numbers, no direct DDL, safe file ops)
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** PASS (Bukan DB prod, ada bukti konkret, tanpa pelonggaran tes, repositori bersih)
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): [PASS / N/A jika fitur baru] - Bukti: [pesan error tes di kode lama]
  - O-08 (Edge Case Atlas): PASS (Sumbu: Who, How much, When, State, Input)
  - O-09 (Damage Assessment): [PASS / Bersih / N/A] - Catatan: [kondisi data lama]
  - O-10 (Full Suite Metrics): PASS - [misal: 100/100 passed, coverage 100%]
  - O-11 (Real Thing Driven): PASS - [misal: diuji via CLI / browser]
  - O-12 (Shipping Env Check): PASS
  - Hunt Loop Final Round: PASS (0 temuan baru di putaran terakhir)
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: PASS (dijalankan di codebase node asli)
  - CHANGELOG.md: PASS (diperbarui di baris teratas)
  - Commit Message Sanity: PASS (steril dari istilah orchestrator/tiket/rencana)
  - O-16 Untested List: [Sebutkan hal yang belum sempat diuji beserta alasannya]
```
