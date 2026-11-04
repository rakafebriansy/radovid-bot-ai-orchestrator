---
id: PLAN-1762347520
ticket_id: TICKET-1762347520
title: Game 5: Roleplay
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 5: Roleplay

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347520](../tickets/TICKET-1762347520-game-roleplay.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun mesin dialog interaktif skenario situasi sehari-hari (Game 5: Roleplay) yang menggabungkan kemampuan generative roleplay Ollama Gemma 3 4b dan sistem evaluasi tata bahasa non-intrusif. AI menghasilkan respons karakter yang imersif dan sekaligus mengekstrak catatan koreksi minor pada payload terstruktur JSON ganda (`character_reply` dan `subtle_tip`), sehingga pengalaman percakapan tetap natural tanpa mematahkan dinamika bermain peran.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Entrypoint router game Discord `/radovid play roleplay`.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, `internal/delivery/discord`.
  - Potensi Efek Samping: Riwayat percakapan bertambah panjang seiring bertambahnya giliran (turn history); context prompt ke Ollama harus dibatasi hingga 6 giliran terakhir untuk menjaga jendela konteks model lokal dan meminimalkan latensi inferensi.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/roleplay.go` | Definisi struktur skenario roleplay, riwayat giliran, dan entitas subtle correction |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/roleplay.go` | Logika orkestrasi dialog turn-based, evaluasi target skenario, dan batas 10 giliran |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/roleplay_test.go` | Unit test state machine dialog, deteksi penyelesaian skenario, dan akumulasi koreksi |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/roleplay.tmpl` | Prompt template dual-channel (in-character conversation + linguistic coaching) |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_roleplay.go` | Handler komponen Discord untuk pemilihan skenario, tombol bicara, dan pesan dialog |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Visual layout percakapan roleplay (tampilan karakter, dialog bubble, subtle tip footer) |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `roleplay_test.go` untuk memverifikasi inisialisasi skenario (misal: "At the Airport"), penambahan dialog per giliran, pemotongan konteks percakapan lampau, dan penghentian otomatis pada giliran ke-10.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan tipe skenario dan status sesi pada `internal/domain/game/roleplay.go`.
   - Bangun template prompt `roleplay.tmpl` yang memerintahkan model Ollama merespons dalam format JSON berisi: `character_response`, `is_goal_reached`, dan `grammar_tip`.
   - Tulis logika `ProcessTurn` pada `internal/usecase/game/roleplay.go` yang memvalidasi input user, mengirim ke Ollama dengan timeout context, dan memperbarui riwayat sesi di repositori.
3. **Langkah 3 (Refactor & Integrasi UI):**
   - Buat representasi visual Discord Embed dengan dialog box bergaya novel visual/rpg: nama persona karakter sebagai Author, balasan ucapan sebagai Description, dan koreksi tata bahasa opsional di bagian Footer/Field tersendiri.
   - Tambahkan tombol "Speak / Reply" yang membuka modal input kalimat bahasa Inggris.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan pengujian input kata kasar / di luar konteks skenario, masukan kalimat yang sangat pendek ("yes", "ok"), dan masukan panjang.
   - Pastikan memory footprint stabil dan sesi dibersihkan saat skenario selesai.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/roleplay.go
package game

type RoleplayScenarioType string

const (
	ScenarioRestaurant RoleplayScenarioType = "restaurant"
	ScenarioAirport    RoleplayScenarioType = "airport"
	ScenarioDoctor     RoleplayScenarioType = "doctor"
	ScenarioHotel      RoleplayScenarioType = "hotel"
)

type DialogueTurn struct {
	TurnNumber  int    `json:"turn_number"`
	Speaker     string `json:"speaker"` // "user" atau "ai"
	Message     string `json:"message"`
	SubtleTip   string `json:"subtle_tip,omitempty"`
}

type RoleplaySessionState struct {
	SessionID     string               `json:"session_id"`
	UserID        string               `json:"user_id"`
	Scenario      RoleplayScenarioType `json:"scenario"`
	CurrentTurn   int                  `json:"current_turn"`
	MaxTurns      int                  `json:"max_turns"` // Default 10
	Turns         []DialogueTurn       `json:"turns"`
	IsGoalReached bool                 `json:"is_goal_reached"`
}
```

```go
// internal/usecase/game/roleplay.go
func (uc *RoleplayUseCase) HandleUserReply(ctx context.Context, sessionID, userText string) (*DialogueTurn, bool, error) {
	state, err := uc.repo.GetRoleplayState(ctx, sessionID)
	if err != nil {
		return nil, false, err
	}
	state.CurrentTurn++
	userTurn := DialogueTurn{TurnNumber: state.CurrentTurn, Speaker: "user", Message: userText}
	state.Turns = append(state.Turns, userTurn)

	aiResp, err := uc.ollamaClient.GenerateRoleplayResponse(ctx, state.Scenario, state.Turns)
	if err != nil {
		return nil, false, err
	}
	state.CurrentTurn++
	aiTurn := DialogueTurn{
		TurnNumber: state.CurrentTurn,
		Speaker:    "ai",
		Message:    aiResp.CharacterResponse,
		SubtleTip:  aiResp.GrammarTip,
	}
	state.Turns = append(state.Turns, aiTurn)
	state.IsGoalReached = aiResp.IsGoalReached || state.CurrentTurn >= state.MaxTurns

	_ = uc.repo.SaveRoleplayState(ctx, state)
	return &aiTurn, state.IsGoalReached, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pengguna yang mencoba jailbreak / instruksi eksplisit di luar peran di-filter oleh system instructions pada prompt template sehingga bot tetap konsisten berada di dalam karakter.
- **Strategi Rollback:** Jika terjadi kegagalan parser respons AI di tengah giliran, sediakan fallback respons karakter generik ("Pardon me, could you rephrase that?") tanpa menambah nomor turn yang gagal.

