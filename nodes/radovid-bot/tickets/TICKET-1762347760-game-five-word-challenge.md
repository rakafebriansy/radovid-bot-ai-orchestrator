---
id: TICKET-1762347760
plan_id: PLAN-1762347760
title: Game 9: Five Word Writing Challenge
status: Todo
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
- [ ] AI Game Master / sistem menyediakan generator 5 kata acak yang berimbang (part-of-speech beragam) dan adaptif terhadap tingkat kesulitan
- [ ] Tampilan tantangan di Discord Embed menyajikan 5 kata target secara visual jelas beserta kelas kata (noun/verb/adj)
- [ ] Input paragraf diterima melalui Discord Modal teks panjang (max 1000 karakter)
- [ ] Verifikasi algoritmik deterministik di sisi Go untuk memeriksa keberadaan kelima kata target (mendukung perbandingan case-insensitive dan variasi infleksi dasar)
- [ ] Evaluasi AI menganalisis koherensi paragraf, kealamian integrasi kata, dan ketepatan tata bahasa
- [ ] Skor akhir dihitung secara transparan: status pemenuhan kelima kata (50%) + skor koherensi & grammar AI (50%)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/five_word.go`
- `../../../../radovid-bot-api/internal/usecase/game/five_word.go`
- `../../../../radovid-bot-api/internal/usecase/game/five_word_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/five_word.tmpl`
- `../../../../radovid-bot-api/internal/delivery/discord/game_five_word.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

