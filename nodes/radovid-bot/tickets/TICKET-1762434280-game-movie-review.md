---
id: TICKET-1762434280
plan_id: PLAN-1762434280
title: "Game 13: Movie Review"
status: Todo
priority: High
labels: [Game, Ollama, AI, Discord, Writing, Discussion]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris,
Saya ingin mengikuti aktivitas diskusi film terstruktur (*Movie Review*) di mana AI memandu saya melalui serangkaian pertanyaan tentang sinopsis, pengembangan karakter, dan opini kritis film favorit saya,
Sehingga saya dapat memperkaya perbendaharaan kata evaluatif, frasa opini formal, dan kemampuan menyusun argumen kritis dalam bahasa Inggris.

Movie Review adalah aktivitas menulis dan berdiskusi terpandu yang dirancang untuk melatih kemampuan diskursus analitis pengguna. Pemain memilih judul film yang diminati, kemudian AI bertindak sebagai moderator diskusi yang mengajukan 3 fase pertanyaan terstruktur:
1. **Plot Premise & Setting:** Menyajikan sinopsis ringkas tanpa membocorkan spoiler (*concise summary*).
2. **Character & Performance Analysis:** Mengulas motivasi tokoh utama atau kualitas akting aktor menggunakan kosakata deskriptif karakter.
3. **Critical Verdict & Recommendations:** Menyampaikan opini kritis dengan frasa argumentatif (*"From my perspective...", "What struck me most was..."*) serta memberikan rating akhir.

Di akhir sesi, AI memberikan umpan balik mendalam terhadap kekayaan ekspresi evaluatif, kejelasan transisi antarkalimat, dan saran sinonim kata sifat sinematik yang lebih tinggi tingkatannya (*advanced adjectives*).

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434280-game-movie-review.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Game Type `movie_review` terdaftar di enum game dan menu slash command `/radovid start`
- [ ] State sesi mencatat 3 fase review sekuensial: `PhasePremise`, `PhaseCharacters`, `PhaseVerdict`
- [ ] AI Game Master (Ollama Gemma 3 4b) menyesuaikan pertanyaan panduan berdasarkan judul film dan genre yang diinput oleh pemain
- [ ] Pemain mengirimkan respon setiap fase melalui Discord Modal input teks panjang
- [ ] Evaluasi akhir AI menghasilkan laporan komprehensif: skor koherensi, evaluasi ekspresi opini, daftar 3-5 kosakata sinematik alternatif (misal: *captivating*, *monotonous*, *thought-provoking*), dan skor keseluruhan (1-10)
- [ ] Integrasi dengan modul progres: memberikan base XP plus bonus proporsional berdasarkan kualitas ulasan
- [ ] Unit testing terhadap alur transisi fase, parser respon evaluasi ulasan film, dan validasi kelengkapan argumen

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/movie_review.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/movie_review_game.go`
- `../../../../radovid-bot-api/internal/usecase/game/movie_review_game_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
