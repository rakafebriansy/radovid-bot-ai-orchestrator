---
id: TICKET-1762347460
plan_id: PLAN-1762347460
title: Game 4: Job Interview
status: Done
priority: High
labels: [Game, Speaking, Roleplay, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin mengikuti simulasi wawancara kerja profesional dalam bahasa Inggris di mana AI bertindak sebagai interviewer sesuai domain karir pilihan saya (Tech, Design, Marketing, Product),
Sehingga saya dapat melatih kemampuan menjawab pertanyaan wawancara teknis/behavioral, menerima evaluasi seketika per jawaban, dan memperoleh asesmen kesiapan kerja di akhir sesi.

Game 4: Job Interview merupakan aktivitas simulasi profesional 5 ronde interaktif. Pengguna memilih domain karir spesifik, kemudian AI Game Master secara dinamis menghasilkan pertanyaan situasional relevan. Pengguna merespons melalui Discord Modal (teks panjang), dan pada tiap ronde AI memberikan umpan balik langsung (analisis kelebihan, area perbaikan tata bahasa, dan kosakata profesional) sebelum beralih ke pertanyaan selanjutnya, yang ditutup dengan laporan evaluasi menyeluruh.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347460-game-job-interview.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Pengguna dapat memilih domain pekerjaan (Software Engineering/Tech, UI/UX Design, Digital Marketing, Product Management) melalui Discord Select Menu
- [x] AI Game Master menghasilkan pertanyaan wawancara adaptif (kombinasi behavioral, technical context, dan problem-solving) sebanyak 5 ronde berturut-turut
- [x] Input jawaban player ditangkap menggunakan Discord Modal dialog dengan batas karakter memadai dan validasi teks non-kosong
- [x] Pada setiap akhir ronde, AI memberikan umpan balik evaluatif singkat (kekuatan respons, koreksi gramatikal, dan saran frasa bisnis)
- [x] Setelah ronde ke-5 tuntas, sistem merangkum keseluruhan performa dalam Discord Embed akhir berisi skor profesionalitas (1-100), kesimpulan kesiapan wawancara, dan ringkasan kosakata kunci
- [x] Integrasi ke state machine sesi game sehingga state terbarui dari `ACTIVE` menjadi `COMPLETED` saat ronde 5 selesai

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/job_interview.go`
- `../../../../radovid-bot-api/internal/domain/game/job_interview_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/job_interview.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/job_interview.go`
- `../../../../radovid-bot-api/internal/usecase/game/job_interview_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Waktu Eksekusi:** 2026-11-05 23:40:00
- **Ringkasan Eksekusi:**
  - Mengimplementasikan Game 4: Job Interview lengkap dari domain hingga delivery Discord:
    1. Domain Game: Menambahkan `GameTypeJobInterview`, entitas `InterviewDomain` (Tech, Design, Marketing, Product), `InterviewRound`, `InterviewRoundFeedback`, `JobInterviewSummary`, dan `JobInterviewSession` dengan state machine 5 ronde berturut-turut.
    2. Prompt & Ollama Engine: Template prompt `job_interview.tmpl`, parser `parseJobInterview` di `ollama.ParseGameMasterOutput`, serta daftar fallback komprehensif `jobInterviewFallbacks` untuk 3 tingkat kesulitan (Easy, Medium, Hard).
    3. Use Case Layer: `JobInterviewUseCase` mengatur inisialisasi sesi wawancara, transisi ronde 1 hingga 5, evaluasi jawaban otomatis berbasis LLM / fallback cerdas, perhitungan akumulasi skor profesionalitas (1-100), asesmen kesiapan kandidat (*Readiness Level*), dan penutupan status sesi game ke `COMPLETED`.
    4. Discord Delivery: Subcommand `/radovid play job-interview` dengan opsi `domain` dan `difficulty`, pemilih domain via Discord Select Menu, pemicu input esai via Discord Modal dialog, penanganan response deferred, dan perenderan kartu umpan balik berkala serta sertifikat asesmen akhir (`BuildJobInterviewSummaryEmbed`).
  - Seluruh pengujian unit test lulus 100% pada 14 package (`go test -count=1 ./...`).
  - Analisis kode `go vet` dan kompilasi biner `go build` bersih tanpa peringatan.
  - Zero-Comment Policy dipatuhi secara konsisten tanpa ada komentar pada seluruh berkas Go baru maupun yang dimodifikasi.
