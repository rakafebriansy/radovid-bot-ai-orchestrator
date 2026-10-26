# System Design Guide & Architecture Template

> **[PRINSIP UTAMA]** System Design adalah cetak biru arsitektur teknis (*architectural blueprint*) yang menetapkan batasan rekayasa perangkat lunak, tumpukan teknologi, dan pola desain agar kode yang dihasilkan AI Agent konsisten, aman, dan dapat diskalakan.

---

## 1. Arsitektur & Komponen Inti System Design

### 1.1 Peran System Design dalam AI Orchestration
1. **Architectural Blueprint:** Menetapkan "bagaimana" sistem dibangun dari sisi rekayasa perangkat lunak (Clean Architecture, MVVM, MVC, Hexagonal, dll.).
2. **Batasan Teknis:** Berfungsi sebagai *technical constraints* agar AI tidak membuat keputusan struktural sembarangan dan mencegah pergeseran kode (*code drift*).

### 1.2 Sepuluh Komponen Spesifikasi Wajib
1. **Arsitektur Perangkat Lunak (*Software Architecture*):** Pola arsitektur dasar yang diterapkan secara konsisten pada seluruh proyek.
2. **Struktur Direktori (*Directory Structure*):** Pemetaan folder standar dan aturan penempatan berkas (`models`, `views`, `controllers`, `services`, `repositories`).
3. **Tumpukan Teknologi (*Tech Stack* & Dependensi):** Bahasa pemrograman, framework, pustaka, versi SDK, dan perkakas pendukung.
4. **Logika & Alur Komponen (Flowchart PlantUML):**
   - **Aplikasi Web / API / Mobile:** Wajib memvisualisasikan alur kerja algoritma komponen utama via PlantUML di `docs/diagrams/flowchart.puml` dan menautkannya ke dokumen.
   - **Game Engine:** Wajib membuat *Micro State Diagram* (misal: *Player State*) PlantUML di `docs/diagrams/state-diagram.puml`.
5. **Skema & Pemodelan Data (*Data Modeling*):**
   - **ERD PlantUML:** Wajib membuat diagram relasi tabel/koleksi di `docs/diagrams/erd.puml` (khusus Backend/Fullstack).
   - **Standar Datetime Basis Data:** Wajib memilih Opsi 1 (Epoch Millis / `BIGINT`) atau Opsi 2 (Timestamp UTC / `TIMESTAMPTZ`) secara konsisten merujuk [database.md](../../global-guidelines/database.md).
   - **Model:** Representasi struktur entitas atau objek internal pada kode.
   - **DTO (*Data Transfer Object*):** Format payload data komunikasi client-server.
6. **Desain Kontrak API (*API Contracts*):** Spesifikasi endpoint, payload request/response, metode autentikasi, serta contoh konkret *Header*, *Request*, dan *Response*.
7. **Arsitektur Deployment (*Hosting & Infrastructure*):** Infrastruktur hosting, skema peladen, penyedia cloud, dan kontainerisasi (Docker).
8. **Integrasi Pihak Ketiga (*Third-Party Services*):** Layanan eksternal seperti payment gateway, analitik, dan penyimpanan cloud.
9. **Konfigurasi Lingkungan (*Environment Variables*):** Daftar variabel `.env` privat beserta fungsi dan validasi fail-fast merujuk [security.md](../../global-guidelines/security.md) dan [coding.md](../../global-guidelines/coding.md).
10. **Keamanan & Penanganan Galat (*Security & Error Handling*):** Protokol enkripsi, otentikasi/RBAC, error logging, dan validasi data.

---

## 2. Alur Perumusan & Validasi Desain

### 2.1 Protokol Perumusan System Design
1. Pastikan PRD (`../../global-docs/prd.md`) telah disahkan terlebih dahulu.
2. AI Agent wajib mengajukan pertanyaan teknis terarah (batasan infrastruktur, preferensi stack, target performa, skema keamanan, struktur data) sebelum merumuskan arsitektur.

### 2.2 Keterkaitan dengan Design System
1. **[WAJIB]** Sebelum AI Agent mengeksekusi antarmuka visual (UI) pada node ini, AI wajib merujuk pedoman visual di `../../global-docs/design-system.md` agar implementasi selaras dengan estetika ekosistem.

---

## 3. Spesifikasi Arsitektur: [GANTI_DENGAN_NAMA_PROYEK]

> **Instruksi AI Agent:** Hapus placeholder judul di atas dan ganti dengan nama proyek atau node aktual. Cantumkan rincian arsitektur teknis, struktur folder, skema data, dan kontrak API node ini di bawah garis pemisah.

---

*(Rincian arsitektur, struktur folder, skema data, serta panduan System Design spesifik untuk node ini dicatatkan di sini)*
