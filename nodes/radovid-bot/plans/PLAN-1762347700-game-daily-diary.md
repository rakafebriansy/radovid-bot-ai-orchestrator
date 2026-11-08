---
id: PLAN-1762347700
ticket_id: TICKET-1762347700
title: Game 8: Daily 5-Sentence Diary
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 8: Daily 5-Sentence Diary

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347700](../tickets/TICKET-1762347700-game-daily-diary.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun fitur jurnal refleksi harian (Game 8: Daily 5-Sentence Diary) yang diakses via command `/radovid diary`. Fitur ini menghadirkan interaksi form Discord Modal dengan 5 input field baris tunggal/multi-baris, menyalurkan data masukan ke usecase `feedback/diary`, mengevaluasi setiap kalimat secara terisolasi via Ollama Gemma 3 4b, mengekstrak 3 rekomendasi kosakata kontekstual, dan mempersistensikan rekaman ke tabel SQLite `daily_diaries` dengan timestamp Epoch Milliseconds (`int64`).
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler Discord `/radovid diary` di `internal/delivery/discord`.
  - Komponen Terdampak: `internal/domain/diary`, `internal/usecase/feedback`, `internal/infrastructure/sqlite`, `internal/infrastructure/ollama`, `internal/delivery/discord`.
  - Potensi Efek Samping: Entri diary pengguna mengandung teks personal; database SQLite harus diproteksi dengan izin baca berkas yang ketat (`0600`) dan evaluasi AI tidak boleh menyimpan data di luar persistensi tabel lokal `daily_diaries`.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/007_create_daily_diaries.sql` | DDL migrasi tabel `daily_diaries` dengan kolom ID UUID dan epoch ms |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/diary/entity.go` | Entitas domain `DiaryEntry`, `SentenceEvaluation`, `DiaryAnalysisResult` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/diary/repository.go` | Interface port `DiaryRepository` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/diary/errors.go` | Definisi domain errors (misal: `ErrIncompleteSentences`) |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/feedback/diary.go` | Logika validasi 5 kalimat, pemrosesan Ollama, dan persistensi database |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/feedback/diary_test.go` | Unit test pemrosesan diary, parser JSON respons Ollama, dan validasi 5 kalimat |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/daily_diary.tmpl` | Prompt template terstruktur untuk analisis 5 kalimat dan ekstraksi 3 kosakata |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/sqlite/diary_repository.go` | Implementasi `DiaryRepository` menggunakan raw SQL `database/sql` |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_diary.go` | Slash command handler `/radovid diary` dan modal submit dispatcher |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Template visual embed kartu jurnal harian dengan 5 blok evaluasi kalimat |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat migrasi SQL `007_create_daily_diaries.sql`.
   - Siapkan unit test `diary_test.go` yang menguji: penolakan bila kurang dari 5 kalimat terisi, pemetaan JSON dari respons LLM, dan penyimpanan ke mock repository.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Buat struct domain pada `internal/domain/diary/entity.go`.
   - Tulis prompt template `daily_diary.tmpl` dengan format output ketat: array 5 elemen untuk `sentences` dan array 3 elemen untuk `vocabularies`.
   - Implementasikan fungsi `ProcessDiary` pada `internal/usecase/feedback/diary.go` yang menggabungkan kalimat menjadi format evaluasi terpadu dan mengonversi respons Ollama ke struct domain.
   - Implementasikan `diary_repository.go` untuk mengeksekusi INSERT ke `daily_diaries`.
3. **Langkah 3 (Refactor & Integrasi Delivery Layer):**
   - Daftarkan slash command `/radovid diary` yang memicu Discord Modal dengan 5 komponen `TextInput` (Sentence 1 s/d 5).
   - Pastikan interaksi menggunakan `Defer()` sebelum memanggil usecase dan mengedit pesan dengan embed kartu jurnal setelah inferensi selesai.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji input kalimat kosong pada baris 3, input teks non-Inggris, dan karakter tanda kutip tunggal/ganda (SQL injection safety check).
   - Verifikasi lulus pengujian 100% dan struktur SQLite mematuhi standar BigInt timestamps.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/diary/entity.go
package diary

type SentenceEvaluation struct {
	Index       int    `json:"index"`
	Original    string `json:"original"`
	Corrected   string `json:"corrected"`
	Explanation string `json:"explanation"`
}

type VocabularyRecommendation struct {
	Word        string `json:"word"`
	Definition  string `json:"definition"`
	Example     string `json:"example"`
}

type DiaryEntry struct {
	ID          string                     `json:"id"`
	UserID      string                     `json:"user_id"`
	Content     string                     `json:"content"`      // 5 kalimat gabungan
	Evaluations []SentenceEvaluation       `json:"evaluations"`
	Vocabularies []VocabularyRecommendation `json:"vocabularies"`
	CreatedAt   int64                      `json:"created_at"`   // Epoch Milliseconds
}
```

```go
// migrations/007_create_daily_diaries.sql
CREATE TABLE IF NOT EXISTS daily_diaries (
    id          TEXT PRIMARY KEY,
    user_id     TEXT NOT NULL REFERENCES users(id),
    content     TEXT NOT NULL,
    ai_feedback TEXT,
    created_at  BIGINT NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_daily_diaries_user_id ON daily_diaries(user_id);
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Jika pengguna memasukkan satu kata berulang di semua kolom teks modal, use case menolak input dengan pesan error bersahabat "Please provide meaningful sentences about your day".
- **Strategi Rollback:** Bila Ollama gagal mengevaluasi entri diary dalam batas 30 detik, konten diary tetap disimpan ke SQLite dengan status `ai_feedback: null`, dan pesan konfirmasi tersimpan dikirimkan ke user dengan catatan evaluasi tertunda.

