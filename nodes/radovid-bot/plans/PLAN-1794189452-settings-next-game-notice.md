---
id: PLAN-1794189452
ticket_id: TICKET-1794189452
title: Implementation Plan - Add Notice in Settings Embed that Difficulty Changes Affect Subsequent Games
status: Completed
priority: Low
---

# Rencana Implementasi: Menambahkan Pemberitahuan Perubahan Level pada `/radovid settings`

## 1. Analisis Arsitektur & Masalah
Pada sistem game Radovid Bot, sesi permainan diinisialisasi dengan tingkat kesulitan user pada saat game dibuat (`StartGame`, `StartSession`). Ketika seorang pengguna mengubah pengaturan profil melalui `/radovid settings` di tengah sesi permainan yang sedang berlangsung, tingkat kesulitan pada state game aktif di memori/database tidak termutasi di tengah jalan agar tidak merusak konsistensi round yang sedang dimainkan.

Namun pada antarmuka `/radovid settings`, belum terdapat teks pemberitahuan yang menginfokan hal ini kepada pengguna, sehingga berpotensi menimbulkan kebingungan apakah prompt game saat itu langsung berubah atau tidak.

## 2. Rencana Perubahan Teknis
1. **`internal/infrastructure/discord/game_templates.go`:**
   - Pada fungsi `BuildSettingsEmbed`, tambahkan field baru ke dalam embed builder:
     ```go
     builder.AddField(
         "ℹ️ Notice",
         "Difficulty changes will take effect starting from your next game session. Any active game currently in progress will finish with its initial difficulty.",
         false,
     )
     ```
2. **`internal/infrastructure/discord/game_templates_test.go`:**
   - Pada unit test `TestBuildSettingsEmbed`, tambahkan assertion yang memvalidasi bahwa embed yang dihasilkan mengandung field pemberitahuan tersebut.
3. **`nodes/radovid-bot/docs/manual-testing-playbook.md`:**
   - Perbarui seksi 4.2 (`/radovid settings`) pada bagian "Tampilan yang Muncul" untuk mencatat keberadaan field `ℹ️ Notice`.

## 3. Rencana Pengujian TDD
1. **Fase RED:** Tambahkan assertion pemberitahuan ke `TestBuildSettingsEmbed` di `game_templates_test.go`. Jalankan `go test` -> Dipastikan gagal (**RED**).
2. **Fase GREEN:** Tambahkan field `ℹ️ Notice` pada `BuildSettingsEmbed` di `game_templates.go`. Jalankan `go test` -> Dipastikan lolos (**GREEN**).
3. Jalankan `go test -race ./...` dan `go vet ./...` untuk memastikan seluruh test suite tetap sehat.
