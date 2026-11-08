---
id: TICKET-1762520000
plan_id: PLAN-1762520000
title: "[BUG] Progression Loop Terputus (XP, Streak & Achievement Tidak Tercatat Saat Game Selesai)"
status: Done
priority: High
severity: Money/Data Loss
labels: [Bug, Backend, Gamification, Discord, Database]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Saat pengguna menyelesaikan salah satu dari 15 mini-game atau mengirim jurnal harian (`/radovid diary`), sistem menghitung `XPReward` di memori dan menampilkannya pada embed Discord (contoh: `+50 XP`). Namun, `AwardXPUseCase`, `RecordDailyActivityUseCase`, dan `CheckAndUnlockUseCase` tidak pernah diinisialisasi di `cmd/bot/main.go` maupun di-inject ke `GameHandler`. Akibatnya, perolehan XP, streak harian, dan pencapaian (*achievements*) tidak pernah disimpan ke SQLite (`user_progress`, `user_xp_log`, `achievements`), sehingga `/radovid progress` selalu menampilkan 0 XP dan 0 Streak.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520000-fix-progression-loop-wiring.md)
*(Gunakan folder `plans/` untuk rincian analisis akar masalah dan diff perbaikan; jaga tiket tetap berfokus pada AC dan DoD).*

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Mainkan mini-game apapun (misal `/radovid play five-word-challenge` atau `/radovid diary`) hingga selesai.
2. Amati embed hasil akhir yang menampilkan perolehan XP.
3. Jalankan perintah `/radovid progress`.
4. Amati bahwa Total XP tetap 0, Streak tetap 0 hari, dan lencana `first_game` tidak terbuka.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Setiap kali sesi mini-game atau diary selesai, bot secara atomik mencatat aktivitas harian (streak), memberikan XP sesuai tingkat kesulitan ke `user_progress` & `user_xp_log`, serta mengevaluasi pembukaan achievement baru (`first_game`, `boss_slayer`, dll.).
- **Aktual (Actual):** XP hanya tampil secara kosmetik pada embed Discord tanpa pernah ditulis ke database SQLite.

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] `AwardXPUseCase`, `RecordDailyActivityUseCase`, dan `CheckAndUnlockUseCase` diinisialisasi di `cmd/bot/main.go` dan di-inject ke `GameHandler`.
- [x] `GameHandler` menyediakan helper terpusat untuk memproses progresi pemain (memastikan profil user ada, mencatat streak harian, memberikan XP, dan mengevaluasi achievement) saat sesi game atau diary selesai.
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/cmd/bot/main.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_diary.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_boss_fight.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_five_word.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_secret_mission.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_transformation.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_movie_review.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_news_reporter.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_dream_house.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_roleplay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_one_minute.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_story_relay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q.go`

---

## 6. AI Execution Log & Output
*Bagian ini WAJIB diisi oleh AI Agent SAAT dan SETELAH memperbaiki bug.*

- **Penyebab Akar (Root Cause):**
  - Use cases gamifikasi (`AwardXPUseCase`, `RecordDailyActivityUseCase`, `CheckAndUnlockUseCase`) telah dibuat pada Fase 4 namun tidak pernah diinstansiasi di `cmd/bot/main.go` maupun dihubungkan ke `GameHandler`. Seluruh handler game hanya menghitung `XPReward` di memori dan menyajikannya di Discord embed tanpa persistensi SQLite ke tabel `user_progress`, `user_xp_log`, dan `achievements`.
- **Langkah Perbaikan:**
  1. Menambahkan method `SetProgressionDependencies(userRepo, awardXP, recordDaily, checkUnlock)` dan helper terpusat `RecordGameCompletion(ctx, params)` pada `internal/delivery/discord/game_handler.go`.
  2. Mengimplementasikan unit test komprehensif pada `internal/delivery/discord/game_handler_test.go` (`TestGameHandler_RecordGameCompletion`).
  3. Membuktikan RED test gagal pada implementasi stub kosong (XP=0, streak=0, achievements=0).
  4. Menyelesaikan logika `RecordGameCompletion` yang memanggil `GetOrCreateByDiscordID`, `RecordDailyActivity`, `AwardXP`, dan `CheckAndUnlockAchievements`.
  5. Menghubungkan seluruh 15 mini-game dan radovid diary (`radovid_diary.go`, `game_boss_fight.go`, `game_five_word.go`, `game_secret_mission.go`, `game_transformation.go`, `game_movie_review.go`, `game_news_reporter.go`, `game_dream_house.go`, `game_job_interview.go`, `game_roleplay.go`, `game_one_minute.go`, `game_story_relay.go`, `interaction_wyr_button.go`, `interaction_taboo_modal.go`, `command_guess_20q.go`, `command_ask_20q.go`) ke `RecordGameCompletion` dan menyertakan achievement embed jika ada unlock baru.
  6. Menginisialisasi `awardXPUseCase`, `recordDailyUseCase`, dan `checkUnlockUseCase` di `cmd/bot/main.go` serta menginjeksi ke `gameHandler`.
  7. Memverifikasi seluruh unit test suite lulus dengan race detector (`go test -race ./...`).
- **Ringkasan Berkas Terdampak:**
  - 1 berkas `cmd/bot/main.go`
  - 17 berkas di `internal/delivery/discord/*.go`
  - 1 berkas test baru `internal/delivery/discord/game_handler_test.go`

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v -run TestGameHandler_RecordGameCompletion ./internal/delivery/discord/...`
- **Output Terminal Aktual:**
  ```text
  === RUN   TestGameHandler_RecordGameCompletion
  --- PASS: TestGameHandler_RecordGameCompletion (0.01s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.255s
  ```
- **Pengujian Penuh Lintas Modul:** `go test -race ./...`
- **Output:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/config	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	1.613s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/http	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/achievement	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/diary	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/feedback	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/gamemaster	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/notification	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/progress	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/session	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/user	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/achievement	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/admin	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/feedback	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/notification	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/progress	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/user	(cached)
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `go test -v -run TestGameHandler_RecordGameCompletion ./internal/delivery/discord/...`
  - Pesan Kegagalan:
    ```text
    expected TotalXP > 0 after game completion, got 0
    expected StreakDays > 0 after game completion, got 0
    expected at least 1 achievement unlocked (e.g. first_game, boss_slayer), got 0
    FAIL	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.270s
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih. Tidak ada baris terkorupsi di SQLite karena tabel `user_progress`, `user_xp_log`, dan `achievements` sebelumnya belum pernah diisi oleh alur delivery Discord.
- **Rincian Temuan:** Skema database dan relasi user ID terbukti siap menampung data tanpa perlu migrasi data retroaktif manual.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):**
  - HG-1 (No Regression): Seluruh test suite lulus 100% (`go test -race ./...`).
  - HG-2 (Clean Build): `go build ./cmd/bot` berhasil tanpa peringatan dan `go vet ./...` bersih.
  - HG-3 (Zero Comments): Kebijakan Zero-Comment ditaati 100% pada semua kode Go baru dan yang diubah.
  - HG-4 (Audit Artifact): Tiket dan plan diperbarui dengan bukti aktual.
- **Must Probe Status (MP-1 s/d MP-7):**
  - MP-1: Boundary values diverifikasi (baseXP <= 0 aman, difficulty mapping diverifikasi).
  - MP-2: Concurrency aman, mutasi data SQLite melalui repository terisolasi.
  - MP-3: Graceful fallback jika progression dependencies bernilai nil.
- **Daftar Belum Diuji (Untested List - O-16):**
  - Real Discord API live network calls (menggunakan unit test dengan mock/in-memory handler & sqlite memory).


