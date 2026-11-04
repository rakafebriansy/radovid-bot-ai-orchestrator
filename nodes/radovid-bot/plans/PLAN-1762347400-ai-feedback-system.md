---
id: PLAN-1762347400
ticket_id: TICKET-1762347400
title: AI Feedback System (Grammar & Vocabulary)
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: AI Feedback System (Grammar & Vocabulary)

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347400](../tickets/TICKET-1762347400-ai-feedback-system.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun subsistem evaluasi teks bahasa Inggris modular (`internal/usecase/feedback`) yang memproses masukan teks pengguna via inferensi lokal Ollama Gemma 3 4b, memparsing struktur JSON berisi koreksi gramatikal, saran pemformatan frasa alami, rekomendasi kosakata tingkat lanjut, serta skor komunikasi (1-10), dan menyimpan rekam jejaknya ke tabel `feedback_logs` di SQLite.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Use case aktivitas game Fase 3 (Job Interview, Roleplay, One Minute Challenge, Story Relay, Daily Diary, Five Word Challenge) serta handler Discord `/radovid feedback`.
  - Komponen Terdampak: `internal/infrastructure/ollama`, `internal/domain/feedback`, `internal/infrastructure/sqlite`, `internal/delivery/discord`.
  - Potensi Efek Samping: Latensi inferensi LLM dapat melampaui default threshold Discord acknowledgement (3 detik), sehingga seluruh panggilan evaluasi harus menggunakan deferred responses (`Defer()`) di layer delivery.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/008_create_feedback_logs.sql` | DDL tabel `feedback_logs` dengan kolom epoch milliseconds |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/feedback/entity.go` | Entitas `FeedbackRequest`, `FeedbackResponse`, `GrammarCorrection`, `VocabularyItem` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/feedback/repository.go` | Definisi interface port `FeedbackRepository` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/feedback/errors.go` | Error domain spesifik evaluasi AI |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback.go` | Orkestrasi use case evaluasi feedback dan fallback builder |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback_test.go` | Unit test table-driven parser, validasi batas skor, dan sanitasi |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/feedback.tmpl` | Prompt template terstruktur penghasil format JSON feedback |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/feedback_client.go` | Adaptor Ollama khusus payload feedback |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/sqlite/feedback_repository.go` | Implementasi konkret `FeedbackRepository` via SQLite raw SQL |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Template visual embed feedback (kartu koreksi & skor komunikasi) |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Rancang migration DDL `008_create_feedback_logs.sql`.
   - Buat unit test `ai_feedback_test.go` yang menguji parsing respons JSON acak/rusak dari LLM dan sanitasi teks mention Discord.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan struct domain pada `internal/domain/feedback/entity.go` tanpa dependensi eksternal.
   - Implementasikan template prompt pada `internal/infrastructure/ollama/prompts/feedback.tmpl` dengan instruksi ketat "JSON-only output".
   - Tulis logika `EvaluateFeedback` di layer usecase dengan timeout kontekstual (default 30 detik) dan fallback response jika LLM mengembalikan format tidak valid.
   - Implementasikan repository SQLite untuk menyimpan log evaluasi ke tabel `feedback_logs`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Rapikan pembuatan Discord Embed builder di `internal/delivery/discord/embeds.go` untuk menampilkan feedback dengan warna tematik (Hijau untuk nilai >= 8, Kuning 5-7, Merah < 5).
   - Pastikan Zero-Comment Policy dan fail-fast diterapkan pada input validation (panjang teks > 2000 karakter ditolak dini).
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji kasus batas teks kosong, teks hanya karakter non-alfanumerik, prompt injection teks user, dan respons LLM parsial.
   - Verifikasi pengujian lulus 100% dan tidak ada race condition saat operasi I/O SQLite concurrency.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/feedback/entity.go
package feedback

type GrammarCorrection struct {
	Original    string `json:"original"`
	Corrected   string `json:"corrected"`
	Explanation string `json:"explanation"`
}

type VocabularyItem struct {
	Word           string `json:"word"`
	Meaning        string `json:"meaning"`
	ContextExample string `json:"context_example"`
}

type FeedbackResponse struct {
	Score               int                 `json:"score"`
	GrammarCorrections  []GrammarCorrection `json:"grammar_corrections"`
	NaturalPhrasing     string              `json:"natural_phrasing"`
	VocabularyEnrichment []VocabularyItem   `json:"vocabulary_enrichment"`
	EncouragementNote   string              `json:"encouragement_note"`
}

type FeedbackRequest struct {
	SessionID  string `json:"session_id"`
	UserID     string `json:"user_id"`
	InputText  string `json:"input_text"`
	GameContext string `json:"game_context"`
	Difficulty string `json:"difficulty"`
}
```

```go
// internal/usecase/feedback/ai_feedback.go
func (uc *FeedbackUseCase) Evaluate(ctx context.Context, req feedback.FeedbackRequest) (*feedback.FeedbackResponse, error) {
	sanitizedText := sanitizeUserInput(req.InputText)
	if len(sanitizedText) == 0 {
		return nil, feedback.ErrEmptyInput
	}
	// Panggilan ke Ollama client dengan timeout konteks
	resp, err := uc.ollamaClient.GenerateFeedback(ctx, sanitizedText, req.GameContext, req.Difficulty)
	if err != nil {
		return uc.buildFallbackResponse(sanitizedText), nil
	}
	_ = uc.repo.SaveFeedbackLog(ctx, req.SessionID, req.UserID, sanitizedText, resp)
	return resp, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Input berisi regex manipulatif atau tag HTML disterilkan; input melebihi 2000 karakter dipotong secara deterministik dengan log peringatan.
- **Strategi Rollback:** Jika terjadi kegagalan skema DDL, rollback tabel menggunakan skrip migrasi `DROP TABLE IF EXISTS feedback_logs;`.

