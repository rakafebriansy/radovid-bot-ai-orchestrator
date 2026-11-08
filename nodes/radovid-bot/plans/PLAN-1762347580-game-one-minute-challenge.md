---
id: PLAN-1762347580
ticket_id: TICKET-1762347580
title: Game 6: One Minute Challenge
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 6: One Minute Challenge

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347580](../tickets/TICKET-1762347580-game-one-minute-challenge.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengembangkan logika Game 6 "One Minute Challenge" dengan mekanisme pengatur waktu (timer) 60 detik berbasis timestamp epoch absolut untuk mencegah desinkronisasi jaringan. Sistem memisahkan evaluasi menjadi dua lapisan: lapisan komputasi analitik Go lokal (menghitung word count, unique words, lexical diversity) dan lapisan AI evaluator inferensi Ollama Gemma 3 4b (menilai koherensi naratif dan memberikan apresiasi atas kecepatan berpikir).
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Slash command `/radovid play one-minute-challenge`.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, `internal/delivery/discord`.
  - Potensi Efek Samping: Discord modal tidak memiliki built-in client countdown visual; penggunaan format timestamp native Discord `<t:EPOCH:R>` di embed deskripsi menjadi solusi standar untuk menghadirkan countdown dinamis di sisi klien Discord tanpa polling websocket berlebihan.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/one_minute.go` | Definisi entitas topik impromptu, batas waktu 60s, dan metrik fluency |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/one_minute.go` | Logika inisialisasi tantangan, validasi durasi submission, dan komputasi metrik leksikal |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/one_minute_test.go` | Unit test kalkulator leksikal, pengecekan overtime deadline, dan validasi skor |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/one_minute.tmpl` | Prompt template generator topik kreatif dan evaluator koherensi fluency |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_one_minute.go` | Komponen interaksi Discord (tombol "Mulai Menulis", modal input, handler submit) |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Template embed tantangan 60 detik dengan visual timer dan scoreboard leksikal |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `one_minute_test.go` untuk menguji algoritma penghitungan `CalculateLexicalDiversity` (tokenisasi kata, normalisasi lowercase, eliminasi tanda baca) dan validasi `IsSubmittedOnTime`.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan struct `OneMinuteChallengeState` dan `FluencyEvaluation` pada `internal/domain/game/one_minute.go`.
   - Implementasikan fungsi generator topik acak pada prompt template `one_minute.tmpl`.
   - Bangun logika use case `SubmitChallenge` yang memeriksa apakah `time.Now().UnixMilli() <= state.DeadlineAt` (dengan toleransi grace period 5 detik untuk latency jaringan).
   - Jalankan kalkulasi jumlah kata dan keanekaragaman leksikal, lalu kirimkan teks ke Ollama untuk evaluasi koherensi fluency.
3. **Langkah 3 (Refactor & Integrasi Delivery Layer):**
   - Rangkai alur interaksi Discord: bot mengirim embed tantangan dengan `<t:TIMESTAMP:R>`, pengguna mengklik tombol "Type Answer" yang membuka Discord Modal, dan hasil dikirimkan via embed evaluasi.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji batas waktu terlampaui (> 65 detik), teks berulang ("cat cat cat cat"), teks minim (< 5 kata), dan teks panjang dalam 60 detik.
   - Pastikan Zero-Comment Policy dan fail-fast logic terpenuhi.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/one_minute.go
package game

type FluencyMetrics struct {
	WordCount        int     `json:"word_count"`
	UniqueWordCount  int     `json:"unique_word_count"`
	LexicalDiversity float64 `json:"lexical_diversity"` // Unique / Total
	CoherenceScore   int     `json:"coherence_score"`   // 1-10 dari AI
	FluencyRank      string  `json:"fluency_rank"`      // Native, Fluent, Developing, Beginner
}

type OneMinuteState struct {
	SessionID   string         `json:"session_id"`
	UserID      string         `json:"user_id"`
	Topic       string         `json:"topic"`
	StartedAt   int64          `json:"started_at"`
	DeadlineAt  int64          `json:"deadline_at"`
	SubmittedAt int64          `json:"submitted_at,omitempty"`
	InputText   string         `json:"input_text,omitempty"`
	Metrics     FluencyMetrics `json:"metrics"`
}
```

```go
// internal/usecase/game/one_minute.go
func (uc *OneMinuteUseCase) EvaluateSubmission(ctx context.Context, sessionID, text string, submitTime int64) (*FluencyMetrics, error) {
	state, err := uc.repo.GetOneMinuteState(ctx, sessionID)
	if err != nil {
		return nil, err
	}
	const gracePeriodMs = 5000
	if submitTime > state.DeadlineAt+gracePeriodMs {
		return nil, ErrDeadlineExceeded
	}
	
	tokens := tokenizeWords(text)
	wordCount := len(tokens)
	uniqueTokens := make(map[string]struct{})
	for _, t := range tokens {
		uniqueTokens[t] = struct{}{}
	}
	lexicalDiv := float64(len(uniqueTokens)) / float64(wordCount)
	
	aiCoherence, _ := uc.ollamaClient.EvaluateFluency(ctx, state.Topic, text)
	metrics := FluencyMetrics{
		WordCount:        wordCount,
		UniqueWordCount:  len(uniqueTokens),
		LexicalDiversity: lexicalDiv,
		CoherenceScore:   aiCoherence.Score,
		FluencyRank:      determineFluencyRank(wordCount, lexicalDiv),
	}
	state.Metrics = metrics
	_ = uc.repo.SaveOneMinuteState(ctx, state)
	return &metrics, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pengguna menyalin teks template eksternal atau menekan tombol spasi panjang ditangani dengan normalisasi string regex `\s+` dan validasi leksikal.
- **Strategi Rollback:** Bila Ollama tidak merespons dalam durasi batas timeout 30s, sistem tetap menyajikan skor analitik berbasis komputasi leksikal lokal Go dan pesan dorongan semangat default.

