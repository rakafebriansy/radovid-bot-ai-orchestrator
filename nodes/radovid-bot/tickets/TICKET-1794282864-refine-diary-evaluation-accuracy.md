---
id: TICKET-1794282864
plan_id: PLAN-1794282864
title: Eliminate LLM Hallucinations and Duplicate Vocabularies in Daily Diary Review
status: Done
priority: High
labels: [Backend, Go, Ollama, PromptEngineering, Quality, Bug]
---

# Deskripsi
Sebagai Pengguna Radovid Bot,
Saya ingin fitur `/radovid diary` memberikan koreksi kalimat, alasan tata bahasa (*explanation*), dan rekomendasi kosakata (*contextual vocabulary*) yang benar-benar akurat tanpa halusinasi contoh prompt atau pengulangan kata yang sudah saya tulis,
Sehingga umpan balik belajar bahasa Inggris dari bot dapat dipercaya, relevan, dan tidak membingungkan pengguna saat menggunakan model lokal berukuran kecil (`qwen2.5:1.5b`).

Konteks Masalah:
Berdasarkan hasil pengujian live `/radovid diary` di Discord menggunakan model lokal `qwen2.5:1.5b`, urutan kalimat 1–5 dan `overall_note` sudah berjalan sesuai urutan, namun ditemukan 4 ketidakakuratan evaluasi:
1. **Halusinasi Penjelasan Akibat Copy-Paste Contoh Prompt:** Pada Sentence 1 (`"i woke up early this morning"` ➔ `"I woke up early this morning"`), model menyalin mentah-mentah teks contoh dari prompt (`"Capitalized 'I', added 'the', and pluralized 'skills'."`), padahal tidak ada kata `'the'` maupun `'skills'` pada kalimat tersebut.
2. **Penambahan Kata yang Tidak Perlu Akibat Contoh Prompt:** Pada Sentence 2 (`"i went to the academy in order to learn coding"`), kalimat sebenarnya sudah benar secara tata bahasa (hanya kurang kapitalisasi `'I'`), tetapi model memaksakan penambahan kata `"skills"` di belakang `"learn coding"` karena terpengaruh contoh `"coding skills"` di dalam prompt.
3. **Penjelasan Tidak Sinkron untuk Koreksi Murni Kapitalisasi/Tanda Baca:** Pada Sentence 3 (`"my friends and i worked..."` ➔ `"My friends and I worked..."`), kata-kata pengguna tidak berubah sama sekali selain huruf kapital (`M` dan `I`), namun model mengarang penjelasan `"fixed verb tense consistency"`. Pada Sentence 4 & 5 (`"i enjoyed..."` ➔ `"I enjoyed..."`, `"trust..."` ➔ `"Trust..."`), model mengubah huruf kapital awal tetapi menulis `"No errors in this sentence."`.
4. **Rekomendasi Vocabulary Mengulang Kata Input Pengguna:** Model merekomendasikan kata `"challenge"` dan `"process"` yang sudah ditulis sendiri oleh pengguna di kalimat ke-3 (`"final challenge project"`) dan ke-5 (`"trust our process"`).

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1794282864-refine-diary-evaluation-accuracy.md)
*(Rincian arsitektur, determinisme parser, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Prompt `daily_diary.tmpl` dibersihkan dari contoh kata spesifik (`"skills"`, `"the"`, `"verb tense consistency"`, dll.) yang memicu perilaku *copy-paste* halusinasi pada model LLM berukuran kecil (`qwen2.5:1.5b`).
- [x] Prompt `daily_diary.tmpl` menginstruksikan model untuk tidak menambahkan kata baru yang mengubah kalimat yang sudah benar secara tata bahasa (seperti mengubah `"learn coding"` menjadi `"learn coding skills"`).
- [x] Parser `diary.go` (`normalizeExplanation`) mendeteksi secara deterministik apabila perbedaan antara kalimat asli (`Original`) dan kalimat koreksi (`Corrected`) **murni hanya berupa huruf kapital dan/atau tanda baca akhir** (seluruh kata identik secara *case-insensitive*). Dalam kondisi tersebut, penjelasan (`Explanation`) wajib di-override secara deterministik menjadi penjelasan kapitalisasi/tanda baca yang akurat (misal: `"Capitalized the sentence and pronoun 'I'."` atau `"Capitalized the start of the sentence."`), sehingga mengeliminasi 100% halusinasi penjelasan LLM pada kalimat yang sudah benar susunan katanya.
- [x] Parser `diary.go` memperluas deteksi kalimat penjelasan kontradiktif (`"No errors in this sentence"`, `"No error in this sentence"`, dll.) pada `isNoCorrectionExplanation`.
- [x] Parser `diary.go` melakukan pemfilteran deterministik pada `Vocabularies` (`filterNewVocabularies`) untuk membuang kata rekomendasi yang sudah muncul sebagai kata utuh di dalam 5 kalimat input pengguna (`originalSentences`), serta menggantinya dengan kosakata kontekstual baru apabila LLM mengulang kata milik pengguna.
- [x] Unit test pada `diary_test.go` dan `embed_test.go` diperbarui untuk menguji kasus: (a) halusinasi penjelasan saat perubahan murni kapitalisasi, (b) string `"No errors in this sentence."`, dan (c) pemfilteran kosakata duplikat terhadap kalimat input pengguna.
- [x] Kebijakan Zero-Comment (`coding.md` Seksi 1.1) dipatuhi 100% pada seluruh berkas Go yang dimodifikasi.
- [x] Seluruh unit test lulus (`go test -race ./...`) tanpa regresi.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/daily_diary.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/diary.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/diary_test.go`
- `CHANGELOG.md`

---

## AI Execution Log & Output

- **Langkah Teknis Tereksekusi:**
  1. Membersihkan contoh bias dari `daily_diary.tmpl` (menghapus `"coding skills"` dan contoh penjelasan literal yang memicu penyalinan halusinasi).
  2. Mengimplementasikan deteksi deterministik perbedaan murni kapitalisasi/tanda baca pada `normalizeExplanation` di `diary.go`.
  3. Memperbaiki dan memperluas `isNoCorrectionExplanation` agar aman terhadap substring seperti `"Corrected"`.
  4. Mengimplementasikan `extractUserWords` dan `filterNewVocabularies` yang menyaring kata rekomendasi agar bebas dari kata yang telah digunakan pengguna, dengan cadangan kosakata refleksi CEFR B2-C1 (`curatedReflectionVocabularies`) untuk menjamin tepat 3 rekomendasi kosakata baru.
  5. Memperbarui `diary_test.go` dan `embed_test.go` untuk memverifikasi penanganan halusinasi, string kontradiktif, dan pemfilteran kosakata duplikat.
  6. Memverifikasi seluruh unit test suite dengan race detector (`go test -race ./...`) dan static analysis (`go vet ./...`).

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v -run TestDiaryUseCase_ProcessDiary ./internal/usecase/feedback/...`
- **Output Terminal:**
  ```text
  === RUN   TestDiaryUseCase_ProcessDiary
  === RUN   TestDiaryUseCase_ProcessDiary/empty_user_id
  === RUN   TestDiaryUseCase_ProcessDiary/fewer_than_5_sentences
  === RUN   TestDiaryUseCase_ProcessDiary/sentence_too_short
  === RUN   TestDiaryUseCase_ProcessDiary/successful_processing_with_llm_output
  === RUN   TestDiaryUseCase_ProcessDiary/normalizes_original_bindings,_subject_pronoun_order,_and_capitalization_explanations
  === RUN   TestDiaryUseCase_ProcessDiary/eliminates_explanation_hallucinations_and_filters_duplicate_vocabularies
  === RUN   TestDiaryUseCase_ProcessDiary/fail-fast_when_llm_returns_error
  === RUN   TestDiaryUseCase_ProcessDiary/fail-fast_when_llm_returns_unparseable_json
  --- PASS: TestDiaryUseCase_ProcessDiary (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/feedback	0.426s
  ```

- **Perintah Full Suite:** `go test -race ./...`
- **Output Terminal:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/config	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	1.638s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/http	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/achievement	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/diary	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/feedback	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/gamemaster	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/notification	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/progress	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/session	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/user	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	1.982s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/achievement	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/admin	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/feedback	2.432s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	3.814s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/notification	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/progress	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/user	(cached)
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** PASS (Zero-comment dipatuhi pada seluruh file Go, no hardcoded secrets, no direct DDL, safe file ops)
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** PASS (Bukan DB prod, ada bukti konkret terminal, tanpa pelonggaran tes, repositori bersih)
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): PASS — Terbukti pada kode lama, `TestDiaryUseCase_ProcessDiary/eliminates_explanation_hallucinations_and_filters_duplicate_vocabularies` gagal dengan pesan `expected deterministic capitalization for sentence 1, got "Capitalized 'I', added 'the', and pluralized 'skills'."`.
  - O-08 (Edge Case Atlas): PASS (Sumbu: input murni huruf kecil, penulisan nama proper noun, tanda baca akhir hilang, semua kosakata LLM duplikat dengan input user, prompt tanpa-contoh).
  - O-09 (Damage Assessment): PASS (Kondisi bersih, struktur DiaryEntry dan skema database SQLite tidak terdampak).
  - O-10 (Full Suite Metrics): PASS — 100% test packages passed (`go test -race ./...`).
  - O-11 (Real Thing Driven): PASS — Diuji via pemanggilan domain use case komprehensif.
  - O-12 (Shipping Env Check): PASS — Format JSON prompt dan fallback terkurasi selaras dengan model lokal Ollama.
  - Hunt Loop Final Round: PASS (0 temuan baru di putaran terakhir).
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: N/A (tidak terdapat direktori `.graphify` di codebase).
  - CHANGELOG.md: PASS (diperbarui di baris teratas).
  - Commit Message Sanity: PASS (steril dari istilah orchestrator).
  - O-16 Untested List: Interaksi jaringan langsung ke endpoint Ollama GPU Windows live (memerlukan container Docker bot aktif di environment WSL).

