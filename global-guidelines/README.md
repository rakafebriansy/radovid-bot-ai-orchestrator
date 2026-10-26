# Guidelines Directory (`global-guidelines/`)

> **[PRINSIP UTAMA]** Repositori batasan teknis (*constraints*), standar koding, dan hukum besi operasional AI Agent untuk menjamin disiplin rekayasa perangkat lunak.

---

## 1. Klasifikasi Pedoman & Hirarki Regulasi

### 1.1 Dua Tingkat Pedoman
1. **Pedoman Universal (`global-guidelines/*.md`):** Standar rekayasa perangkat lunak baku (koding, database, keamanan, testing, dll.) yang berlaku mengikat lintas-bahasa dan lintas-platform.
2. **Pedoman Spesifik Node (`nodes/[nama-node]/guidelines/project-context.md`):** Aturan eksklusif dan konvensi lokal hasil pemindaian basis kode spesifik node terkait.

### 1.2 Prinsip Single Source of Truth (SSoT)
1. Direktori `global-guidelines/` adalah **sumber kebenaran tunggal tertinggi** untuk seluruh aturan operasional sistem.
2. Berkas `main.md` di setiap node berfungsi sebagai **dispatcher (pengarah)** yang merujuk ke pedoman di direktori ini tanpa menduplikasi isinya.
3. **[PRIORITAS REGULASI]** Jika terjadi perbedaan tafsir atau kontradiksi antara `main.md` dengan berkas di `global-guidelines/`, maka **berkas pedoman di direktori ini adalah rujukan yang menang**.

---

## 2. Siklus Pembelajaran Berkelanjutan (The Retrospective Loop)

1. Sistem pedoman ini terintegrasi langsung dengan direktori evaluasi `nodes/[nama-node]/retrospectives/`.
2. Saat AI Agent menyelesaikan anomali kronis pada suatu node, pemecahan masalah tersebut diekstraksi secara permanen ke dalam `project-context.md` milik node tersebut.
3. Entri yang telah dikonversi ke pedoman wajib dihapus dari `RETROSPECTIVE.md` demi efisiensi konsumsi token.
