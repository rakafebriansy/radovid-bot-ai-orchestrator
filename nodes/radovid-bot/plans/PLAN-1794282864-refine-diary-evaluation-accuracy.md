---
id: PLAN-1794282864
ticket_id: TICKET-1794282864
title: Eliminate LLM Hallucinations and Duplicate Vocabularies in Daily Diary Review
status: Completed
priority: High
---

# Rencana Implementasi Teknis: Eliminate LLM Hallucinations and Duplicate Vocabularies in Daily Diary Review

## 1. Analisis Kebutuhan & Akar Masalah
Pada pengujian live fitur `/radovid diary` dengan model lokal `qwen2.5:1.5b`, teridentifikasi empat kelemahan perilaku model kecil terhadap struktur prompt dan parser saat ini:

1. **Prompt Example Echoing (Copy-Paste Hallucination):**
   Model berukuran 1.5B parameter cenderung menyalin literal contoh yang tertulis di dalam `daily_diary.tmpl` (seperti `(e.g. "Capitalized 'I', added 'the', and pluralized 'skills'.")` dan `plural nouns ("coding skills")`) ke dalam output JSON meskipun kalimat pengguna tidak memiliki masalah tersebut.
2. **Over-Correction pada Kalimat Valid:**
   Kalimat `"i went to the academy in order to learn coding"` yang secara tata bahasa sudah tepat (kecuali huruf kapital `'i'`) diubah secara tidak perlu menjadi `"I went to the academy in order to learn coding skills"` karena bias contoh `"coding skills"` pada prompt.
3. **Ketidaksinkronan Explanation pada Perubahan Murni Kapitalisasi:**
   Ketika pengguna memasukkan kalimat yang susunan kata dan tata bahasanya sudah 100% benar namun ditulis dengan huruf kecil (misal `"i woke up early this morning"`, `"my friends and i worked on our final challenge project"`, `"i enjoyed this day a lot"`, `"trust our process"`), `Original` dan `Corrected` hanya berbeda pada huruf kapital (`i` ➔ `I`, `my` ➔ `My`, `trust` ➔ `Trust`). Namun model `1.5b` menghasilkan penjelasan yang tidak relevan (`"added 'the' and pluralized 'skills'"`, `"fixed verb tense consistency"`, atau `"No errors in this sentence."`).
4. **Duplikasi Kosakata (Vocabulary Echoing):**
   Model merekomendasikan kata `"challenge"` dan `"process"` di dalam array `"vocabularies"`, padahal kedua kata tersebut sudah digunakan secara eksplisit oleh pengguna di dalam kalimat ke-3 dan ke-5.

## 2. Strategi Perubahan Arsitektural
1. **Sanitasi Prompt Tanpa Contoh Bias (`daily_diary.tmpl`):**
   - Hapus seluruh contoh literal spesifik (`"coding skills"`, `"went to the academy"`, `"added 'the', and pluralized 'skills'"`) dari `daily_diary.tmpl`.
   - Instruksikan dengan tegas: jangan menambah atau mengganti kata jika tata bahasa kalimat sudah benar; hanya perbaiki kesalahan tata bahasa nyata dan kapitalisasi.
   - Minta 5 kandidat kosakata tingkat menengah/lanjut di `"vocabularies"` yang relevan dengan topik namun tidak muncul di kalimat pengguna, sehingga parser Go memiliki cadangan kandidat jika ada kata yang terfilter.
