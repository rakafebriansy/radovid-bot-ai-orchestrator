---
id: PLAN-1762434340
ticket_id: TICKET-1762434340
title: "Game 14: News Reporter"
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 14: News Reporter

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434340](../tickets/TICKET-1762434340-game-news-reporter.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengimplementasikan game simulasi peliputan berita "News Reporter" (`news_reporter`), menghubungkan generator peristiwa 5W1H Ollama Gemma 3 4b dengan antarmuka modal Discord, serta merancang engine evaluasi laras bahasa jurnalistik dan kelengkapan fakta wacana.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler perintah slash Discord (`command_play_wyr.go`) dan interaction event router (`game_handler.go`, `cmd/bot/main.go`).
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, prompt registry di `internal/infrastructure/ollama`, template visual di `internal/infrastructure/discord`, dan modul akumulasi skor XP.
  - Penanganan Error & Fallback: Evaluasi 5W1H divalidasi dengan parser terstruktur dan dataset fallback deterministik multi-kategori (Environmental, Science & Tech, Archaeology, Unusual Discovery) untuk menjamin ketahanan terhadap kegagalan LLM.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/news_reporter.go` | Model NewsDispatch, FactSheet5W1H, NewsAssessment, NewsReporterState, skoring & XP |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/news_reporter_test.go` | Unit test domain model, validasi naskah (25-2000 chars), skoring & XP |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeNewsReporter = "news_reporter"` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity_test.go` | Unit test validasi GameTypeNewsReporter |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/news_reporter.tmpl` | Prompt template Ollama pembuat evaluasi naskah berita 5W1H |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` | Unit test rendering prompt news reporter |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parser respons terstruktur `parseNewsReporter` dan `ParseNewsReporterEvaluation` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parsing output AI news reporter |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Fallback data evaluasi berita deterministik `newsReporterFallbacks` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Unit test fallback news reporter |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/news_reporter.go` | Use case orkestrasi skenario berita, pemrosesan naskah, dan verifikasi fakta |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/news_reporter_test.go` | Unit test use case, pemanggilan Ollama, skoring, fallback, dan perolehan XP |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Embed builder memo breaking news (`BuildNewsReporterDispatchEmbed`) dan hasil siaran (`BuildNewsReporterEvaluationEmbed`) |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test format embed berita Discord |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Pendaftaran subcommand slash `news-reporter` dengan opsi `category` dan `difficulty` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_test.go` | Unit test definisi subcommand slash news reporter |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Routing subcommand `news-reporter` ke `HandlePlayNewsReporter` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Injeksi `newsReporterUseCase` ke `GameHandler` dan delegator component/modal |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_news_reporter.go` | Handler tombol "Siarkan Berita" dan modal editor naskah berita |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_news_reporter_test.go` | Unit test delivery handler Discord news reporter |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Inisialisasi use case dan registrasi handler interaction prefix `newsreporter:` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/usecase/game/news_reporter_game_test.go`.
   - Rancang pengujian evaluasi: masukkan naskah yang mencakup seluruh fakta 5W1H dan naskah yang melewatkan fakta penting (misal: tidak menyebut lokasi). Verifikasi pemetaan field `missing_facts`.
   - Pastikan tes berstatus RED.
2. **Langkah 2 (Domain Modeling):**
   - Definisikan struct `FactSheet5W1H`: `Who`, `What`, `When`, `Where`, `Why`, `How`.
   - Definisikan struct `NewsReporterState`: `Headline`, `Facts FactSheet5W1H`, `BroadcastScript string`, `Assessment NewsAssessment`.
3. **Langkah 3 (Use Case & Prompting):**
   - Buat template prompt di `prompts.go` yang menghasilkan lembar fakta terstruktur dalam format JSON dari model Gemma 3 4b.
   - Buat template prompt evaluasi naskah yang mengembalikan: `{ "covered_5w1h": 6, "formal_register_score": 8, "grammar_score": 9, "feedback": "...", "model_script": "..." }`.
   - Sambungkan dengan `AwardXP` dengan formula penghargaan berbasis kelengkapan fakta dan kefasihan tata bahasa formal.
4. **Langkah 4 (Delivery Gate & Format Visual):**
   - Rancang Discord embed bertema "🚨 BREAKING NEWS DISPATCH" dengan warna merah/oranye peringatan.
   - Tampilkan contoh naskah ideal (*model news script*) dari AI pada tab penutup embed sebagai panduan belajar pengguna.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/news_reporter.go
package game

type FactSheet5W1H struct {
	Who   string `json:"who"`
	What  string `json:"what"`
	When  string `json:"when"`
	Where string `json:"where"`
	Why   string `json:"why"`
	How   string `json:"how"`
}

type NewsAssessment struct {
	FactsCoveredCount   int      `json:"facts_covered_count"`
	MissingFacts        []string `json:"missing_facts"`
	FormalRegisterScore int      `json:"formal_register_score"`
	GrammarScore        int      `json:"grammar_score"`
	FeedbackNotes       string   `json:"feedback_notes"`
	ExemplaryScript     string   `json:"exemplary_script"`
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain menginput naskah yang sangat pendek (misal hanya 1 kalimat "A fire happened"). Evaluator harus menangani gracefully dengan menandai daftar fakta yang hilang tanpa memicu crash parser.
- **Strategi Rollback:** Apabila generator Ollama menghasilkan lembar fakta yang tidak lengkap, fallback generator statis lokal (hardcoded safe scenarios) digunakan agar sesi game tidak terhenti mendadak.
