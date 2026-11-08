---
id: TICKET-1762347700
plan_id: PLAN-1762347700
title: Game 8: Daily 5-Sentence Diary
status: Done
priority: High
labels: [Feature, Writing, Diary, Backend, Discord, Database]
---

# Deskripsi
Sebagai Pembelajar Bahasa Inggris di Radovid Bot,
Saya ingin menulis catatan harian (diary) sebanyak 5 kalimat dalam bahasa Inggris melalui perintah `/radovid diary` dan menerima analisis koreksi per kalimat serta saran 3 kosakata baru,
Sehingga saya dapat membangun kebiasaan menulis harian yang konsisten dan memperluas repertoar kosakata bahasa Inggris saya secara personal.

Game 8: Daily 5-Sentence Diary adalah fitur refleksi harian di mana pengguna mengisi form modal Discord yang terdiri atas 5 input baris kalimat tentang aktivitas hari mereka. Sistem mengirimkan teks ke Ollama untuk dievaluasi secara terperinci per kalimat (koreksi kesalahan tata bahasa, saran pemilihan kata), menyertakan rekomendasi 3 kosakata baru yang sangat kontekstual dengan topik diary hari itu, serta menyimpan entri ke dalam database SQLite (`daily_diaries`) sebagai riwayat jurnal belajar.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347700-game-daily-diary.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Slash command `/radovid diary` menampilkan form Discord Modal dengan 5 input field teks wajib (Sentence 1 s/d Sentence 5)
- [x] Validasi bahwa kelima kalimat terisi dengan konten substansial (bukan spasi kosong atau duplikasi karakter)
- [x] Evaluasi AI menganalisis setiap kalimat secara granular: menampilkan bentuk awal, versi revisi alami, serta penjelasan tata bahasa singkat
- [x] AI merekomendasikan tepat 3 kosakata baru (advanced/idiomatic vocabulary) lengkap dengan definisi ringkas dan contoh penerapannya dalam konteks diary
- [x] Migrasi database `006_create_daily_diaries.sql` membuat tabel `daily_diaries` dengan ID UUID dan created_at bertipe epoch milliseconds
- [x] Implementasi repositori SQLite untuk menyimpan konten diary dan feedback AI, serta mendukung query riwayat diary pengguna
- [x] Hasil feedback disajikan secara elegan melalui Discord Embed kartu jurnal

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/006_create_daily_diaries.sql`
- `../../../../radovid-bot-api/internal/domain/diary/entity.go`
- `../../../../radovid-bot-api/internal/domain/diary/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/diary/repository.go`
- `../../../../radovid-bot-api/internal/domain/diary/errors.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/diary_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/diary_repository_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/daily_diary.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/diary.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/diary_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_diary.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_diary_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Completed
- **Executed At:** 2026-11-06 08:38:00
- **Summary:**
  - Menulis migrasi database DDL `migrations/006_create_daily_diaries.sql` membuat tabel `daily_diaries` dengan indeks pada `user_id` dan `created_at`.
  - Mengimplementasikan domain model Diary `internal/domain/diary/`: `SentenceEvaluation`, `VocabularyRecommendation`, `DiaryAnalysis`, `DiaryEntry`, fungsi `ValidateSentences` (validasi 5 kalimat, batas 5-300 karakter, filtering teks repetitif/duplikat), dan unit test domain `entity_test.go`.
  - Mengimplementasikan prompt template `internal/infrastructure/ollama/prompts/daily_diary.tmpl` dengan JSON schema granular untuk 5 kalimat dan 3 rekomendasi kosakata, beserta verifikasi unit test `embed_test.go`.
  - Mengimplementasikan `internal/infrastructure/database/diary_repository.go` berbasis SQLite (`modernc.org/sqlite`) yang mendukung persistensi `Save`, retrieval `GetByID`, dan riwayat kronologis `GetHistoryByUserID` beserta unit test `diary_repository_test.go`.
  - Mengimplementasikan Usecase layer `internal/usecase/feedback/diary.go` (`DiaryUseCase`) mencakup sanitasi teks, pemrosesan inferensi Ollama, parsing output JSON cerdas (`parseDiaryJSON`), fallback robust saat Ollama gagal/timeout (`buildFallbackDiaryAnalysis`), dan penyimpanan ke database. Unit test `diary_test.go` lulus 100%.
  - Mengimplementasikan helper visual Discord Embed `BuildDailyDiaryEmbed` pada `internal/infrastructure/discord/game_templates.go` dan unit test `game_templates_test.go`.
  - Mengimplementasikan delivery Discord handler `internal/delivery/discord/radovid_diary.go` (membuka form modal 5 input baris dan memproses submission modal), mendaftarkan slash command `/radovid diary` di `command_def.go` dan `handler.go`, serta menambahkan alias di `/radovid play diary`.
  - Mengintegrasikan router prefix `diary:` pada `cmd/bot/main.go`.
  - Seluruh pengujian lolos 100% (`go test -count=1 ./...`), `go vet ./...` bersih 0 isu, `go build ./...` sukses, dan Zero-Comment Policy dipatuhi secara mutlak di seluruh file Go.
