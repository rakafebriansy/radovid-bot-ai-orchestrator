---
id: PLAN-1762347400
ticket_id: TICKET-1762347400
title: AI Feedback System (Grammar & Vocabulary)
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: AI Feedback System (Grammar & Vocabulary)

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347400](../tickets/TICKET-1762347400-ai-feedback-system.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun subsistem evaluasi teks bahasa Inggris modular (`internal/usecase/feedback`) yang memproses masukan teks pengguna via inferensi lokal Ollama Gemma 3 4b, memparsing struktur JSON berisi koreksi gramatikal, saran pemformatan frasa alami, rekomendasi kosakata tingkat lanjut, serta skor komunikasi (1-10), dan menyimpan rekam jejaknya ke tabel `feedback_logs` di SQLite.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Use case aktivitas game Fase 3 (Job Interview, Roleplay, One Minute Challenge, Story Relay, Daily Diary, Five Word Challenge) serta handler Discord `/radovid feedback`.
  - Komponen Terdampak: `internal/infrastructure/ollama`, `internal/domain/feedback`, `internal/infrastructure/database`, `internal/infrastructure/discord`.
  - Potensi Efek Samping: Latensi inferensi LLM dapat melampaui default threshold Discord acknowledgement (3 detik), sehingga seluruh panggilan evaluasi harus menggunakan deferred responses (`Defer()`) di layer delivery.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/005_create_feedback_logs.sql` | DDL tabel `feedback_logs` dengan kolom epoch milliseconds |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/feedback/errors.go` | Error domain spesifik evaluasi AI |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/feedback/entity.go` | Entitas `FeedbackRequest`, `FeedbackResponse`, `GrammarCorrection`, `VocabularyItem`, `FeedbackLog` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/feedback/entity_test.go` | Unit test normalisasi skor batas [1, 10] dan entitas |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/feedback/repository.go` | Definisi interface port `feedback.Repository` |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/feedback.tmpl` | Prompt template terstruktur penghasil format JSON feedback |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback.go` | Orkestrasi use case evaluasi feedback, sanitasi input, parser JSON, dan fallback builder |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback_test.go` | Unit test table-driven parser, validasi batas skor, dan sanitasi |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/feedback_repository.go` | Implementasi konkret `feedback.Repository` via SQLite raw SQL |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/feedback_repository_test.go` | Unit test SQLite feedback repository CRUD dan order sorting |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Template visual embed feedback (`FeedbackSummaryEmbed`, `BuildFeedbackEmbed`) |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test visual embed feedback dan skema warna |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Rancang migration DDL `005_create_feedback_logs.sql`.
   - Buat unit test `ai_feedback_test.go` yang menguji parsing respons JSON acak/rusak dari LLM dan sanitasi teks mention Discord.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan struct domain pada `internal/domain/feedback/entity.go` tanpa dependensi eksternal.
   - Implementasikan template prompt pada `internal/infrastructure/ollama/prompts/feedback.tmpl` dengan instruksi ketat "JSON-only output".
   - Tulis logika `EvaluateFeedback` di layer usecase dengan timeout kontekstual (default 30 detik) dan fallback response jika LLM mengembalikan format tidak valid.
   - Implementasikan repository SQLite untuk menyimpan log evaluasi ke tabel `feedback_logs`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Implementasikan Discord Embed builder di `internal/infrastructure/discord/game_templates.go` untuk menampilkan feedback dengan warna tematik (Hijau untuk nilai >= 8, Kuning 5-7, Merah < 5).
   - Pastikan Zero-Comment Policy dan fail-fast diterapkan pada input validation (panjang teks > 2000 karakter ditolak dini).
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji kasus batas teks kosong, teks hanya karakter non-alfanumerik, prompt injection teks user, dan respons LLM parsial.
   - Verifikasi pengujian lulus 100% dan tidak ada race condition saat operasi I/O SQLite concurrency.

---

## 5. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Input berisi regex manipulatif atau tag HTML disterilkan; input melebihi 2000 karakter ditolak dengan `ErrInputTooLong`.
- **Strategi Rollback:** Jika terjadi kegagalan skema DDL, rollback tabel menggunakan skrip migrasi `DROP TABLE IF EXISTS feedback_logs;`.
