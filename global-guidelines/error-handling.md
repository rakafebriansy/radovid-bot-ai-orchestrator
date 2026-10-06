# Error Handling & Anti-Hallucination Policy

Dokumen ini adalah barikade utama untuk mencegah perilaku halusinasi, asumsi keliru, atau percobaan *debugging* secara "membabi buta" oleh AI Agent.

## Aturan "Stop-and-Ask" (Anti-Looping)
1. **Batas Percobaan Maksimal (Max Retry Rule):** Apabila Anda mencoba memperbaiki sebuah kegagalan (contoh: *build error*, peringatan linting berulang, kegagalan *test case*) dan gagal menyelesaikannya dalam **2 kali iterasi penyelesaian**, Anda **DIWAJIBKAN MUTLAK** untuk berhenti meraba-raba (*blind guessing*).
2. **Kewajiban Melapor:** Setelah menyentuh batas percobaan tersebut, Anda harus secara proaktif melaporkan kegagalan tersebut kepada pengguna. Paparkan *log error* terakhir secara utuh, sampaikan hipotesis mengapa hal ini sangat persisten, dan mintalah panduan lebih spesifik.
3. **Kewajiban Pencatatan Retrospektif:** Selain melapor kepada pengguna, Anda **WAJIB** mencatatkan detail kegagalan persisten ini ke dalam log pembelajaran mandiri di `nodes/[nama-node]/retrospectives/RETROSPECTIVE.md` (mengikuti pedoman di folder tersebut). Ini untuk memastikan jejak masalah terekam spesifik untuk *node* ini.

## Larangan Inisiatif Liar (No Wild Initiative)
1. **Dilarang Menebak Keputusan:** Anda **DILARANG KERAS** mengambil inisiatif liar, menebak-nebak, atau membuat keputusan arsitektural/fungsional di luar cakupan instruksi pengguna jika terdapat informasi yang kurang atau ambigu.
2. **Wajib Bertanya:** Jika Anda kebingungan dalam mengambil keputusan, merasa kurang konteks, atau menghadapi titik percabangan (*branching point*) yang tidak diatur di dalam pedoman mana pun, Anda **WAJIB BERHENTI** dan bertanya kepada pengguna melalui *prompt*. Inisiatif yang salah dan merugikan sangat dikutuk!

## Larangan Mengasumsikan Keberhasilan
1. **Wajib Memvalidasi Eksekusi:** Anda **DILARANG KERAS** melontarkan klaim "Kode sudah berhasil diperbaiki" atau "Seharusnya sudah berfungsi" secara teoretis tanpa eksekusi bukti konkret.
2. Jika Anda memodifikasi kode fungsional, Anda harus menjalankan validasi di lapis bawah secara mandiri (misalnya dengan menjalankan terminal linting, mengeksekusi kompilasi lokal, atau menjalankan skrip *test case* yang relevan). Hanya ketika indikator terminal mencetak status bersih (*pass/success*), barulah Anda boleh mengonfirmasi ke pengguna.

## Verifikasi Eksistensi File (No Blind Imports)
1. **Dilarang Menebak Path:** Sebelum Anda (AI Agent) merujuk (*import/require*), mengedit secara *inline*, atau membongkar struktur file apa pun yang belum pernah Anda sentuh di sesi ini, Anda **WAJIB** melakukan pengecekan awal.
2. Gunakan perintah pemindaian terminal seperti `ls`, pemanggilan fungsi pelacakan direktori, atau penelusuran *grep* untuk memvalidasi bahwa lokasi direktori atau struktur berkas (*file/folder*) yang akan Anda kerjakan benar-benar eksis secara fisik di sistem.

## Codebase State Preservation & Architectural Integrity

