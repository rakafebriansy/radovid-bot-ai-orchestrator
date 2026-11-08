---
id: TICKET-1762520060
plan_id: PLAN-1762520060
title: "[BUG] Inkonsistensi Format User Primary Key (usr-<discordID> vs <discordID>) Memicu Foreign Key Error"
status: Done
priority: High
severity: Money/Data Loss
labels: [Bug, Database, SQLite, Backend]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Terdapat inkonsistensi format primary key `users.id` antara `UserRepository` dan `SessionRepository`:
- `UserRepository.GetOrCreateByDiscordID` dan `UpdateDifficulty` membuat record di tabel `users` dengan `id = "usr-" + discordID` dan `discord_id = discordID`.
- `SessionRepository.CreateSession` dan `AddParticipant` mengeksekusi `INSERT OR IGNORE INTO users (id, discord_id, username, ...) VALUES (p.UserID, p.UserID, p.UserID, ...)` menggunakan raw `discordID`, lalu memasukkan `p.UserID` ke `session_participants.user_id` yang memiliki constraint `REFERENCES users(id)` dengan `PRAGMA foreign_keys = ON`.
Jika pengguna menjalankan `/radovid settings` atau `/radovid progress` terlebih dahulu, baris di `users` memiliki `id = "usr-<discordID>"`. Saat pengguna tersebut memulai game, `INSERT OR IGNORE` diabaikan karena `UNIQUE(discord_id)` sudah ada, dan `INSERT INTO session_participants` gagal total dengan `FOREIGN KEY constraint failed`. Sebaliknya, jika pengguna bermain game lebih dulu, `username` mereka disimpan sebagai angka ID Discord mentah.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520060-fix-user-pk-foreign-key-mismatch.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Panggil `UserRepository.GetOrCreateByDiscordID(ctx, "12345", "Geralt")` atau jalankan `/radovid settings`.
2. Panggil `SessionRepository.CreateSession` dengan participant `UserID: "12345"`.
3. Amati kegagalan SQLite: `failed to insert participants: FOREIGN KEY constraint failed`.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Pembuatan sesi game dan penambahan partisipan berjalan mulus baik sebelum maupun sesudah pengguna mengakses `/radovid settings` atau `/radovid progress`, serta tidak menimpa `username` asli pengguna dengan ID numerik.
- **Aktual (Actual):** SQLite melempar error `FOREIGN KEY constraint failed` karena ketidakcocokan `users.id` (`usr-12345` vs `12345`).

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] `SessionRepository` dan `UserRepository` menggunakan skema resolusi `users.id` yang konsisten dan menghormati baris user yang sudah ada berdasarkan `discord_id` maupun `id`.
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/infrastructure/database/session_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/session_repository_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/user_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/user_repository_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_settings.go`

---

## 6. AI Execution Log & Output
*Bagian ini WAJIB diisi oleh AI Agent SAAT dan SETELAH memperbaiki bug.*

- **Penyebab Akar (Root Cause):**
  - `UserRepository.GetOrCreateByDiscordID` dan `UpdateDifficulty` membuat user dengan format `ID = "usr-" + discordID` dan `DiscordID = discordID`.
  - Sementara `SessionRepository.CreateSession` dan `AddParticipant` melakukan `INSERT OR IGNORE` dengan `ID = UserID` (Discord ID numerik) dan memasukkan `p.UserID` ke `session_participants.user_id` yang memiliki foreign key `REFERENCES users(id)`.
  - Jika pengguna lebih dulu mengakses `/radovid settings` atau `/radovid progress`, row `users` tersimpan dengan `id = "usr-<discordID>"`. Saat membuat game session, `INSERT OR IGNORE` diabaikan karena `UNIQUE(discord_id)` sudah ada, dan `INSERT INTO session_participants` gagal total dengan `FOREIGN KEY constraint failed`. Sebaliknya jika sesi game dibuat lebih dulu, `username` tertulis sebagai Discord ID dan tidak pernah diperbarui saat `GetOrCreateByDiscordID` dipanggil kemudian.
- **Langkah Perbaikan:**
  1. Menambahkan dua unit test baru: `TestSessionRepository_CreateSession_AfterUserRepositoryCreation` di `session_repository_test.go` dan `TestUserRepository_SessionCompatibility` di `user_repository_test.go`.
  2. Memverifikasi kegagalan tes (RED) pada kode lama: `FOREIGN KEY constraint failed (787)` dan `expected username to be updated to LearnerGamer, got: discord-init-from-game`.
  3. Memperbarui `UserRepository.GetOrCreateByDiscordID` dan `UpdateDifficulty` agar menstandarkan `u.ID = cleanDiscordID` (sehingga `users.id == users.discord_id`) dan memperbarui `username` jika record sebelumnya dibuat oleh sesi game dengan placeholder ID numerik.
  4. Memperbarui `SessionRepository.CreateSession` dan `AddParticipant` agar melakukan pengecekan keberadaan user berdasarkan `discord_id OR id`, menyelesaikan `resolvedUserID`, serta mencoba migrasi `UPDATE users SET id = discord_id WHERE discord_id = ?` jika terdapat format ID lama.
  5. Memperbarui `radovid_settings.go` agar tidak lagi menyematkan prefix statis `"usr-"`.
  6. Menjalankan seluruh pengujian `database` dan test suite lintas modul dengan race detector (`go test -race ./...`) 100% PASS.
- **Ringkasan Berkas Terdampak:**
  - `internal/infrastructure/database/user_repository.go`
  - `internal/infrastructure/database/user_repository_test.go`
  - `internal/infrastructure/database/session_repository.go`
  - `internal/infrastructure/database/session_repository_test.go`
  - `internal/delivery/discord/radovid_settings.go`

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v -run "TestSessionRepository_CreateSession_AfterUserRepositoryCreation|TestUserRepository_SessionCompatibility" ./internal/infrastructure/database/...`
- **Output Terminal Aktual:**
  ```text
  === RUN   TestSessionRepository_CreateSession_AfterUserRepositoryCreation
  --- PASS: TestSessionRepository_CreateSession_AfterUserRepositoryCreation (0.01s)
  === RUN   TestUserRepository_SessionCompatibility
  --- PASS: TestUserRepository_SessionCompatibility (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	0.623s
  ```
