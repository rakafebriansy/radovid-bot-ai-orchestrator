# Retrospectives Directory (`retrospectives/`)

> **[PRINSIP UTAMA]** Pusat evaluasi diri (*Self-Healing*) AI Agent untuk merekam, menganalisis, dan mengonversi kegagalan teknis menjadi aturan permanen proyek.

---

## 1. Alur Kerja Evaluasi Retrospektif

### 1.1 Lima Tahap Siklus Self-Healing
1. **Pencatatan Hambatan Kronis:** Saat menemui jalan buntu teknis, gagal setelah batas percobaan (*Max Retry Rule*), salah tafsir instruksi, atau ditegur pengguna, AI Agent WAJIB mencatat detail kejadian ke `RETROSPECTIVE.md`.
2. **Pencatatan Resolusi Konflik:** Catat detail konflik penggabungan cabang (*merge conflict*) beserta langkah penyelesaiannya ke `MERGE_HISTORY.md`.
3. **Analisis Forensik:** Jabarkan gejala masalah (*symptoms*), konteks pemicu, dan solusi definitif yang memecahkannya.
4. **Konversi ke Pedoman Permanen:** Entri evaluasi berkala WAJIB dikonversikan menjadi aturan teknis baku yang dicatatkan ke `guidelines/project-context.md`.
5. **Pembersihan Log (Token Optimization):** Entri log di `RETROSPECTIVE.md` yang telah berhasil dikonversi dan dipindahkan ke `guidelines/project-context.md` WAJIB DIHAPUS permanen dari `RETROSPECTIVE.md` agar ukuran berkas tetap minimal.

### 1.2 Ketentuan Pembacaan Berkas
1. AI Agent wajib membaca `RETROSPECTIVE.md` dan `MERGE_HISTORY.md` sebelum memulai tugas krusial HANYA JIKA berkas tersebut memuat entri permasalahan aktif (bukan template kosong).
2. Hal ini diberlakukan demi efisiensi konsumsi token dan mencegah perulangan galat yang pernah terselesaikan.
