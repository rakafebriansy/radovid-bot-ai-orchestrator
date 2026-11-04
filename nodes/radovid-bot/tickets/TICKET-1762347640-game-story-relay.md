---
id: TICKET-1762347640
plan_id: PLAN-1762347640
title: Game 7: Story Relay
status: Todo
priority: High
labels: [Game, Storytelling, Multiplayer, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin bermain merangkai cerita kolaboratif bergantian bersama teman dan AI Game Master (Story Relay),
Sehingga kami dapat melatih kemampuan menulis naratif bahasa Inggris, memicu kreativitas dengan plot twist tak terduga dari AI, dan menerima umpan balik gramatikal tanpa memutus keseruan cerita.

Game 7: Story Relay adalah aktivitas kolaboratif berbasis giliran (turn-based) untuk 2 pemain dan 1 AI Game Master. AI mengawali cerita dengan paragraf pembuka yang dramatis. Pemain pertama menyumbangkan kelanjutan cerita (minimal 1-2 kalimat), AI merespons dengan menjembatani cerita serta menyisipkan plot twist yang menarik, lalu giliran berpindah ke pemain kedua. Siklus ini berulang hingga babak klimaks, di mana AI menyusun epilog cerita utuh disertai evaluasi tata bahasa untuk masing-masing kontribusi pemain.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347640-game-story-relay.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Manajemen antrean giliran strictly turn-based antara Player 1, AI Game Master, dan Player 2
- [ ] AI Game Master mengawali sesi dengan paragraf pembuka (story hook) yang kaya narasi dan deskripsi latar
- [ ] Validasi kontribusi pemain via Discord Modal: minimal 1 kalimat dan maksimal 300 karakter per giliran
- [ ] AI secara cerdas menyambung cerita pemain, menjaga konsistensi logika naratif, serta menyisipkan plot twist tak terduga
- [ ] Setiap giliran pemain dianalisis secara background untuk mendeteksi kesalahan tata bahasa yang signifikan
- [ ] Pada akhir sesi (setelah 6 putaran kombinasi), bot merangkum seluruh cerita menjadi satu novellet utuh beserta kartu review bahasa untuk kedua pemain

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/story_relay.go`
- `../../../../radovid-bot-api/internal/usecase/game/story_relay.go`
- `../../../../radovid-bot-api/internal/usecase/game/story_relay_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/story_relay.tmpl`
- `../../../../radovid-bot-api/internal/delivery/discord/game_story_relay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

