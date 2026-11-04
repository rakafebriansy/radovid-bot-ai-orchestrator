---
id: PLAN-1762434340
ticket_id: TICKET-1762434340
title: "Game 14: News Reporter"
status: Draft
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
  - Modul / Fungsi Pemanggil: Handler perintah slash Discord (`radovid_start.go`) dan interaction event router.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, prompt registry di `internal/infrastructure/ollama`, dan modul akumulasi skor XP.
  - Potensi Efek Samping: Evaluasi 5W1H membutuhkan ketelitian AI dalam membedakan ada/tidaknya fakta spesifik; format JSON output dari LLM harus divalidasi dengan ketat agar tidak menimbulkan kegagalan unmarshal.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/news_reporter.go` | Model NewsDispatch, FactSheet5W1H, BroadcastReport, NewsAssessment |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeNewsReporter = "news_reporter"` |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/news_reporter_game.go` | Use case orkestrasi skenario berita, pemrosesan naskah, dan verifikasi fakta |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/news_reporter_game_test.go` | Unit test kelengkapan 5W1H dan validasi skor jurnalisme |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go` | Prompt pembuat memo berita 5W1H dan prompt reviewer siaran berita |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go` | Integrasi game type news reporter ke pilihan slash command |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go` | Handler tombol "Mulai Siaran" dan modal editor naskah berita |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Format embed telegram berita mendesak (breaking news) dan kartu evaluasi |

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
