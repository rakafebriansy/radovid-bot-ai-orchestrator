---
id: TICKET-1794189577
plan_id: PLAN-1794189577
title: Clean Command Typography, Optimize Newlines, and Remove Redundant Emojis
status: Done
priority: Medium
labels: [Discord, UX, Typography, CleanCode, Presentation]
---

# Deskripsi
Sebagai pengguna dan pemain Radovid Bot,
Saya ingin teks balasan dari setiap perintah disajikan secara bersih, rapi, dengan pemanfaatan baris baru (*newline*) yang optimal untuk kenyamanan membaca, serta bebas dari icon/emoji dekoratif yang tidak perlu,
Sehingga tampilan pesan bot terlihat profesional, modern, mudah dipindai (*scannable*), dan tidak padat/penuh (*cluttered*).

Konteks masalah:
Sebelumnya, banyak judul embed dan nama field pada perintah bot memuat emoji dekoratif berlebihan (misal: `📖`, `❓`, `🎮`, `🎯`, `💡`, `⚙️`, `📊`, `📝`, `🏆`, dll.), serta teks perintah dan opsi game tertulis rapat dengan satu newline sehingga terasa padat di layar pengguna. Pengguna meminta optimalisasi newline di seluruh perintah agar teks bernapas (*spaced readability*) dan eliminasi icon-icon yang tidak esensial.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1794189577-clean-command-typography-and-spacing.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Embed `/radovid help` (`radovid_help.go`) dibersihkan dari emoji dekoratif pada nama field (`Core Commands`, `20 Questions Game Commands`, dll.) dan memanfaatkan double-newline (`\n\n`) antar entri perintah dan game.
- [x] Embed `/radovid settings` (`BuildSettingsEmbed`) dibersihkan dari emoji dekoratif judul/field (`English Learning Settings`, `Proficiency Levels (CEFR)`, `Notice`) dan memformat daftar level dengan jarak newline yang nyaman.
- [x] Embed `/radovid progress` (`BuildProgressEmbed`) dibersihkan dari emoji dekoratif judul/field (`Level & Proficiency`, `Active Streak`, `XP Progress`, `Badges Unlocked`) dan bar progress diberi jarak newline dari angka statistik.
- [x] Embed `/radovid diary` (`BuildDailyDiaryEmbed`) dibersihkan dari emoji dekoratif pada judul/field dan hasil evaluasi per kalimat dipisahkan dengan newline yang rapi.
- [x] Judul dan field pada mini-games (20 Questions, Taboo, Would You Rather, Job Interview, Roleplay, One Minute, Five Word, Story Relay, Secret Mission, Transformation Race, Movie Review, News Reporter, Dream House, Daily Challenge) dibersihkan dari emoji dekoratif berlebih.
- [x] Seluruh unit test terkait diperbarui dan dipastikan lulus 100%.
- [x] Dokumen `manual-testing-playbook.md` diselaraskan dengan tipografi bersih baru.
- [x] Kebijakan Zero-Comment pada seluruh berkas Go tetap terjaga 100%.
- [x] Seluruh unit test suite lulus (`go test -race ./...`) dan build binary berhasil.

## Delivery Gate Verification
- `go test -race ./...` in `radovid-bot-api`: PASS (all 25 packages).
- `go vet ./...` in `radovid-bot-api`: PASS (0 warnings).
- Zero-comment compliance check: PASS (0 comments in Go code).
- `go build ./cmd/bot`: PASS.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
- `docs/manual-testing-playbook.md`
- `CHANGELOG.md`
