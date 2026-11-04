---
id: TICKET-1762261060
plan_id: PLAN-1762261060
title: Game Session State Machine
status: Todo
priority: High
labels: [Backend, Core, Session, Database, SQLite]
---

# Deskripsi
Sebagai sistem manajemen permainan Radovid Bot,
Saya ingin memiliki engine state machine untuk siklus hidup sesi game beserta persistensi SQLite dan ticker pemantau timeout otomatis,
Sehingga transisi status sesi (PENDING -> ACTIVE -> COMPLETED / CANCELLED) terkelola secara ketat, konsisten di database, dan sesi yang ditinggalkan oleh pemain dapat dibersihkan secara otomatis.

Latar belakang teknis: Sesi permainan di Discord bersifat stateful dan multi-tahap (pemain membuat lobi, pemain lain bergabung, game berlangsung beberapa ronde, hingga selesai atau ditinggalkan). Tanpa state machine formal dengan transisi tervalidasi dan persistensi ke tabel SQLite (`game_sessions` dan `session_participants`), status game bisa mengalami race condition, state desynchronization, atau ghost sessions yang memakan memori server saat pemain tidak melanjutkan permainan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762261060-game-session-state-machine.md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Definisi domain model `GameSession`, `SessionStatus` (`pending`, `active`, `completed`, `cancelled`), dan `SessionParticipant` di layer domain.
- [ ] Validasi transisi state machine ketat:
  - `PENDING` -> `ACTIVE` (saat game dimulai atau kuota pemain terpenuhi).
  - `PENDING` -> `CANCELLED` (saat dibatalkan host atau timeout lobi).
  - `ACTIVE` -> `COMPLETED` (saat game tuntas normal).
  - `ACTIVE` -> `CANCELLED` (saat dibatalkan atau sesi terabaikan).
  - State `COMPLETED` dan `CANCELLED` bersifat terminal; transisi keluar dari status terminal melempar `ErrInvalidStateTransition`.
- [ ] File migrasi SQL untuk tabel `game_sessions` dan `session_participants` menggunakan epoch milliseconds untuk timestamp.
- [ ] Repository interface `session.Repository` dan implementasi konkret SQLite dengan query berparameter (mencegah SQL injection).
- [ ] Background goroutine ticker untuk mendeteksi sesi kedaluwarsa (session timeout handler) dan mengubah statusnya menjadi `CANCELLED`.
- [ ] Unit test komprehensif untuk aturan transisi state machine dan operasi CRUD repository.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/002_create_game_sessions.sql`
- `../../../../radovid-bot-api/migrations/003_create_session_participants.sql`
- `../../../../radovid-bot-api/internal/domain/session/entity.go`
- `../../../../radovid-bot-api/internal/domain/session/errors.go`
- `../../../../radovid-bot-api/internal/domain/session/repository.go`
- `../../../../radovid-bot-api/internal/domain/session/entity_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/session_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/session_repository_test.go`
- `../../../../radovid-bot-api/internal/usecase/session/session_manager.go`
- `../../../../radovid-bot-api/internal/usecase/session/timeout_ticker.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. Pending execution.
- **Ringkasan File Terpengaruh:**
  - Pending execution.
- **Catatan & Keputusan Arsitektural:**
  - Pending execution.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v ./internal/domain/session/... ./internal/infrastructure/sqlite/...`
- **Output Terminal:**
  ```text
  Pending execution.
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** Pending
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** Pending
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): Pending
  - O-08 (Edge Case Atlas): Pending
  - O-09 (Damage Assessment): Pending
  - O-10 (Full Suite Metrics): Pending
  - O-11 (Real Thing Driven): Pending
  - O-12 (Shipping Env Check): Pending
  - Hunt Loop Final Round: Pending
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: Pending
  - CHANGELOG.md: Pending
  - Commit Message Sanity: Pending
  - O-16 Untested List: Pending
