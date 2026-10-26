# Dependency & Package Management Policy

Pedoman ini mengatur standar pengelolaan pustaka eksternal (third-party dependencies) guna mencegah pembengkakan ukuran aplikasi (bloat), kerentanan keamanan, konflik resolusi versi, dan rusaknya integritas riwayat dependensi.

---

## 1. Tata Kelola & Justifikasi Penambahan Dependensi (Dependency Governance)

### 1.1 Larangan Instalasi Otonom Tanpa Justifikasi
AI Agent DILARANG menginstal paket eksternal baru (via npm, pnpm, yarn, pip, composer, pub, cargo, go get) secara mandiri tanpa justifikasi teknis yang jelas.

### 1.2 Kriteria Evaluasi Pustaka
Sebelum merekomendasikan paket baru, AI Agent wajib mengevaluasi 3 kriteria berikut:
1. **Native vs External Logic:** Jelaskan pertimbangan mengapa paket pihak ketiga lebih menguntungkan dibanding menulis logika mandiri (native logic / YAGNI). Hindari memasang dependensi mikro untuk fungsi sepele.
2. **Reputasi & Pemeliharaan:** Prioritaskan pustaka standar industri yang aktif dipelihara, teruji, dan memiliki rekam jejak keamanan baik dibanding paket niche yang tidak dikenal.
3. **Kesesuaian Platform & Arsitektur:** Verifikasi kompatibilitas target platform sesuai spesifikasi di `nodes/[nama-node]/docs/system-design.md` (contoh: dilarang memasang pustaka manipulasi DOM Web pada lingkungan Flutter Mobile/Desktop murni).

### 1.3 Protokol Izin Mutasi Dependensi (State Mutation Protocol)
Instalasi paket eksternal tergolong **Operasi Modifikasi State**:
- Sesuai [safe-file-operations.md](./safe-file-operations.md), AI Agent dilarang langsung meminta permission untuk perintah instalasi terminal sebelum mencantumkan nama paket, alasan, dan dampaknya di dalam Implementation Plan serta memperoleh persetujuan pengguna.

---

## 2. Integritas Lockfile & Resolusi Konflik (Lockfile Integrity)

### 2.1 Larangan Menghapus Berkas Pengunci (Lockfile)
Berkas pengunci versi absolut (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `pubspec.lock`, `poetry.lock`, `Gemfile.lock`, `Cargo.lock`, `go.sum`) adalah dokumen integritas yang pantang dirusak.
- Dilarang merekomendasikan atau menghapus lockfile sebagai jalan pintas saat menghadapi konflik dependensi atau kegagalan instalasi.

### 2.2 Prosedur Resolusi Konflik yang Benar
Jika terjadi konflik ketidakcocokan versi (peer dependency / resolution conflict):
1. Telusuri akar masalah pada berkas manifes utama (`package.json`, `pubspec.yaml`, `pyproject.toml`, `Cargo.toml`).
2. Selaraskan batasan versi (semver constraints) yang saling berbenturan secara eksplisit.
3. Jalankan pembaruan terarah (misal: `npm update <package>` atau audit resolver) tanpa melenyapkan riwayat deterministik lockfile.
