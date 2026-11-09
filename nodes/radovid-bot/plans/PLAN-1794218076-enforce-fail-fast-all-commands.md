---
id: PLAN-1794218076
ticket_id: TICKET-1794218076
title: Enforce Fail-Fast Policy Across All Radovid Bot Commands
status: Completed
priority: High
---

# Rencana Implementasi Teknis: Enforce Fail-Fast Policy Across All Radovid Bot Commands

## 1. Analisis Kebutuhan & Akar Masalah
Sesuai regulasi pada [coding.md](../../../global-guidelines/coding.md) Seksi 2.1 (*Fail-Fast Policy*), sistem dilarang menggunakan silent fallback, dummy defaults, atau mock data pada kode produksi ketika integrasi layanan eksternal (Ollama LLM) mengalami kegagalan.

Saat ini ditemukan pola anti-pattern:
1. `ProcessDiary` pada `internal/usecase/feedback/diary.go` mengeksekusi `buildFallbackDiaryAnalysis` jika LLM error/timeout, yang menyetel `Corrected = Original` untuk seluruh 5 kalimat. Akibatnya, kalimat yang memiliki kesalahan fatal dianggap valid dan diberi tanda centang hijau.
2. `FeedbackUseCase.Evaluate` pada `internal/usecase/feedback/ai_feedback.go` mengeksekusi `buildFallbackResponse` dengan skor statis 7 dan array koreksi kosong.
3. Seluruh use case mini-game AI (`game_master.go`, `movie_review.go`, `news_reporter.go`, `dream_house.go`, `five_word.go`, `one_minute.go`, `boss_fight.go`, `taboo_evaluator.go`, `transformation.go`) memanggil `ollama.GetFallbackResponse` ketika generate LLM gagal.

## 2. Strategi Perubahan (Fail-Fast Architecture)
1. **Eliminasi Silent Fallback:**
   - Hapus pemanggilan dan fungsi `buildFallbackDiaryAnalysis` pada `diary.go`.
   - Hapus pemanggilan dan fungsi `buildFallbackResponse` pada `ai_feedback.go`.
   - Ganti pemanggilan `ollama.GetFallbackResponse` pada seluruh use case evaluasi game menjadi return explicit `error` bertipe descriptive (`fmt.Errorf("AI service unavailable: %w", err)`).
2. **Perketat Prompt Evaluasi Jurnal:**
   - Perbarui `daily_diary.tmpl` dengan instruksi ketat untuk mendeteksi kesalahan preposisi gerund, kapitalisasi, modal auxiliary verb, dan run-on sentence.
3. **Penyampaian Pesan Galat yang Jujur ke Pengguna:**
   - Pastikan layer delivery Discord menangkap error ini dan mengirimkan pesan kesalahan informatif (*"AI language evaluation service is currently unavailable. Please try again shortly."*) alih-alih menampilkan hasil palsu.
4. **Pembaruan Unit Test:**
   - Ubah seluruh skenario uji di `usecase/` agar menguji kegagalan fail-fast secara eksplisit (memastikan error tidak bernilai nil saat LLM gagal).
5. **Kepatuhan Aturan:**
   - Memastikan Zero-Comment Policy (`coding.md` Seksi 1.1) dipatuhi pada seluruh perubahan berkas Go.

## 3. Langkah Eksekusi Terinci
1. **Fase 1 (Feedback & Diary Use Cases):**
   - Refactor `diary.go` dan `ai_feedback.go` untuk melempar error saat `llmClient.Generate` atau unmarshal JSON gagal.
   - Refactor `daily_diary.tmpl` untuk instruksi evaluasi tata bahasa yang ketat.
2. **Fase 2 (Game AI Use Cases):**
   - Refactor `game_master.go`, `movie_review.go`, `news_reporter.go`, `dream_house.go`, `five_word.go`, `one_minute.go`, `boss_fight.go`, `taboo_evaluator.go`, dan `transformation.go`.
3. **Fase 3 (Unit Test & Verification):**
   - Perbarui unit test pada masing-masing use case.
   - Jalankan `go test -race ./...` di repositori `radovid-bot-api` untuk memverifikasi 100% kelulusan.
4. **Fase 4 (Dokumentasi):**
   - Perbarui status tiket dan catat entri pada `CHANGELOG.md`.

## 4. Hasil Eksekusi & Status Selesai
Seluruh 4 fase telah tereksekusi dengan sukses:
- Seluruh fungsi fallback diam-diam dan mock heuristic telah dibersihkan dari use cases `diary.go`, `ai_feedback.go`, `game_master.go`, `movie_review.go`, `news_reporter.go`, `dream_house.go`, `five_word.go`, `one_minute.go`, `boss_fight.go`, `taboo_evaluator.go`, dan `transformation.go`.
- Prompt `daily_diary.tmpl` diperketat dengan aturan audit grammar objektif.
- Seluruh unit test suite lulus 100% (`go test -race ./...` dan `go test -count=1 ./...`).
- Aturan Zero-Comment dipatuhi 100% pada semua berkas Go yang dimodifikasi.
