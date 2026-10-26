# Edge Case Atlas: Peta Skenario Batas & Kasus Ekstrem

> **NAVIGASI PENGUJIAN EKSTREM (O-08):**
> Saat menjalankan siklus Hunt Loop, AI Agent WAJIB menelusuri setiap sumbu dalam Atlas ini untuk menggali minimal 10 skenario batas ekstrem konkret pada tahap Chatter sebelum menyatakan kode selesai.
>
> Pengujian fungsional biasa (happy path) hanya menguji apa yang diharapkan berjalan. Edge Case Atlas memaksa verifikasi skenario tak terduga yang sering lolos ke lingkungan produksi.

---

## 1. Alur Kerja Eksplorasi Kasus Batas (Hunt Loop Integration)

1. **Deklarasi Terbuka (Chatter):** Sebutkan minimal 10 skenario ekstrem konkret secara terbuka di terminal/chat sebelum pengujian dimulai. Dilarang hanya menyatakan kalimat umum seperti *"mungkin ada edge cases"* tanpa merinci kasus spesifik.
2. **Reproduksi & Sematkan (Reproduce & Pin):** Buktikan kegagalan setiap skenario secara nyata, lalu sematkan (pin) kasus yang terbukti menjadi unit test otomatis permanen yang mematuhi aturan O-07 pada [testing.md](./testing.md) (terbukti gagal di kode lama, lulus setelah perbaikan).
3. **Pencatatan Bukti:** Dokumentasikan ringkasan sumbu yang dieksplorasi ke dalam seksi `AI Execution Log & Output` pada tiket tugas (butir O-08) atau laporan audit [hunt-findings.md](../../global-docs/templates/hunt-findings.md).

---

## 2. Sembilan Sumbu Kasus Ekstrem Universal

Telusuri 9 sumbu ini untuk setiap perubahan kode, modul, atau antarmuka:

### 2.1 Who (Aktor & Identitas)
Meneliti siapa yang berinteraksi dengan sistem dan konteks otentikasinya:
- **Peran & Status:** Pengguna terotentikasi, tamu (guest), hak akses berjenjang (admin vs member), akun dinonaktifkan atau dihapus di tengah sesi aktif.
- **Dua Peran pada Subjek Sama:** Pembeli mengirimkan tiket hadiah ke rekan, namun membuka tautan klaim di peramban pribadinya yang sedang login sebagai pembeli (konflik validasi kepemilikan).
- **Konkurensi Aktor:** Dua pengguna berbeda mengedit atau mengklaim record yang sama pada detik yang sama.
- **Kepemilikan Terinferensi vs Tersimpan:** Sistem mengasumsikan pembayar adalah pemilik. Perubahan pada field pembayaran dapat memindahkan kepemilikan objek secara senyap dan tidak disengaja.
- **Kredensial Tertukar:** Pengguna memegang token, tautan klaim, atau berkas yang ditujukan untuk entitas lain.

### 2.2 How Much (Jumlah, Nilai & Kuantitas)
Meneliti batasan kuantitas dan kalkulasi angka:
- **Batas Numerik:** Nilai nol (0), satu (1), batas maksimum tipe data, integer overflow, bilangan negatif, desimal tak berhingga.
- **Pembagian Pecahan:** Membagi nominal Rp100.000 menjadi 3 termin cicilan (33.334 + 33.333 + 33.333). Pastikan alokasi sisa selisih presisi dan total akhir tetap seimbang.
- **Diskon & Saldo:** Nilai diskon melebihi total tagihan, saldo pas-pasan menutup transaksi, voucher bernilai negatif.
- **Batas Total vs Kuota Baris:** Kapasitas aula 200 kursi menjual 3 kategori tiket yang masing-masing dibatasi 200 kursi (risiko overselling hingga 600 tiket).
- **Representasi Nilai Nol:** Nilai angka `0` atau array kosong yang keliru ditampilkan sebagai status "Penuh" atau "Galat Sistem".

### 2.3 When (Waktu, Kronologi & Durasi)
Meneliti kronologi peristiwa dan penanganan waktu:
- **Batas Temporal:** Transaksi sebelum jam buka, tepat di detik penutupan, melintasi tengah malam, tahun kabisat, pergeseran Daylight Saving Time (DST).
- **Normalisasi Zona Waktu:** Perbedaan zona waktu lokal pengguna vs server. Wajib dinormalisasi ke format UTC atau Epoch sesuai [database.md](./database.md).
- **Peristiwa di Luar Urutan (Out of Order):** Notifikasi webhook pembayaran tiba setelah pesanan kedaluwarsa (expired). Uang masuk namun pesanan hangus tanpa refund otomatis.
- **Duplikasi Kejadian:** Webhook yang di-retry otomatis oleh payment gateway; penekanan tombol pembayaran ganda (double-click) dalam rentang milidetik.
- **Layar Usang (Stale State):** Form pemesanan dibuka 2 jam lalu, tombol Submit baru ditekan setelah stok barang habis dibeli pengguna lain.

### 2.4 What State (Siklus Hidup & Status Objek)
Meneliti variasi siklus hidup entitas bisnis:
- **Siklus Status Menyeluruh:** Uji seluruh variasi status di luar happy path (`Pending`, `Expired`, `Cancelled`, `Partially Refunded`, `Overdue`, `Archived`), bukan hanya `Draft` dan `Published`.
- **Eksibisi Informasi Berdasarkan State:** Pesanan yang belum dibayar dilarang merender QR code akses masuk arena yang dapat disalahgunakan layaknya tiket resmi.
- **Kegagalan Parsial (Partial Failures):** Penulisan database lokal berhasil namun panggilan API eksternal gagal. Cegah kondisi data terpotong tanpa konfirmasi akhir.
- **Data Warisan Skema Lama:** Memvalidasi bagaimana catatan data yang dibuat oleh skema database versi lama diproses oleh kode baru.

