# Local Implementation Plans Directory (`plans/`)

> **[PRINSIP UTAMA]** Pusat penyimpanan rencana teknis terperinci (*Implementation Plans*) sebelum mutasi state kode. Berfungsi sebagai cetak biru rekayasa detail, terpisah secara ketat dari tiket tugas.

---

## 1. Pemisahan Tanggung Jawab: Tiket vs Rencana Teknis

| Dimensi | Tiket Tugas (`tickets/`) | Rencana Implementasi (`plans/`) |
| :--- | :--- | :--- |
| **Audiens & Bahasa** | Stakeholder, Product Owner, AI Agent | Engineer, Arsitek, AI Agent |
| **Fokus Konten** | *User Stories*, *Acceptance Criteria* (DoD), daftar berkas terdampak | Analisis arsitektur mendalam, langkah perbaikan sekuensial, *diff preview*, mitigasi risiko |
| **Format Wajib** | Ringkas, kriteria berbasis checklist | Teknis, mendalam, menyertakan pemetaan dependensi |
| **Larangan Konten** | **DILARANG** memuat instruksi teknis berbelit-belit, pseudocode panjang, atau diff kode raksasa | **DILARANG** menggantikan fungsi acceptance criteria atau user story |

---

## 2. Tata Kelola & Kebijakan Berkas Rencana

### 2.1 Konvensi Penamaan Berkas & Pasangan Beriringan (Inseparable Pair)
Setiap berkas rencana baru WAJIB dibuat beriringan dengan berkas tiket pasangannya (`../tickets/TICKET-[EPOCH_TIMESTAMP]-[judul].md`) menggunakan timestamp UNIX epoch (dalam detik) dan judul deskriptif (*kebab-case*) yang selaras guna menghindari tabrakan (*collision*) saat banyak agen AI bekerja bersamaan:
```text
PLAN-[EPOCH_TIMESTAMP]-[JUDUL-SINGKAT-KEBAB-CASE].md
```
- **Contoh Valid:** `PLAN-1730421820-jwt-authentication-flow.md` berpasangan dengan `TICKET-1730421820-jwt-authentication-flow.md`.

### 2.2 Siklus Hidup Rencana
1. **Penyusunan (Draft):** Disusun sebelum melakukan mutasi state (edit file, instalasi dependensi, migrasi DDL).
2. **Review & Approval:** AI Agent memaparkan rencana kepada pengguna dan menunggu persetujuan eksplisit.
3. **Penyimpanan Permanen:** Berkas rencana disimpan di direktori `plans/` ini sebagai dokumentasi historis rekayasa sistem yang dapat ditinjau kembali di masa depan.

---

## 3. Format Baku Pembuatan Rencana (Boilerplate Template)

Salin struktur Markdown di bawah ini saat membuat berkas rencana baru:

```markdown
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
- **Tujuan Teknis:** [Penjelasan detail apa yang akan diubah dan mengapa pendekatan ini dipilih]
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: `...`
  - Komponen Terdampak: `...`
  - Potensi Efek Samping: `...`

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `path/ke/file-baru.ext` | Menambahkan utilitas / komponen baru |
| `MODIFY` | `path/ke/file-lama.ext` | Memperbarui logika bisnis atau routing |
| `DELETE` | `path/ke/file-usang.ext` | Menghapus kode yang sudah tidak dipakai |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `path/ke/test.ext`.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Tulis kode produksi minimal untuk meloloskan tes.
3. **Langkah 3 (Refactor & Integrasi):**
   - Rapikan struktur kode sesuai `coding.md` (Zero-Comment Policy, Fail-Fast).
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan pengujian kasus batas ekstrem dan verifikasi gerbang serah terima.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
*(Tampilkan cuplikan diff fokus sebelum mutasi berkas diterapkan)*

```typescript
// Contoh diff terencana
+ export function validateToken(token: string): TokenPayload { ... }
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** [Rujuk edge-case-atlas.md]
- **Strategi Rollback:** [Langkah pengembalian state jika terjadi galat fatal, misal: git reset --hard]
```

