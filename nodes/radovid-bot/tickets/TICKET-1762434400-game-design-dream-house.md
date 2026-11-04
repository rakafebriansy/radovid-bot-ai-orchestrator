---
id: TICKET-1762434400
plan_id: PLAN-1762434400
title: "Game 15: Design Your Dream House"
status: Todo
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
- [ ] Game Type `design_dream_house` terdaftar di enum game dan pilihan menu `/radovid start`
- [ ] Sesi game mengelola 3 tahapan desain terstruktur: Eksterior & Lanskap, Tata Ruang Spasial Interior, dan Ruang Khusus Personal
- [ ] Pemain menginput konsep deskriptif tiap tahapan melalui formulir Discord Modal
- [ ] AI Game Master (Ollama Gemma 3 4b) memindai penggunaan preposisi spasial dan istilah estetika arsitektural
- [ ] AI evaluator menghasilkan umpan balik terstruktur: skor deskripsi spasial (1-10), skor kekayaan kosakata sensorik (1-10), daftar 3 kata sifat arsitektural lanjutan baru, dan rangkuman sintetis (*architectural synthesis narration*)
- [ ] Penambahan reward XP gamifikasi berdasarkan kelengkapan deskripsi dan kekayaan leksikal
- [ ] Unit testing terhadap alur state mesin tahapan desain, ekstraksi terminologi spasial, dan struktur embed respons

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/dream_house.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/dream_house_game.go`
- `../../../../radovid-bot-api/internal/usecase/game/dream_house_game_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
