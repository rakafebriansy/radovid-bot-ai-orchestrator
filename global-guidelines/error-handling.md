# Error Handling & Anti-Hallucination Policy

Pedoman ini mengatur protokol penanganan galat, pembatasan inisiatif AI Agent, dan disiplin debugging analitis guna mencegah halusinasi, perbaikan spekulatif, dan degradasi arsitektur.

---

## 1. Protokol Eskalasi & Interaksi Pengguna (Stop-and-Ask Policy)

### 1.1 Batas Percobaan Maksimal (Max Retry Rule: 2 Iterasi)
Jika perbaikan galat (build error, linter warning, kegagalan test suite) tidak terselesaikan dalam **2 kali iterasi perbaikan**, AI Agent **WAJIB BERHENTI**:
1. Hentikan percobaan acak atau tebakan spekulatif.
2. Laporkan kegagalan kepada pengguna secara transparan: sertakan log terminal aktual, analisis penyebab persisten, dan minta arahan spesifik.
3. Catat detail kegagalan dan jalan buntu teknis tersebut ke dalam `nodes/[nama-node]/retrospectives/RETROSPECTIVE.md` agar terekam sebagai pembelajaran lokal node.

### 1.2 Larangan Inisiatif Liar (No Wild Initiative)
1. **Dilarang Menebak Keputusan:** Dilarang mengambil keputusan arsitektural, fungsional, atau bisnis di luar cakupan instruksi pengguna saat menghadapi informasi ambigu atau kurang lengkap.
2. **Kewajiban Bertanya (Stop-and-Ask):** Jika menghadapi titik percabangan logika yang tidak diatur dalam PRD atau pedoman, wajib berhenti dan meminta konfirmasi pengguna melalui prompt.

---

## 2. Integritas State, Pembuktian & Validasi Kode (State Integrity & Evidence)

### 2.1 Pembuktian Eksekusi Konkret (No Assumed Success)
1. **Dilarang Klaim Berdasarkan Asumsi:** Dilarang mengklaim "kode sudah diperbaiki" atau "seharusnya berfungsi" tanpa bukti eksekusi riil.
2. **Verifikasi Bukti Terminal:** Setiap perubahan kode wajib divalidasi via perintah terminal (linting, kompilasi build, atau test suite). Konfirmasi keberhasilan ke pengguna hanya sah jika terminal mencetak status lulus (PASS/Success).

### 2.2 Verifikasi Eksistensi Berkas (No Blind Imports)
1. **Dilarang Menebak Path:** Sebelum merujuk (import/require), memodifikasi, atau memindahkan berkas yang belum disentuh dalam sesi aktif, wajib memverifikasi keberadaannya secara fisik.
2. Gunakan perintah pemindaian pasif (`ls`, `list_dir`, `grep`, `find`) untuk memastikan berkas dan folder target benar-benar eksis.

### 2.3 Pelestarian State Arsitektur (State Preservation)
1. **Kepatuhan Operasi Berkas:** Dilarang menggunakan perintah destruktif yang menghapus perubahan working tree (`git checkout <file>`, `git restore <file>`, `git clean`, `sed -i`, atau `write_to_file` dengan overwrite tanpa persetujuan). Seluruh aturan pada [safe-file-operations.md](./safe-file-operations.md) berlaku mutlak.
2. **Perbaiki ke Depan (Fix Forward):** Jika terjadi kerusakan akibat kesalahan pengeditan, perbaiki baris spesifik tersebut secara cermat. Dilarang me-reset paksa keseluruhan berkas ke commit lama tanpa menampilkan `git diff` dan memperoleh persetujuan pengguna.
3. **Toleransi Nol untuk Regresi:** Jika menemukan penamaan, struktur modul, atau arsitektur kembali ke versi lama yang salah, segera hentikan tugas dan perbaiki regresi tersebut seketika. Integritas state arsitektur selalu lebih utama daripada kecepatan menyelesaikan tugas (State Integrity > Task Momentum).

---

## 3. Disiplin Debugging & Penanganan Galat (Anti-Slop Debugging & Fail-Fast)

### 3.1 Prinsip Fail-Fast & Larangan Menyembunyikan Galat (No Error Masking)

> **FAIL-FAST OVER SILENT SURVIVAL:**
> Aplikasi yang melempar runtime error secara gamblang jauh lebih berharga daripada aplikasi yang tetap berjalan secara semu namun menyerap data palsu atau rusak.

1. **Dilarang Membungkam Galat (No Silent Catch):**
   - [TERLARANG] Menangkap galat (`catch (e) {}`) lalu secara diam-diam mengembalikan nilai dummy/mock tanpa melempar ulang galat (rethrow) atau menyajikan status error transparan.
   - [WAJIB] Jika dependensi eksternal (API, Database, Storage, SDK) gagal atau tidak tersedia, sistem wajib melempar runtime error eksplisit atau menampilkan Error State UI yang jujur kepada pengguna.
2. **Larangan Defensive Mocking Saat Debugging:**
   - Dilarang menyisipkan data statis/hardcoded sebagai penambal darurat saat aplikasi atau test gagal. Telusuri dan selesaikan akar masalah pada berkas konfigurasi `.env`, koneksi jaringan, atau penanganan galat strukturalnya.

### 3.2 Standar Pesan Galat Runtime & Antarmuka
1. **Pesan Runtime Informatif:** Setiap galat yang dilempar wajib memberikan konteks spesifik: apa yang hilang, nilai apa yang diekspektasikan, dan berkas/variabel mana yang perlu diperiksa developer (contoh: `"[Config Error] NEXT_PUBLIC_API_URL is required but was not provided in .env.local"`).
2. **Pesan UI Berorientasi Pemulihan (Actionable Recovery):** Pesan galat pada antarmuka pengguna wajib menjelaskan apa yang terjadi dan tindakan solusi konkretnya (contoh: *"Nomor telepon belum lengkap. Masukkan 10–13 digit angka dan coba lagi"*, BUKAN hanya *"Input invalid"*).

### 3.3 Larangan Shotgun Debugging & Halusinasi API

> **DEBUGGING DENGAN ANALISIS, BUKAN SPEKULASI:**
> Dilarang melakukan shotgun debugging (mengubah puluhan baris kode secara acak di banyak file dengan harapan salah satunya berhasil).

1. **Dilarang Mengarang Signature API (No Hallucinated APIs/Props):**
   - Dilarang menebak nama method, property, atau prop dari pustaka pihak ketiga yang tidak didukung versi terpasang.
   - Selalu periksa deklarasi tipe (`.d.ts`), `package.json`, atau dokumentasi resmi sebelum menggunakan fungsi baru.
2. **Satu Hipotesis pada Satu Waktu:**
   - Formulasikan satu hipotesis terarah berdasarkan analisis stack trace dan log terminal. Terapkan perbaikan terfokus dan uji hasilnya secara terukur.
3. **Pembersihan Workaround Gagal:**
   - Jika suatu perbaikan eksperimen tidak menyelesaikan masalah, bersihkan sisa kode tersebut sebelum mencoba pendekatan berikutnya. Dilarang menumpuk workaround cacat di codebase.