- **Pengujian Penuh Lintas Modul:** `go test -race ./...`
- **Output:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/config	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	1.597s
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
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	2.607s
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
  - Perintah: `go test -v -run TestSessionRepository_CreateSession_AfterUserRepositoryCreation ./internal/infrastructure/database/...`
  - Pesan Kegagalan:
    ```text
    === RUN   TestSessionRepository_CreateSession_AfterUserRepositoryCreation
        session_repository_test.go:281: expected CreateSession to succeed after user created via user repo, got: failed to insert participants: constraint failed: FOREIGN KEY constraint failed (787)
    --- FAIL: TestSessionRepository_CreateSession_AfterUserRepositoryCreation (0.01s)
    FAIL
    ```
  - Perintah 2: `go test -v -run TestUserRepository_SessionCompatibility ./internal/infrastructure/database/...`
  - Pesan Kegagalan 2:
    ```text
    === RUN   TestUserRepository_SessionCompatibility
        user_repository_test.go:147: expected username to be updated to LearnerGamer, got: discord-init-from-game
    --- FAIL: TestUserRepository_SessionCompatibility (0.00s)
    FAIL
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih. Tidak ada baris terkorupsi di database produksi karena sistem belum dirilis ke publik.
- **Rincian Temuan:** Skema resolusi fleksibel pada `SessionRepository` mendukung migrasi otomatis atau fallback ID lama jika terdapat row dengan prefix `usr-`.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):**
  - HG-1 (No Regression): Seluruh test suite lulus 100% (`go test -race ./...`).
  - HG-2 (Clean Build): `go build ./cmd/bot` berhasil tanpa peringatan dan `go vet ./...` bersih.
  - HG-3 (Zero Comments): Kebijakan Zero-Comment ditaati 100% pada semua kode Go baru dan yang diubah.
  - HG-4 (Audit Artifact): Tiket dan plan diperbarui dengan bukti aktual.
- **Must Probe Status (MP-1 s/d MP-7):**
  - MP-1: Boundary values diverifikasi (user baru, user eksisting, username update).
  - MP-2: Concurrency & foreign keys aman di level transaksi SQLite.
- **Daftar Belum Diuji (Untested List - O-16):**
  - Discord gateway event live (diuji dengan unit test in-memory SQLite).


