---
id: TICKET-1762347700
plan_id: PLAN-1762347700
title: Game 8: Daily 5-Sentence Diary
status: Todo
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
- [ ] Slash command `/radovid diary` menampilkan form Discord Modal dengan 5 input field teks wajib (Sentence 1 s/d Sentence 5)
- [ ] Validasi bahwa kelima kalimat terisi dengan konten substansial (bukan spasi kosong atau duplikasi karakter)
- [ ] Evaluasi AI menganalisis setiap kalimat secara granular: menampilkan bentuk awal, versi revisi alami, serta penjelasan tata bahasa singkat
- [ ] AI merekomendasikan tepat 3 kosakata baru (advanced/idiomatic vocabulary) lengkap dengan definisi ringkas dan contoh penerapannya dalam konteks diary
- [ ] Migrasi database `007_create_daily_diaries.sql` membuat tabel `daily_diaries` dengan ID UUID dan created_at bertipe epoch milliseconds
- [ ] Implementasi repositori SQLite untuk menyimpan konten diary dan feedback AI, serta mendukung query riwayat diary pengguna
- [ ] Hasil feedback disajikan secara elegan melalui Discord Embed kartu jurnal

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/007_create_daily_diaries.sql`
- `../../../../radovid-bot-api/internal/domain/diary/entity.go`
- `../../../../radovid-bot-api/internal/domain/diary/repository.go`
- `../../../../radovid-bot-api/internal/domain/diary/errors.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/diary.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/diary_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/daily_diary.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/diary_repository.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_diary.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

