---
id: TICKET-1762347520
plan_id: PLAN-1762347520
title: Game 5: Roleplay
status: Done
priority: High
labels: [Game, Speaking, Roleplay, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin melakukan simulasi percakapan interaktif berbasis giliran (turn-based) dalam berbagai skenario kehidupan sehari-hari (seperti di restoran, bandara, hotel, atau klinik dokter),
Sehingga saya dapat melatih kelancaran percakapan spontan dalam bahasa Inggris sementara AI memberikan koreksi tata bahasa secara halus tanpa merusak imersi situasi.

Game 5: Roleplay menyajikan skenario situasional realistis di mana AI mengambil persona karakter lawan bicara (misalnya pelayan restoran, petugas imigrasi, atau dokter). Pengguna berdialog bolak-balik hingga maksimal 10 giliran atau hingga objektif skenario tercapai. Sistem dirancang untuk memberikan koreksi gramatikal minor secara non-intrusif pada elemen visual pendukung, kemudian menyajikan evaluasi bahasa menyeluruh di akhir sesi.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347520-game-roleplay.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Tersedia katalog skenario sehari-hari (Restaurant, Airport Check-in, Hotel Reservation, Doctor's Appointment, Shopping Inquiry)
- [x] AI Game Master menginisialisasi skenario pembuka dengan konteks yang jelas, tujuan misi percakapan, serta persona karakter yang konsisten
- [x] Dialog berlangsung secara turn-based (pengguna memasukkan kalimat ucapan via tombol balasan / modal Discord)
- [x] Koreksi tata bahasa dan frasa disajikan secara halus (subtle correction) pada footer embed atau side-note tanpa memutus dialog persona AI
- [x] Sesi berakhir secara otomatis saat tujuan skenario terselesaikan atau telah mencapai batas maksimum 10 putaran dialog
- [x] Embed penutup sesi menyajikan ringkasan performa dialog, daftar koreksi grammar yang terkumpul, dan skor naturalness percakapan

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/roleplay.go`
- `../../../../radovid-bot-api/internal/usecase/game/roleplay.go`
- `../../../../radovid-bot-api/internal/usecase/game/roleplay_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/roleplay.tmpl`
- `../../../../radovid-bot-api/internal/delivery/discord/game_roleplay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Completed
- **Timestamp:** 2026-11-06 00:05:00
- **Hasil Eksekusi:**
  1. Membuat entitas domain skenario dan riwayat percakapan `internal/domain/game/roleplay.go` serta unit test komprehensif `roleplay_test.go`. Katalog 5 skenario (`restaurant`, `airport`, `hotel`, `doctor`, `shopping`) dengan batas 10 turn dan kalkulasi skor naturalness.
  2. Menyusun Ollama prompt template `internal/infrastructure/ollama/prompts/roleplay.tmpl` yang menegakkan delimiter `REPLY:`, `TIP:`, dan `GOAL_REACHED:`.
  3. Mengimplementasikan parser `parseRoleplay` di `internal/infrastructure/ollama/parser.go` dan bank fallback skenario situasional `roleplayFallbacks` di `internal/infrastructure/ollama/fallback.go`.
  4. Membangun use case `internal/usecase/game/roleplay.go` (`RoleplayUseCase`) yang mencakup pemotongan konteks percakapan lampau (windowing 6 turn), evaluasi early goal, transisi sesi, dan unit test `roleplay_test.go`.
  5. Menambahkan Discord Embed template `BuildRoleplayScenarioSelectEmbed`, `BuildRoleplayTurnEmbed`, dan `BuildRoleplaySummaryEmbed` pada `internal/infrastructure/discord/game_templates.go` serta pengujian pada `game_templates_test.go`.
  6. Mengimplementasikan delivery Discord handler `internal/delivery/discord/game_roleplay.go` dan unit test `game_roleplay_test.go`.
  7. Mendaftarkan subcommand `roleplay` di `command_def.go`, menghubungkan perutean di `command_play_wyr.go` dan `game_handler.go`, serta wiring komponen di `cmd/bot/main.go`.
  8. Seluruh pengujian (`go test -count=1 ./...`), `go vet ./...`, dan `go build ./...` lulus 100% dengan mematuhi Zero-Comment Policy.

