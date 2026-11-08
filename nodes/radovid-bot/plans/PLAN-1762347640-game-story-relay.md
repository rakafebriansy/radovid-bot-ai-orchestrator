---
id: PLAN-1762347640
ticket_id: TICKET-1762347640
title: Game 7: Story Relay
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 7: Story Relay

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347640](../tickets/TICKET-1762347640-game-story-relay.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun modul kolaborasi multiplayer Game 7 "Story Relay" yang mengatur pergiliran naratif antara Player A, AI Game Master, dan Player B. AI memanfaatkan Ollama Gemma 3 4b untuk berperan ganda: sebagai Co-Writer yang meneruskan alur cerita dengan plot twist tak terduga, dan sebagai Language Coach yang mencatat evaluasi gramatikal di background tanpa memutus jalannya narasi cerita.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Router aktivitas `/radovid play story-relay` di `internal/delivery/discord`.
  - Komponen Terdampak: `internal/domain/game`, `internal/domain/session`, `internal/usecase/game`, `internal/infrastructure/ollama`, `internal/delivery/discord`.
  - Potensi Efek Samping: Giliran pemain kedua bisa terhenti jika pemain pertama AFK; usecase harus dilengkapi turn timeout (misal: 120 detik per giliran) yang otomatis melepaskan giliran atau menutup sesi dengan aman.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/story_relay.go` | Entitas `StoryRelaySession`, segmen cerita (`StorySegment`), dan queue antrean pemain |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/story_relay.go` | Logika pergantian giliran multiplayer, penyusunan cerita utuh, dan integrasi plot twist AI |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/story_relay_test.go` | Unit test perpindahan giliran, validasi kepemilikan giliran (turn ownership), dan kompilasi cerita |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/story_relay.tmpl` | Prompt template naratif AI: kelanjutan cerita, penyisipan plot twist, dan koreksi bahasa tersembunyi |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_story_relay.go` | Handler tombol giliran, interaksi modal Discord, dan event giliran berikutnya |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Layout visual embed buku cerita (Story Book embed) dengan kutipan kontributor |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `story_relay_test.go` yang memvalidasi giliran: jika Player A mengirim saat giliran Player B, sistem mengembalikan `ErrNotYourTurn`. Uji juga agregasi segmen menjadi cerita final.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan struct `StoryRelayState` pada `internal/domain/game/story_relay.go`.
   - Susun prompt template `story_relay.tmpl` dengan instruksi penyisipan plot twist dramatis dan menjaga konsistensi latar dunia fiksi.
   - Tulis logika `SubmitSentence` di `internal/usecase/game/story_relay.go`: memvalidasi panjang kalimat, memanggil Ollama untuk menghasilkan fragmen cerita berikutnya, mengevaluasi kesalahan bahasa, dan mengalihkan giliran ke player berikutnya.
3. **Langkah 3 (Refactor & Integrasi UI):**
   - Susun embed Discord dengan menampilkan bab-bab sebelumnya sebagai quote block, segmen terkini dari AI dengan highlight, serta menyebutkan giliran pemain aktif menggunakan tag Discord mention.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji skenario jika salah satu pemain disconnect, input kalimat kosong, atau input teks bahasa non-Inggris.
   - Verifikasi tidak ada race condition saat dua pemain menekan tombol secara bersamaan menggunakan mutex/transaction lock di repositori.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/story_relay.go
package game

type ContributorRole string

const (
	RolePlayerOne ContributorRole = "player_1"
	RolePlayerTwo ContributorRole = "player_2"
	RoleGameMaster ContributorRole = "game_master"
)

type StorySegment struct {
	SegmentNumber int             `json:"segment_number"`
	ContributorID string          `json:"contributor_id"`
	Role          ContributorRole `json:"role"`
	Text          string          `json:"text"`
	GrammarNotes  string          `json:"grammar_notes,omitempty"`
}

type StoryRelayState struct {
	SessionID       string         `json:"session_id"`
	PlayerOneID     string         `json:"player_one_id"`
	PlayerTwoID     string         `json:"player_two_id"`
	ActivePlayerID  string         `json:"active_player_id"`
	CurrentTurn     int            `json:"current_turn"`
	MaxTurns        int            `json:"max_turns"` // e.g., 6 rounds
	Segments        []StorySegment `json:"segments"`
	IsCompleted     bool           `json:"is_completed"`
}
```

```go
// internal/usecase/game/story_relay.go
func (uc *StoryRelayUseCase) SubmitTurn(ctx context.Context, sessionID, userID, sentence string) (*StorySegment, error) {
	state, err := uc.repo.GetStoryRelayState(ctx, sessionID)
	if err != nil {
		return nil, err
	}
	if state.ActivePlayerID != userID {
		return nil, ErrNotYourTurn
	}
	
	// Catat kontribusi pemain
	state.CurrentTurn++
	playerSegment := StorySegment{
		SegmentNumber: state.CurrentTurn,
		ContributorID: userID,
		Role:          uc.resolveRole(state, userID),
		Text:          sentence,
	}
	state.Segments = append(state.Segments, playerSegment)
	
	// AI Game Master menyambung cerita dengan plot twist
	aiResp, err := uc.ollamaClient.ContinueStoryRelay(ctx, state.Segments)
	if err != nil {
		return nil, err
	}
	state.CurrentTurn++
	aiSegment := StorySegment{
		SegmentNumber: state.CurrentTurn,
		ContributorID: "ai_game_master",
		Role:          RoleGameMaster,
		Text:          aiResp.ContinuationText,
		GrammarNotes:  aiResp.GrammarNotesOnPrevious,
	}
	state.Segments = append(state.Segments, aiSegment)
	
	// Rotasi giliran ke pemain berikutnya
	state.ActivePlayerID = uc.toggleNextPlayer(state, userID)
	if state.CurrentTurn >= state.MaxTurns {
		state.IsCompleted = true
	}
	_ = uc.repo.SaveStoryRelayState(ctx, state)
	return &aiSegment, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain yang mengirimkan teks spam atau vulgar disaring secara otomatis; kalimat yang tidak logis tetap diarahkan kembali oleh kemampuan grounding konteks Ollama.
- **Strategi Rollback:** Apabila pemain kedua tidak merespons dalam 120 detik, pemain pertama memiliki opsi untuk mengubah mode sesi menjadi Single Player (AI bertindak sebagai pemain kedua secara otomatis).

