---
id: PLAN-1762520480
ticket_id: TICKET-1762520480
title: "Release Mutex During LLM HTTP Calls & Bound In-Memory Game State Maps"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Release Mutex During LLM Calls & Bound State Maps

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520480](../tickets/TICKET-1762520480-fix-game-mutex-lock-and-state-leak.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #9)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Testing](../../../global-guidelines/testing.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:**
  1. Melepaskan `uc.statesMu` sebelum melakukan panggilan jaringan ke Ollama di `job_interview.go`, `roleplay.go`, dan `story_relay.go` (selaras dengan pola di `news_reporter.go` dan `dream_house.go`).
  2. Menerapkan pembersihan otomatis (*pruning*) terhadap sesi yang sudah berstatus `IsFinished == true` saat kapasitas map melampaui ambang batas atau saat sesi baru dimulai, sehingga penggunaan memori tetap terbatas (*bounded*) tanpa merusak pemeriksaan `GetState` sesaat setelah permainan berakhir.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `JobInterviewUseCase.SubmitAnswer`, `RoleplayUseCase.SubmitTurn`, `StoryRelayUseCase.SubmitTurn`, serta `StartGame` di 11 game use case.
  - Komponen Terdampak: `internal/usecase/game/*.go`.
  - Potensi Efek Samping: Wajib memastikan pembacaan/modifikasi slice atau struct di bawah lock menggunakan salinan yang aman dari data race (`go test -race`).

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/job_interview.go` | Melepas lock sebelum `evaluateAnswer` & `GenerateGameContent`, mengunci kembali saat update state, dan memangkas state selesai |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/roleplay.go` | Melepas lock sebelum panggilan LLM dan mengunci kembali saat mutasi state |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/story_relay.go` | Melepas lock sebelum panggilan LLM dan mengunci kembali saat mutasi state |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/*.go` | Menambahkan pembersihan state `IsFinished` ketika ukuran map melampaui batas kapasitas (`maxRetainedStates`) |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/job_interview_test.go` | Menambahkan unit test konkurensi non-blocking saat LLM lambat dan uji batas memori state map |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Buat unit test di `job_interview_test.go` dengan mock LLM yang tidur selama `300ms` pada Sesi A, lalu ukur waktu eksekusi `GetState` pada Sesi B yang dipanggil paralel (harus selesai `< 50ms`).
   - Jalankan `go test` dan buktikan tes **GAGAL (RED)** karena `GetState` terblokir `>= 300ms` pada kode lama.
2. **Langkah 2 (Implementasi Inti):**
   - Di `job_interview.go`, `roleplay.go`, dan `story_relay.go`, salin field yang diperlukan saat memegang lock, panggil `uc.statesMu.Unlock()`, jalankan evaluasi LLM di luar lock, lalu `uc.statesMu.Lock()` kembali untuk memvalidasi state belum selesai dan menerapkan perubahan.
   - Tambahkan pembersihan sesi yang sudah `IsFinished` saat jumlah entry di `uc.states` melebihi batas (`MaxActiveStateCache = 500`).
3. **Langkah 3 (Refactor & Integrasi):**
   - Jalankan `go test -race ./internal/usecase/game/...` dan pastikan 0 data race serta 100% PASS.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
uc.statesMu.Lock()
state, exists := uc.states[cleanSessionID]
if !exists {
	uc.statesMu.Unlock()
	return nil, nil, domain.ErrSessionNotFound
}
if state.IsFinished {
	uc.statesMu.Unlock()
	return nil, nil, domain.ErrInterviewAlreadyEnded
}
currentIndex := state.CurrentQuestionIndex
currentQuestion := state.Questions[currentIndex]
domainField := state.Domain
difficulty := state.Difficulty
uc.statesMu.Unlock()

eval, err := uc.evaluateAnswer(ctx, domainField, difficulty, currentQuestion, cleanAnswer)
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Dua request paralel untuk `sessionID` yang sama ditangani dengan aman karena setelah mengunci kembali (`uc.statesMu.Lock()`), kode memverifikasi ulang `!state.IsFinished` dan `state.CurrentQuestionIndex == currentIndex`.
- **Strategi Rollback:** Revert perubahan pada `internal/usecase/game/`.