### 2.5 Where (Lingkungan & Mesin Eksekusi)
Meneliti perbedaan runtime dan infrastruktur:
- **Perbedaan Engine Uji vs Produksi:** SQLite mengizinkan tipe data longgar yang ditolak ketat oleh PostgreSQL/MySQL; batas panjang identifier indeks database.
- **Karakteristik Klien:** Layar sempit 320px vs monitor ultra-wide, orientasi lanskap, dark mode, koneksi lambat 3G, dan mode offline total sesuai [ui-and-assets.md](./ui-and-assets.md).
- **Proses Berumur Panjang (Long-Lived Workers):** Background worker atau cron yang mempertahankan variabel memori antar-eksekusi (risiko memory leak atau state tercemar).
- **Isolasi Sesi & Host:** Perbedaan penanganan cookie dan session antara `localhost`, `127.0.0.1`, dan domain produksi.

### 2.6 Which Input (Bentuk, Tipe & Karakter Data)
Meneliti variasi bentuk payload dan masukan:
- **Ekstrem Karakter:** String kosong, spasi murni, teks ribuan karakter, karakter khusus Unicode, emoji ganda, bahasa teks kanan-ke-kiri (RTL), tag HTML/script injection, angka dengan leading zeros (`00123`).
- **Nilai Kosong vs Absen:** Perbedaan semantik antara `null` (disetel kosong), `undefined` (properti absen), dan angka `0`. Wajib mematuhi Fail-Fast Policy pada [coding.md](./coding.md) (dilarang silent fallback).
- **Format Berkas Variatif:** Dokumen lampiran dari aplikasi versi lama, berkas CSV dengan pemisah titik koma alih-alih koma.

### 2.7 How Many (Volume & Skala Data)
Meneliti skala volume pemrosesan:
- **Kuantitas Kumpulan:** Koleksi kosong (`[]`), elemen tunggal (`[item]`), kumpulan data tepat satu halaman paginasi, dan data melebihi batas halaman.
- **Masalah Kueri N+1:** Kueri database yang berlipat ganda secara eksponensial seiring bertambahnya baris data yang ditampilkan.
- **Batas Rate Limit Kolektif:** Kumpulan pengguna di balik satu alamat IP kantor atau reverse proxy yang sama memicu pembatasan kuota (rate limiting).

### 2.8 What Else Changes (Efek Samping & Mutasi Sistem)
Meneliti dampak sekunder dari suatu aksi:
- **Notifikasi & Komunikasi:** Email atau pesan terkirim tepat satu kali, kepada penerima yang benar, dan dalam bahasa yang sesuai.
- **Keseimbangan Buku Besar:** Setiap pengurangan saldo wajib memiliki pencatatan penambahan di sisi lawan; pembalikan transaksi (reversal) mengembalikan nilai secara seimbang.
- **Mutasi Berkas & Kamus Bahasa:** Perubahan state mematuhi [safe-file-operations.md](./safe-file-operations.md). Sinkronisasi teks baru ke seluruh kamus bahasa i18n sesuai [localization.md](./localization.md).
- **Jejak Audit:** Aksi sensitif tercatat di log audit keamanan tanpa membocorkan kredensial atau data privat.

### 2.9 What the User Will Actually Do (Perilaku Nyata Pengguna)
Meneliti interaksi acak manusia asli:
- **Interaksi Acak & Barbar:** Menekan tombol Back peramban di tengah proses pembayaran, refresh halaman berulang kali, membuka aplikasi di dua tab bersamaan, menutup form di tengah proses lalu kembali keesokan harinya.
- **Verifikasi Teks Harfiah:** Seluruh pesan kesalahan, instruksi form, dan status yang tampil di layar wajib mencerminkan realitas kondisi sistem yang sebenarnya.

---

## 3. Heuristik Penemu Kasus (Pemicu Eksplorasi)

Gunakan 4 teknik ini jika membutuhkan inspirasi menggali 10 skenario batas pada tahap Chatter:

### 3.1 Analisis Nilai Batas (Boundary Value Analysis)
Uji variasi nilai ekstrem: $\text{min}-1$, $\text{min}$, $\text{min}+1$, $\text{max}-1$, $\text{max}$, dan $\text{max}+1$. Terapkan pada angka numerik, panjang karakter string, batas ukuran upload berkas, durasi waktu kedaluwarsa, dan kuota keranjang belanja.

### 3.2 Kombinasi Silang Antar-Sumbu (Pairwise Combinations)
Silangkan dua sumbu berbeda untuk mendeteksi bug interaksi: silangkan aktor dengan state; silangkan state dengan zona waktu; silangkan input dengan mode offline. Mayoritas bug fatal terjadi akibat pertemuan 2 kondisi spesifik yang berjalan serentak.

### 3.3 Asersi Metamorfik (Metamorphic Assertions)
Jika nilai kembalian eksak sulit diprediksi, ujilah relasi logisnya: total cicilan wajib sama dengan total tagihan; filter pencarian yang dipersempit tidak boleh menghasilkan data yang lebih banyak daripada filter awal; operasi pengurutan dua kali menghasilkan output yang identik dengan pengurutan satu kali.

### 3.4 Injeksi Kegagalan Dependensi (Failure Injection)
Simulasikan skenario ketika dependensi eksternal gagal merespons: gateway pembayaran timeout, server email menolak koneksi, atau disk penyimpanan penuh sesaat setelah transaksi tercatat.
