---
id: TICKET-1762434220
plan_id: PLAN-1762434220
title: "Game 12: Sentence Transformation Race"
status: Todo
priority: High
labels: [Game, Ollama, AI, Discord, SpeedRace, Grammar]
---

# Deskripsi
Sebagai Pemain / Peserta Pembelajaran Bahasa Inggris,
Saya ingin berkompetisi dalam balapan transformasi kalimat cepat (*Sentence Transformation Race*) di mana AI memberikan kalimat acuan dan instruksi transformasi tata bahasa (seperti aktif ke pasif, langsung ke tidak langsung, atau pergeseran tenses),
Sehingga kelincahan berpikir gramatikal dan ketepatan sintaksis saya terasah di bawah batasan waktu yang kompetitif.

Sentence Transformation Race menguji ketangkasan peserta dalam merekonstruksi kalimat bahasa Inggris sesuai aturan tata bahasa formal tanpa mengubah makna intinya. AI Game Master menyajikan kalimat awal bersama direktif transformasi:
- *Active to Passive Voice* (atau sebaliknya).
- *Direct Speech to Reported (Indirect) Speech*.
- *Tense Shifting* (contoh: Simple Present -> Present Perfect Continuous).
- *Conditional Transformations* (contoh: First Conditional -> Second Conditional).

Setiap ronde dibatasi waktu (misal: 45-60 detik). Pemain yang mengirimkan transformasi dengan akurasi tata bahasa tertinggi dan kecepatan waktu pengiriman tercepat mendapatkan skor tertinggi dan bonus XP kecepatan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434220-game-sentence-transformation-race.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Game Type `sentence_transformation` terdaftar di enum game dan menu `/radovid start`
- [ ] AI Game Master mengenerate tantangan kalimat awal beserta kategori transformasi yang valid sesuai level kesulitan
- [ ] Timer countdown visual ditampilkan pada embed ronde (misal: 45 detik) dengan pencatatan timestamp waktu mulai dan waktu submit respon pemain
- [ ] AI evaluator memverifikasi ketepatan gramatikal hasil transformasi, ekuivalensi semantik (makna kalimat tidak berubah), dan mengembalikan skor akurasi (1-100)
- [ ] Rumus penilaian mengombinasikan akurasi sintaksis dan faktor kecepatan: $\text{RoundScore} = \text{Accuracy} \times \left(1.0 + \frac{\text{TimeRemaining}}{\text{TotalTime}} \times 0.5\right)$
- [ ] Embed hasil ronde menyajikan perbandingan kalimat asli, transformasi pemain, kunci transformasi ideal dari AI, dan rincian skor
- [ ] Unit test algoritma skoring kecepatan, parser response evaluasi AI, dan validasi jenis transformasi

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/transformation.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/transformation_game.go`
- `../../../../radovid-bot-api/internal/usecase/game/transformation_game_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
