# Safe File Operations — Prohibited Destructive Commands

> 🔴 **PELANGGARAN FATAL:** Mengeksekusi perintah-perintah yang dilarang di dokumen ini adalah
> **PELANGGARAN FATAL** yang setara dengan menghapus pekerjaan pengguna secara sepihak. Agent yang
> melanggar rule ini dianggap telah mengorbankan kepercayaan pengguna dan integritas proyek.

## Tujuan

Rule ini melindungi user dari kehilangan kesempatan me-review perubahan file sebelum diterapkan.
Perintah-perintah di bawah bersifat **destruktif dan tidak reversible secara langsung**, sehingga
berpotensi menimpa atau menghapus konten tanpa review yang memadai.

> ⚠️ **Hubungan dengan `error-handling.md`:** Dokumen `error-handling.md` menyebut larangan
> `git checkout <file>` dan `git restore <file>` dalam konteks *tool failure*. Dokumen ini
> **memperluas dan menggantikan** cakupan tersebut — larangan berlaku **secara absolut**, bukan
> hanya saat tool failure. Jika ada kontradiksi, **dokumen ini yang menang**.

---

## ⚖️ Protokol Permintaan Izin: Read-Only vs Modifikasi State (State Mutation Protocol)

> 🛡️ **PRINSIP TRANSPARANSI PERUBAHAN STATE:**
> AI Agent **DILARANG KERAS** langsung meminta *permission* untuk perintah yang memodifikasi *state* tanpa memaparkan rencana terlebih dahulu.

### 1. Operasi Read-Only (Boleh Langsung Meminta Izin / Dijalankan Mandiri)
- **Definisi:** Segala perintah terminal atau pemanggilan *tool* yang bersifat inspeksi pasif, membaca isi dokumen, memindai direktori, atau mengumpulkan konteks sistem tanpa mengubah status (*state*), isi berkas, skema, atau data apa pun.
- **Daftar Operasi Read-Only yang Diizinkan Langsung:**
  - **Navigasi & Pelacakan Direktori:** `ls`, `dir`, `pwd`, `tree`, `find` (pencarian), `which`, `graphify query`
  - **Pembacaan Konten Berkas:** `head`, `tail`, `cat`, `less`, `sed -n` (mode cetak/read-only tanpa `-i`), `awk` (tanpa *file redirection* / *overwrite*), `grep`, `rg`, `view_file`, `list_dir`, `read_url_content`
  - **Inspeksi Status Kontrol Versi (VCS):** `git status`, `git log`, `git diff`, `git branch -a`, `git show`
  - **Inspeksi Basis Data:** Kueri `SELECT` non-locking, `EXPLAIN`, inspeksi skema baca (*read-only schema inspection*)
- **Kebijakan Izin:** AI Agent **DIPERBOLEHKAN** langsung meminta *permission* eksekusi (atau langsung mengeksekusi secara otonom jika tool mendukung) untuk operasi-operasi *read-only* ini demi kelancaran pengumpulan konteks awal.

### 2. Operasi Modifikasi State (DILARANG LANGSUNG MEMINTA PERMISSION TANPA RENCANA)
- **Definisi:** Segala tindakan atau perintah yang berpotensi mengubah status (*state mutation*), memodifikasi *codebase*, menyisipkan berkas, menghapus berkas, mengubah konfigurasi, merombak skema basis data (DDL), atau memanipulasi data produksi/tabel (DML).
- **Cakupan Operasi Modifikasi State:**
  - **Manipulasi Berkas & Struktur:** Menambah file baru (`write_to_file`, `touch`), memodifikasi file (`replace_file_content`, `multi_replace_file_content`), menghapus file (`rm`), memindahkan/menggandakan file (`mv`, `cp`), membuat direktori (`mkdir`).
  - **Konfigurasi & Dependensi:** Menginstal paket (`npm install`, `pip install`, `cargo add`, dll.), mengubah file environment (`.env`), mengedit berkas konfigurasi sistem (`package.json`, `tsconfig.json`, `docker-compose.yml`).
  - **Basis Data & Skema:** Menjalankan migrasi DDL (`CREATE`, `ALTER`, `DROP`), menyisipkan data *seed*, mengubah tabel (INSERT/UPDATE/DELETE/TRUNCATE).
  - **Lingkungan & Layanan:** Menjalankan *build script* yang menghasilkan artefak baru, menjalankan *service container*, atau merestart peladen.
