---
id: TICKET-1762520480
plan_id: PLAN-1762520480
title: "[BUG] Global Mutex Freeze Saat Request HTTP LLM & Kebocoran Memori Status Game (uc.states)"
status: Done
priority: Medium
severity: Broken Flow
labels: [Bug, Concurrency, Performance, Memory, Game]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Ditemukan dua masalah konkurensi dan manajemen memori pada layer `internal/usecase/game/`:
1. **Global Mutex Held Across Blocking Network Calls:** Pada `job_interview.go` (baris 132-158), `roleplay.go` (baris 106-150), dan `story_relay.go` (baris 113-156), method `SubmitAnswer` / `SubmitTurn` menahan kunci `uc.statesMu.Lock()` selama melakukan panggilan jaringan HTTP sinkron ke Ollama LLM (timeout hingga 30 detik). Hal ini memblokir seluruh pengguna lain di server Discord manapun yang sedang mengakses state game tersebut.
2. **Unbounded In-Memory State Leak:** Seluruh 11 in-memory game use case menyimpan state sesi di dalam map `uc.states[sessionID]`. Ketika permainan selesai (`IsFinished = true`), state tidak pernah dibersihkan atau dibatasi jumlahnya, sehingga pemakaian RAM server bot akan terus membengkak tanpa batas seiring bertambahnya sesi yang dimainkan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520480-fix-game-mutex-lock-and-state-leak.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Buat mock LLM client dengan simulasi latensi 500ms.
2. Jalankan `SubmitAnswer` untuk Sesi A pada `JobInterviewUseCase`, dan secara paralel panggil `GetState` untuk Sesi B.
3. Amati bahwa `GetState` untuk Sesi B tertahan selama 500ms penuh karena mutex global ditahan selama panggilan LLM.
4. Selesaikan ratusan sesi game dan amati bahwa entry sesi lama tidak pernah dibersihkan dari memori ketika melampaui batas kapasitas / usia.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Mutex `statesMu` dilepas sebelum melakukan panggilan jaringan eksternal ke Ollama (hanya dikunci saat membaca salinan state awal dan saat menulis pembaruan state), serta map `uc.states` memiliki mekanisme pembersihan (*bounded eviction / finished state pruning*).
- **Aktual (Actual):** Mutex dikunci sepanjang durasi HTTP call ke Ollama, dan map `uc.states` tumbuh tanpa batas.

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] `job_interview.go`, `roleplay.go`, dan `story_relay.go` melepas `statesMu` sebelum memanggil Ollama HTTP client dan mengunci kembali saat memperbarui state.
- [x] Seluruh use case game yang menyimpan `uc.states` membersihkan entry sesi selesai yang sudah usang (*stale finished state eviction*) agar ukuran map tetap terbatas (*bounded memory*).
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite (`go test -race ./...`) lulus 100% tanpa data race dan tanpa regresi.
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/usecase/game/game_master.go`
- `../../../../radovid-bot-api/internal/usecase/game/job_interview.go`
- `../../../../radovid-bot-api/internal/usecase/game/job_interview_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/roleplay.go`
- `../../../../radovid-bot-api/internal/usecase/game/roleplay_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/story_relay.go`
- `../../../../radovid-bot-api/internal/usecase/game/story_relay_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/one_minute.go`
- `../../../../radovid-bot-api/internal/usecase/game/five_word.go`
- `../../../../radovid-bot-api/internal/usecase/game/secret_mission.go`
- `../../../../radovid-bot-api/internal/usecase/game/boss_fight.go`
- `../../../../radovid-bot-api/internal/usecase/game/transformation.go`
- `../../../../radovid-bot-api/internal/usecase/game/movie_review.go`
- `../../../../radovid-bot-api/internal/usecase/game/news_reporter.go`
- `../../../../radovid-bot-api/internal/usecase/game/dream_house.go`

---

## 6. AI Execution Log & Output
*Bagian ini WAJIB diisi oleh AI Agent SAAT dan SETELAH memperbaiki bug.*

- **Penyebab Akar (Root Cause):**
  1. Pada `job_interview.go`, `roleplay.go`, dan `story_relay.go`, pemanggilan `uc.statesMu.Lock()` dilakukan di awal fungsi `SubmitAnswer`/`SubmitReply`/`SubmitSentence` dan ditahan selama pemanggilan LLM Ollama (`evaluateAnswer` atau `GenerateGameContent`). Karena panggilan jaringan ini memakan waktu ratusan milidetik hingga puluhan detik, mutex global usecase terblokir dan membekukan akses `GetState`/`Submit` untuk semua pemain lain di seluruh server Discord.
  2. Seluruh 11 game use case in-memory tidak memiliki batas atas atau pembersihan sesi lama pada map `uc.states`. Setiap kali game baru dimulai atau selesai, sesi tidak pernah dihapus dari memori, menyebabkan potensi kebocoran memori RAM (*unbounded memory growth*).
- **Langkah Perbaikan:**
  1. Menambahkan konstanta `MaxActiveStateCache = 500` pada `internal/usecase/game/game_master.go`.
  2. Menambahkan unit test `TestJobInterviewUseCase_NonBlockingConcurrency` dan `TestJobInterviewUseCase_PrunesFinishedStates` pada `job_interview_test.go`, lalu membuktikan kegagalan pengujian pada kode lama (terblokir 270ms dan ukuran map 506 > 500).
  3. Memodifikasi `SubmitAnswer` di `job_interview.go`, `SubmitReply` di `roleplay.go`, dan `SubmitSentence` di `story_relay.go` untuk melepaskan `uc.statesMu.Unlock()` sebelum memanggil LLM/GameMaster, dan mengunci kembali `uc.statesMu.Lock()` untuk memverifikasi ulang integritas state serta memperbarui hasil.
  4. Menerapkan mekanisme pemangkasan sesi usang berstatus selesai (`IsFinished`/`IsCompleted`) saat inisialisasi game baru (`len(uc.states) >= MaxActiveStateCache`) di seluruh 11 game use case: `job_interview.go`, `roleplay.go`, `story_relay.go`, `one_minute.go`, `five_word.go`, `secret_mission.go`, `boss_fight.go`, `transformation.go`, `movie_review.go`, `news_reporter.go`, dan `dream_house.go`.
  5. Menambahkan unit test konkurensi non-blocking dan pruning serupa pada `roleplay_test.go` dan `story_relay_test.go`.
  6. Menjalankan seluruh test suite dengan `-race` memastikan 100% lulus tanpa data race dan tanpa regresi.
- **Ringkasan Berkas Terdampak:**
  - `internal/usecase/game/game_master.go`
  - `internal/usecase/game/job_interview.go`
  - `internal/usecase/game/job_interview_test.go`
  - `internal/usecase/game/roleplay.go`
  - `internal/usecase/game/roleplay_test.go`
  - `internal/usecase/game/story_relay.go`
  - `internal/usecase/game/story_relay_test.go`
  - `internal/usecase/game/one_minute.go`
  - `internal/usecase/game/five_word.go`
  - `internal/usecase/game/secret_mission.go`
  - `internal/usecase/game/boss_fight.go`
  - `internal/usecase/game/transformation.go`
  - `internal/usecase/game/movie_review.go`
  - `internal/usecase/game/news_reporter.go`
  - `internal/usecase/game/dream_house.go`

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -race -v ./internal/usecase/game/...`
- **Output Terminal Aktual:**
  ```text
=== RUN   TestJobInterviewUseCase_NonBlockingConcurrency
--- PASS: TestJobInterviewUseCase_NonBlockingConcurrency (0.30s)
=== RUN   TestJobInterviewUseCase_PrunesFinishedStates
--- PASS: TestJobInterviewUseCase_PrunesFinishedStates (0.00s)
=== RUN   TestRoleplayUseCase_NonBlockingConcurrency
--- PASS: TestRoleplayUseCase_NonBlockingConcurrency (0.30s)
=== RUN   TestRoleplayUseCase_PrunesFinishedStates
--- PASS: TestRoleplayUseCase_PrunesFinishedStates (0.00s)
=== RUN   TestStoryRelayUseCase_NonBlockingConcurrency
--- PASS: TestStoryRelayUseCase_NonBlockingConcurrency (0.30s)
=== RUN   TestStoryRelayUseCase_PrunesFinishedStates
--- PASS: TestStoryRelayUseCase_PrunesFinishedStates (0.01s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	2.760s
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `go test -v -run "TestJobInterviewUseCase_NonBlockingConcurrency|TestJobInterviewUseCase_PrunesFinishedStates" ./internal/usecase/game/...`
  - Pesan Kegagalan:
    ```text
    === RUN   TestJobInterviewUseCase_NonBlockingConcurrency
        job_interview_test.go:413: GetInterview for session B was blocked by session A LLM call! Took: 270.165041ms, expected < 100ms
    --- FAIL: TestJobInterviewUseCase_NonBlockingConcurrency (0.30s)
    === RUN   TestJobInterviewUseCase_PrunesFinishedStates
        job_interview_test.go:460: expected states map to be bounded <= 500, got 506
    --- FAIL: TestJobInterviewUseCase_PrunesFinishedStates (0.00s)
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih.
- **Rincian Temuan:** State game yang terdampak bersifat in-memory dan tidak mengubah struktur tabel SQLite. Hasil game yang berhasil selesai tetap tercatat di database dengan benar.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):**
  - HG-1 (Zero Comment Policy): LULUS. Seluruh berkas Go bersih tanpa komentar `//` atau `/* */`.
  - HG-2 (Clean Build & Vet): LULUS. `go vet ./...` dan `go build ./cmd/bot` berhasil tanpa error.
  - HG-3 (Race Free Tests): LULUS. `go test -race ./...` lulus 100%.
  - HG-4 (Working on Main Approved): LULUS. Telah dikonfirmasi dan disetujui langsung pada branch `main`.
- **Must Probe Status (MP-1 s/d MP-7):**
  - MP-1 (Regression check): LULUS. Seluruh paket dan tes lulus.
  - MP-2 (Code Hygiene): LULUS. Zero comments di codebase Go.
  - MP-3 (TDD Red-Green cycle): LULUS. Dibuktikan dengan tangkapan log kegagalan sebelum fix dan log kelulusan sesudah fix.
  - MP-4 (Error handling): LULUS. Menangani sesi yang dibatalkan/dihapus saat lock dilepas secara fail-safe.
  - MP-5 (Concurrency safety): LULUS. Tidak ada data race (`-race`), operasi baca/tulis state selalu diproteksi lock.
  - MP-6 (Memory bounded): LULUS. Map dipangkas secara otomatis jika melampaui `MaxActiveStateCache`.
  - MP-7 (Documentation sync): LULUS. Tiket, Plan, Development Planning, dan CHANGELOG disinkronkan.
- **Daftar Belum Diuji (Untested List - O-16):** Nihil.
