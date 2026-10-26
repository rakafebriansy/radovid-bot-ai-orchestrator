# Laporan Temuan Sesi Hunt: hunt-[NNN]-[YYYY-MM-DD]

- **Target Audit:** [Fitur, branch, modul, atau proses yang diaudit]
- **Lingkungan Pengujian:** [Sandbox lokal / Test DB terisolasi; konfirmasi bukan prod/dev riil (O-01, O-02)]
- **Jumlah Putaran (Rounds):** [n putaran]
- **Putaran Terakhir Menemukan Nol Kasus Baru:** [Ya / Tidak]

---

## Tabel Temuan (Findings)

*Temuan diurutkan berdasarkan skala impak (O-13): Kehilangan Uang/Data $\rightarrow$ Keamanan/Privasi $\rightarrow$ Alur Bisnis Patah $\rightarrow$ Kosmetik/Tampilan.*

| # | Keparahan (Severity) | Aturan | Skenario Kasus (Aktor, Input, State, Urutan) | Cara Reproduksi | Bukti (Evidence — O-03) |
|---|---|---|---|---|---|
| 1 | Kehilangan Uang / Data | O-09 | | | |
| 2 | Keamanan / Privasi | O-01 | | | |
| 3 | Alur Bisnis Patah | O-04 | | | |
| 4 | Kosmetik / Teks / UI | O-15 | | | |

---

## Penilaian Kerusakan Data Masa Lalu (Damage Assessment — O-09)

- **Temuan Data Rusak:** [Apakah ada data eksisting yang sudah terlanjur korup akibat perilaku lama? Berapa banyak record/entitas terdampak?]
- **Status Tindakan:** [Sudah diperbaiki / Memerlukan skrip migrasi / Perlu izin pengguna]

---

## Daftar Pengujian yang Belum Dilakukan (Untested List — O-16)

*Deklarasikan secara eksplisit aspek yang sengaja belum/tidak diuji beserta alasannya:*
- [Contoh: Pengujian pada perangkat iOS nyata belum dilakukan karena keterbatasan simulator macOS]
- [Contoh: Webhook payment gateway produksi belum diuji karena ketiadaan sandbox live]

---

## Tindak Lanjut Perbaikan (Follow-Up Post-Fix)

*(Bagian ini ditambahkan setelah pengembang/pengguna memilih temuan yang akan diperbaiki)*

- **Nomor Temuan Terpilih:** [misal: #1, #2]
- **Nomor Temuan Diperbaiki:** [misal: #1, #2 — masing-masing menyertakan bukti tes gagal di kode lama (O-07)]
- **Hasil Test Suite Lengkap:** [misal: 245/245 tests passed, coverage 100% (O-10)]
- **Evaluasi Delivery Gate:** [PASS / FAIL]
