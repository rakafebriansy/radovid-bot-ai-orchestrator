---
id: TICKET-1762261300
plan_id: PLAN-1762261300
title: "Game 3: 20 Questions"
status: Done
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
- [x] AI Game Master menentukan satu topik/benda/orang/tempat rahasia dalam bahasa Inggris dan menyimpannya di internal state sesi (terisolasi dan tidak terlihat di Discord).
- [x] Pemain dapat mengajukan pertanyaan berbasis ya/tidak menggunakan command `/radovid ask <question>` atau tombol interaktif.
- [x] AI mengevaluasi pertanyaan terhadap topik rahasia dan memberikan jawaban pasti: "Yes", "No", atau "Sometimes" disertai penjelasan singkat dalam bahasa Inggris.
- [x] Counter sisa pertanyaan (misal: "17 / 20 questions remaining") ditampilkan secara jelas pada setiap Discord Embed respons.
- [x] Pemain dapat mengajukan tebakan kata menggunakan command `/radovid guess <answer>`.
- [x] Game selesai dengan status kemenangan (*WIN*) jika tebakan pemain benar sebelum kuota 20 pertanyaan habis.
- [x] Game selesai dengan status kekalahan (*GAME OVER*) jika kuota 20 pertanyaan habis tanpa ada tebakan benar.
- [x] Di akhir permainan, AI Game Master mengungkap topik rahasia (*reveal*) dan menyajikan ringkasan evaluasi perjalanan deduksi pemain dalam bahasa Inggris.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/twenty_questions_entity.go`
- `../../../../radovid-bot-api/internal/domain/game/twenty_questions_entity_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/twenty_questions.go`
- `../../../../radovid-bot-api/internal/usecase/game/twenty_questions_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output

- **Langkah Teknis Tereksekusi:**
  1. Membuat model domain `twenty_questions_entity.go` dan suite unit test `twenty_questions_entity_test.go` untuk menangani kuota 20 pertanyaan, pencatatan pertanyaan, validasi semantik, tebakan kata, serta state mutasi thread-safe.
  2. Mengimplementasikan `TwentyQuestionsUseCase` di `twenty_questions.go` dan suite komprehensif `twenty_questions_test.go` dengan integrasi AI Game Master untuk evaluasi yes/no/sometimes/invalid, normalisasi string tebakan, serta proteksi kerahasiaan topik.
  3. Mendaftarkan sub-command `/radovid ask` dan `/radovid guess` serta sub-command group `play 20-questions` di `command_def.go` dan `handler.go`.
  4. Mengimplementasikan layer delivery Discord untuk 20 Questions:
     - `command_play_20q.go`: inisiasi sesi, lobby matching, buttons ("Ask a Question", "Make a Guess"), modal launchers, dan routing modal submit.
     - `command_ask_20q.go`: eksekusi tanya jawab dengan countdown kuota dan pengungkapan jika habis.
     - `command_guess_20q.go`: eksekusi tebakan langsung, pengungkapan topik saat menang, dan penyelesaian sesi.
  5. Menulis unit test lengkap untuk layer delivery: `command_play_20q_test.go`, `command_ask_20q_test.go`, dan `command_guess_20q_test.go`.
  6. Memperbarui wiring bot di `cmd/bot/main.go` dan meregistrasikan prefix `tq:` untuk interaction component dan modal.
  7. Memvalidasi Zero-Comment Policy, `go vet`, dan `go test -count=1 ./...` (100% PASS).

- **Ringkasan File Terpengaruh:**
  - `radovid-bot-api/internal/domain/game/entity.go`
  - `radovid-bot-api/internal/domain/game/twenty_questions_entity.go`
  - `radovid-bot-api/internal/domain/game/twenty_questions_entity_test.go`
  - `radovid-bot-api/internal/usecase/game/twenty_questions.go`
  - `radovid-bot-api/internal/usecase/game/twenty_questions_test.go`
  - `radovid-bot-api/internal/delivery/discord/command_def.go`
  - `radovid-bot-api/internal/delivery/discord/command_test.go`
  - `radovid-bot-api/internal/delivery/discord/handler.go`
  - `radovid-bot-api/internal/delivery/discord/game_handler.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_20q.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`
  - `radovid-bot-api/internal/delivery/discord/command_ask_20q.go`
  - `radovid-bot-api/internal/delivery/discord/command_ask_20q_test.go`
  - `radovid-bot-api/internal/delivery/discord/command_guess_20q.go`
  - `radovid-bot-api/internal/delivery/discord/command_guess_20q_test.go`
  - `radovid-bot-api/cmd/bot/main.go`

- **Catatan & Keputusan Arsitektural:**
  - Topik rahasia tidak pernah dikirimkan pada Discord Embed selama permainan berlangsung; hanya diekspos saat kemenangan tercapai atau kuota 20 pertanyaan habis.
  - Pertanyaan terbuka (*open-ended*) dievaluasi sebagai `AnswerInvalidQuestion` tanpa mengurangi kuota pertanyaan agar pemain tidak dirugikan.
  - Normalisasi tebakan mencakup case-insensitivity, pembersihan tanda baca, dan trimming artikel bahasa Inggris ("a", "an", "the").

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -count=1 ./...`
- **Output Terminal:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/config	0.161s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.813s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	0.133s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/gamemaster	0.403s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/session	0.940s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/user	1.215s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	1.525s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord	0.281s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	1.322s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	1.357s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	1.671s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	1.709s
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** Lolos (Zero comments, no magic numbers, fail-fast checks).
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** Lolos (Unit tests cover entity, usecase, delivery).
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): Lolos
  - O-08 (Edge Case Atlas): Lolos (Invalid open-ended questions, article prefixes, empty inputs)
  - O-09 (Damage Assessment): Lolos (Zero regression on WYR and Taboo games)
  - O-10 (Full Suite Metrics): Lolos (100% tests passing across all packages)
  - O-11 (Real Thing Driven): Lolos (Discord interaction payloads and modal components verified)
  - O-12 (Shipping Env Check): Lolos (`go vet` clean, `go build` clean)
  - Hunt Loop Final Round: Lolos
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: Tidak ada `.graphify` di root proyek.
  - CHANGELOG.md: Updated.
  - Commit Message Sanity: Sesuai instruksi.
  - O-16 Untested List: Tidak ada untested path kritis.