- **Aturan Mutlak Permintaan Permission Untuk Menjalankan Command Terminal:**
  AI Agent **DILARANG KERAS** langsung melontarkan dialog izin eksekusi (*permission prompt*) untuk operasi modifikasi state tanpa menyelesaikan 3 prasyarat berurutan berikut:
  1. **Paparkan Rencana (*Implementation Plan*) Secara Transparan:** Jelaskan berkas apa saja yang akan dibuat, diubah, atau dihapus, perintah modifikasi apa yang akan dijalankan, dan apa dampaknya.
  2. **Tampilkan Pratinjau (*Diff Preview*):** Tunjukkan pratinjau potongan kode atau skrip yang akan diterapkan.
  3. **Tunggu Persetujuan Rencana dari Pengguna:** Hanya setelah pengguna secara eksplisit menyetujui rencana tersebut, barulah AI diizinkan meminta *permission* eksekusi untuk perintah modifikasi state terkait.

---

## ❌ Perintah yang DILARANG TOTAL

Agent **TIDAK BOLEH** menjalankan perintah-perintah berikut dalam kondisi apapun.

---

### 1. Script Perubahan Massal (JavaScript, Python, Shell, dll.)

Contoh yang dilarang:
```
node fix-all.js
python migrate.py
bash bulk-rename.sh
ts-node transform.ts
```

**Alasan**: Script semacam ini memodifikasi banyak file sekaligus tanpa memberikan kesempatan
user melihat perubahan per-file secara detail sebelum diterapkan.

**Alternatif wajib**:
- Gunakan tool `replace_file_content` atau `multi_replace_file_content` untuk setiap file secara eksplisit
- Jika perubahan massal benar-benar diperlukan, buat terlebih dahulu `implementation_plan.md`
  yang menjabarkan setiap file yang akan berubah beserta diff-nya, lalu **tunggu persetujuan
  eksplisit user** sebelum mengeksekusi satu baris pun

---

### 2. In-place File Edit (`sed -i`, `perl -i`, `awk` overwrite, `tee` destruktif)

Contoh yang dilarang:
```bash
sed -i 's/foo/bar/g' file.ts          # sed in-place
sed -i '' 's/old/new/' **/*.js        # sed in-place (macOS)
perl -i -pe 's/foo/bar/g' file.ts     # perl in-place
awk '{gsub(/foo/,"bar"); print}' f > f # overwrite via redirect ke file yang sama
tee file.ts <<< "new content"         # overwrite via tee
```

**Alasan**: Semua varian di atas memodifikasi file langsung di disk tanpa menampilkan preview
perubahan kepada user terlebih dahulu.

**Alternatif wajib**:
- Gunakan tool `replace_file_content` atau `multi_replace_file_content` yang transparan dan
  menampilkan diff sebelum diterapkan
- Untuk preview: jalankan `sed 's/foo/bar/g' file.ts` (tanpa `-i`) atau `perl -pe '...' file.ts`
  untuk melihat output, kemudian terapkan via file edit tool — **bukan** via flag in-place

---

### 3. `git checkout <file>` (Membuang Perubahan Working Tree)

Contoh yang dilarang:
```bash
git checkout -- src/index.ts
git checkout HEAD src/
git checkout <branch> -- path/to/file
```

**Alasan**: Perintah ini **menghapus permanen** perubahan yang belum di-commit pada file yang
ditarget tanpa konfirmasi user. Perhatikan: `git checkout <branch>` (berpindah branch tanpa
path) **tidak termasuk** larangan ini — yang dilarang adalah varian yang menyertakan path/file.

