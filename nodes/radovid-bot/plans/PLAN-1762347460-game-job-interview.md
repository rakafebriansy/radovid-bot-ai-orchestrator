---
id: PLAN-1762347460
ticket_id: TICKET-1762347460
title: Game 4: Job Interview
status: Draft
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
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, `internal/delivery/discord`.
  - Potensi Efek Samping: Interaksi modal Discord memiliki timeout submit dari sisi klien Discord jika user tidak mengirimkan respons dalam waktu lama; state ronde harus disimpan di SQLite agar sesi tidak corrupt jika bot restart di tengah ronde.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/job_interview.go` | Definisi enum domain karir, struct interview state, dan ronde interview |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/job_interview.go` | Logika state machine 5 ronde wawancara, pemanggilan prompt pertanyaan & penilaian |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/job_interview_test.go` | Unit test otomasi ronde wawancara dan kalkulasi agregasi skor |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/job_interview.tmpl` | Prompt template pewawancara profesional sesuai persona karir |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview.go` | Handler interaksi tombol, select menu domain karir, dan modal submit Discord |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/modals.go` | Registrasi custom ID Discord modal untuk input jawaban wawancara |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Template visual embed kartu pertanyaan wawancara, per-round feedback, dan sertifikat asesmen |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `job_interview_test.go` yang memvalidasi siklus 5 ronde: Round 1 hingga 5, transisi ronde, pencegahan pengiriman jawaban ganda per ronde, dan kalkulasi skor agregat.
   - Verifikasi pengujian berstatus RED.
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan `JobInterviewSession`, `InterviewDomain`, dan `RoundFeedback` pada `internal/domain/game/job_interview.go`.
   - Susun prompt template `job_interview.tmpl` yang menginstruksikan model Gemma 3 4b memposisikan diri sebagai pewawancara teknis/HR berstandar internasional.
   - Implementasikan fungsi `StartInterview`, `SubmitAnswer`, dan `FinalizeInterview` di `internal/usecase/game/job_interview.go`.
3. **Langkah 3 (Refactor & Integrasi Delivery Layer):**
   - Integrasikan komponen Discord Select Menu untuk memilih peran karir, tombol "Answer Question" yang memicu Discord Modal, serta render embed informatif.
   - Pastikan response menggunakan Deferred Message (`InteractionResponseDeferredChannelMessageWithSource`) selama pemrosesan inferensi Ollama.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji skenario jawaban kosong, jawaban di luar topik wawancara, karakter ekstrem (> 1000 karakter), dan pembatalan sesi di tengah ronde.
   - Verifikasi clean build tanpa CGO via `CGO_ENABLED=0 go build ./cmd/bot`.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/job_interview.go
package game

type InterviewDomain string

const (
	DomainTech       InterviewDomain = "tech"
	DomainDesign     InterviewDomain = "design"
	DomainMarketing  InterviewDomain = "marketing"
	DomainProduct    InterviewDomain = "product"
)

type InterviewRound struct {
	RoundNumber int    `json:"round_number"`
	Question    string `json:"question"`
	Answer      string `json:"answer"`
	Feedback    string `json:"feedback"`
	Score       int    `json:"score"`
}

type JobInterviewState struct {
	SessionID    string           `json:"session_id"`
	UserID       string           `json:"user_id"`
	Domain       InterviewDomain  `json:"domain"`
	CurrentRound int              `json:"current_round"`
	MaxRounds    int              `json:"max_rounds"`
	Rounds       []InterviewRound `json:"rounds"`
	FinalSummary string           `json:"final_summary"`
}
```

```go
// internal/usecase/game/job_interview.go
func (uc *JobInterviewUseCase) SubmitAnswer(ctx context.Context, sessionID string, answer string) (*InterviewRound, bool, error) {
	state, err := uc.sessionRepo.GetInterviewState(ctx, sessionID)
	if err != nil {
		return nil, false, err
	}
	if state.CurrentRound > state.MaxRounds {
		return nil, false, ErrInterviewAlreadyFinished
	}
	roundFeedback, err := uc.ollamaClient.EvaluateInterviewAnswer(ctx, state.Domain, state.Rounds[state.CurrentRound-1].Question, answer)
	if err != nil {
		return nil, false, err
	}
	state.Rounds[state.CurrentRound-1].Answer = answer
	state.Rounds[state.CurrentRound-1].Feedback = roundFeedback.Feedback
	state.Rounds[state.CurrentRound-1].Score = roundFeedback.Score
	
	isFinished := state.CurrentRound >= state.MaxRounds
	if !isFinished {
		state.CurrentRound++
		nextQ, _ := uc.ollamaClient.GenerateInterviewQuestion(ctx, state.Domain, state.CurrentRound)
		state.Rounds = append(state.Rounds, InterviewRound{RoundNumber: state.CurrentRound, Question: nextQ})
	}
	_ = uc.sessionRepo.SaveInterviewState(ctx, state)
	return &state.Rounds[state.CurrentRound-1], isFinished, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Respons jawaban modal dibatasi maksimal 1500 karakter; jawaban dengan indikasi trolling/gibberish dideteksi oleh prompt evaluator dengan skor rendah dan arahan profesional.
- **Strategi Rollback:** Apabila terjadi pemutusan koneksi Discord di tengah 5 ronde, pengguna dapat melanjutkan sesi dalam kurun waktu 15 menit melalui `/radovid play job-interview resume` sebelum sesi ditandai `CANCELLED`.

