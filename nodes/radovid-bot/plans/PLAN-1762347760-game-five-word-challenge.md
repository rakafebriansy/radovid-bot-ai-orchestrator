---
id: PLAN-1762347760
ticket_id: TICKET-1762347760
title: Game 9: Five Word Writing Challenge
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 9: Five Word Writing Challenge

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347760](../tickets/TICKET-1762347760-game-five-word-challenge.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengimplementasikan Game 9 "Five Word Writing Challenge" dengan arsitektur validasi berlapis: (1) validasi deterministik lokal Go untuk memeriksa apakah seluruh 5 kosakata acak benar-benar digunakan dalam paragraf (termasuk toleransi bentuk kata kerja lampau/-ing dan jamak -s/-es), dan (2) evaluasi kualitatif semantik via Ollama Gemma 3 4b untuk menilai apakah 5 kata tersebut dirangkai secara bermakna atau hanya dijejalkan secara acak (spamming).
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Entrypoint router Discord `/radovid play five-word-challenge`.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, `internal/delivery/discord`.
  - Potensi Efek Samping: Deteksi kata bisa salah (false negative) jika pemain menggunakan bentuk kata turunan (misal: "run" menjadi "running"). Modul verifikasi kata di domain layer harus mendukung matching stem/root dasar tanpa ketergantungan library NLP CGO yang berat.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeFiveWordChallenge` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/five_word.go` | Definisi struct `TargetWord`, `PartOfSpeech`, `FiveWordResult`, `FiveWordState`, katalog kata per difficulty, dan algoritma verifikasi kata infleksi |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/five_word_test.go` | Unit test algoritma pencocokan kata (word presence matcher), case-insensitivity, tanda baca, dan scoring |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/five_word.tmpl` | Prompt template penilaian koherensi integrasi kata dalam paragraf |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` | Verifikasi rendering prompt `five_word.tmpl` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parser respons evaluasi LLM untuk Five Word Challenge |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parser respons Five Word Challenge |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Data fallback evaluasi Five Word Challenge jika Ollama offline |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Unit test fallback data Five Word Challenge |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/five_word.go` | Usecase `FiveWordUseCase`: generator 5 kata berimbang, delegasi evaluasi ke LLM/fallback, dan agregasi skor |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/five_word_test.go` | Unit test usecase table-driven untuk `StartGame` dan `SubmitParagraph` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Builder Discord Embed: kartu tantangan 5 kata dan scorecard checklist hasil evaluasi |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test embed builder untuk Five Word Challenge |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Registrasi slash subcommand `/radovid play five-word-challenge` dengan opsi `difficulty` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Perutean slash command untuk `five-word-challenge` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Integrasi `FiveWordUseCase` ke handler utama game |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_five_word.go` | Handler tombol "Write Paragraph" dan Discord modal submit paragraf |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_five_word_test.go` | Unit test handler interaction Discord untuk Five Word Challenge |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_test.go` | Unit test pendaftaran subcommand Discord |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Inisialisasi usecase, dependency injection, dan registrasi prefix modal/button `fiveword:` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `five_word_test.go` yang menguji fungsi `VerifyWordInclusion(targets []string, paragraph string)` terhadap variasi kasus: penulisan huruf besar/kecil, bentuk jamak, tanda baca yang menempel pada kata ("whisper," vs "whisper"), dan deteksi kata yang hilang.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Buat entitas `FiveWordChallengeState` dan `TargetWord` di `internal/domain/game/five_word.go`.
   - Implementasikan fungsi pembersih teks dan word boundary regex matcher `\bword\b` di usecase layer.
   - Susun prompt template `five_word.tmpl` yang menginstruksikan model Ollama menganalisis koherensi paragraf dan memberikan skor (1-10) serta saran penyempurnaan alur cerita.
   - Satukan logika `SubmitParagraph`: kalkulasi skor kata yang lolos verifikasi (10 poin per kata = max 50 poin) + skor kualitas AI (max 50 poin).
3. **Langkah 3 (Refactor & Integrasi Delivery Layer):**
   - Rancang antarmuka Discord: embed tantangan menampilkan 5 badge kata (contoh: 🏷️ *Breeze*, 🏷️ *Labyrinth*, 🏷️ *Whisper*, 🏷️ *Glimpse*, 🏷️ *Reluctant*), tombol "Submit Paragraph" membuka form modal, dan hasil evaluasi menampilkan checklist status kata [✅/❌].
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji masukan paragraf kosong, masukan yang hanya berisi 5 kata itu saja tanpa kalimat utuh, dan masukan melebihi 1000 karakter.
   - Verifikasi tidak ada kebocoran alokasi memori atau dependensi eksternal tak terdokumentasi.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/five_word.go
package game

type TargetWord struct {
	Word        string `json:"word"`
	PartOfSpeech string `json:"part_of_speech"`
	IsUsed      bool   `json:"is_used"`
}

type FiveWordResult struct {
	WordsUsedCount int          `json:"words_used_count"` // 0 - 5
	TargetWords    []TargetWord `json:"target_words"`
	CoherenceScore int          `json:"coherence_score"`  // 1 - 10
	GrammarScore   int          `json:"grammar_score"`    // 1 - 10
	TotalScore     int          `json:"total_score"`      // 0 - 100
	FeedbackNotes  string       `json:"feedback_notes"`
}

type FiveWordState struct {
	SessionID   string       `json:"session_id"`
	UserID      string       `json:"user_id"`
	Words       []TargetWord `json:"words"`
	Submission  string       `json:"submission,omitempty"`
	Result      *FiveWordResult `json:"result,omitempty"`
}
```

```go
// internal/usecase/game/five_word.go
func (uc *FiveWordUseCase) CheckWordPresence(targets []TargetWord, text string) ([]TargetWord, int) {
	lowerText := strings.ToLower(text)
	usedCount := 0
	result := make([]TargetWord, len(targets))
	for i, tw := range targets {
		result[i] = tw
		pattern := fmt.Sprintf(`(?i)\b%s(s|es|ed|ing)?\b`, regexp.QuoteMeta(tw.Word))
		matched, _ := regexp.MatchString(pattern, lowerText)
		if matched {
			result[i].IsUsed = true
			usedCount++
		}
	}
	return result, usedCount
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pengguna yang hanya menuliskan daftar kelima kata tanpa membentuk kalimat (misal: "breeze labyrinth whisper glimpse reluctant") akan dideteksi oleh skor koherensi Ollama dengan nilai 1/10 dan peringatan "Not a coherent paragraph".
- **Strategi Rollback:** Jika terjadi kegagalan parser respons Ollama, skor tetap dihitung berdasarkan verifikasi kata programatik Go (50 poin jika semua kata hadir) ditambah fallback nilai koherensi default.

