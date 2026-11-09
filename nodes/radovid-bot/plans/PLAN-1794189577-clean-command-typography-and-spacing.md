---
id: PLAN-1794189577
ticket_id: TICKET-1794189577
title: Implementation Plan - Clean Command Typography, Optimize Newlines, and Remove Redundant Emojis
status: Completed
priority: Medium
---

# Rencana Implementasi: Pembersihan Tipografi, Spasi Newline, dan Eliminasi Emoji Dekoratif

## 1. Analisis Desain Antarmuka & UX
Dalam antarmuka obrolan Discord, keterbacaan (*readability*) dan kemudahan pemindaian informasi (*scannability*) sangat bergantung pada:
- **Hierarki Spasi Visual:** Blok teks yang berdempetan tanpa jarak baris ekstra (`\n\n`) antar entri atau instruksi membuat pengguna lelah membaca dan melewatkan informasi penting.
- **Kerapian Tipografi:** Penggunaan emoji dekoratif berlebihan pada setiap judul embed dan header field menimbulkan polusi visual (*visual noise*) yang mengaburkan pesan utama.

## 2. Rencana Perubahan Teknis
1. **`internal/delivery/discord/radovid_help.go`:**
   - Bersihkan header field: `Core Commands`, `20 Questions Game Commands`, `Multiplayer Games (/radovid play <game>)`, `Solo Practice Games (/radovid play <game>)`, `Learning Guidelines`.
   - Gunakan `\n\n` antar setiap perintah dan setiap game agar memiliki ruang bernapas.
2. **`internal/infrastructure/discord/game_templates.go`:**
   - `BuildSettingsEmbed`: Bersihkan judul menjadi `English Learning Settings`, header field menjadi `Proficiency Levels (CEFR)` dan `Notice`. Bersihkan icon dari list panduan CEFR dan berikan `\n\n` antar level.
   - `BuildProgressEmbed`: Bersihkan judul menjadi `English Learning Progress — %s`, header field menjadi `Level & Proficiency`, `Active Streak`, `XP Progress (Total: %d XP)`, dan `Badges Unlocked (%d)`. Tambahkan jarak `\n\n` antara bar progress dan teks XP.
   - `BuildDailyDiaryEmbed`: Bersihkan judul menjadi `Daily 5-Sentence Diary — Language Analysis` / `Daily Diary — %s`. Bersihkan header field (`Sentence-by-Sentence Language Feedback`, `Recommended Contextual Vocabulary`, `Reflection & Encouragement`). Pisahkan evaluasi kalimat dengan `\n\n`.
   - Embed umum & mini-games: Bersihkan icon dekoratif pada `SessionStartEmbed`, `FeedbackSummaryEmbed`, `BuildFiveWordChallengeEmbed`, `BuildJobInterviewDomainSelectEmbed`, `BuildRoleplayScenarioSelectEmbed`, `BuildOneMinuteChallengeEmbed`, `BuildSecretMissionBriefingEmbed`, `BuildTransformationRoundEmbed`, `BuildMovieReviewPhaseEmbed`, dan `BuildDailyChallengeEmbed`.
3. **Command Embeds & Tests (20 Questions, Taboo, Would You Rather):**
   - Hapus emoji judul dekoratif di `command_play_20q.go`, `command_ask_20q.go`, `command_guess_20q.go`, `command_play_taboo.go`, dan `command_play_wyr.go`.
   - Selaraskan unit test pada masing-masing test file.
4. **Dokumentasi `docs/manual-testing-playbook.md`:**
   - Perbarui seluruh representasi judul dan header field yang telah dibersihkan.

## 3. Rencana Pengujian TDD
1. Jalankan unit test sebelum dan sesudah perubahan untuk memastikan seluruh string assertion selaras.
2. Pastikan tidak ada field Discord yang melebihi batas 1024 karakter.
3. Jalankan `go test -race ./...` dan `go vet ./...`.
