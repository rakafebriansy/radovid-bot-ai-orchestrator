---
id: PLAN-1762434220
ticket_id: TICKET-1762434220
title: "Game 12: Sentence Transformation Race"
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 12: Sentence Transformation Race

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434220](../tickets/TICKET-1762434220-game-sentence-transformation-race.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun mini-game "Sentence Transformation Race" (`sentence_transformation`) yang menuntut pemain mengubah struktur gramatikal kalimat berbahasa Inggris di bawah batas waktu tertentu, dengan penilaian kombinasi ketepatan sintaksis dan kecepatan pengiriman jawaban.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Discord interaction command `/radovid play transformation-race` dan modal submit handler.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, kalkulator skor kecepatan, dan XP system.
  - Potensi Efek Samping: Jeda waktu jaringan (network latency) antara pengiriman Discord modal dan server diperhitungkan dengan toleransi *grace period* 3 detik agar tidak membatalkan jawaban valid di detik terakhir.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/transformation.go` | Model TransformationType, Challenge, RoundResult, TransformationState, CalculateSpeedScore, ValidateTransformationSubmission |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/transformation_test.go` | Unit test domain model, skoring kecepatan, dan validasi |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeSentenceTransformation = "sentence_transformation"` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity_test.go` | Test registrasi game type |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/transformation.tmpl` | Prompt template Gemma 3 4b delimiter format |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` | Verifikasi embed template transformation.tmpl |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parser delimiter respons evaluasi transformasi AI |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parser response evaluasi AI |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Fallback tantangan deterministik berdasarkan difficulty |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Unit test fallback transformation |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/transformation.go` | Use case orkestrasi sesi, submit, evaluasi, timer scoring |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/transformation_test.go` | Unit test use case transformasi kalimat |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Embed round, round result, and summary |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test embed visual transformasi |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Registrasi subcommand `transformation-race` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Router command `transformation-race` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Injeksi TransformationUseCase dan delegator component/modal |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_transformation.go` | Handlers command, component (`open_modal`, `next`, `summary`), dan modal submit |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_transformation_test.go` | Unit test handlers Discord delivery |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Wiring use case dan router component/modal |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Domain Modeling & Formula):**
   - Registrasi `GameTypeSentenceTransformation = "sentence_transformation"`.
   - Buat enum jenis transformasi: `TransformActiveToPassive`, `TransformPassiveToActive`, `TransformDirectToReported`, `TransformReportedToDirect`, `TransformTenseShift`, `TransformConditionalShift`.
   - Implementasikan fungsi murni `CalculateSpeedScore(accuracy int, elapsedMs int64, durationMs int64) (int, float64)` dengan grace period 3000ms.
2. **Langkah 2 (Ollama Prompts, Parser & Fallback):**
   - Buat template prompt `transformation.tmpl` dengan delimiter tags.
   - Buat parser `parseSentenceTransformation` di `parser.go`.
   - Tambahkan fallback deterministik per level kesulitan di `fallback.go`.
3. **Langkah 3 (Use Case & State Orchestration):**
   - Implementasikan `TransformationUseCase` dengan metode `StartGame`, `GetState`, `SubmitTransformation`.
   - Orkestrasi flow: start round -> tampilkan prompt di embed -> terima submission via modal -> hitung selisih waktu -> panggil evaluasi Ollama -> award XP proporsional.
4. **Langkah 4 (Discord Templates & Handlers):**
   - Buat embed builder `BuildTransformationRoundEmbed`, `BuildTransformationRoundResultEmbed`, dan `BuildTransformationGameSummaryEmbed`.
   - Implementasikan command handler `/radovid play transformation-race`, component handler (`open_modal`, `next`, `summary`), dan modal submit handler.
5. **Langkah 5 (Verification & Zero Comments):**
   - Verifikasi 100% PASS seluruh pengujian unit di semua package (`go test -count=1 ./...`).
   - Verifikasi Zero-Comment Policy (tidak ada komentar `//` atau `/* */` di kode Go baru).

---

## 5. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain mengirim kalimat identik dengan kalimat asal tanpa melakukan transformasi. Sistem mengenali kesamaan verbatim dan mengembalikan error validasi / skor akurasi 0.
- **Strategi Kecepatan & Latensi:** Waktu komputasi AI evaluasi tidak mengurangi poin pemain; waktu submit diukur seketika saat event interaksi Discord diterima.
- **Grace Period:** Grace period 3000ms diberikan untuk mengompensasi delay jaringan Discord modal submission.
