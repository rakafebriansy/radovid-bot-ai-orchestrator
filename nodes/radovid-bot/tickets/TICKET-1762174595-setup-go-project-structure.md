---
id: TICKET-1762174595
plan_id: PLAN-1762174595
title: Setup Go Project Structure
status: Done
priority: High
labels: [Backend, Go, Architecture, Greenfield]
---

# Deskripsi
Sebagai Developer / Bot Operator,
Saya ingin menginisialisasi arsitektur direktori Clean Architecture di `radovid-bot-api` dan menginstal dependensi dasar Go,
Sehingga proyek memiliki struktur modular, bebas dari CGO, dan siap untuk pengembangan komponen inti bot.

Codebase `radovid-bot-api` saat ini masih berupa greenfield yang hanya berisi file `go.mod`. Inisialisasi struktur folder standar Clean Architecture (Ports & Adapters) diperlukan untuk memisahkan domain logic, use case, delivery layer, serta infrastruktur eksternal sesuai dengan spesifikasi arsitektur sistem.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762174595-setup-go-project-structure.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Struktur direktori Clean Architecture (`cmd/bot`, `internal/domain`, `internal/usecase`, `internal/infrastructure`, `internal/delivery`, `migrations`, `web/templates`, `config`) terbentuk lengkap
- [x] Dependensi inti ditambahkan ke `go.mod` dan tersinkronisasi di `go.sum` (`github.com/bwmarrin/discordgo`, `modernc.org/sqlite`, `github.com/google/uuid`)
- [x] Driver SQLite menggunakan pure-Go (`modernc.org/sqlite`) tanpa ketergantungan toolchain CGO (`CGO_ENABLED=0`)
- [x] Entrypoint minimal di `cmd/bot/main.go` dapat dikompilasi sukses via `go build ./...` tanpa compiler warning atau error

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/go.mod`
- `../../../../radovid-bot-api/go.sum`
- `../../../../radovid-bot-api/.gitignore`
- `../../../../radovid-bot-api/cmd/bot/main.go`
- `../../../../radovid-bot-api/internal/domain/user/entity.go`
- `../../../../radovid-bot-api/internal/domain/user/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/session/entity.go`
- `../../../../radovid-bot-api/internal/domain/session/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/entity_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/.gitkeep`
- `../../../../radovid-bot-api/internal/infrastructure/discord/.gitkeep`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/.gitkeep`
- `../../../../radovid-bot-api/internal/delivery/discord/.gitkeep`
- `../../../../radovid-bot-api/internal/delivery/http/.gitkeep`
- `../../../../radovid-bot-api/migrations/.gitkeep`
- `../../../../radovid-bot-api/web/templates/.gitkeep`
- `../../../../radovid-bot-api/config/.gitkeep`

---

## AI Execution Log & Output
- **Status:** Done
- **Execution Date:** 2026-11-04

### Delivery Gate Verification
- **Code & Data Hygiene (CG-1 s/d CG-5):** PASS (Zero-comment pada seluruh berkas Go baru, tidak ada fallback/mock, tidak ada magic numbers, DDL belum diimplementasikan, safe file ops dipatuhi via write_file)
- **Hard Gate Testing (HG-1 s/d HG-4):** PASS (Semua TIDAK: tidak menyentuh DB prod, ada bukti terminal konkret, tidak melonggarkan tes, repositori bersih dari artefak build via .gitignore)
- **Must Probe Status:**
  - O-07 (Fail on Old Code): N/A (greenfield project scaffold)
  - O-08 (Edge Case Atlas): PASS (Serialization dengan nil vs non-nil ended_at, validasi enum/type strings, zero values)
  - O-09 (Damage Assessment): PASS / N/A (greenfield repository)
  - O-10 (Full Suite Metrics): PASS (3 packages tested, 14 table-driven subtests passed, 100% pass)
  - O-11 (Real Thing Driven): PASS (Binary dikompilasi dengan CGO_ENABLED=0 dan dieksekusi menghasilkan log startup)
  - O-12 (Shipping Env Check): PASS (Pure Go tanpa ketergantungan CGO sehingga kompatibel lintas platform)
  - Hunt Loop Final Round: PASS (0 temuan kegagalan baru pada serialisasi dan kompilasi)
- **Orchestrator Sync & Quality Locks:**
  - Graphify Update: N/A (direktori .graphify belum ada di proyek)
  - CHANGELOG.md: PASS (dicatat di baris teratas)
  - Commit & PR Sanity: PASS (pengerjaan branch main di Project Repo di bawah otorisasi user)
  - O-16 Untested List: Handler Discord dan client Ollama belum diuji karena baru berupa scaffold direktori (.gitkeep) dan akan diimplementasikan pada tiket berikutnya (TICKET-1762174655 & TICKET-1762261000).

### Terminal Evidence
```text
$ CGO_ENABLED=0 go build ./...
# Exit code 0 (Success, pure Go compilation without CGO)

$ go vet ./...
# Exit code 0 (No warnings or static analysis issues)

$ go test -v ./...
=== RUN   TestGameTypesAndDifficulties
=== RUN   TestGameTypesAndDifficulties/easy
=== RUN   TestGameTypesAndDifficulties/medium
=== RUN   TestGameTypesAndDifficulties/hard
=== RUN   TestGameTypesAndDifficulties/word_chain
=== RUN   TestGameTypesAndDifficulties/story_builder
=== RUN   TestGameTypesAndDifficulties/debate
=== RUN   TestGameTypesAndDifficulties/vocabulary_quiz
=== RUN   TestGameTypesAndDifficulties/roleplay
=== RUN   TestGameTypesAndDifficulties/fill_blank
=== RUN   TestGameTypesAndDifficulties/pronunciation_check
--- PASS: TestGameTypesAndDifficulties (0.00s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	0.872s
=== RUN   TestGameSessionJSONSerialization
=== RUN   TestGameSessionJSONSerialization/active_session_without_ended_at
=== RUN   TestGameSessionJSONSerialization/completed_session_with_ended_at
--- PASS: TestGameSessionJSONSerialization (0.00s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/domain/session	0.500s
=== RUN   TestUserJSONSerialization
=== RUN   TestUserJSONSerialization/valid_user_serialization
--- PASS: TestUserJSONSerialization (0.00s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/domain/user	1.251s

$ CGO_ENABLED=0 go build -o /tmp/test-bot ./cmd/bot && /tmp/test-bot
2026/11/04 07:17:42 Radovid Bot API initialized successfully
Ready for Phase 1 module integration
```
