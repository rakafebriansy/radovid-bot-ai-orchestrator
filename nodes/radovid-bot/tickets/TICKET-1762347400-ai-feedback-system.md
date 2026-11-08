---
id: TICKET-1762347400
plan_id: PLAN-1762347400
title: AI Feedback System (Grammar & Vocabulary)
status: Done
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
- [x] Definisi entitas domain `FeedbackRequest`, `FeedbackResponse`, `GrammarCorrection`, dan `VocabularyItem` pada package domain feedback
- [x] Implementasi structured prompt template Ollama khusus mode feedback evaluator dengan output berformat JSON terstandarisasi
- [x] Parser parser JSON dengan validasi fail-safe dan fallback respon graceful jika Ollama mengembalikan teks non-JSON atau malformed
- [x] Integrasi use case `EvaluateFeedback` dengan context timeout dan sanitasi input (menghapus Discord mentions, strip markdown manipulatif, batas max 2000 karakter)
- [x] Skema database `feedback_logs` via migration SQL dan implementasi repository SQLite untuk mencatat riwayat evaluasi teks pengguna
- [x] Unit test table-driven komprehensif untuk sanitasi input, JSON parsing, skor bounded validation (1-10), dan error fallback

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/005_create_feedback_logs.sql`
- `../../../../radovid-bot-api/internal/domain/feedback/errors.go`
- `../../../../radovid-bot-api/internal/domain/feedback/entity.go`
- `../../../../radovid-bot-api/internal/domain/feedback/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/feedback/repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/feedback.tmpl`
- `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/feedback_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/feedback_repository_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Waktu Eksekusi:** 2026-11-05 23:25:00
- **Ringkasan Eksekusi:**
  - Telah mengimplementasikan seluruh subsistem AI Feedback System untuk koreksi grammar dan kosakata dengan Clean Architecture:
    1. Domain Feedback: Entitas `GrammarCorrection`, `VocabularyItem`, `FeedbackResponse`, `FeedbackRequest`, `FeedbackLog`, metode `NormalizeScore()`, interface `Repository`, dan domain errors.
    2. Migrasi Database: `005_create_feedback_logs.sql` untuk persistensi log umpan balik pengguna dengan timestamp epoch milidetik dan indexing.
    3. Template Prompt Ollama: `internal/infrastructure/ollama/prompts/feedback.tmpl` yang mewajibkan model mengeluarkan format JSON terstruktur untuk grammar, phrasing, vocab, score, dan encouragement.
    4. Usecase Layer: `internal/usecase/feedback/ai_feedback.go` mencakup sanitasi teks (menghapus mention Discord, regex non-alfanumerik ekstrem, strip markdown berbahaya, pembatasan 2000 karakter), eksekusi context timeout (30s), parsing JSON otomatis dengan pembersihan markdown block, normalisasi skor (1-10), dan pembentukan graceful fallback response jika inferensi LLM gagal.
    5. Database Layer: `internal/infrastructure/database/feedback_repository.go` dengan raw SQL SQLite untuk penyimpanan dan query log umpan balik berdasarkan user ID, session ID, dan feedback ID.
    6. Discord UI Embeds: `FeedbackSummaryEmbed` dan `BuildFeedbackEmbed` pada `internal/infrastructure/discord/game_templates.go` dengan pewarnaan tematik adaptif (Hijau >= 8, Kuning 5-7, Merah < 5).
  - Seluruh unit test lulus 100% (`go test -count=1 ./...`).
  - Zero-Comment Policy dipatuhi sepenuhnya di seluruh file Go baru.
