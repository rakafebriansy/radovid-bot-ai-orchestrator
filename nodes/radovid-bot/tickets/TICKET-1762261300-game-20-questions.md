---
id: TICKET-1762261300
plan_id: PLAN-1762261300
title: "Game 3: 20 Questions"
status: Todo
priority: High
labels: [Backend, Game, Feature, Discord, Ollama]
---

# Deskripsi
Sebagai pemain Discord yang ingin melatih kemampuan menyusun kalimat tanya (interrogative sentences) dalam bahasa Inggris,
Saya ingin memainkan game "20 Questions" di mana AI Game Master menyimpan topik rahasia dan saya serta rekan pemain mengajukan pertanyaan ya/tidak hingga maksimal 20 pertanyaan untuk menebak kata rahasia tersebut,
Sehingga kami dapat mengasah keterampilan merumuskan pertanyaan deduktif, mengenali kata sifat dan kategori objek dalam bahasa Inggris, serta berkolaborasi memecahkan teka-teki.

Latar belakang teknis: Game 3 ("20 Questions") membutuhkan pengelolaan state tersembunyi yang aman (topik rahasia tidak boleh bocor ke embed pemain), penghitung sisa pertanyaan (*questions counter*) yang terkelola secara atomik dari 20 hingga 0, klasifikasi respons AI ("Yes", "No", atau "Sometimes") terhadap pertanyaan pemain, mekanisme penanganan tebakan akhir (*guess submission*), serta pengungkapan (*reveal*) dan evaluasi akhir permainan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762261300-game-20-questions.md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] AI Game Master menentukan satu topik/benda/orang/tempat rahasia dalam bahasa Inggris dan menyimpannya di internal state sesi (terisolasi dan tidak terlihat di Discord).
- [ ] Pemain dapat mengajukan pertanyaan berbasis ya/tidak menggunakan command `/radovid ask <question>` atau tombol interaktif.
- [ ] AI mengevaluasi pertanyaan terhadap topik rahasia dan memberikan jawaban pasti: "Yes", "No", atau "Sometimes" disertai penjelasan singkat dalam bahasa Inggris.
- [ ] Counter sisa pertanyaan (misal: "17 / 20 questions remaining") ditampilkan secara jelas pada setiap Discord Embed respons.
- [ ] Pemain dapat mengajukan tebakan kata menggunakan command `/radovid guess <answer>`.
- [ ] Game selesai dengan status kemenangan (*WIN*) jika tebakan pemain benar sebelum kuota 20 pertanyaan habis.
- [ ] Game selesai dengan status kekalahan (*GAME OVER*) jika kuota 20 pertanyaan habis tanpa ada tebakan benar.
- [ ] Di akhir permainan, AI Game Master mengungkap topik rahasia (*reveal*) dan menyajikan ringkasan evaluasi perjalanan deduksi pemain dalam bahasa Inggris.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/twenty_questions_entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/twenty_questions.go`
- `../../../../radovid-bot-api/internal/usecase/game/twenty_questions_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. Pending execution.
- **Ringkasan File Terpengaruh:**
  - Pending execution.
- **Catatan & Keputusan Arsitektural:**
  - Pending execution.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v ./internal/domain/game/... ./internal/usecase/game/...`
- **Output Terminal:**
  ```text
  Pending execution.
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** Pending
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** Pending
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): Pending
  - O-08 (Edge Case Atlas): Pending
  - O-09 (Damage Assessment): Pending
  - O-10 (Full Suite Metrics): Pending
  - O-11 (Real Thing Driven): Pending
  - O-12 (Shipping Env Check): Pending
  - Hunt Loop Final Round: Pending
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: Pending
  - CHANGELOG.md: Pending
  - Commit Message Sanity: Pending
  - O-16 Untested List: Pending
