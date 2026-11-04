---
id: TICKET-1762347400
plan_id: PLAN-1762347400
title: AI Feedback System (Grammar & Vocabulary)
status: Todo
priority: High
labels: [Backend, AI, Ollama, Clean-Architecture, Evaluation]
---

# Deskripsi
Sebagai Player / Pembelajar Bahasa Inggris di Radovid Bot,
Saya ingin menerima evaluasi komprehensif berupa koreksi tata bahasa (grammar corrections), saran pengungkapan yang lebih alami (natural phrasing), serta pengayaan kosakata (vocabulary enrichment) dari respons tulisan saya,
Sehingga saya memahami kesalahan sintaksis saya dan mampu meningkatkan kualitas komunikasi bahasa Inggris secara bertahap.

Sistem Radovid Bot memerlukan service evaluator terdedikasi yang memanfaatkan model lokal Ollama (Gemma 3 4b) untuk menganalisis teks pengguna. Evaluator ini harus mampu mengembalikan struktur data yang terstandarisasi (JSON) mencakup koreksi grammar, alternatif frasa yang lebih idiomatik, rekomendasi kosakata tingkat lanjut, serta skor komunikasi keseluruhan (1-10) tanpa merusak flow interaksi game.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347400-ai-feedback-system.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Definisi entitas domain `FeedbackRequest`, `FeedbackResponse`, `GrammarCorrection`, dan `VocabularyItem` pada package domain feedback
- [ ] Implementasi structured prompt template Ollama khusus mode feedback evaluator dengan output berformat JSON terstandarisasi
- [ ] Parser parser JSON dengan validasi fail-safe dan fallback respon graceful jika Ollama mengembalikan teks non-JSON atau malformed
- [ ] Integrasi use case `EvaluateFeedback` dengan context timeout dan sanitasi input (menghapus Discord mentions, strip markdown manipulatif, batas max 2000 karakter)
- [ ] Skema database `feedback_logs` via migration SQL dan implementasi repository SQLite untuk mencatat riwayat evaluasi teks pengguna
- [ ] Unit test table-driven komprehensif untuk sanitasi input, JSON parsing, skor bounded validation (1-10), dan error fallback

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/008_create_feedback_logs.sql`
- `../../../../radovid-bot-api/internal/domain/feedback/entity.go`
- `../../../../radovid-bot-api/internal/domain/feedback/repository.go`
- `../../../../radovid-bot-api/internal/domain/feedback/errors.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/feedback.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/feedback_client.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/feedback_repository.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

