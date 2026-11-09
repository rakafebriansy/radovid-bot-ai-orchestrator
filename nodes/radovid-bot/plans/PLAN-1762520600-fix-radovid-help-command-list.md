---
id: PLAN-1762520600
ticket_id: TICKET-1762520600
title: Implementation Plan - Fix Radovid Help Command to List All Commands and Mini-Games
status: Completed
priority: Medium
---

# Rencana Implementasi: Melengkapi Seluruh Perintah & Mini-Games pada `/radovid help`

## 1. Analisis Arsitektur & Masalah
Saat ini, `HandleRadovidHelp` di `internal/delivery/discord/radovid_help.go` hanya menghasilkan 3 fields statis:
- `/radovid start`
- `/radovid help`
- `Learning Guidelines`

Seluruh perintah yang telah diimplementasikan sejak Fase 2 hingga Fase 5 belum terdokumentasi di dalam respon bantuan bot:
- `/radovid settings [level]`
- `/radovid progress`
- `/radovid diary`
- `/radovid ask question:` & `/radovid guess answer:`
- 15 Mini-Games di bawah `/radovid play <game>`:
  1. `would-you-rather`
  2. `taboo`
  3. `20-questions`
  4. `job-interview`
  5. `roleplay`
  6. `one-minute-challenge`
  7. `story-relay`
  8. `five-word-challenge`
  9. `secret-mission`
  10. `grammar-boss-fight`
  11. `transformation-race`
  12. `movie-review`
  13. `news-reporter`
  14. `dream-house`
  15. `diary`

## 2. Struktur Embed Baru
Agar embed tidak melebihi batasan Discord (1024 karakter per field value dan 6000 karakter per embed), struktur embed dipecah menjadi 5 field modular:
1. `📖 Core Commands` (Core utilities & profiling)
2. `❓ 20 Questions Game Commands` (`/radovid ask` & `/radovid guess`)
3. `🎮 Multiplayer Games (/radovid play <game>)` (Game 2-pemain)
4. `🎯 Solo Practice Games (/radovid play <game>)` (Game latihan mandiri & menulis)
5. `💡 Learning Guidelines` (Prinsip belajar interaktif santai)

## 3. Rencana Pengujian TDD (Red-Green Cycle)
1. **Fase RED:** Buat unit test `radovid_help_test.go` yang memanggil `BuildHelpEmbed()`.
   - Menguji keberadaan seluruh perintah dan 15 mini-game serta memverifikasi panjang setiap field < 1024 karakter.
   - Terbukti GAGAL (**RED**) pada kode lama.
2. **Fase GREEN:** Tulis `BuildHelpEmbed()` di `radovid_help.go` dan integrasikan ke `HandleRadovidHelp`.
   - Lulus 100% (**GREEN**).
3. **Fase REFACTOR & VERIFIKASI:**
   - Jalankan `go test -race ./...`.
   - Jalankan `go vet ./...`.
   - Pastikan Zero-Comment Policy dipatuhi 100%.

## 4. Target Mutasi Berkas
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
|---|---|---|
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_help_test.go` | Unit test verifikasi Red-Green kelengkapan katalog bantuan |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go` | Pembangunan embed lengkap mencakup seluruh perintah dan 15 mini-game |
