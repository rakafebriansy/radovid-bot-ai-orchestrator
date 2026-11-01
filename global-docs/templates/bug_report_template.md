---
id: TICKET-[EPOCH_TIMESTAMP]
plan_id: PLAN-[EPOCH_TIMESTAMP]
title: [BUG] [Judul Singkat]
status: Todo
priority: High # Siklus Valid: Low, Medium, High
severity: Money/Data Loss # Pilihan: Money/Data Loss | Security/Privacy | Broken Flow | Wording/UI
labels: [Bug]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
[Penjelasan detail anomali atau kegagalan yang terjadi.]

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-[EPOCH_TIMESTAMP]-[judul-singkat].md)
*(Gunakan folder `plans/` untuk rincian analisis akar masalah dan diff perbaikan; jaga tiket tetap berfokus pada AC dan DoD).*

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Buka halaman X / Panggil endpoint Y.
2. Masukkan data input Z.
3. Amati kegagalan yang terjadi.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** [Perilaku yang seharusnya terjadi jika sistem normal.]
- **Aktual (Actual):** [Perilaku anomali yang terjadi beserta cuplikan pesan galat.]

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [ ] Bug berhasil dieliminasi dan tidak dapat direproduksi lagi.
- [ ] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [ ] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [ ] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [ ] Lolos verifikasi penuh [Delivery Gate](../../global-guidelines/delivery-gate.md).
- [ ] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `path/ke/file.ext`

---

## 6. AI Execution Log & Output
*Bagian ini WAJIB diisi oleh AI Agent SAAT dan SETELAH memperbaiki bug.*

- **Penyebab Akar (Root Cause):**
  - [Analisis teknis penyebab utama bug.]
- **Langkah Perbaikan:**
  1. ...
  2. ...
- **Ringkasan Berkas Terdampak:**
  - `path/ke/file/...`

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `npm run test` / `pytest` / dll.
- **Output Terminal Aktual:**
  ```text
  [Lampirkan ringkasan output pengujian terminal]
  ```

### Verifikasi Regresi (Disiplin O-07)
- [ ] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `[command]`
  - Pesan Kegagalan: `[pesan error yang membuktikan bug tertangkap]`
- [ ] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** [Bersih / Ditemukan data korup dan telah diperbaiki / Butuh migrasi manual]
- **Rincian Temuan:** [Penjelasan kondisi entitas database/storage terdampak]

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):** PASS (Bukan DB prod, ada bukti konkret, tanpa pelonggaran tes, repo bersih)
- **Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): PASS (Tercatat di atas)
  - O-08 (Edge Case Atlas): PASS (Sumbu dieksplorasi)
  - O-09 (Damage Assessment): PASS (Tercatat di atas)
  - O-10 (Full Suite Metrics): PASS - [misal: 145/145 passed, coverage 100%]
  - O-11 (Real Thing Driven): PASS - [diuji via CLI / browser]
  - O-12 (Shipping Env): PASS
  - Hunt Loop Final Round: PASS (0 temuan baru)
- **Daftar Belum Diuji (Untested List - O-16):**
  - [Sebutkan skenario yang belum diuji beserta alasannya]
