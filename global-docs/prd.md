# PRD — Radovid Bot (Discord)

**Versi:** 0.1 (Draft)
**Terakhir diperbarui:** 2026-11-03
**Status:** In Review

---

## Daftar Isi

1. [Tujuan & Latar Belakang](#1-tujuan--latar-belakang)
2. [Alur Pengguna (User Flow)](#2-alur-pengguna-user-flow)
3. [Diagram Interaksi Sistem Global](#3-diagram-interaksi-sistem-global)
4. [Kebutuhan Fungsional](#4-kebutuhan-fungsional)
5. [Kebutuhan Non-Fungsional](#5-kebutuhan-non-fungsional)
6. [Kriteria Penerimaan](#6-kriteria-penerimaan)
7. [Asumsi & Keterbatasan](#7-asumsi--keterbatasan)
8. [Di Luar Cakupan (Out of Scope)](#8-di-luar-cakupan-out-of-scope)
9. [Peta Jalan Fase](#9-peta-jalan-fase)

---

## 1. Tujuan & Latar Belakang

### Latar Belakang

Banyak orang ingin meningkatkan kemampuan bahasa Inggris mereka tetapi gagal karena satu alasan sederhana: tidak ada kesempatan untuk praktik yang terasa alami dan tidak menegangkan. Kelas formal terasa kaku, aplikasi solo terasa sepi, dan percakapan spontan dengan native speaker terasa terlalu intimidatif bagi pemula.

Radovid Bot hadir untuk dua orang yang ingin belajar bersama — menggunakan Discord, platform yang sudah mereka gunakan sehari-hari. Alih-alih menambah satu aplikasi baru ke rutinitas, Radovid Bot masuk ke dalam ruang yang sudah familiar dan mengubah sesi chatting biasa menjadi latihan bahasa yang terstruktur.

### Tujuan Produk

Radovid Bot adalah Discord bot yang berperan sebagai AI English learning companion. Tujuan spesifiknya:

- **Konsistensi:** Membuat dua pengguna memiliki kebiasaan belajar bahasa Inggris harian yang terjaga, bukan sporadis.
- **Praktik aktif:** Mendorong pengguna untuk langsung menulis dan berbicara dalam bahasa Inggris, bukan hanya mempelajari teori.
- **Feedback langsung:** Memberikan koreksi dan saran yang konstruktif segera setelah pengguna menulis, tanpa menghakimi.
- **Engagement berkelanjutan:** Menjaga motivasi melalui elemen permainan (XP, streak, achievement) yang terasa relevan, bukan sekadar dekorasi.

### Target Pengguna

- **Jumlah:** Dua orang per sesi (pasangan belajar / *study partner*).
- **Level:** Pemula hingga lower-intermediate (A1–B1 CEFR).
- **Konteks:** Pengguna aktif Discord yang ingin menjadikan belajar bahasa Inggris sebagai bagian dari rutinitas harian mereka.
- **Motivasi utama:** Belajar bersama teman, bukan sendirian. Lebih suka belajar dengan bermain daripada metode konvensional.

### Filosofi Desain

- **Speaking first:** Prioritas utama adalah membuat pengguna terbiasa mengekspresikan diri dalam bahasa Inggris, meskipun belum sempurna.
- **Learn by doing:** Setiap aktivitas adalah latihan langsung, bukan pelajaran pasif.
- **Beginner friendly:** Kesalahan adalah bagian dari proses. Feedback tidak pernah mengkritik, selalu membangun.
- **Low friction:** Memulai sesi harus semudah mengetik satu perintah.

---

## 2. Alur Pengguna (User Flow)

> Lihat diagram lengkap: [`diagrams/user-journey.puml`](diagrams/user-journey.puml)

### Alur Utama: Memulai Sesi Belajar

```
Pengguna mengetik /radovid start
  → Bot menampilkan daftar aktivitas (15 pilihan + kategori)
  → Pengguna memilih aktivitas via button atau command
  → Bot mengkonfirmasi: "Waiting for partner..." atau langsung mulai jika sudah ada dua pengguna
  → Bot mengirimkan instruksi aktivitas dalam embed
  → Sesi berjalan (game loop spesifik per aktivitas)
  → Sesi berakhir → Bot menampilkan summary (skor, XP didapat, feedback)
  → Bot menawarkan: mainkan lagi / pilih aktivitas lain / selesai
```

### Alur: Koreksi Bahasa Real-time

```
Pengguna menulis respons dalam bahasa Inggris
  → Bot menganalisis teks (via Ollama Gemma 3 4b)
  → Jika ada kesalahan: bot mengirim correction embed secara inline
    (format: apa yang salah → kenapa salah → versi yang benar)
  → Jika tidak ada kesalahan: bot memberi konfirmasi singkat / poin feedback
  → Permainan berlanjut
```

### Alur: Cek Progress Harian

```
Pengguna mengetik /radovid progress
  → Bot menampilkan embed: streak hari ini, XP total, aktivitas terakhir, achievement terbaru
  → Pengguna bisa klik tombol "Lihat Detail" untuk breakdown per aktivitas
```

### Alur: Daily Challenge

```
Bot mengirim daily challenge otomatis ke channel yang dikonfigurasi (via cronjob)
  → Pengguna merespons langsung di channel
  → Bot menilai dan memberi XP
  → Streak +1 jika diselesaikan sebelum reset harian (00:00 WIB atau timezone yang dikonfigurasi)
```

---

## 3. Diagram Interaksi Sistem Global

> Lihat diagram lengkap: [`diagrams/system-activity.puml`](diagrams/system-activity.puml)

### Komponen Sistem

```
[Discord Client]
    ↕ Discord Gateway (WebSocket)
[Radovid Bot — Golang / DiscordGo]
    ├── Command Handler (slash commands, message events)
    ├── Game Engine (state machine per sesi)
    ├── AI Interface (HTTP → Ollama)
    ├── Data Layer (SQLite)
    └── Admin Dashboard (HTMX, HTTP server internal)
        
[Ollama — Gemma 3 4b]
    → Dijalankan di server yang sama
    → Dipanggil via HTTP localhost

[SQLite]
    → Satu file database
    → Menyimpan: user data, sesi, progress, XP, achievement
```

### Alur Permintaan Tipikal

1. Discord mengirim event ke bot via WebSocket.
2. Command handler memproses event dan mengidentifikasi perintah/aktivitas.
3. Game engine memuat state sesi yang relevan dari SQLite.
4. Jika diperlukan analisis AI: bot mengirim HTTP request ke Ollama (localhost).
5. Ollama mengembalikan respons teks.
6. Bot memformat respons menjadi Discord embed dan mengirimnya kembali ke channel.
7. State sesi diperbarui di SQLite.

---

## 4. Kebutuhan Fungsional

### 4.1 Sistem Sesi & Game Engine

- Bot harus bisa membuat, mengelola, dan mengakhiri sesi aktivitas per channel.
- Satu channel hanya boleh memiliki satu sesi aktif pada satu waktu.
- State sesi harus persisten: jika bot restart, sesi yang sedang berjalan harus bisa dilanjutkan atau dinyatakan selesai dengan graceful.
- Setiap sesi mencatat: peserta, aktivitas yang dipilih, durasi, skor (jika ada), dan timestamp.

### 4.2 Aktivitas (15 Game)

#### 4.2.1 Job Interview

- Bot berperan sebagai interviewer.
- Bot mengajukan pertanyaan interview dalam bahasa Inggris (disesuaikan dengan difficulty level).
- Pengguna menjawab; bot mengevaluasi jawaban dari sisi bahasa (grammar, vocabulary, clarity) dan memberikan feedback.
- Sesi terdiri dari 5–10 pertanyaan per ronde.
- Di akhir, bot memberikan ringkasan: area kekuatan dan area yang perlu ditingkatkan.

#### 4.2.2 Would You Rather?

- Bot menampilkan dua pilihan situasi dalam bahasa Inggris.
- Kedua pengguna memilih dan menjelaskan pilihan mereka dalam bahasa Inggris (minimal 2 kalimat).
- Bot mengoreksi bahasa dan memicu diskusi lanjutan dengan pertanyaan follow-up.

#### 4.2.3 Roleplay

- Bot menentukan skenario (misalnya: memesan makanan di restoran, complain ke customer service, bertanya arah).
- Bot berperan sebagai karakter dalam skenario.
- Satu atau dua pengguna berinteraksi dengan bot dalam bahasa Inggris.
- Bot menjaga konsistensi karakter dan mengoreksi bahasa di akhir setiap giliran.

#### 4.2.4 Taboo

- Bot memilih kata target secara acak.
- Satu pengguna menjelaskan kata tersebut dalam bahasa Inggris tanpa menggunakan kata-kata terlarang (yang juga ditentukan bot).
- Pengguna lain menebak.
- Bot memvalidasi apakah kata terlarang digunakan dan mencatat waktu.

#### 4.2.5 20 Questions

- Bot (atau salah satu pengguna, bergantian) memikirkan sebuah objek/tokoh/tempat.
- Pengguna lain mengajukan pertanyaan yes/no untuk menebak.
- Bot memvalidasi pertanyaan (apakah benar-benar yes/no question) dan mencatat jumlah pertanyaan yang digunakan.

#### 4.2.6 One Minute Challenge

- Bot memberi topik; pengguna harus menulis sebanyak mungkin kalimat dalam satu menit.
- Timer berjalan di sisi bot (countdown via embed yang diperbarui, atau pesan terpisah).
- Bot menghitung jumlah kalimat valid dan memberikan skor.

#### 4.2.7 Story Relay

- Bot memulai sebuah cerita dengan 1–2 kalimat pertama.
- Pengguna secara bergantian menambahkan 1–2 kalimat untuk melanjutkan cerita.
- Bot memastikan alur cerita konsisten dan mengoreksi bahasa setelah setiap giliran.
- Cerita berakhir setelah N ronde (dapat dikonfigurasi, default: 10 ronde).

#### 4.2.8 Secret Mission: Write to Win

- Setiap pengguna mendapat "secret mission" tersembunyi: kata atau frasa tertentu yang harus dimasukkan secara alami ke dalam percakapan.
- Pengguna berdiskusi topik tertentu dalam bahasa Inggris.
- Di akhir sesi, bot mengungkapkan mission masing-masing dan menilai apakah berhasil dimasukkan dengan natural.

#### 4.2.9 Daily 5-Sentence Diary

- Setiap pengguna menulis 5 kalimat tentang hari mereka dalam bahasa Inggris.
- Bot mengoreksi setiap kalimat satu per satu.
- Bot memberikan summary: pola kesalahan yang sering muncul, kosakata baru yang berhasil digunakan.
- Aktivitas ini berkontribusi ke streak harian.

#### 4.2.10 Five Word Writing Challenge

- Bot memberi 5 kata acak (kata benda, kata kerja, kata sifat, dsb.).
- Pengguna harus menulis sebuah paragraf pendek (minimal 5 kalimat) yang menggunakan kelima kata tersebut secara natural.
- Bot menilai: apakah semua kata digunakan? Apakah paragraf koheren? Koreksi grammar.

#### 4.2.11 Grammar Boss Fight

- Bot berperan sebagai "Grammar Boss" dengan poin kesehatan (HP).
- Bot mengirim kalimat dengan kesalahan grammar.
- Pengguna harus mengidentifikasi dan memperbaiki kesalahan untuk "menyerang" boss.
- Setiap jawaban benar mengurangi HP boss. Jawaban salah tidak menambah HP boss tapi tidak mengurangi juga.
- Boss dikalahkan ketika HP = 0.
- Difficulty: jumlah dan kompleksitas kesalahan per kalimat.

#### 4.2.12 Sentence Transformation Race

- Bot menampilkan kalimat dan instruksi transformasi (misalnya: ubah ke passive voice, ubah ke past perfect, negate kalimat ini).
- Pengguna berlomba mengirim jawaban tercepat yang benar.
- Bot menilai kebenaran jawaban; poin diberikan ke pengguna pertama yang benar.

#### 4.2.13 Movie Review

- Bot meminta pengguna memilih atau menerima film acak.
- Pengguna menulis review dalam bahasa Inggris (minimal 5 kalimat): sinopsis singkat, pendapat, rekomendasi.
- Bot mengoreksi bahasa dan memberi feedback tentang struktur review.

#### 4.2.14 News Reporter

- Bot memberikan headline berita (fiktif atau nyata yang aman).
- Pengguna harus "melaporkan" berita tersebut seolah menjadi news reporter: menulis beberapa kalimat dalam format berita.
- Bot mengoreksi bahasa dan memberi feedback tentang kejelasan laporan.

#### 4.2.15 Design Your Dream House

- Bot memandu pengguna mendeskripsikan rumah impian mereka dalam bahasa Inggris: ruang-ruang apa saja, letaknya, tampilannya, dll.
- Bot mengajukan pertanyaan panduan (misalnya: "What does your living room look like?", "How many floors does it have?").
- Bot mengoreksi bahasa dan mendorong pengguna untuk menggunakan vocabulary baru yang relevan (furniture, architecture terms).

### 4.3 Fitur Bersama (Shared Features)

#### 4.3.1 AI Game Master

- Semua aktivitas ditenagai oleh Ollama Gemma 3 4b yang berjalan secara lokal.
- Bot mengirim prompt yang sesuai konteks ke Ollama untuk mendapatkan: pertanyaan baru, evaluasi jawaban, koreksi grammar, feedback naratif.
- Respons dari Ollama diformat dan divalidasi sebelum dikirim ke Discord.
- Bot harus menangani timeout atau kegagalan dari Ollama secara graceful (pesan error yang informatif, tidak crash).

#### 4.3.2 Conversation Prompts

- Bot memiliki kumpulan conversation prompts yang dapat digunakan di luar aktivitas terstruktur.
- Pengguna bisa mengetik `/radovid prompt` untuk mendapat topik diskusi acak.
- Prompts dikategorikan berdasarkan topik: daily life, hobbies, opinions, hypotheticals, culture.
- Difficulty level memengaruhi kompleksitas prompt yang diberikan.

#### 4.3.3 English Correction & Feedback

- Setiap teks bahasa Inggris yang ditulis pengguna dalam konteks sesi aktif dapat dianalisis bot.
- Format koreksi standar:
  - **Original:** teks asli pengguna (yang salah)
  - **Corrected:** versi yang benar
  - **Why:** penjelasan singkat kenapa salah (dalam Bahasa Indonesia, maksimal 2 kalimat)
  - **Tip:** saran tambahan jika relevan
- Koreksi ditampilkan dalam embed terpisah, tidak mengganggu alur permainan.
- Jika tidak ada kesalahan, bot tetap memberi feedback positif singkat.

#### 4.3.4 Difficulty Levels

- Tiga level: **Beginner**, **Intermediate**, **Advanced**.
- Default: Beginner.
- Level memengaruhi: kompleksitas vocabulary dalam prompt, panjang minimum jawaban, dan standar evaluasi grammar.
- Pengguna bisa mengubah level kapan saja via `/radovid difficulty [level]`.
- Level disimpan per pengguna di SQLite.

#### 4.3.5 Learning Progress Tracking

- Setiap sesi, setiap aktivitas, dan setiap koreksi yang diterima pengguna dicatat di SQLite.
- Data yang dilacak:
  - Total sesi yang diselesaikan (per aktivitas dan keseluruhan)
  - Jumlah koreksi yang diterima (per jenis kesalahan: grammar, vocabulary, structure)
  - XP yang dikumpulkan per hari/minggu/total
  - Streak harian (berapa hari berturut-turut pengguna aktif)
  - Achievement yang didapat
- Pengguna bisa melihat progress via `/radovid progress`.
- Admin bisa melihat progress semua pengguna via dashboard HTMX.

#### 4.3.6 Gamification (XP, Streaks, Achievements)

**XP System:**
- Setiap aktivitas yang diselesaikan memberikan XP (jumlah bervariasi per aktivitas dan difficulty).
- Bonus XP untuk: menyelesaikan aktivitas tanpa kesalahan grammar besar, memenangkan kompetisi (Sentence Transformation Race, dll.), mempertahankan streak.
- XP tidak di-reset; terakumulasi selamanya.

**Streak:**
- Streak +1 jika pengguna menyelesaikan minimal satu aktivitas atau daily diary dalam sehari.
- Streak reset ke 0 jika satu hari terlewat.
- Bot mengirim notifikasi pengingat jika pengguna belum aktif dan streak terancam (waktu pengingat dapat dikonfigurasi).

**Achievements:**
- Contoh achievement: "First Session" (selesaikan sesi pertama), "Grammar Defender" (1 sesi tanpa kesalahan grammar besar), "7-Day Streak" (streak 7 hari berturut-turut), "Storyteller" (selesaikan 10 Story Relay), dll.
- Achievement ditampilkan di profil pengguna dan diumumkan di channel saat pertama kali didapat.

---

## 5. Kebutuhan Non-Fungsional

### 5.1 Performa

- Respons bot untuk command non-AI (misalnya `/radovid progress`) harus muncul dalam **< 500ms**.
- Respons yang melibatkan Ollama (AI analysis, feedback) harus muncul dalam **< 10 detik** dalam kondisi normal (server tidak sedang under load berat).
- Bot tidak boleh memblokir Discord Gateway heartbeat karena pemrosesan AI yang lama. Semua panggilan ke Ollama harus dilakukan secara asinkron (goroutine).

### 5.2 Reliabilitas

- Bot harus me-reconnect otomatis ke Discord jika koneksi WebSocket terputus.
- Ollama down tidak boleh menyebabkan bot crash; bot menampilkan pesan error yang jelas dan tetap berjalan untuk fungsi non-AI.
- State sesi yang sedang berjalan harus bertahan melalui restart bot (disimpan ke SQLite sebelum shutdown).

### 5.3 Skalabilitas

- **Bukan prioritas untuk v1.0.** Desain awal untuk satu server Discord dengan dua pengguna aktif.
- SQLite sudah cukup untuk kapasitas ini. Migrasi ke database lain bukan bagian dari roadmap jangka pendek.

### 5.4 Keamanan

- Token bot Discord disimpan di environment variable, tidak di-hardcode.
- Dashboard HTMX admin hanya dapat diakses dari localhost atau jaringan internal (tidak terekspos ke publik).
- Input dari pengguna Discord yang dikirim ke Ollama harus di-sanitasi untuk menghindari prompt injection yang berbahaya.
- SQLite database file harus memiliki permission yang tepat (tidak world-readable).

### 5.5 Maintainability

- Codebase mengikuti konvensi Go standar (gofmt, golangci-lint).
- Setiap komponen utama (command handler, game engine, AI interface, data layer) dipisahkan ke dalam package yang berbeda.
- Semua konfigurasi (token, channel ID, Ollama URL, timezone, dll.) dibaca dari file konfigurasi atau environment variable — tidak ada magic number/string di dalam kode.

### 5.6 Observability

- Logging terstruktur (JSON atau key=value) untuk semua event penting: command diterima, sesi dimulai/diakhiri, error dari Ollama, dll.
- Log level dapat dikonfigurasi (DEBUG, INFO, WARN, ERROR).
- Dashboard HTMX menampilkan status bot (uptime, jumlah sesi aktif, status koneksi Ollama) secara real-time.

---

## 6. Kriteria Penerimaan

### 6.1 Aktivitas

- [ ] Semua 15 aktivitas dapat dimulai via command Discord tanpa error.
- [ ] Setiap aktivitas memiliki kondisi selesai yang jelas (berhasil menyelesaikan semua ronde, atau pengguna mengetik `/radovid stop`).
- [ ] Setiap aktivitas memberikan summary di akhir sesi (skor, XP, feedback ringkas).

### 6.2 AI & Feedback

- [ ] Bot memberikan koreksi grammar untuk setiap respons pengguna dalam bahasa Inggris yang mengandung kesalahan.
- [ ] Format koreksi konsisten (Original / Corrected / Why / Tip) di semua aktivitas.
- [ ] Bot menangani Ollama timeout (> 15 detik) dengan pesan error yang jelas tanpa crash.

### 6.3 Gamification

- [ ] XP bertambah setelah setiap sesi selesai, tercatat di SQLite, dan ditampilkan di summary embed.
- [ ] Streak bertambah jika pengguna aktif hari itu dan ter-reset jika satu hari dilewati.
- [ ] Achievement muncul di channel Discord saat pertama kali didapatkan pengguna.

### 6.4 Dashboard Admin

- [ ] Dashboard HTMX dapat diakses di localhost dan menampilkan: daftar pengguna, XP masing-masing, streak, dan total sesi.
- [ ] Dashboard menampilkan status koneksi ke Ollama (UP/DOWN).
- [ ] Dashboard dapat dikunci dengan password sederhana (basic auth atau env-based token check).

### 6.5 Kualitas Kode

- [ ] `golangci-lint` berjalan tanpa error atau warning pada konfigurasi default.
- [ ] Unit test mencakup: game engine state transitions, XP calculation, streak logic, dan Ollama HTTP client (dengan mock).
- [ ] Coverage minimal 70% untuk package inti (game engine, data layer, AI interface).

---

## 7. Asumsi & Keterbatasan

### Asumsi

- **Dua pengguna:** Desain sesi diasumsikan untuk tepat dua pengguna. Fitur multi-user (lebih dari dua) tidak dirancang dan tidak diuji.
- **Satu server Discord:** Bot dirancang untuk digunakan di satu server (guild) Discord. Fitur multi-guild (mengelola lebih dari satu server) bukan bagian dari desain awal.
- **Koneksi internet stabil:** Bot dan Ollama berada di server yang sama; diasumsikan koneksi ke Discord Gateway stabil. Bot reconnect otomatis menangani gangguan sementara.
- **Ollama tersedia:** Ollama dengan model Gemma 3 4b diasumsikan sudah ter-install dan berjalan di server yang sama sebelum bot dijalankan. Setup Ollama bukan bagian dari tanggung jawab bot.
- **Pengguna memahami perintah dasar Discord:** Pengguna diasumsikan dapat menggunakan slash commands di Discord.
- **Bahasa target adalah Bahasa Inggris:** Semua evaluasi, koreksi, dan prompt berbahasa Inggris. Bot tidak mendukung bahasa target lain.

### Keterbatasan

- **Kualitas AI bergantung pada Gemma 3 4b:** Akurasi koreksi grammar dan kualitas feedback dibatasi oleh kemampuan model. Model lokal ini tidak sekuat GPT-4 atau Gemini Pro untuk tugas linguistik kompleks.
- **Tidak ada audio processing:** Bot tidak men-generate atau menganalisis audio. Aktivitas "speaking" difasilitasi via teks di Discord; pengguna yang ingin latihan speaking sungguhan harus melakukannya sendiri di voice channel.
- **SQLite untuk single-node:** Database SQLite tidak cocok untuk deployment multi-instance. Ini sesuai dengan scope saat ini (satu server) tetapi menjadi bottleneck jika scope berubah.
- **Tidak ada antrian request ke Ollama:** Jika banyak request AI dikirim bersamaan, mereka akan diproses secara berurutan oleh Ollama. Untuk dua pengguna ini tidak masalah, tapi jadi catatan teknis.
- **Streak timezone:** Streak reset menggunakan timezone yang dikonfigurasi di server. Jika dua pengguna berada di timezone berbeda, ada ambiguitas "hari ini". Diasumsikan keduanya menggunakan timezone yang sama (WIB by default).

---

## 8. Di Luar Cakupan (Out of Scope)

Hal-hal berikut **tidak** menjadi bagian dari proyek Radovid Bot, setidaknya sampai v2.0 kecuali disebutkan eksplisit:

- **Voice synthesis / Text-to-Speech:** Bot tidak akan menghasilkan audio dalam bentuk apapun.
- **Speech recognition / Voice-to-Text:** Bot tidak menganalisis input suara dari pengguna.
- **Mobile atau web app terpisah:** Antarmuka pengguna hanya Discord. Dashboard HTMX adalah tool admin internal, bukan aplikasi untuk end-user.
- **Sistem pembayaran atau subscription:** Radovid Bot adalah tool personal, tidak ada monetisasi dalam roadmap saat ini.
- **Multi-guild deployment:** Bot tidak dirancang untuk digunakan di banyak server Discord sekaligus.
- **Lebih dari dua pengguna per sesi:** Tidak ada mode grup atau kelas.
- **Kurikulum terstruktur:** Tidak ada silabus, modul, atau jalur belajar yang harus diikuti secara berurutan. Pengguna bebas memilih aktivitas.
- **Integrasi dengan platform eksternal:** Tidak ada integrasi dengan Duolingo, Anki, atau platform belajar lain.
- **Dashboard yang dapat diakses pengguna Discord:** Dashboard HTMX hanya untuk admin (operator server), bukan untuk pengguna bot.
- **Notifikasi push / email:** Reminder hanya dikirim via pesan Discord di channel yang dikonfigurasi.
- **Analitik yang kompleks:** Tidak ada visualisasi grafik, tren harian, atau analisis agregat lintas waktu di v1.0. Data mentah tersedia di SQLite; dashboard menampilkan snapshot sederhana.

---

## 9. Peta Jalan Fase

### MVP v1.0 — Fungsional Dasar

**Target:** Bot bisa dijalankan, dua pengguna bisa bermain, AI feedback berfungsi.

**Cakupan:**
- Setup infrastruktur: bot Discord dengan DiscordGo, koneksi ke Ollama, SQLite.
- 5 aktivitas awal: **Daily 5-Sentence Diary**, **Job Interview**, **Story Relay**, **Grammar Boss Fight**, **Sentence Transformation Race**.
- English Correction & Feedback (format standar) untuk semua aktivitas.
- Difficulty Levels (Beginner / Intermediate / Advanced) — implementasi dasar.
- XP system dan Streak (tanpa achievement).
- Command dasar: `/radovid start`, `/radovid stop`, `/radovid progress`.
- Dashboard HTMX: halaman status (uptime, Ollama status, daftar pengguna dan XP) — read-only.
- Logging terstruktur ke stdout.
- Unit test untuk game engine dan XP/streak logic.

**Tidak termasuk v1.0:** Achievement system, Conversation Prompts, 10 aktivitas lainnya, daily challenge otomatis.

---

### v1.1 — Aktivitas Lengkap & Gamification

**Target:** Semua 15 aktivitas tersedia; achievement dan daily challenge aktif.

**Cakupan:**
- 10 aktivitas tambahan: **Would You Rather?**, **Roleplay**, **Taboo**, **20 Questions**, **One Minute Challenge**, **Secret Mission: Write to Win**, **Five Word Writing Challenge**, **Movie Review**, **News Reporter**, **Design Your Dream House**.
- Achievement system lengkap (minimal 15 achievement).
- Conversation Prompts (`/radovid prompt`).
- Daily Challenge otomatis via cronjob internal.
- Notifikasi streak reminder (dapat dikonfigurasi jam pengingatnya).
- Dashboard HTMX: tambah halaman activity history dan detail per pengguna.
- Perbaikan kualitas: review prompt engineering untuk Ollama, perbaikan edge cases dari v1.0.

---

### v2.0 — Polish & Extensibility

**Target:** Pengalaman yang lebih halus; fondasi untuk kemungkinan ekspansi.

**Cakupan:**
- Leaderboard antara dua pengguna (siapa yang punya XP lebih banyak minggu ini).
- Export progress ke format yang dapat dibaca (JSON atau teks ringkasan) via command.
- Konfigurasi aktivitas yang lebih fleksibel (admin bisa mengatur jumlah ronde, durasi, dll. via dashboard).
- Evaluasi untuk migrasi dari SQLite jika diperlukan (sebagai planning item, bukan implementasi langsung).
- Peninjauan ulang kualitas AI feedback berdasarkan penggunaan aktual v1.0–v1.1.
- Dokumentasi operasional: cara deploy, cara konfigurasi, cara update model Ollama.

---

*Dokumen ini adalah living document. Perubahan signifikan harus didiskusikan dan dicatat di bagian changelog atau commit message yang relevan.*
