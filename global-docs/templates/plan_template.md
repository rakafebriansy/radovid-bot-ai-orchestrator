---
id: PLAN-[EPOCH_TIMESTAMP]
ticket_id: TICKET-[EPOCH_TIMESTAMP]
title: [Judul Rencana Teknis Implementasi]
status: Draft # Siklus Valid: Draft -> Approved -> In Progress -> Completed
created_at: [ISO-Date/Timestamp]
---

# Rencana Implementasi Teknis: [Judul Tugas]

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-[EPOCH]](../tickets/TICKET-[EPOCH]-[judul-singkat].md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../global-guidelines/coding.md) | [Safe File Operations](../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** [Penjelasan detail apa yang akan diubah dan mengapa pendekatan teknis ini dipilih]
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: `...`
  - Komponen Terdampak: `...`
  - Potensi Efek Samping: `...`

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `path/ke/file-baru.ext` | Menambahkan modul/komponen baru |
| `MODIFY` | `path/ke/file-lama.ext` | Memperbarui logika bisnis atau antarmuka |
| `DELETE` | `path/ke/file-usang.ext` | Menghapus kode tidak terpakai |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Siapkan unit test pada `path/ke/test.ext`.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Tulis kode produksi minimal hingga tes lulus (GREEN).
3. **Langkah 3 (Refactor & Integrasi):**
   - Terapkan Zero-Comment Policy, Fail-Fast, dan isolasi modul.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan minimal 10 kasus batas merujuk `edge-case-atlas.md` dan tuntaskan delivery gate.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```typescript
// Cuplikan diff sebelum eksekusi
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** [Rujuk edge-case-atlas.md]
- **Strategi Rollback:** [Instruksi pemulihan jika terjadi kegagalan]

