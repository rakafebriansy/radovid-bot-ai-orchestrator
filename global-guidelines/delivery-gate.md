# Delivery Gate: Gerbang Verifikasi Penyerahan Tugas

> **HUKUM BESI PENYERAHAN TUGAS (HANDOVER DISCIPLINE):**
> Dokumen ini adalah gerbang pemeriksaan wajib (mandatory exit gate) sebelum AI Agent mengubah status tiket tugas menjadi `Done`, menyatakan pekerjaan selesai, atau menyerahkan hasil kepada pengguna. 
>
> **Jika ditemukan satu saja butir pemeriksaan yang gagal (FAIL), pekerjaan DILARANG diserahkan!** Perbaiki kegagalan tersebut, jalankan ulang pengujian, dan pastikan seluruh gerbang berstatus **PASS** dengan bukti nyata.

---

## 1. Filosofi & Peran Delivery Gate
Delivery Gate berfungsi sebagai **pintu keluar tunggal (consolidated exit gate)** dari seluruh pedoman di `global-guidelines/`. Tujuannya memastikan pengguna tidak akan pernah menemukan masalah atau bug yang seharusnya dapat dicegah dan ditemukan sendiri oleh AI. Ringkasan pesan chat biasa atau commit message tidak dapat menggantikan verifikasi gerbang ini.

---

## 2. Empat Pilar Gerbang Pemeriksaan (Delivery Gate Pillars)
Setiap penyelesaian tiket atau instruksi implementasi wajib mengevaluasi 4 pilar berikut secara berurutan:

### 2.1 Gerbang 1: Kepatuhan Standar Rekayasa Kode (Code & Data Hygiene)
Seluruh pertanyaan pemeriksaan kode di bawah ini **WAJIB DIJAWAB "TIDAK"**:

| Kode | Pertanyaan Pemeriksaan | Aturan Rujukan | Syarat Mutlak |
|---|---|---|---|
| **CG-1** | Apakah ada komentar teks baru yang disisipkan di dalam source code? | [coding.md](./coding.md) (1.1) | **TIDAK.** Wajib zero-comment pada kode baru; kode harus self-documenting. |
| **CG-2** | Apakah ada silent fallback, nilai dummy default, atau data mock di kode produksi? | [coding.md](./coding.md) (2.1) | **TIDAK.** Wajib fail-fast dan membaca sumber data dinamis riil. |
| **CG-3** | Apakah ada angka mentah (magic numbers) tanpa named constants atau Enum? | [coding.md](./coding.md) (2.2) | **TIDAK.** Seluruh angka bernilai fungsional/bisnis wajib diabstraksikan. |
| **CG-4** | Apakah ada eksekusi DDL langsung pada live database tanpa berkas migrasi? | [database.md](./database.md) (2.1) | **TIDAK.** Seluruh perubahan skema wajib via file migrasi terstruktur. |
| **CG-5** | Apakah ada operasi file destruktif atau mutasi state tanpa persetujuan pengguna? | [safe-file-operations.md](./safe-file-operations.md) | **TIDAK.** Wajib memaparkan Implementation Plan dan disetujui pengguna. |

---

### 2.2 Gerbang 2: Integritas Pengujian & Batas Aman (Hard Gate Testing)
Seluruh pertanyaan keamanan pengujian di bawah ini **WAJIB DIJAWAB "TIDAK"**:

| Kode | Pertanyaan Pemeriksaan | Aturan Rujukan | Syarat Mutlak |
|---|---|---|---|
| **HG-1** | Apakah ada tes atau skrip yang dijalankan terhadap database produksi/development riil? | [testing.md](./testing.md) (O-01, O-02) | **TIDAK.** Wajib menggunakan sandbox, mock terisolasi, atau database khusus testing. |
| **HG-2** | Apakah ada klaim status `PASS` berdasarkan tebakan, asumsi, atau tanpa bukti terminal? | [testing.md](./testing.md) (O-03) | **TIDAK.** Setiap klaim PASS wajib menyertakan log command dan output terminal konkret. |
| **HG-3** | Apakah ada test case yang dihapus, di-skip, atau dilonggarkan asersinya agar hijau? | [testing.md](./testing.md) (O-05) | **TIDAK.** Integritas tes dilarang dikompromikan (anti-overfitting). |
| **HG-4** | Apakah ada file sementara, kredensial uji, atau data sampah tertinggal di repositori? | [testing.md](./testing.md) (O-06) | **TIDAK.** Repositori wajib bersih kembali dari artefak pengujian. |

---

### 2.3 Gerbang 3: Kedalaman Eksplorasi & Validasi (Must Probe & Hunt Loop)
Seluruh pertanyaan kedalaman pengujian di bawah ini **WAJIB DIJAWAB "YA"**:

| Kode | Pertanyaan Pemeriksaan | Aturan Rujukan | Syarat Mutlak |
|---|---|---|---|
| **MP-1** | Jika perbaikan bug: apakah tes baru terbukti GAGAL di kode lama sebelum fix diterapkan? | [testing.md](./testing.md) (O-07) | **YA.** Wajib membuktikan kegagalan terjadi karena akar bug tersebut. |
| **MP-2** | Apakah seluruh 9 sumbu Edge Case Atlas telah dieksplorasi (atau dideklarasikan alasannya jika dilewati)? | [edge-case-atlas.md](./edge-case-atlas.md) (O-08) | **YA.** Wajib mengeksplorasi kondisi batas ekstrem di luar happy path. |
| **MP-3** | Apakah ada pemeriksaan potensi data lama yang terlanjur korup akibat bug masa lalu? | [testing.md](./testing.md) (O-09) | **YA.** Perbaikan yang hanya melindungi data masa depan adalah setengah matang. |
| **MP-4** | Apakah full test suite proyek telah dieksekusi dengan metrik angka presisi? | [testing.md](./testing.md) (O-10) | **YA.** Laporkan angka presisi (contoh: "145/145 passed, coverage 100%"). |
| **MP-5** | Apakah produk nyata (the real thing) telah dijalankan langsung minimal 1 kali (CLI/API/UI)? | [testing.md](./testing.md) (O-11) | **YA.** Interaksi nyata wajib memvalidasi output di luar unit test. |
| **MP-6** | Apakah perbedaan environment pengujian dengan target rilis (shipping env) telah diperhitungkan? | [testing.md](./testing.md) (O-12) | **YA.** Waspadai perbedaan dialek DB, env variables, atau dimensi layar. |
| **MP-7** | Apakah putaran terakhir siklus Hunt Loop menghasilkan NOL skenario kegagalan baru? | [testing.md](./testing.md) (Hunt Loop) | **YA.** Dilarang berhenti saat masih ditemukan celah ekstrem. |

---

### 2.4 Gerbang 4: Tata Kelola Mutu & Sinkronisasi Orkestrator (Governance & Handover Sync)
Pemeriksaan integritas tata kelola kualitas dan sinkronisasi ekosistem orchestrator:

1. **Perangkingan Dampak Risiko (Impact Ranking — O-13):**  
   Klasifikasikan seluruh temuan risiko berdasarkan hierarki urgensi:
   Kehilangan Uang/Data -> Keamanan/Privasi -> Alur Bisnis Patah -> Tampilan/UI/Teks.
2. **Uji Deterministik (Deterministic Tests — O-14):**  
   Pengujian dilarang flaky. Gunakan frozen clock untuk manipulasi waktu, seeded random untuk keacakan, dilarang `sleep()` arbitrer, dan isolasi mutasi data antar-tes.
3. **Asersi Substantif (Assert What Matters — O-15):**  
   Asersi wajib memeriksa mutasi state nyata dan nilai bisnis, bukan sekadar status HTTP 200 atau `toBeDefined()`.
4. **Daftar Hal yang Belum Diuji (Untested List — O-16):**  
   Wajib mencantumkan secara jujur daftar apa saja yang belum/tidak sempat diuji beserta alasannya.
5. **Sinkronisasi Graphify (Graphify Codebase Sync):**  
   Wajib mengeksekusi `graphify update` di dalam direktori codebase node asli setelah modifikasi kode selesai. DILARANG membuat atau memperbarui `.graphify` di dalam direktori `ai-orchestrator-template/`.
6. **Pencatatan CHANGELOG:**  
   Seluruh pembaruan wajib dicatatkan di baris teratas `CHANGELOG.md` node terkait sesuai format templat resmi.
7. **Kebersihan Pesan Git Commit:**  
   Pesan commit wajib mematuhi konvensi Semantic Commit dan steril 100% dari istilah internal orchestrator (dilarang menyebutkan nama node, nomor tiket, atau prompt).

---

## 3. Format Dokumentasi di Tiket (`AI Execution Log`)

Sebelum mengubah status tiket menjadi `Done`, AI Agent **WAJIB MENYALIN & MENGISI** rekaman Delivery Gate berikut ke dalam seksi `AI Execution Log & Output` pada file tiket:

```markdown
### Delivery Gate Verification
- **Code & Data Hygiene (CG-1 s/d CG-5):** PASS (Zero-comment, no fallback/mock, no magic numbers, no direct DDL, safe file ops)
- **Hard Gate Testing (HG-1 s/d HG-4):** PASS (Semua TIDAK: bukan DB prod, ada bukti konkret, tanpa melonggarkan tes, repositori bersih)
- **Must Probe Status:**
  - O-07 (Fail on Old Code): [PASS / N/A jika fitur baru] — Bukti: [pesan error tes di kode lama]
  - O-08 (Edge Case Atlas): PASS (Sumbu: Who, How much, When, State, Input)
  - O-09 (Damage Assessment): [PASS / Bersih / N/A] — Catatan: [kondisi data lama]
  - O-10 (Full Suite Metrics): PASS — [Contoh: 145/145 passed, 100% coverage]
  - O-11 (Real Thing Driven): PASS — [Contoh: diverifikasi via curl endpoint / browser preview]
  - O-12 (Shipping Env Check): PASS — [Contoh: konfigurasi target dev/prod selaras]
  - Hunt Loop Final Round: PASS (0 temuan baru di putaran terakhir)
- **Orchestrator Sync & Quality Locks:**
  - Graphify Update: PASS (dijalankan di codebase node asli)
  - CHANGELOG.md: PASS (diperbarui di baris teratas)
  - Commit Message Sanity: PASS (steril dari istilah orchestrator/tiket)
  - O-16 Untested List: [Sebutkan hal yang belum sempat diuji beserta alasannya]
```