**Alternatif wajib**:
- Tampilkan `git diff <file>` dan tunjukkan hasilnya kepada user
- Jelaskan secara eksplisit konten apa yang akan hilang permanen
- Gunakan `view_file` untuk menunjukkan isi file saat ini vs versi target
- Tunggu konfirmasi eksplisit dari user sebelum melanjutkan

---

### 4. `git restore <file>` (Membuang Perubahan Working Tree)

Contoh yang dilarang:
```bash
git restore src/index.ts
git restore --staged src/
git restore --source=HEAD src/config.ts
```

**Alasan**: Identik dengan `git checkout <file>` — perubahan yang belum di-commit akan hilang
secara permanen dan tidak bisa di-undo.

**Alternatif wajib**:
- Jalankan `git diff <file>` terlebih dahulu dan tampilkan hasilnya kepada user
- Jelaskan secara eksplisit konten apa yang akan hilang
- Tunggu konfirmasi eksplisit dari user sebelum melanjutkan

---

### 5. `git clean` (Menghapus File Untracked)

Contoh yang dilarang:
```bash
git clean -f
git clean -fd
git clean -fdx
```

**Alasan**: Menghapus permanen file-file yang belum di-track Git (untracked files). File yang
belum pernah di-`git add` tidak bisa dikembalikan — tidak ada di staging maupun history.

**Alternatif wajib**:
- Tampilkan `git clean -n` (dry run) kepada user terlebih dahulu untuk menunjukkan file apa yang
  akan dihapus
- Tunggu konfirmasi eksplisit dari user

---

### 6. `git stash drop` / `git stash clear` (Menghapus Stash)

Contoh yang dilarang:
```bash
git stash drop
git stash drop stash@{0}
git stash clear
```

**Alasan**: Menghapus permanen perubahan yang tersimpan di stash. Setelah di-drop, stash tidak
bisa dipulihkan.

**Alternatif wajib**:
- Tampilkan isi stash dengan `git stash show -p stash@{N}` terlebih dahulu
- Tunjukkan hasilnya kepada user dan tunggu konfirmasi eksplisit sebelum menghapus

---

### 7. `write_to_file` dengan `Overwrite: true` (Tool Native Agent)

Contoh yang dilarang:
```
write_to_file(path="src/config.ts", overwrite=True, content="...")
```

**Alasan**: Tool ini menimpa **seluruh isi file** sekaligus — lebih destruktif dari `sed -i`
karena menghapus konten yang tidak termasuk dalam perubahan yang dimaksud. Tidak ada diff yang
ditampilkan sebelum eksekusi.

**Alternatif wajib**:
- Gunakan `replace_file_content` atau `multi_replace_file_content` untuk mengedit bagian
  spesifik yang perlu diubah
- Gunakan `write_to_file` dengan `Overwrite: true` **hanya** untuk file yang baru dibuat atau
  file yang **seluruh isinya** memang ingin diganti secara sadar dan sudah disetujui user
- Jika ragu, tampilkan diff rencana perubahan kepada user terlebih dahulu

---

## ✅ Prinsip Umum: Utamakan Operasi Reversible

Ketika ragu, selalu pilih pendekatan yang:
1. **Show before apply** — tampilkan perubahan kepada user sebelum diterapkan
2. **Reversible** — gunakan Git staging, branch baru, atau backup eksplisit agar bisa di-undo
3. **Granular** — satu file pada satu waktu, bukan bulk sekaligus
4. **Transparan** — user dapat melihat dengan jelas apa yang akan berubah

---

## Cara Menangani Edge Case

Jika ada situasi di mana perintah-perintah di atas tampaknya merupakan satu-satunya solusi praktis:

1. **Jangan jalankan perintahnya**
2. Jelaskan kepada user mengapa Anda tidak bisa melanjutkan dengan cara yang ada
3. Tampilkan dry run / preview output jika tersedia (misal: `git clean -n`, `sed` tanpa `-i`)
4. Tawarkan alternatif reversible yang tersedia
5. Tunggu instruksi eksplisit dari user sebelum mengambil tindakan apapun
