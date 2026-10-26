# Continuous Integration & Delivery (CI/CD) Standard

Pedoman ini mengatur standar otomasi pipeline CI/CD (GitHub Actions, GitLab CI, Bitbucket Pipelines) guna menjamin keandalan integrasi kode, efisiensi eksekusi build, dan penegakan gerbang kualitas otomatis sebelum kode digabungkan ke branch utama.

---

## 1. Pemicu & Tata Kelola Pipeline (Triggers & Branch Governance)

### 1.1 Konfigurasi Pemicu Otomatis (Event Triggers)
Pipeline wajib terkonfigurasi untuk berjalan secara otomatis pada:
1. Event `push` ke branch utama (`main`, `master`, `develop`).
2. Event `pull_request` yang menargetkan branch utama.

### 1.2 Kebijakan Gerbang Pemblokir (Blocking PR Gate)
Pull Request DILARANG digabungkan (merge) jika terdapat satu pun tahapan job yang berstatus gagal (FAIL). Pipeline CI/CD berfungsi sebagai gerbang verifikasi mutlak sebelum integrasi kode.

---

## 2. Urutan Tahapan Pekerjaan (Sequential Pipeline Stages)

Urutan pekerjaan (jobs) wajib mematuhi sekuens logis berikut:

```text
[1. Install Dependencies] ──► [2. Lint & Formatting] ──► [3. Typecheck & Build] ──► [4. Automated Tests]
```

### 2.1 Instalasi Dependensi Deterministik
Gunakan perintah instalasi berbasis lockfile beku guna menjamin konsistensi paket:
- Node.js: `npm ci` atau `pnpm install --frozen-lockfile` (dilarang menggunakan `npm install` biasa di lingkungan CI).
- Python: `pip install -r requirements.txt` atau instalasi berbasis lockfile Poetry.
- Flutter / Dart: `flutter pub get`.

### 2.2 Format Kode & Analisis Statis (Linting)
Eksekusi linter dan formatter standar:
- Verifikasi pemformatan kode (Prettier, Black, Gofmt).
- Analisis statis dan type checking (ESLint, `tsc --noEmit`, Pylint).

### 2.3 Kompilasi Build
Eksekusi build dalam mode produksi guna memverifikasi ketiadaan impor putus, circular dependencies, atau galat kompilasi.

### 2.4 Eksekusi Pengujian Otomatis (Automated Testing)
Jalankan seluruh test suite secara headless dan terisolasi sesuai [testing.md](./testing.md):
- Seluruh unit test dan integration test wajib lulus (100% PASS).
- Tampilkan ringkasan metrik cakupan pengujian (test coverage) pada log CI.

---

## 3. Optimasi Performa & Keamanan Pipeline (Optimization & Security)

### 3.1 Mekanisme Caching Dependensi
Wajib mengonfigurasi cache untuk direktori dependensi pihak ketiga (`node_modules`, `~/.npm`, `~/.cache/pip`, `.gradle/caches`, `target/`) guna memangkas durasi eksekusi pada putaran build berulang.

### 3.2 Pengelolaan Rahasia Lingkungan (Secret Management)
1. Seluruh kunci API, token deploy, dan connection string wajib disuplai melalui fitur Encrypted Secrets repositori (seperti GitHub Secrets).
2. Dilarang mencetak rahasia lingkungan secara mentah ke log konsol terminal sesuai [security.md](./security.md).
