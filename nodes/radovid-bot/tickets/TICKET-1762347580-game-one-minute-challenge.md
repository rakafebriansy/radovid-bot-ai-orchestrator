---
id: TICKET-1762347580
plan_id: PLAN-1762347580
title: Game 6: One Minute Challenge
status: Todo
priority: High
labels: [Game, Fluency, Writing, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin mengikuti tantangan kelancaran berekspresi (One Minute Challenge) dengan topik tak terduga dan batas waktu ketat 60 detik,
Sehingga saya dapat melatih kecepatan berpikir dan kelancaran (fluency) menyusun kalimat bahasa Inggris secara spontan tanpa rasa takut salah.

Game 6: One Minute Challenge menitikberatkan pada aspek kelancaran (fluency) dan produksi bahasa Inggris spontan, bukan pada kesempurnaan gramatikal semata. Bot menyajikan topik acak yang menarik, mengaktifkan timer 60 detik di Discord, dan menerima paragraf ekspresif pengguna. AI kemudian mengevaluasi kuantitas kata (word count), kekayaan kosakata (lexical variety), koherensi gagasan, dan memberikan skor kelancaran secara objektif.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347580-game-one-minute-challenge.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] AI Game Master menghasilkan topik impromptu yang kreatif dan memancing imajinasi secara dinamis
- [ ] Timer 60 detik ditampilkan di Discord Embed menggunakan format Discord relative timestamp (`<t:TIMESTAMP:R>`)
- [ ] Tombol submit memicu modal penulisan teks dengan validasi batas waktu (submission setelah deadline ditolak atau diberi penalti)
- [ ] Sistem menghitung metrik kuantitatif: total jumlah kata (word count) dan rasio variasi leksikal (unique words ratio)
- [ ] AI mengevaluasi aspek kualitatif: kelancaran alur pemikiran, koherensi paragraf, serta saran ekspresi lanjutan
- [ ] Hasil evaluasi disajikan dalam embed terstruktur dengan badge skor kelancaran (Speed & Fluency Score)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/one_minute.go`
- `../../../../radovid-bot-api/internal/usecase/game/one_minute.go`
- `../../../../radovid-bot-api/internal/usecase/game/one_minute_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/one_minute.tmpl`
- `../../../../radovid-bot-api/internal/delivery/discord/game_one_minute.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

