# Internationalization (i18n) & Localization Standard

Pedoman ini mengatur standar arsitektur internasionalisasi (i18n) dan pengelolaan kamus terjemahan guna menjamin skalabilitas multi-bahasa, konsistensi penamaan kunci, dan kemudahan pemeliharaan.

---

## 1. Arsitektur Berkas Kamus Terjemahan (Dictionary Architecture)

### 1.1 Pemisahan Berbasis Namespace
Struktur berkas kamus terjemahan (JSON, YAML, XML) wajib dipecah berdasarkan namespace atau domain fitur:
- Contoh struktur: `locales/en/common.json`, `locales/en/auth.json`, `locales/en/settings.json`.
- Dilarang menggabungkan seluruh teks antarmuka ke dalam satu berkas raksasa tunggal (monolithic dictionary).

### 1.2 Konvensi Penamaan Kunci Terjemahan
1. Terapkan konvensi penamaan kunci secara konsisten di seluruh modul: gunakan `snake_case` (misal: `login_button_title`) atau `camelCase` (misal: `loginButtonTitle`).
2. Hindari menggunakan teks kalimat bahasa asli sebagai kunci (dilarang menggunakan `"Click Here"` sebagai key objek).

---

## 2. Tata Kelola Bahasa & Sinkronisasi Kamus (Language Governance)

### 2.1 Bahasa Bawaan & Cadangan (Fallback Language)
1. **Bahasa Cadangan (Fallback):** Sistem i18n wajib mengonfigurasi Bahasa Inggris (`en`) sebagai bahasa cadangan utama jika entri teks pada bahasa target yang dipilih tidak ditemukan.
2. **Kesesuaian Preferensi Proyek:** Selaras dengan [coding.md](./coding.md), default teks aplikasi adalah Bahasa Inggris kecuali jika pengguna secara eksplisit meminta bahasa lain dan telah dicatat pada `nodes/[nama-node]/guidelines/project-context.md`.

### 2.2 Sinkronisasi Kamus Lintas Bahasa
1. **Kelengkapan Kunci Multi-Bahasa:** Setiap penambahan atau perubahan kunci teks baru wajib disinkronkan ke seluruh berkas bahasa yang didukung proyek guna mencegah teks hilang (missing translation keys).
2. **Larangan String Antarmuka Hardcoded:** Pada modul yang mengaktifkan sistem i18n, dilarang menyisipkan teks antarmuka secara statis langsung di dalam komponen visual; seluruh label dan pesan wajib dipanggil via hook atau fungsi translasi.
