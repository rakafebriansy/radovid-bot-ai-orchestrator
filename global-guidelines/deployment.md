# Deployment & Release Standard

Pedoman ini mengatur standar perilisan dan deployment lintas platform guna menjamin keamanan kredensial, keandalan performa build, kepatuhan versioning, dan stabilitas layanan di lingkungan produksi.

---

## 1. Verifikasi Universal Pra-Rilis (Pre-Deployment Checklist)

Terlepas dari jenis platform (Web, Mobile, Game, atau API), seluruh butir pemeriksaan berikut wajib dipenuhi sebelum proses perilisan:

### 1.1 Gerbang Pengujian & Delivery Gate
AI Agent DILARANG KERAS mengeksekusi skrip deployment jika pengujian belum mencapai status lulus (PASS):
1. Seluruh test suite wajib berstatus 100% PASS dan mematuhi [testing.md](./testing.md).
2. Wajib lolos seluruh evaluasi checklist pra-serah terima di [delivery-gate.md](./delivery-gate.md).
3. Jika melibatkan perubahan skema database, migrasi DDL terstruktur wajib dieksekusi terlebih dahulu sesuai [database.md](./database.md).

### 1.2 Audit Konfigurasi & Kredensial (Environment & Secret Audit)
1. **Peralihan URL:** Pastikan seluruh endpoint dan URL pengembangan (localhost, staging) telah dialihkan ke URL produksi.
2. **Larangan Kredensial Hardcoded:** Kredensial, kunci API, token otentikasi, dan connection string wajib dibaca dari Environment Variables produksi sesuai [security.md](./security.md) dan [coding.md](./coding.md).

### 1.3 Optimasi Build Produksi (Build Optimization)
Build wajib dieksekusi dengan konfigurasi mode rilis (production/release flag):
1. Aktifkan minifikasi kode (minification), tree-shaking, dan kompresi bundle.
2. Hapus seluruh logging internal dan debug output yang tidak diperlukan pada rilis publik.

### 1.4 Tata Kelola Versi (Semantic Versioning)
Tingkatkan nomor versi aplikasi pada berkas manifes terkait (`package.json`, `pubspec.yaml`, `build.gradle`, `Cargo.toml`) secara disiplin mengikuti kaidah Semantic Versioning (SemVer):
- `MAJOR`: Perubahan arsitektur dengan breaking changes.
- `MINOR`: Penambahan fitur baru yang backward-compatible.
- `PATCH`: Perbaikan bug backward-compatible.

---

## 2. Standar Deployment Spesifik Platform (Platform-Specific Standards)

### 2.1 Aplikasi Web & API Server (Backend)
1. **Containerization Multi-Stage:** Bungkus aplikasi ke dalam Docker container untuk konsistensi lingkungan. Konfigurasikan `Dockerfile` menggunakan *multi-stage build* untuk meminimalkan ukuran image rilis final.
2. **Reverse Proxy & SSL:** Jalankan server aplikasi di balik reverse proxy standar industri (Nginx, Caddy) dengan enkripsi SSL/TLS aktif.
3. **Dynamic Port Binding:** Wajib mengikat port jaringan yang dialokasikan oleh environment runtime peladen (misal: `process.env.PORT`), dilarang mengunci port secara statis/hardcoded.

### 2.2 Aplikasi Mobile (iOS & Android)
1. **Distribusi Sertifikat (Code Signing):** Pastikan penandatanganan aplikasi menggunakan sertifikat distribusi resmi (Distribution Keystore untuk Android, Distribution Provisioning Profile untuk iOS), bukan sertifikat development/debug.
2. **Sanitasi Izin (Permission Sanitization):** Periksa kembali manifes perizinan (`AndroidManifest.xml`, `Info.plist`). Hapus seluruh izin akses sensitif yang hanya digunakan selama debugging lokal.
3. **Format Distribusi Modern:** Pada platform Android, prioritaskan pembuatan paket distribusi berformat Android App Bundle (`.aab`) ketimbang `.apk` mentah untuk optimasi ukuran unduhan pengguna.

### 2.3 Pengembangan Game (Game Development)
1. **Pembersihan Aset (Asset Stripping):** Konfigurasikan build pipeline untuk membuang aset yang tidak direferensikan (unused asset stripping) pada scene rilis akhir.
2. **Penyesuaian Arsitektur Target:** Sesuaikan arsitektur kompilasi biner (misal: ARM64, x86_64) secara presisi dengan spesifikasi sistem operasi perangkat target sasaran.