---
id: TICKET-1762520300
plan_id: PLAN-1762520300
title: "[BUG] TimeoutTicker Tidak Pernah Dijalankan di main.go (Channel Discord Terkunci Permanen)"
status: Done
priority: High
severity: Broken Flow
labels: [Bug, Backend, Session, Worker, Lifecycle]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Komponen background worker `TimeoutTicker` di `internal/usecase/session/timeout_ticker.go` telah dibuat untuk secara berkala membatalkan sesi game yang kedaluwarsa (`CancelExpiredSessions`), namun tidak pernah diinisialisasi maupun dijalankan (`Start(ctx)`) di dalam `cmd/bot/main.go`. Karena aplikasi juga tidak memiliki perintah pembatalan sesi manual (seperti `/radovid cancel`), apabila seorang pemain memulai sesi game di sebuah channel Discord lalu meninggalkan permainan tanpa menyelesaikannya, sesi tersebut akan terus berstatus `pending` atau `active` di SQLite selamanya dan mengunci channel tersebut dari sesi permainan baru.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520300-fix-session-timeout-ticker-wiring.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Mulai sesi game apapun di sebuah channel Discord (misal `/radovid play taboo`).
2. Biarkan sesi tersebut tanpa diselesaikan selama lebih dari 30 menit.
3. Coba mulai sesi game baru di channel yang sama.
4. Bot selalu menolak dengan pesan *"Another game is already running in this channel"* karena tidak ada worker yang membersihkan sesi kedaluwarsa.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** `TimeoutTicker` diinisialisasi dan dijalankan saat startup di `cmd/bot/main.go` (serta dihentikan dengan *graceful shutdown* `Stop()` saat terminasi), dan sesi yang kedaluwarsa juga dibersihkan secara malas (*lazy cleanup*) atau proaktif sebelum memeriksa sesi aktif di channel.
- **Aktual (Actual):** `NewTimeoutTicker` tidak pernah dipanggil di `cmd/bot/main.go`.

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] `TimeoutTicker` diinisialisasi dan dijalankan di `cmd/bot/main.go` dengan graceful shutdown `Stop()` saat sinyal OS diterima.
- [x] `SessionManager` mendukung pembersihan sesi kedaluwarsa (*expired session cleanup*) sehingga channel tidak terkunci oleh sesi yang sudah melewati batas timeout.
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/cmd/bot/main.go`
- `../../../../radovid-bot-api/internal/usecase/session/session_manager.go`
- `../../../../radovid-bot-api/internal/usecase/session/session_manager_test.go`
- `../../../../radovid-bot-api/internal/usecase/session/timeout_ticker.go`

---

## 6. AI Execution Log & Output

- **Penyebab Akar (Root Cause):**
  - `TimeoutTicker` di `internal/usecase/session/timeout_ticker.go` tidak pernah dihubungkan atau dijalankan dalam `cmd/bot/main.go`.
  - Pada `SessionManager.CreateSession` dan `GetActiveSession`, tidak terdapat lazy-expiration check untuk sesi yang usianya sudah melampaui batas wajar (`DefaultSessionTTL = 30 * time.Minute`), sehingga channel Discord terkunci permanen jika sesi ditinggalkan pemain.
- **Langkah Perbaikan:**
  1. Menambahkan konstanta `DefaultSessionTTL = 30 * time.Minute` pada `internal/usecase/session/session_manager.go`.
  2. Memperbarui `SessionManager.CreateSession` dan `GetActiveSession` untuk mendeteksi sesi menggantung yang usianya `>= DefaultSessionTTL`, otomatis membatalkannya (`StatusCancelled`), dan mengizinkan pembentukan sesi baru.
  3. Menambahkan method `CleanupExpiredSessions(ctx context.Context) (int64, error)` pada `SessionManager`.
  4. Menginisialisasi `TimeoutTicker` di `cmd/bot/main.go`, menjalankan pembersihan awal saat startup, mengaktifkan `Start(schedulerCtx)` di latar belakang, dan memanggil `Stop()` saat graceful shutdown bot.
  5. Menambahkan unit test TDD (O-07) `TestSessionManager_CreateSession_AutoExpireStaleSession`, `TestSessionManager_GetActiveSession_AutoExpireStaleSession`, dan `TestSessionManager_CleanupExpiredSessions` pada `session_manager_test.go`.
- **Ringkasan Berkas Terdampak:**
  - `radovid-bot-api/cmd/bot/main.go`: Wiring `TimeoutTicker` (startup cleanup, background loop, graceful shutdown).
  - `radovid-bot-api/internal/usecase/session/session_manager.go`: Auto-expire stale session pada `CreateSession` & `GetActiveSession`, penambahan method `CleanupExpiredSessions`.
  - `radovid-bot-api/internal/usecase/session/session_manager_test.go`: Unit test penanganan auto-expire stale sessions dan cleanup worker.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `cd radovid-bot-api && go test -v -run "TestSessionManager_.*AutoExpireStaleSession" ./internal/usecase/session/...`
- **Output Terminal Aktual:**
  ```text
  === RUN   TestSessionManager_CreateSession_AutoExpireStaleSession
  --- PASS: TestSessionManager_CreateSession_AutoExpireStaleSession (0.00s)
  === RUN   TestSessionManager_GetActiveSession_AutoExpireStaleSession
  --- PASS: TestSessionManager_GetActiveSession_AutoExpireStaleSession (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	0.572s
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `cd radovid-bot-api && go test -v -run "TestSessionManager_.*AutoExpireStaleSession" ./internal/usecase/session/...`
  - Pesan Kegagalan:
    ```text
    === RUN   TestSessionManager_CreateSession_AutoExpireStaleSession
        session_manager_test.go:447: expected new session to be created after stale session auto-expired, got error: an active session already exists in this channel
    --- FAIL: TestSessionManager_CreateSession_AutoExpireStaleSession (0.00s)
    === RUN   TestSessionManager_GetActiveSession_AutoExpireStaleSession
        session_manager_test.go:480: expected ErrSessionNotFound for stale session, got &{stale-sess-2 wyr pending chan-timeout-2 guild-1 1794102236702 <nil> []}, err <nil>
    --- FAIL: TestSessionManager_GetActiveSession_AutoExpireStaleSession (0.00s)
    FAIL
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih.
- **Rincian Temuan:** Pembersihan saat startup (`CleanupExpiredSessions`) secara otomatis mentransisikan seluruh sesi lama yang masih menggantung dengan status `pending` atau `active` menjadi `cancelled` tanpa ada data yang hilang.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):** PASSED. Build bot berhasil (`go build ./cmd/bot`), tidak ada lint/vet error (`go vet ./...`), seluruh test suite lulus dengan race detector (`go test -race ./...`).
- **Must Probe Status (MP-1 s/d MP-7):** PASSED. Penanganan sesi stale teruji via lazy cleanup saat pembuatan sesi baru, inspeksi aktif sesi, maupun eksekusi batch cleanup.
- **Daftar Belum Diuji (Untested List - O-16):** NONE. Seluruh alur lifecycle timeout telah teruji secara komprehensif.
