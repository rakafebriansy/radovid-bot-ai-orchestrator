---
id: PLAN-1762261180
ticket_id: TICKET-1762261180
title: "Game 1: Would You Rather?"
status: Completed
created_at: 2026-11-04T00:15:00+07:00
---

# Rencana Implementasi Teknis: Game 1 - Would You Rather?

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261180](../tickets/TICKET-1762261180-game-would-you-rather.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengimplementasikan gameplay lengkap "Would You Rather?" yang mengintegrasikan AI Game Master, Discord Interactive Components (ActionRow + Buttons), dan persistensi ronde SQLite. Game ini menyajikan dilema komparatif dalam bahasa Inggris, menangkap respon tombol dari 2 peserta sesi tanpa tabrakan event, menghasilkan tanggapan evaluatif AI terhadap alasan pilihan, serta menjamin minimal 3 ronde per sesi permainan.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Discord Interaction Handler (`handler.go`), Slash Command `/radovid play`.
  - Komponen Terdampak: `internal/usecase/game/would_you_rather.go`, `internal/delivery/discord`, tabel `game_rounds`.
  - Potensi Efek Samping: Interaksi button race condition jika kedua pemain menekan tombol pada milidetik yang sama; dimitigasi dengan mutex sinkronisasi state in-memory atau atomic update database.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/004_create_game_rounds.sql` | DDL skema tabel game_rounds untuk riwayat sesi |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/wyr_entity.go` | Domain struct WYRRound, WYRChoice, WYRSummary |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/round_repository.go` | Interface RoundRepository |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/sqlite/round_repository.go` | Implementasi SQLite untuk pencatatan ronde |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go` | Logika usecase start round, submit choice, AI commentary, completion |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/would_you_rather_test.go` | Unit test alur multi-ronde dan validasi voting ganda |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Slash command router `/radovid play would-you-rather` |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go` | Interaction handler untuk Button click event Opsi A dan B |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Database Migration & Domain Model):**
   - Buat migrasi SQL `004_create_game_rounds.sql` dengan relasi `REFERENCES game_sessions(id)`.
   - Definisikan struct `WYRRound` dan entitas domain pilihan pemain.
2. **Langkah 2 (Implementasi Round Repository):**
   - Buat implementasi `round_repository.go` untuk menyimpan teks prompt, opsi, pilihan tiap user, dan status ronde.
3. **Langkah 3 (Red Test Use Case Would You Rather):**
   - Tulis unit test `would_you_rather_test.go` yang mensimulasikan:
     - Ronde 1 dimulai -> AI menghasilkan Option A dan Option B.
     - Pemain 1 memilih Option A -> pilihan tercatat, status pending player 2.
     - Pemain 1 mencoba vote lagi -> gagal dengan `ErrAlreadyVoted`.
     - Pemain 2 memilih Option B -> ronde selesai, AI commentary digenerate.
     - Melanjutkan hingga Ronde 3 tuntas -> flag `IsSessionCompleted` aktif.
   - Buktikan tes RED.
4. **Langkah 4 (Implementasi Use Case):**
   - Tulis `would_you_rather.go` yang mengorkestrasikan `GameMasterService`, `SessionRepository`, dan `RoundRepository`.
   - Pastikan tes usecase GREEN.
5. **Langkah 5 (Delivery Discord Delivery Adapter):**
   - Tulis `command_play_wyr.go` untuk registrasi command dan respon pesan embed pembuka.
   - Tulis `interaction_wyr_button.go` untuk memproses klik button dengan custom ID `wyr_vote:<session_id>:<round_number>:<option>`.
   - Respon Discord diperbarui secara in-place via `discordgo.InteractionResponseUpdateMessage`.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```go
package game

import (
	"context"
	"errors"
	"time"
)

var (
	ErrAlreadyVoted     = errors.New("participant has already cast a vote for this round")
	ErrRoundNotActive   = errors.New("current round is not accepting votes")
	ErrNotInSession     = errors.New("user is not an active participant in this session")
)

type WYRChoice string

const (
	ChoiceA WYRChoice = "A"
	ChoiceB WYRChoice = "B"
)

type PlayerVote struct {
	UserID    string
	Choice    WYRChoice
	VotedAt   int64
}

type WYRRound struct {
	ID          string
	SessionID   string
	RoundNumber int
	Question    string
	OptionA     string
	OptionB     string
	Commentary  string
	Votes       map[string]PlayerVote
	IsFinished  bool
}

type WouldYouRatherUseCase struct {
	sessionRepo   SessionRepository
	roundRepo     RoundRepository
	gameMaster    GameMasterService
}

func (uc *WouldYouRatherUseCase) SubmitVote(ctx context.Context, sessionID, userID string, roundNumber int, choice WYRChoice) (*WYRRound, error) {
	round, err := uc.roundRepo.GetBySessionAndRound(ctx, sessionID, roundNumber)
	if err != nil {
		return nil, err
	}
	if round.IsFinished {
		return nil, ErrRoundNotActive
	}
	if _, exists := round.Votes[userID]; exists {
		return nil, ErrAlreadyVoted
	}
	round.Votes[userID] = PlayerVote{
		UserID:  userID,
		Choice:  choice,
		VotedAt: time.Now().UnixMilli(),
	}
	return round, uc.roundRepo.Save(ctx, round)
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:**
  - Non-participant mencoba klik tombol: Diperiksa pada layer delivery & usecase; jika Discord ID tidak ada di daftar participant session, kembalikan respon error ephemeral tanpa mengubah status vote.
  - Kegagalan AI commentary: Jika generate commentary timeout, gunakan template komentar default bahasa Inggris tanpa membatalkan ronde permainan.
  - Ronde kurang dari 3: Sistem memblokir perintah selesai dini kecuali host secara eksplisit memilih tombol surrender/cancel session.
- **Strategi Rollback:**
  - Hapus data ronde terkait sesi pada database atau drop migrasi `004` jika terjadi schema regression.