2. **Determinisme Penjelasan Kapitalisasi di Go (`diary.go`):**
   - Pada `normalizeExplanation(orig, corr, expl string)`:
     Bandingkan token kata `orig` dan `corr` setelah tanda baca akhir (`.!?`) dihapus.
     Apabila `strings.EqualFold(origBase, corrBase)` bernilai `true` (yang berarti **tidak ada satu pun kata yang berubah, ditambah, atau dihapus** antara kalimat asli dan kalimat koreksi — murni perbedaan huruf kapital dan/atau tanda baca akhir), jangan pernah gunakan teks penjelasan bebas dari LLM.
     Sebaliknya, hasilkan penjelasan secara deterministik berdasarkan perbedaan aktual antara `orig` dan `corr`:
     - Jika kata ganti `"i"` berubah menjadi `"I"` dan huruf awal kalimat dikapitalisasi: `"Capitalized the start of the sentence and pronoun 'I'."`
     - Jika hanya kata ganti `"i"` yang dikapitalisasi: `"Capitalized the pronoun 'I'."`
     - Jika huruf awal kalimat dikapitalisasi: `"Capitalized the first word of the sentence."`
     - Jika hanya tanda baca akhir yang ditambahkan: `"Added ending punctuation."`
   - Perluas `isNoCorrectionExplanation` untuk mencakup variasi seperti `"no errors in this sentence"`, `"no error in this sentence"`, dan `"this sentence is correct"`.
3. **Filter Deterministik Kosakata Duplikat di Go (`diary.go`):**
   - Bangun himpunan kata (*word set*) dari seluruh token kata di dalam `originalSentences` (case-insensitive).
   - Saring daftar `resp.Vocabularies` agar setiap kata rekomendasi yang sudah ada di dalam himpunan kata input pengguna dibuang.
   - Batasi hasil akhir maksimal 3 kosakata unik yang belum pernah ditulis pengguna, dengan daftar kosakata kontekstual cadangan (*curated B2/C1 reflection vocabulary*) apabila seluruh kandidat dari model 1.5B merupakan duplikat dari kata input pengguna.

## 3. Langkah Eksekusi Terinci
1. **Langkah 1 — Pembaruan Prompt (`daily_diary.tmpl`):**
   - Tulis ulang aturan 1–5 di `daily_diary.tmpl` secara ringkas, tegas, dan bebas dari contoh kata spesifik yang memicu *copy-paste*.
2. **Langkah 2 — Normalisasi Deterministik (`diary.go`):**
   - Implementasikan deteksi perubahan murni kapitalisasi/tanda baca pada `normalizeExplanation`.
   - Implementasikan `filterNewVocabularies` untuk membuang kosakata yang duplikat dengan kata di `originalSentences` dan menjaga tepat 3 rekomendasi kosakata baru.
3. **Langkah 3 — Pengujian Unit (`diary_test.go` & `embed_test.go`):**
   - Tambahkan skenario pengujian dengan tepat 5 kalimat nyata dari kasus pengguna (`"i woke up early this morning"`, `"i went to the academy in order to learn coding"`, `"my friends and i worked on our final challenge project"`, `"i enjoyed this day a lot"`, `"trust our process"`) beserta respons JSON quirky dari `qwen2.5:1.5b` untuk memastikan seluruh halusinasi penjelasan dan kosakata duplikat (`"challenge"`, `"process"`) tereliminasi.
   - Jalankan `go test -race ./...` di `radovid-bot-api`.

## 4. Hasil Eksekusi & Status Selesai
Seluruh 3 langkah telah tereksekusi dengan sukses:
- Prompt `daily_diary.tmpl` telah dibersihkan dari contoh spesifik yang memicu bias copy-paste halusinasi pada model LLM kecil.
- `normalizeExplanation` pada `diary.go` mendeteksi perbedaan murni kapitalisasi/tanda baca secara deterministik dan mengeliminasi 100% halusinasi LLM pada kalimat yang susunan katanya sudah benar.
- `filterNewVocabularies` pada `diary.go` berhasil menyaring kata-kata yang sudah ditulis pengguna dan melengkapi slot dengan kosakata B2/C1 kontekstual terkurasi.
- Seluruh unit test suite lulus 100% (`go test -race ./...` dan `go vet ./...`).
- Aturan Zero-Comment dipatuhi 100% pada semua berkas Go yang dimodifikasi.

