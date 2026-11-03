# Prototypes Directory (`prototypes/`)

> **[PRINSIP UTAMA]** Ruang kanvas visual mandiri (*Rapid HTML-based Previewing*) untuk merancang, memoles, dan memvalidasi antarmuka pengguna sebelum ditranslasikan ke dalam kerangka kerja kode produksi (*production framework*).

---

## 1. Konsep & Kebijakan Prototipe

### 1.1 Metodologi Rapid HTML-based Previewing
1. **Validasi Tanpa Hambatan:** Hindari menyusun antarmuka awal langsung pada framework target yang memakan waktu kompilasi berat (Flutter, SwiftUI, React Native). Seluruh desain visual awal WAJIB dirancang menggunakan teknologi web dasar (**HTML, CSS, Canvas**).
2. **Iterasi Cepat:** Memungkinkan penyesuaian tata letak, warna, dan interaktivitas secara instan di peramban sebelum menyentuh kode produksi aplikasi.
3. **Translasi Terisolasi:** Setelah purwarupa HTML disetujui pengguna, barulah struktur visual diterjemahkan ke dalam komponen native (seperti *Widget* Flutter atau *View* SwiftUI).

### 1.2 Tiga Aturan Mutlak Single-File Policy
1. **[WAJIB] Single-File Mandiri:** Setiap rancangan komponen atau layar HARUS disatukan ke dalam satu berkas `.html` tunggal (contoh: `pos-dashboard.html`).
2. **[TERLARANG] Memecah Berkas Eksternal:** Dilarang memisahkan file `.css` atau `.js` eksternal. Seluruh gaya penataan wajib ditulis di dalam blok internal `<style>`, dan logika interaksi mockup di dalam blok internal `<script>`.
3. **[TERLARANG] Koneksi Backend/DB:** Prototipe berfungsi murni sebagai kanvas statis visual. DILARANG memanggil API produksi, websocket, atau menghubungkan ke basis data.

---

## 2. Struktur Direktori & Organisasi Berkas

### 2.1 Hierarki Folder Baku
Kelola berkas prototipe secara terstruktur:

```text
prototypes/
├── pages/         # Layar penuh (full-page layout), contoh: pos-dashboard.html
└── components/    # Komponen mandiri (micro-UI elements), contoh: item-card.html
```

### 2.2 Modularitas DOM & Kesiapan Translasi
1. Rancang struktur DOM secara presisi, semantik, dan modular.
2. Manfaatkan Flexbox dan CSS Grid proporsional agar elemen antarmuka dapat diekstraksi secara langsung menjadi komponen framework produksi tanpa perombakan struktur logika.
