---
id: TICKET-1762347760
plan_id: PLAN-1762347760
title: Game 9: Five Word Writing Challenge
status: Done
priority: High
labels: [Game, Writing, Vocabulary, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin ditantang menulis satu paragraf bahasa Inggris yang padu dan bermakna dengan menggunakan 5 kata acak yang diberikan oleh bot,
Sehingga saya dapat melatih fleksibilitas penggunaan kosakata baru, struktur kalimat yang variatif, serta kemampuan menyusun konteks tulisan yang koheren.

Game 9: Five Word Writing Challenge adalah latihan menulis kreatif ringkas. Bot membangkitkan 5 kata bahasa Inggris acak yang menarik (kombinasi kata benda, kata kerja, dan kata sifat). Pemain kemudian menyusun satu paragraf utuh yang wajib menyertakan kelima kata tersebut. Sistem melakukan verifikasi algoritmik untuk memastikan keberadaan kata-kata target, dan kemudian menggunakan AI Ollama untuk menilai koherensi logika cerita, ketepatan kolokasi, serta kualitas tata bahasa paragraf yang dihasilkan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347760-game-five-word-challenge.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] AI Game Master / sistem menyediakan generator 5 kata acak yang berimbang (part-of-speech beragam) dan adaptif terhadap tingkat kesulitan
- [x] Tampilan tantangan di Discord Embed menyajikan 5 kata target secara visual jelas beserta kelas kata (noun/verb/adj)
- [x] Input paragraf diterima melalui Discord Modal teks panjang (max 1000 karakter)
- [x] Verifikasi algoritmik deterministik di sisi Go untuk memeriksa keberadaan kelima kata target (mendukung perbandingan case-insensitive dan variasi infleksi dasar)
- [x] Evaluasi AI menganalisis koherensi paragraf, kealamian integrasi kata, dan ketepatan tata bahasa
- [x] Skor akhir dihitung secara transparan: status pemenuhan kelima kata (50%) + skor koherensi & grammar AI (50%)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/game/five_word.go`
- `../../../../radovid-bot-api/internal/domain/game/five_word_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/five_word.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/five_word.go`
- `../../../../radovid-bot-api/internal/usecase/game/five_word_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_five_word.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_five_word_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
- **Status:** Completed
- **Branch:** `main` (sesuai konfirmasi eksplisit pengguna)
- **Ringkasan Delivery Gate:**
  1. *Higienitas Kode:* Zero-comment policy diterapkan penuh pada seluruh kode Go baru; tidak ada komentar non-fungsional; fail-fast validation diterapkan; seluruh path dokumentasi mematuhi kebijakan relative path.
  2. *Hard Gate Testing:* Seluruh unit test Go pada layer domain, infrastructure ollama, infrastructure discord, usecase, dan delivery discord berstatus 100% PASS (`go test ./...` sukses tanpa regresi).
  3. *Must Probe Loop:* 14 kasus ekstrem dievaluasi dan diuji (input kosong/whitespace, teks terlalu pendek < 20 karakter, batas maksimum 1000 karakter, case-insensitivity, tanda baca menempel, batasan kata boundary terhadap substring palsu, variasi bentuk infleksi jamak/lampau/gerund per kelas kata, toleransi offline fallback Ollama, isolasi pengguna submission).
  4. *Kompilasi & Vet:* `go build ./cmd/bot` dan `go vet ./...` sukses tanpa galat dan tanpa peringatan.
- **Untested List:**
  - Koneksi live ke Discord Gateway menggunakan bot token asli pada lingkungan Discord produksi (diuji melalui unit tests dengan mock sessions dan simulated interaction payloads).
  - Inferensi langsung ke instance Ollama lokal langsung pada port 11434 (diuji melalui integration test mock LLM Client dan static fallback generation test).

