# Direktori Laporan Hunt (`hunt/`)

> **[PRINSIP UTAMA]** Arsip resmi hasil pengujian eksplorasi mendalam (*exploratory testing*) dan pembuktian ketahanan sistem secara proaktif sebelum rilis.

---

## 1. Tata Kelola & Klasifikasi Audit

### 1.1 Perbedaan `hunt/` vs `retrospectives/`
| Direktori | Sifat & Fokus | Waktu Penggunaan |
|---|---|---|
| **`retrospectives/RETROSPECTIVE.md`** | **Reaktif:** Mencatat jalan buntu teknis (*blockers*), kebingungan arsitektur, dan kegagalan berulang saat batas retry terlampaui. | Terjadi saat koding aktif mengalami kebuntuan error berulang. |
| **`hunt/`** | **Proaktif:** Mencatat investigasi celah ekstrem (*edge cases*), pengujian batas, dan audit ketahanan sistem. | Sesi audit kualitas berkala, evaluasi QA mandiri, atau sebelum rilis besar. |

### 1.2 Konvensi Penamaan Berkas
Setiap laporan sesi hunt WAJIB menggunakan penomoran berurut:
`hunt-NNN-YYYY-MM-DD.md`
- **Contoh Valid:** `hunt-001-2026-10-26.md`, `hunt-002-2026-11-04.md`

### 1.3 Format Templat Wajib
Setiap laporan hunt baru WAJIB menyalin dan mematuhi format baku di:
`../../global-docs/templates/hunt-findings.md`

---

## 2. Alur Kerja Sesi Hunt (Tujuh Tahap Baku)

1. **Inisiasi Berkas:** Buat file `hunt-NNN-YYYY-MM-DD.md` baru dengan menyalin templat `hunt-findings.md`.
2. **Eksplorasi Kasus (Hunt Loop):** Jalankan putaran investigasi berbasis 9 sumbu [Edge Case Atlas](../../global-guidelines/edge-case-atlas.md); sebutkan minimal 10 skenario batas di putaran pertama.
3. **Dokumentasikan Temuan:** Catat anomali ke tabel temuan terurut berdasarkan dampak (O-13: Kehilangan Uang/Data -> Keamanan/Privasi -> Alur Rusak -> Kosmetik/UI).
4. **Damage Assessment (O-09):** Periksa integritas data historis terhadap potensi kerusakan akibat perilaku versi lama.
5. **Untested List (O-16):** Deklarasikan daftar skenario yang belum diuji beserta alasannya.
6. **Laporkan ke Pengguna:** Paparkan ringkasan temuan kepada pengguna untuk penentuan prioritas perbaikan.
7. **Perbaikan & Delivery Gate:** Terapkan perbaikan, buktikan tes gagal di kode lama (O-07), jalankan seluruh suite (O-10), dan evaluasi kelulusan [Delivery Gate](../../global-guidelines/delivery-gate.md).
