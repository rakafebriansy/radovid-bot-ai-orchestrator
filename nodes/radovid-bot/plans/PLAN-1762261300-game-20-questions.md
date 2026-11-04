---
id: PLAN-1762261300
ticket_id: TICKET-1762261300
title: "Game 3: 20 Questions"
status: Draft
created_at: 2026-11-04T00:15:00+07:00
---

# Rencana Implementasi Teknis: Game 3 - 20 Questions

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261300](../tickets/TICKET-1762261300-game-20-questions.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengimplementasikan mesin game "20 Questions" interaktif di Discord yang menjaga kerahasiaan topik rahasia pada session state, mengelola pengurangan counter pertanyaan secara deterministik dari 20 hingga 0, memvalidasi semantik pertanyaan via AI Game Master untuk menghasilkan respon ("Yes", "No", "Sometimes") yang akurat, serta memfasilitasi tebakan langsung pemain beserta evaluasi akhir sesi.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler slash command Discord (`/radovid play 20-questions`, `/radovid ask`, `/radovid guess`).
  - Komponen Terdampak: `internal/usecase/game/twenty_questions.go`, `internal/domain/game`, `internal/delivery/discord`.
  - Potensi Efek Samping: Kebocoran topik rahasia pada log sistem atau embed Discord; dicegah dengan memisahkan struct response publik dari entitas internal game session.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/twenty_questions_entity.go` | Domain struct TQSession, TQQAItem, TQAnswerType enum |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/twenty_questions.go` | Usecase start game, submit question, submit guess, reveal logic |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/twenty_questions_test.go` | Unit test alur counter pengurangan, limit 20 pertanyaan, dan evaluasi tebakan |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go` | Handler Discord slash command inisiasi sesi 20 Questions |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q.go` | Handler Discord slash command pengajuan pertanyaan ya/tidak |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q.go` | Handler Discord slash command pengajuan tebakan topik rahasia |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Domain Model & Red Test 20 Questions):**
   - Tulis `twenty_questions_entity.go` yang memodelkan kuota awal (20), histori daftar pertanyaan beserta jawaban, status tertebak (*isGuessed*), dan topik rahasia.
   - Tulis `twenty_questions_test.go` yang mensimulasikan:
     - Game baru dimulai: `RemainingQuestions == 20`.
     - Pemain bertanya 1 pertanyaan: `RemainingQuestions == 19`, respon AI tercatat.
     - Pertanyaan ke-20 diajukan: jika tidak ada tebakan benar, status berubah menjadi `GameOver` dan topik terungkap (*revealed*).
     - Tebakan benar diajukan di pertanyaan ke-5: status berubah menjadi `PlayerWon`, game selesai seketika.
   - Buktikan tes RED.
2. **Langkah 2 (Implementasi Logika Usecase):**
   - Tulis `twenty_questions.go` dengan orkestrasi:
     - `AskQuestion(ctx context.Context, sessionID, userID, question string) (*TQQAItem, int, error)`
     - `SubmitGuess(ctx context.Context, sessionID, userID, guess string) (bool, string, error)`
   - Implementasikan fungsi pemanggilan Ollama untuk mengklasifikasi pertanyaan terhadap topik rahasia dengan prompt terarah (menghasilkan YES, NO, atau SOMETIMES).
   - Pastikan tes unit GREEN.
3. **Langkah 3 (Discord Delivery Slash Commands):**
   - Buat `command_play_20q.go` yang menginisiasi sesi dan menampilkan embed sambutan AI Game Master.
   - Buat `command_ask_20q.go` yang membaca opsi string `question` dan memperbarui embed di Discord dengan counter tersisa.
   - Buat `command_guess_20q.go` yang membaca opsi string `answer` dan menampilkan status kemenangan atau kegagalan tebakan.
4. **Langkah 4 (Refactor & Delivery Gate):**
   - Verifikasi isolasi data rahasia (topik rahasia tidak pernah dikirimkan pada payload embed sampai game dinyatakan selesai).
   - Cek kepatuhan Zero-Comment Policy dan fail-fast validation.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```go
package game

import (
	"context"
	"errors"
	"strings"
	"sync"
)

var (
	ErrNoQuestionsRemaining = errors.New("maximum question limit reached")
	ErrGameAlreadyOver       = errors.New("the 20 questions game has already concluded")
)

type TQAnswerType string

const (
	AnswerYes       TQAnswerType = "Yes"
	AnswerNo        TQAnswerType = "No"
	AnswerSometimes TQAnswerType = "Sometimes"
)

type TQQAItem struct {
	QuestionNumber int
	QuestionText   string
	Answer         TQAnswerType
	Explanation    string
	AskedByUserID  string
}

type TQGameState struct {
	mu                 sync.RWMutex
	SessionID          string
	SecretTopic        string
	Category           string
	QuestionsRemaining int
	QALog              []TQQAItem
	IsOver             bool
	IsWon              bool
}

func (s *TQGameState) DecrementQuestions() (int, error) {
	s.mu.Lock()
	defer s.mu.Unlock()
	if s.IsOver {
		return s.QuestionsRemaining, ErrGameAlreadyOver
	}
	if s.QuestionsRemaining <= 0 {
		s.IsOver = true
		return 0, ErrNoQuestionsRemaining
	}
	s.QuestionsRemaining--
	if s.QuestionsRemaining == 0 {
		s.IsOver = true
	}
	return s.QuestionsRemaining, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:**
  - Pertanyaan bukan berbasis Yes/No (contoh: "What color is it?"): AI Game Master diinstruksikan dalam prompt untuk mendeteksi open-ended questions dan menjawab "Please ask a yes/no question" tanpa mengurangi counter pertanyaan.
  - Pertanyaan dikirim berturut-turut sebelum respons AI selesai: Mutex locking pada session state memastikan decrement counter berjalan teratur tanpa race condition.
  - Tebakan memiliki perbedaan ejaan kecil atau perbedaan artikel ("the dog" vs "dog"): Normalisasi string (lowercase, trim "a ", "an ", "the ") dilakukan sebelum pencocokan deterministik, disertai evaluasi sinonim AI jika mendekati benar.
- **Strategi Rollback:**
  - Batalkan perubahan via Git dan bersihkan session game yang aktif jika terjadi inkonsistensi status.
