# Product Requirements Document (PRD) Guide & Template

> **[PRINSIP UTAMA]** PRD berfungsi sebagai instruksi induk (*master prompt*) dan sumber kebenaran tunggal (*single source of truth*) fungsionalitas produk sebelum penyusunan arsitektur teknis dan penulisan kode.

---

## 1. Arsitektur & Komponen Inti PRD

### 1.1 Peran PRD dalam AI Orchestration
1. **Master Prompt & Ground Truth:** Menetapkan fungsionalitas, perilaku sistem yang diharapkan, dan kriteria kesuksesan produk bagi AI Agent.
2. **Pencegahan Deviasi:** Memberikan gambaran makro (*big picture*) agar pengembangan tidak menyimpang dari tujuan produk.

### 1.2 Sembilan Komponen Spesifikasi Wajib
1. **Tujuan & Latar Belakang (*Objective & Background*):** Masalah utama yang dipecahkan, target pengguna (*user personas*), dan justifikasi bisnis.
2. **Alur Pengguna (*User Flow / Journey*):** Langkah interaksi pengguna ujung-ke-ujung (*end-to-end*).
   - **[WAJIB] Visualisasi PlantUML:** Buat diagram alur di `global-docs/diagrams/user-journey.puml` dan tautkan ke dokumen.
3. **Diagram Interaksi Sistem Global:**
   - **Web / Mobile / Desktop:** Wajib membuat *Use Case Diagram* atau *Activity Diagram* PlantUML.
   - **Game:** Wajib membuat *Macro State Diagram* (Menu State, Play State, Pause, Game Over) PlantUML.
   - Simpan berkas diagram `.puml` di `global-docs/diagrams/` dan cantumkan tautannya.
4. **Kebutuhan Fungsional (*Functional Requirements*):** Daftar rinci kapabilitas dan aksi sistem (contoh: otentikasi, filter data, kalkulasi transaksi).
5. **Kebutuhan Non-Fungsional (*Non-Functional Requirements*):** Standar performa (waktu respon, load time), keamanan, ketersediaan, dan dukungan platform.
6. **Kriteria Penerimaan (*Acceptance Criteria*):** Kondisi batas dan skenario kelulusan fitur (termasuk penanganan kasus ekstrem) agar tugas divalidasi selesai.
7. **Asumsi & Keterbatasan (*Assumptions & Constraints*):** Asumsi dasar pengembangan dan batasan teknologi atau bisnis yang mengikat.
8. **Di Luar Cakupan (*Out of Scope*):** Daftar eksplisit fitur yang sengaja tidak dikerjakan pada iterasi aktif guna menjaga fokus eksekusi.
9. **Peta Jalan Fase (*Milestone Breakdown*):** Pembagian fase rilis berurut (v1.0 MVP, v1.1 Lanjutan) untuk memandu prioritas tiket.

### 1.3 Arsitektur Multi-Node
1. **Global Scope:** Visi ekosistem, personas lintas-sistem, dan asumsi menyeluruh.
2. **Node-Specific Scope:** Isolasi kebutuhan fungsional, batasan teknis, dan kriteria penerimaan ke dalam sub-bab masing-masing node (`frontend-app`, `backend-api`, dll.).

---

## 2. Alur Perumusan & Kewajiban Pipeline

### 2.1 Protokol Perumusan PRD
1. Jelaskan konsep dasar, visi, dan alur aplikasi kepada AI Agent.
2. AI Agent wajib mengajukan pertanyaan klarifikasi terarah hingga seluruh parameter dan batasan terpetakan lengkap sebelum menyusun PRD.

### 2.2 Gerbang Wajib Pipeline (Pipeline Gate)
1. **[TERLARANG]** Menulis kode produksi sebelum arsitektur teknis terbentuk.
2. **[WAJIB]** Setelah PRD disahkan, lanjutkan penyusunan arsitektur teknis merujuk pada `nodes/[nama-node]/docs/system-design.md` milik masing-masing node.

---

## 3. Spesifikasi Proyek: [GANTI_DENGAN_NAMA_PROYEK]

> **Instruksi AI Agent:** Hapus placeholder judul di atas dan ganti dengan nama proyek aktual. Cantumkan rincian kebutuhan produk di bawah garis pemisah ini.

---

*(Isi Product Requirements Document spesifik untuk proyek atau ekosistem ini dicatatkan di sini)*
