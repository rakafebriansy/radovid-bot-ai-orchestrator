# Safe File Operations: Prohibited Destructive Commands

> **[FATAL]** Mengeksekusi perintah terlarang dalam dokumen ini menghapus kesempatan pengguna meninjau (review) kode dan setara dengan perusakan sepihak.
> **[PRIORITAS REGULASI]** Dokumen ini memperluas dan menganulir klausul terkait pada [error-handling.md](./error-handling.md). Berlaku mutlak pada seluruh siklus eksekusi AI Agent. Jika terjadi kontradiksi aturan, dokumen ini adalah rujukan tertinggi.

---

## 1. Protokol Otorisasi Operasi: Read-Only vs Mutasi State

AI Agent DILARANG meminta izin terminal (permission prompt) untuk mutasi state sebelum memaparkan rencana implementasi secara transparan.

### 1.1 Operasi Read-Only (Inspeksi Pasif)
Operasi pembacaan dokumen, inspeksi direktori, atau penelusuran konteks tanpa mengubah berkas, skema, maupun data. AI Agent **DIPERBOLEHKAN** meminta izin atau mengeksekusi secara otonom untuk investigasi:
- **Navigasi & Struktur Direktori:** `ls`, `dir`, `pwd`, `tree`, `find`, `which`, `graphify query`
- **Pembacaan Berkas:** `head`, `tail`, `cat`, `less`, `sed -n` (tanpa `-i`), `awk` (tanpa redirect overwrite), `grep`, `rg`, `view_file`, `list_dir`, `read_url_content`
- **Inspeksi Git:** `git status`, `git log`, `git diff`, `git branch -a`, `git show`
- **Inspeksi Database:** Kueri `SELECT` non-locking, `EXPLAIN`, inspeksi skema baca

### 1.2 Operasi Mutasi State (State Mutation)
Tindakan yang mengubah berkas, basis kode, konfigurasi, skema database (DDL), atau data:
- **Manipulasi Berkas & Direktori:** Penambahan (`write_to_file`, `touch`), pengeditan (`replace_file_content`, `multi_replace_file_content`), penghapusan (`rm`), pemindahan/penyalinan (`mv`, `cp`), pembuatan direktori (`mkdir`).
- **Konfigurasi & Dependensi:** Instalasi paket (`npm install`, `pip install`, `cargo add`), mutasi `.env`, modifikasi konfigurasi (`package.json`, `tsconfig.json`, `docker-compose.yml`).
- **Basis Data & Skema:** Migrasi DDL (`CREATE`, `ALTER`, `DROP`), seeding data, mutasi DML massal (`INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`).
- **Layanan & Runtime:** Build script penghasil artefak, inisiasi service container, restart peladen.

### 1.3 Prasyarat Otorisasi Sebelum Modifikasi (Tiga Tahap Mutlak)
AI Agent DILARANG memunculkan dialog izin eksekusi terminal untuk mutasi state sebelum menuntaskan 3 tahap berurutan:
1. **Paparkan Rencana (*Implementation Plan*):** Rincikan berkas yang dibuat/diubah/dihapus, perintah yang dijalankan, dan konsekuensi teknisnya.
2. **Tampilkan Pratinjau (*Diff Preview*):** Tunjukkan cuplikan kode, diff terfokus, atau skrip yang akan diterapkan.
3. **Tunggu Persetujuan Eksplisit:** Ajukan izin eksekusi terminal hanya setelah pengguna menyetujui rencana dan pratinjau.

---

## 2. Katalog Perintah Destruktif Terlarang Mutlak

AI Agent DILARANG mengeksekusi perintah berikut dalam kondisi apa pun tanpa konfirmasi tertulis eksplisit pengguna:

### 2.1 Skrip Transformasi Massal (Bulk Mutation Scripts)
Contoh terlarang:
```bash
node fix-all.js
python migrate.py
bash bulk-rename.sh
ts-node transform.ts
```
- **Rasional:** Mengubah banyak berkas sekaligus tanpa memberikan visibilitas diff per-berkas sebelum eksekusi.
- **Alternatif Wajib:** Gunakan `replace_file_content` atau `multi_replace_file_content` per berkas secara granular. Jika modifikasi massal tidak terhindarkan, susun dokumen rencana implementasi berisi rincian diff tiap berkas dan tunggu persetujuan pengguna.

