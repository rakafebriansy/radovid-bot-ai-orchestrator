---
id: PLAN-1762347460
ticket_id: TICKET-1762347460
title: Game 4: Job Interview
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 4: Job Interview

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347460](../tickets/TICKET-1762347460-game-job-interview.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengimplementasikan alur logika Game 4 "Job Interview" berbasis 5 ronde terstruktur. AI bertindak sebagai Lead Recruiter / Hiring Manager yang mengajukan pertanyaan berbobot berdasarkan domain karir terpilih (Tech, Design, Marketing, Product). Komunikasi masukan pengguna ditangani via Discord Modal dialog untuk mendukung penulisan esai naratif multi-kalimat, dan evaluasi menggunakan template AI evaluator per ronde serta akumulasi metrik di akhir sesi.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Router slash command `/radovid play job-interview` atau command picker di `internal/delivery/discord`.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, `internal/infrastructure/discord`, `internal/delivery/discord`.
  - Potensi Efek Samping: Interaksi modal Discord memiliki timeout submit dari sisi klien Discord jika user tidak mengirimkan respons dalam waktu lama; state ronde disimpan di in-memory cache dan disinkronkan ke state machine SQLite agar siklus sesi berakhir `COMPLETED` secara deterministik.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeJobInterview` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/job_interview.go` | Definisi enum domain karir, struct interview state, dan ronde interview |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/job_interview_test.go` | Unit test domain entitas job interview dan transisi state ronde |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/job_interview.go` | Logika state machine 5 ronde wawancara, pemanggilan prompt pertanyaan & penilaian |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/job_interview_test.go` | Unit test otomasi ronde wawancara dan kalkulasi agregasi skor |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/job_interview.tmpl` | Prompt template pewawancara profesional sesuai persona karir |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` | Unit test verifikasi rendering prompt template job interview |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parsing output AI khusus format interview (`QUESTION`, `CATEGORY`, `GUIDELINE`) |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parsing output AI wawancara kerja |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Bank pertanyaan fallback 5 ronde untuk 3 tingkat kesulitan |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Unit test fallback bank soal job interview |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Template visual embed kartu pertanyaan wawancara, select menu domain, per-round feedback, dan sertifikat asesmen |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test visual embed job interview |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Subcommand `/radovid play job-interview` dengan opsi domain dan difficulty |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Dispatch subcommand job interview pada router slash command |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Integrasi `interviewUseCase` ke handler game Discord |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview.go` | Handler interaksi tombol, select menu domain karir, dan modal submit Discord |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview_test.go` | Unit test delivery Discord job interview |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Wiring dependency injection usecase job interview dan pendaftaran router prefix `interview:` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat entitas domain `job_interview.go` dan unit test `job_interview_test.go`.
   - Implementasikan template prompt `job_interview.tmpl`, parser `parseJobInterview`, dan bank soal fallback di package `ollama`.
2. **Langkah 2 (Implementasi Inti Usecase):**
   - Implementasikan usecase `JobInterviewUseCase` dengan method `StartInterview`, `SubmitAnswer`, dan `GetInterview`.
   - Uji alur 5 ronde lengkap, pembatasan jawaban ganda, dan transisi status sesi game ke `COMPLETED` pada `job_interview_test.go`.
3. **Langkah 3 (Integrasi Delivery Layer):**
   - Buat template Discord Embed dan komponen Select Menu / Button di `game_templates.go`.
   - Daftarkan subcommand `job-interview` pada `command_def.go` dan router dispatch `command_play_wyr.go`.
   - Implementasikan handler Discord `HandlePlayJobInterview`, `HandleJobInterviewComponent`, dan `HandleJobInterviewModal` pada `game_job_interview.go`.
   - Wiring biner utama di `cmd/bot/main.go`.
4. **Langkah 4 (Delivery Gate):**
   - Verifikasi pengujian lulus 100% via `go test -count=1 ./...`.
   - Verifikasi `go vet ./...` dan `go build ./...`.

---

## 5. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Respons jawaban modal dibatasi maksimal 1500 karakter; sanitasi mention Discord dan normalisasi spasi diterapkan.
- **Fail-Safe LLM:** Jika LLM mengalami keterlambatan atau jaringan gagal, sistem secara otomatis mengaktifkan fallback evaluator dan bank soal cadangan sehingga flow permainan tidak terhenti.
