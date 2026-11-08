---
id: TICKET-1762433980
plan_id: PLAN-1762433980
title: /radovid progress Command
status: Done
priority: High
labels: [Frontend, Discord, SlashCommand, Gamification]
---

# Deskripsi
Sebagai Pengguna Discord,
Saya ingin menjalankan perintah slash `/radovid progress` untuk melihat ringkasan visual statistik belajar bahasa Inggris saya,
Sehingga saya dapat memantau level saat ini, progres XP menuju level berikutnya melalui progress bar grafis Unicode, jumlah streak aktif, dan daftar pencapaian yang telah saya raih.

Perintah `/radovid progress` adalah antarmuka visual utama bagi pengguna untuk menginspeksi kemajuan belajar mereka. Tampilan harus disajikan dalam format Discord Embed yang bersih, elegan, dan informatif, memuat indikator level, progress bar Unicode (misal: `[████████░░░░░░] 56%`), streak hari berturut-turut, total sesi game yang diselesaikan, dan deretan lencana penghargaan yang berhasil dibuka.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762433980-radovid-progress-command.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Perintah slash `/radovid progress` terdaftar di router Discord command framework
- [x] Embed Discord menyajikan visualisasi statistik: Level, Total XP, XP saat ini vs XP menuju level berikutnya, Current Streak (dengan emoji 🔥), dan Total Games Completed
- [x] Tersedia utilitas generator Unicode Progress Bar murni Go yang fleksibel (panjang bar dapat dikonfigurasi, menghitung persentase akurat 0% - 100%)
- [x] Menampilkan daftar lencana pencapaian (Achievements) yang telah dibuka berupa deretan ikon emoji dan nama lencana, atau placeholder edukatif jika belum ada
- [x] Penanganan pengguna baru (user yang belum pernah bermain) secara otomatis membuat profil inisial (Level 1, 0 XP, 0 Streak) tanpa menampilkan error
- [x] Respons dikirimkan dalam format ephemeral atau channel embed sesuai preferensi konteks pemanggilan
- [x] Unit testing pada generator progress bar (uji nilai 0%, 50%, 100%, pembagian dengan nol, dan nilai batas overflow)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_progress.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_progress_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/progress_bar.go`
- `../../../../radovid-bot-api/internal/delivery/discord/progress_bar_test.go`
- `../../../../radovid-bot-api/internal/usecase/progress/get_progress.go`
- `../../../../radovid-bot-api/internal/usecase/progress/get_progress_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Waktu Eksekusi:** 2026-11-07 21:55:00
- **Rangkuman Eksekusi:**
  1. Generator Unicode Progress Bar `RenderProgressBar` dibangun di `internal/delivery/discord/progress_bar.go` dengan perlindungan pembagian nol, batas overflow, dan clamping rasio `[0.0, 1.0]`. Unit test diuji komprehensif di `progress_bar_test.go` (100% PASS).
  2. Use case `GetProgressUseCase` (`GetProgressSummary`) diimplementasikan di `internal/usecase/progress/get_progress.go` untuk menghitung level dasar, batas XP level berikutnya, span XP, sisa XP, streak aktif, dan daftar badge yang diraih. Pengguna baru otomatis diinisialisasi melalui `GetOrCreateByDiscordID` dan `GetOrCreateProgress`. Unit test use case diuji di `get_progress_test.go` (100% PASS).
  3. Template visual Discord Embed `BuildProgressEmbed` diimplementasikan di `internal/infrastructure/discord/game_templates.go` menampilkan level, streak (🔥), progress bar Unicode, persentase, sisa XP, serta deretan lencana pencapaian atau pesan edukatif jika belum ada lencana. Unit test diverifikasi di `game_templates_test.go` (100% PASS).
  4. Sub-command `progress` didaftarkan pada router slash command `/radovid` di `command_def.go` dan dipetakan di `handler.go`.
  5. Handler interaksi `HandleProgressCommand` dan wrapper delegator `HandleRadovidProgress` diimplementasikan di `radovid_progress.go` dan diuji di `radovid_progress_test.go` (100% PASS).
  6. Dependensi use case diintegrasikan ke `GameHandler` dan di-wire di `cmd/bot/main.go`.
  7. Seluruh rangkaian test suite (21 package) lulus 100%, `go vet` bersih dari peringatan, `go build ./cmd/bot` berhasil, dan Zero-Comment Policy dipatuhi penuh.
