---
id: TICKET-1762347460
plan_id: PLAN-1762347460
title: Game 4: Job Interview
status: Todo
priority: High
labels: [Game, Speaking, Roleplay, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin mengikuti simulasi wawancara kerja profesional dalam bahasa Inggris di mana AI bertindak sebagai interviewer sesuai domain karir pilihan saya (Tech, Design, Marketing),
Sehingga saya dapat melatih kemampuan menjawab pertanyaan wawancara teknis/behavioral, menerima evaluasi seketika per jawaban, dan memperoleh asesmen kesiapan kerja di akhir sesi.

Game 4: Job Interview merupakan aktivitas simulasi profesional 5 ronde interaktif. Pengguna memilih domain karir spesifik, kemudian AI Game Master secara dinamis menghasilkan pertanyaan situasional relevan. Pengguna merespons melalui Discord Modal (teks panjang), dan pada tiap ronde AI memberikan umpan balik langsung (analisis kelebihan, area perbaikan tata bahasa, dan kosakata profesional) sebelum beralih ke pertanyaan selanjutnya, yang ditutup dengan laporan evaluasi menyeluruh.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347460-game-job-interview.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Pengguna dapat memilih domain pekerjaan (Software Engineering/Tech, UI/UX Design, Digital Marketing, Product Management) melalui Discord Select Menu
- [ ] AI Game Master menghasilkan pertanyaan wawancara adaptif (kombinasi behavioral, technical context, dan problem-solving) sebanyak 5 ronde berturut-turut
- [ ] Input jawaban player ditangkap menggunakan Discord Modal dialog dengan batas karakter memadai dan validasi teks non-kosong
- [ ] Pada setiap akhir ronde, AI memberikan umpan balik evaluatif singkat (kekuatan respons, koreksi gramatikal, dan saran frasa bisnis)
- [ ] Setelah ronde ke-5 tuntas, sistem merangkum keseluruhan performa dalam Discord Embed akhir berisi skor profesionalitas (1-100), kesimpulan kesiapan wawancara, dan ringkasan kosakata kunci
- [ ] Integrasi ke state machine sesi game sehingga state terbarui dari `ACTIVE` menjadi `COMPLETED` saat ronde 5 selesai

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/job_interview.go`
- `../../../../radovid-bot-api/internal/usecase/game/job_interview.go`
- `../../../../radovid-bot-api/internal/usecase/game/job_interview_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/job_interview.tmpl`
- `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview.go`
- `../../../../radovid-bot-api/internal/delivery/discord/modals.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

