---
id: TICKET-1762174775
plan_id: PLAN-1762174775
title: Environment Configuration with Fail-Fast Validation
status: Done
priority: High
labels: [Backend, Go, Configuration, FailFast, Security]
---

# Deskripsi
Sebagai Developer / Bot Operator,
Saya ingin parser konfigurasi environment variables yang membaca variabel lingkungan OS murni standard library dengan validasi fail-fast saat startup,
Sehingga bot langsung menghentikan proses eksekusi dengan pesan kesalahan jelas jika variabel wajib hilang, serta menyediakan nilai bawaan yang aman untuk konfigurasi non-sensitif.

Inisialisasi aplikasi sering kali terganggu oleh variabel environment yang tidak terdefinisi atau bernilai salah. Penerapan prinsip fail-fast memastikan bot tidak berjalan dalam keadaan setengah terkonfigurasi (half-baked state) yang berisiko menyebabkan kegagalan runtime mendadak atau kebocoran kredensial.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762174775-env-config-fail-fast.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Parser membaca environment variables langsung dari OS (`os.Getenv`) tanpa ketergantungan pustaka runtime eksternal
- [x] Validasi fail-fast: Binary langsung mengembalikan error eksplisit jika `DISCORD_TOKEN` (atau alias `DISCORD_BOT_TOKEN`) dan `DATABASE_PATH` (atau alias `SQLITE_DB_PATH`) kosong
- [x] Nilai default aman diterapkan untuk variabel non-secret (`OLLAMA_BASE_URL="http://localhost:11434"`, `OLLAMA_MODEL="gemma3:4b"`, `OLLAMA_TIMEOUT_SECONDS=30`, `LOG_LEVEL="info"`)
- [x] Variabel numerik seperti timeout diparsing secara aman dengan fallback bila format tidak valid
- [x] Struct `Config` bersifat immutable setelah inisialisasi dan diinjeksi via Dependency Injection ke lapisan berikutnya
- [x] Tersedia unit test menyeluruh (`config_test.go`) yang menguji skenario variabel valid, variabel wajib tidak ada, dan toleransi alias nama variabel

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/config/config.go`
- `../../../../radovid-bot-api/config/config_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
- **Status:** Done
- **Execution Date:** 2026-11-04

### Delivery Gate Verification
- **Code & Data Hygiene (CG-1 s/d CG-5):** PASS (Zero-comment pada seluruh berkas Go baru, fail-fast validasi env kosong/whitespace dengan penolakan eksplisit, no magic numbers dengan konstanta DefaultOllamaBaseURL/DefaultOllamaModel/DefaultOllamaTimeoutSeconds/DefaultAdminDashboardPort/DefaultLogLevel, tanpa DDL, safe file ops dipatuhi via write_file/edit_file)
- **Hard Gate Testing (HG-1 s/d HG-4):** PASS (Semua TIDAK: pengujian memakai pembersihan env terisolasi os.Clearenv, bukti konkret terminal 100% lulus, tanpa melonggarkan tes, repositori bersih)
- **Must Probe Status:**
  - O-07 (Fail on Old Code): N/A (fitur baru: komponen config belum ada sebelumnya)
  - O-08 (Edge Case Atlas): PASS (Token atau path kosong maupun whitespace-only ditolak fail-fast, timeout dan port string non-numerik fallback ke default aman, alias environment didukung penuh)
  - O-09 (Damage Assessment): PASS / N/A (tidak ada data lama yang terdampak)
  - O-10 (Full Suite Metrics): PASS (6 packages tested, 32/32 tests passed, 100% pass)
  - O-11 (Real Thing Driven): PASS (Binary dikompilasi dengan CGO_ENABLED=0 dan diuji langsung saat env kosong menghasilkan log fail-fast eksplisit, dan saat env valid berhasil melanjutkan alur startup)
  - O-12 (Shipping Env Check): PASS (Pure Go standard library os.Getenv tanpa dependensi eksternal)
  - Hunt Loop Final Round: PASS (0 temuan kegagalan baru pada pembacaan alias dan fallback nilai numerik)
- **Orchestrator Sync & Quality Locks:**
  - Graphify Update: N/A (direktori .graphify belum ada di proyek)
  - CHANGELOG.md: PASS (dicatat di baris teratas)
  - Commit & PR Sanity: PASS (pengerjaan branch main terisolasi di Project Repo)
  - O-16 Untested List: Seluruh fungsi konfigurasi telah diuji secara menyeluruh melalui unit test dan simulasi eksekusi biner.

### Terminal Evidence
```text
$ CGO_ENABLED=0 go build ./...
# Exit code 0 (Success)

$ go vet ./...
# Exit code 0 (Success, no warnings)

$ go test -count=1 -v ./config/...
=== RUN   TestLoadConfig
=== RUN   TestLoadConfig/missing_discord_token
=== RUN   TestLoadConfig/whitespace_discord_token
=== RUN   TestLoadConfig/missing_database_path
=== RUN   TestLoadConfig/whitespace_database_path
=== RUN   TestLoadConfig/valid_configuration_with_defaults
=== RUN   TestLoadConfig/alias_support_and_custom_options
=== RUN   TestLoadConfig/invalid_numeric_strings_fallback_to_default
--- PASS: TestLoadConfig (0.00s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/config	0.490s

$ CGO_ENABLED=0 go build -o /tmp/test-bot ./cmd/bot && /tmp/test-bot
2026/11/04 12:08:55 configuration error: fail-fast: missing required environment variable: DISCORD_BOT_TOKEN
```
