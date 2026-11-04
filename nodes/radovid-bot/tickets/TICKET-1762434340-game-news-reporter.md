---
id: TICKET-1762434340
plan_id: PLAN-1762434340
title: "Game 14: News Reporter"
status: Todo
priority: High
labels: [Game, Ollama, AI, Discord, Speaking, FormalEnglish]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris Tingkat Menengah ke Atas,
Saya ingin memainkan simulasi "News Reporter" di mana AI Game Master memberikan serangkaian fakta peristiwa mentah (5W1H) dan saya harus menyusun serta menyiarkan laporan berita formal dalam bahasa Inggris,
Sehingga keterampilan berbicara/menulis menggunakan register formal, struktur piramida terbalik, dan gaya bahasa jurnalistik saya meningkat secara signifikan.

News Reporter melatih ketangkasan merangkai informasi acak menjadi siaran berita yang terstruktur dan berwibawa (*journalistic broadcast format*). Alur permainan mencakup:
1. AI Game Master mengenerate memo berita mendadak (*breaking news dispatch*) yang memuat fakta mentah 5W1H: *Who* (Siapa), *What* (Apa yang terjadi), *When* (Kapan waktu peristiwa), *Where* (Di mana lokasi), *Why* (Mengapa/Pemicu), dan *How* (Bagaimana prosesnya).
2. Pemain berperan sebagai jurnalis lapangan atau pembawa berita studio, menyusun naskah siaran dalam bahasa Inggris menggunakan konvensi jurnalistik: pembuka formal (*"Good evening, this is [Name] reporting live from..."*), kalimat utama/lead sentence, penyampaian fakta secara runtut, dan kalimat penutup (*"Back to you in the studio"*).
3. AI mengevaluasi: ketepatan pencakupan fakta 5W1H, pemilihan kata formal (*journalistic register*), penggunaan pasif yang tepat (*passive voice for objectivity*), dan kohesi antarparagraf.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434340-game-news-reporter.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Game Type `news_reporter` terdaftar di enum game dan pilihan menu `/radovid start`
- [ ] AI Game Master (Ollama Gemma 3 4b) menghasilkan skenario breaking news yang bervariasi (isu lingkungan, sains & teknologi, penemuan arkeologi, kejadian unik) lengkap dengan poin fakta 5W1H
- [ ] Pemain menginput naskah siaran melalui Discord Modal interaktif
- [ ] AI evaluator memvalidasi kelengkapan fakta 5W1H (apakah ada fakta kunci yang tertinggal?)
- [ ] AI memberikan skor register formal (1-10) dan akurasi tata bahasa (1-10) dengan analisis pemakaian passive voice dan transisi formal
- [ ] Menyajikan revisi model broadcast berita yang ideal (*exemplary news script*) sebagai bahan perbandingan belajar pemain
- [ ] Unit testing terhadap pembuatan skenario berita, ekstraksi komponen 5W1H, dan parsing evaluasi berita dari model AI

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/news_reporter.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/news_reporter_game.go`
- `../../../../radovid-bot-api/internal/usecase/game/news_reporter_game_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
