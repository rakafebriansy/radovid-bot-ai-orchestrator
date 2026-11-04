---
id: PLAN-1762261060
ticket_id: TICKET-1762261060
title: Game Session State Machine
status: Draft
created_at: 2026-11-04T00:15:00+07:00
---

# Rencana Implementasi Teknis: Game Session State Machine

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261060](../tickets/TICKET-1762261060-game-session-state-machine.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengimplementasikan finite state machine (FSM) untuk siklus hidup game session pada layer domain (`internal/domain/session`) dengan persistensi transaksi ke SQLite (`internal/infrastructure/sqlite`). Sistem ini menjamin integritas transisi status (`pending` -> `active` -> `completed` / `cancelled`), mengaitkan pemain melalui entitas `session_participants`, serta membersihkan sesi terlantar melalui goroutine worker `timeout_ticker.go`.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler `/radovid start`, `/radovid join`, `/radovid end`, dan use case game interaktif.
  - Komponen Terdampak: `internal/domain/session`, `internal/infrastructure/sqlite`, skema database SQLite.
  - Potensi Efek Samping: Potensi lock contention pada SQLite jika transaksi write tidak ditutup dengan benar; diantisipasi dengan mode SQLite WAL dan query transaksional terisolasi.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/002_create_game_sessions.sql` | Skema DDL tabel game_sessions dengan index |
| `CREATE` | `../../../../radovid-bot-api/migrations/003_create_session_participants.sql` | Skema DDL tabel session_participants composite PK |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/session/entity.go` | Domain struct GameSession, SessionParticipant, SessionStatus enum |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/session/errors.go` | Definisi domain error terkait state machine |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/session/repository.go` | Interface SessionRepository (Port) |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/session/entity_test.go` | Unit test validasi transisi state machine domain |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/sqlite/session_repository.go` | Adapter implementasi SQLite SessionRepository |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/sqlite/session_repository_test.go` | Integration test SQLite menggunakan in-memory DB |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/session/session_manager.go` | Use case orkestrator pembuatan, join, dan mutasi state sesi |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/session/timeout_ticker.go` | Worker background berkala untuk auto-cancellation sesi timeout |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan Skema Migrasi SQL):**
   - Buat migrasi `002_create_game_sessions.sql` dan `003_create_session_participants.sql` sesuai standar [system-design.md](../docs/system-design.md) (kolom timestamp menggunakan BIGINT epoch ms).
2. **Langkah 2 (Domain Model & Red Test State Machine):**
   - Tulis `entity_test.go` yang memvalidasi transisi:
     - NewSession (PENDING) -> Start (ACTIVE)
     - ACTIVE -> Complete (COMPLETED)
     - ACTIVE -> Cancel (CANCELLED)
     - PENDING -> Cancel (CANCELLED)
     - Ilegal: COMPLETED -> ACTIVE (harus return `ErrInvalidStateTransition`)
     - Ilegal: CANCELLED -> COMPLETED (harus return `ErrInvalidStateTransition`)
   - Tulis `entity.go`, `errors.go`, dan `repository.go` pada `internal/domain/session` hingga seluruh unit test berstatus GREEN.
3. **Langkah 3 (Implementasi SQLite Repository):**
   - Tulis `session_repository_test.go` dengan `sqlite.Open(":memory:")` untuk menguji penyimpanan, query session aktif berdasarkan channel ID, dan update status.
   - Tulis `session_repository.go` dengan query SQL parameterized.
4. **Langkah 4 (Implementasi Use Case & Timeout Worker):**
   - Buat `session_manager.go` untuk menangani pembuatan sesi, penambahan partisipan, dan validasi duplikasi sesi aktif di channel yang sama (`ErrSessionAlreadyActive`).
   - Buat `timeout_ticker.go` dengan `time.NewTicker`, context cancellation untuk graceful shutdown, dan pemanggilan repository untuk membatalkan sesi kadaluwarsa (> 15 menit tanpa aktivitas).
5. **Langkah 5 (Refactor & Delivery Gate):**
   - Verifikasi tidak ada komentar non-fungsional, pastikan error handling tidak diabaikan (`err != nil`), dan periksa eksekusi tes 100% lulus.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```go
package session

import (
	"errors"
	"time"
)

type Status string

const (
	StatusPending   Status = "pending"
	StatusActive    Status = "active"
	StatusCompleted Status = "completed"
	StatusCancelled Status = "cancelled"
)

var (
	ErrInvalidStateTransition = errors.New("invalid session state transition")
	ErrSessionNotFound        = errors.New("game session not found")
	ErrSessionAlreadyActive   = errors.New("an active session already exists in this channel")
	ErrMaxParticipantsReached = errors.New("maximum participants reached for this session")
)

type ParticipantRole string

const (
	RoleHost   ParticipantRole = "host"
	RolePlayer ParticipantRole = "player"
)

type SessionParticipant struct {
	SessionID string
	UserID    string
	Role      ParticipantRole
	JoinedAt  int64
}

type GameSession struct {
	ID           string
	GameType     string
	Status       Status
	ChannelID    string
	GuildID      string
	CreatedAt    int64
	EndedAt      *int64
	Participants []SessionParticipant
}

func (s *GameSession) Start() error {
	if s.Status != StatusPending {
		return ErrInvalidStateTransition
	}
	s.Status = StatusActive
	return nil
}

func (s *GameSession) Complete() error {
	if s.Status != StatusActive {
		return ErrInvalidStateTransition
	}
	s.Status = StatusCompleted
	now := time.Now().UnixMilli()
	s.EndedAt = &now
	return nil
}

func (s *GameSession) Cancel() error {
	if s.Status == StatusCompleted || s.Status == StatusCancelled {
		return ErrInvalidStateTransition
	}
	s.Status = StatusCancelled
	now := time.Now().UnixMilli()
	s.EndedAt = &now
	return nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:**
  - Multiple `/radovid start` pada channel Discord yang sama: `session_manager.go` mengecek keberadaan sesi berstatus `pending` atau `active` di channel tersebut, menolak pembuatan sesi baru dengan `ErrSessionAlreadyActive`.
  - Shutdown bot saat sesi masih `active`: Ticker atau startup recovery menandai sesi yang melampaui timeout batas sebagai `cancelled`.
  - Transisi state konkuren: Operasi update status di database menggunakan conditional SQL `WHERE id = ? AND status = ?` untuk memastikan proteksi optimistik concurrency.
- **Strategi Rollback:**
  - Jalankan script SQL rollback untuk drop tabel jika diperlukan, atau batalkan commit Git.