### 🚫 Larangan Keras (Anti-Patterns)
1. **Destructive File Operations:** JANGAN PERNAH menggunakan perintah atau tool yang menimpa/menghapus file tanpa review eksplisit dari user. Ini mencakup (namun tidak terbatas pada) `git checkout <file>`, `git restore <file>`, `git clean`, `sed -i`, dan `write_to_file` dengan `Overwrite: true` pada file yang sudah ada. Lihat **[safe-file-operations.md](./safe-file-operations.md)** untuk daftar lengkap larangan dan alternatif wajibnya — dokumen tersebut adalah **satu-satunya sumber kebenaran** untuk aturan ini dan berlaku **secara absolut**, bukan hanya saat tool failure.
2. **Conscious Disregard (Pengabaian Sadar):** JANGAN PERNAH mengabaikan regresi atau inkonsistensi yang terlihat pada *codebase*. Pemikiran seperti "Ini tidak penting sekarang, kerjakan saja tugas utamanya dulu" adalah **PELANGGARAN FATAL**.

### ✅ Tindakan yang Diwajibkan (Best Practices)
1. **Fix Forward (Perbaiki ke Depan):** Jika terjadi kerusakan file akibat kesalahan tool, perbaiki baris yang rusak tersebut secara spesifik dan teliti menggunakan tool pengedit teks. Jangan me-reset keseluruhan file ke *state* Git sebelumnya kecuali Anda 100% yakin tidak ada pekerjaan yang hilang — dan bahkan dalam kasus itu, **wajib tampilkan `git diff` kepada user dan tunggu konfirmasi** sebelum mengeksekusi.
2. **Zero Tolerance for Regressions (Toleransi Nol untuk Regresi):** Jika saat membaca file Anda menyadari bahwa arsitektur, penamaan (misal: nama komponen/variabel), atau struktur kembali ke versi lama yang salah, **HENTIKAN** tugas saat ini. Perbaiki regresi tersebut seketika itu juga demi menghormati kesepakatan arsitektur sebelumnya.
3. **State Integrity > Task Momentum (Integritas State lebih utama dari Momentum Tugas):** Mempertahankan kebenaran arsitektur secara keseluruhan selalu lebih penting daripada buru-buru menyelesaikan satu *micro-task*.

## Prinsip Fail-Fast & Larangan Menyembunyikan Galat (No Error Masking)

> 💥 **FAIL-FAST OVER SILENT SURVIVAL:**
> Sebuah aplikasi yang melempar *runtime error* secara gamblang jauh lebih berharga daripada aplikasi yang tetap berjalan (*silent survival*) namun memakan data palsu/mock secara sembunyi-sembunyi.

1. **Dilarang Membungkam Galat (No Silent Catch):**
   - **TERLARANG:** Menangkap galat (`catch (e) {}`) lalu secara diam-diam mengembalikan nilai dummy/mock default tanpa melempar ulang galat (*rethrow*) atau menyajikan status error yang transparan.
   - **WAJIB:** Jika dependensi eksternal (API endpoint, query database, layanan cloud, variabel lingkungan) tidak tersedia atau mengembalikan kegagalan, biarkan sistem melempar galat eksplisit atau tampilkan *Error State UI* yang jujur kepada pengguna.
2. **Larangan Defensive Mocking Saat Debugging:**
   - Ketika Anda mendapati kegagalan saat menjalankan aplikasi atau *test*, Anda **DILARANG KERAS** menyisipkan data statis/hardcoded sebagai "penambal darurat" agar aplikasi terlihat berjalan.
   - Telusuri dan perbaiki akar masalahnya (*root cause*): periksa file konfigurasi `.env`, validasi integrasi jaringan, atau sesuaikan penanganan galatnya secara struktural.
3. **Kewajiban Pesan Galat Deskriptif:**
   - Setiap *runtime error* yang dilempar wajib memberikan konteks spesifik: apa yang hilang, nilai apa yang diekspektasikan, dan file/variabel apa yang harus diperiksa developer (contoh: `"[Config Error] NEXT_PUBLIC_API_URL is required but was not provided in .env.local"` atau `"[Config Error] DATABASE_URL is missing"`).
