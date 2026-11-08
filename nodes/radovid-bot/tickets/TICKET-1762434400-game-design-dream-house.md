---
id: TICKET-1762434400
plan_id: PLAN-1762434400
title: "Game 15: Design Your Dream House"
status: Done
priority: High
labels: [Game, Ollama, AI, Discord, CreativeWriting, SpatialVocabulary]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris,
Saya ingin mengikuti game kreatif "Design Your Dream House" di mana saya merancang dan mendeskripsikan rumah impian saya langkah demi langkah,
Sehingga perbendaharaan kata deskriptif arsitektur, preposisi spasial (*prepositions of place*), dan kosakata estetika sensorik saya berkembang secara mendalam.

Design Your Dream House adalah aktivitas penulisan deskriptif imajinatif terpandu. Pemain dipandu melalui proses bertahap untuk memvisualisasikan hunian impian mereka dalam bahasa Inggris:
1. **Exterior & Surroundings:** Menentukan gaya arsitektur (misal: *minimalist Scandinavian*, *rustic Victorian cottage*), latar geografis (*perched upon a cliff*, *tucked away in a tranquil pine forest*), dan material luar bangunan.
2. **Interior Layout & Spatial Relations:** Merinci tata ruang, penataan cahaya alami, dan penempatan ruangan menggunakan preposisi spasial (*adjacent to*, *nestled beside*, *overlooking the courtyard*).
3. **Signature Sanctuary Room:** Mendeskripsikan satu ruangan khusus yang paling personal (misal: perpustakaan pribadi, studio musik, observatorium atap) lengkap dengan detail atmosfer dan estetika.

AI Game Master bertindak sebagai kurator arsitektur yang menganalisis kekayaan kosakata visual/spasial, memberikan saran kata sifat yang lebih kaya nuansa (*evocative adjectives*), dan merangkum deskripsi pemain menjadi narasi arsitektural yang elegan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434400-game-design-dream-house.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Game Type `design_dream_house` terdaftar di enum game dan pilihan menu `/radovid play dream-house`
- [x] Sesi game mengelola 3 tahapan desain terstruktur: Eksterior & Lanskap, Tata Ruang Spasial Interior, dan Ruang Khusus Personal
- [x] Pemain menginput konsep deskriptif tiap tahapan melalui formulir Discord Modal
- [x] AI Game Master (Ollama Gemma 3 4b) memindai penggunaan preposisi spasial dan istilah estetika arsitektural
- [x] AI evaluator menghasilkan umpan balik terstruktur: skor deskripsi spasial (1-10), skor kekayaan kosakata sensorik (1-10), daftar 3 kata sifat arsitektural lanjutan baru, dan rangkuman sintetis (*architectural synthesis narration*)
- [x] Penambahan reward XP gamifikasi berdasarkan kelengkapan deskripsi dan kekayaan leksikal
- [x] Unit testing terhadap alur state mesin tahapan desain, ekstraksi terminologi spasial, format embed Discord, parser respons AI, dan use case orkestrasi

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/dream_house.go`
- `../../../../radovid-bot-api/internal/domain/game/dream_house_test.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/entity_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/dream_house.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/dream_house.go`
- `../../../../radovid-bot-api/internal/usecase/game/dream_house_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_dream_house.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_dream_house_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
- **Status:** Completed
- **Tanggal:** 2026-11-08 08:15:00
- **Ringkasan Eksekusi:**
  1. Menambahkan enum `GameTypeDesignDreamHouse = "design_dream_house"` pada `internal/domain/game/entity.go` beserta unit test di `entity_test.go`.
  2. Mengimplementasikan model domain `internal/domain/game/dream_house.go` (`HouseDesignPhase`, `SpatialPrepositions`, `DetectSpatialPrepositions`, `HouseThemeOption`, `GetDefaultHouseThemes`, `ArchitecturalAssessment`, `HouseDesignState`, `GetHousePhaseGuidingPrompt`, `ValidateHouseDescription`, `CalculateDreamHouseScore`, `CalculateDreamHouseXP`) dan unit test di `dream_house_test.go` (100% PASS).
  3. Menyusun prompt template Ollama `internal/infrastructure/ollama/prompts/dream_house.tmpl` dengan unit test rendering di `embed_test.go`.
  4. Menambahkan parser respons terstruktur `parseDreamHouse` dan helper konversi domain `ParseDreamHouseEvaluation` pada `internal/infrastructure/ollama/parser.go` serta pengujian di `parser_test.go`.
  5. Menambahkan skenario fallback deterministik arsitektur per tingkat kesulitan `dreamHouseFallbacks` pada `internal/infrastructure/ollama/fallback.go` dan pengujian di `fallback_test.go`.
  6. Mengimplementasikan use case layer `DreamHouseUseCase` pada `internal/usecase/game/dream_house.go` mencakup state machine multi-fase (Exterior, Interior, Sanctuary, Completed), evaluasi LLM Ollama dengan timeout & fallback, serta penambahan XP proporsional. Unit test lengkap di `dream_house_test.go` (100% PASS).
  7. Membangun Discord embed visual `BuildDreamHousePhaseEmbed` dan `BuildDreamHouseEvaluationEmbed` di `internal/infrastructure/discord/game_templates.go` beserta pengujian di `game_templates_test.go`.
  8. Menambahkan subcommand slash `dream-house` (opsi `theme` dan `difficulty`) di `command_def.go` dan router di `command_play_wyr.go`.
  9. Mengimplementasikan delivery handlers `HandlePlayDreamHouse`, `HandleDreamHouseComponentInteraction`, dan `HandleDreamHouseModalSubmit` pada `internal/delivery/discord/game_dream_house.go` beserta pengujian di `game_dream_house_test.go`.
  10. Menginjeksi `dreamHouseUseCase` ke `GameHandler`, memperbarui seluruh test suite delivery handler, dan mendaftarkan prefix `dreamhouse:` di `cmd/bot/main.go`.
  11. Verifikasi kelulusan 100% seluruh test suite pada 23 packages di `radovid-bot-api`, `go vet ./...`, dan kepatuhan mutlak zero-comment policy.