### 2.2 Manipulasi Berkas In-Place (sed -i, perl -i, awk redirect, tee)
Contoh terlarang:
```bash
sed -i 's/foo/bar/g' file.ts          # sed in-place
sed -i '' 's/old/new/' **/*.js        # sed in-place (macOS)
perl -i -pe 's/foo/bar/g' file.ts     # perl in-place
awk '{gsub(/foo/,"bar"); print}' f > f # overwrite via redirect ke berkas yang sama
tee file.ts <<< "new content"         # overwrite via tee
```
- **Rasional:** Memodifikasi berkas langsung di disk tanpa mekanisme diff preview kepada pengguna.
- **Alternatif Wajib:** Gunakan `replace_file_content` atau `multi_replace_file_content`. Jalankan `sed` atau `perl` tanpa flag `-i` untuk inspeksi output ke stdout, lalu terapkan perubahan via tool pengedit berkas terisolasi.

### 2.3 Pembatalan Modifikasi Git Tanpa Konfirmasi (git checkout & git restore per-file)
Contoh terlarang:
```bash
git checkout -- src/index.ts
git checkout HEAD src/
git checkout <branch> -- path/to/file
git restore src/index.ts
git restore --staged src/
git restore --source=HEAD src/config.ts
```
- **Rasional:** Menghapus permanen perubahan unstaged maupun staged pada berkas target tanpa persetujuan pengguna. *(Pengecualian: `git checkout <branch>` murni untuk navigasi antar-cabang diperbolehkan).*
- **Alternatif Wajib:** Jalankan `git diff <file>` untuk meninjau perubahan, sampaikan bagian yang akan hilang, gunakan `view_file`, dan tunggu konfirmasi eksplisit pengguna sebelum membatalkan modifikasi.

### 2.4 Pembersihan Berkas Untracked (git clean)
Contoh terlarang:
```bash
git clean -f
git clean -fd
git clean -fdx
```
- **Rasional:** Menghapus permanen berkas untracked tanpa riwayat pemulihan Git.
- **Alternatif Wajib:** Jalankan dry-run `git clean -n` terlebih dahulu untuk memaparkan daftar berkas yang akan terhapus, lalu minta persetujuan eksplisit pengguna.

### 2.5 Penghapusan Git Stash (git stash drop & clear)
Contoh terlarang:
```bash
git stash drop
git stash drop stash@{0}
git stash clear
```
- **Rasional:** Menghapus snapshot stash secara permanen tanpa opsi pemulihan.
- **Alternatif Wajib:** Tampilkan konten stash dengan `git stash show -p stash@{N}`, paparkan detailnya, dan tunggu konfirmasi eksplisit sebelum eksekusi penghapusan.

### 2.6 Penimpaan Berkas Total (write_to_file Overwrite: true)
Contoh terlarang:
```python
write_to_file(path="src/config.ts", overwrite=True, content="...")
```
- **Rasional:** Menimpa seluruh isi berkas sekaligus tanpa visualisasi diff terfokus, berisiko menghapus kode penting di luar cakupan tugas.
- **Alternatif Wajib:** Gunakan `replace_file_content` atau `multi_replace_file_content` untuk perubahan spesifik. Gunakan `write_to_file` dengan `Overwrite: true` HANYA untuk berkas baru atau jika penggantian total berkas telah disetujui dalam rencana implementasi.

---

## 3. Tata Kelola Reversibilitas & Resolusi Situasi Khusus

### 3.1 Empat Pilar Tindakan Reversible
1. **Show Before Apply:** Paparkan rencana atau diff kepada pengguna sebelum eksekusi mutasi.
2. **Reversibility:** Utamakan operasi yang dapat di-rollback via Git staging, branch terisolasi, atau pencadangan eksplisit.
3. **Granularitas:** Terapkan modifikasi satu berkas per langkah; hindari script transformasi massal tak terkendali.
4. **Transparansi:** Berikan detail baris kode yang diubah beserta rasional dampaknya secara objektif.

### 3.2 Protokol Resolusi Deadlock & Situasi Buntu
Jika perintah destruktif tampaknya merupakan satu-satunya solusi teknis:
1. **[TERLARANG]** Jangan eksekusi perintah destruktif secara sepihak.
2. Jelaskan kendala teknis dan penyebab pendekatan standar tidak berfungsi.
3. Jalankan dan paparkan hasil simulasi / dry-run (`git clean -n`, `git stash show -p`, `sed` tanpa `-i`, `git diff`).
4. Ajukan opsi mitigasi yang aman dan dapat dipulihkan (*reversible*).
5. Tunggu instruksi tertulis eksplisit dari pengguna sebelum mengeksekusi tindakan apa pun.
