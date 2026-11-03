# Panduan Spesifik Node: Radovid Bot

> **[PRINSIP UTAMA]** Pedoman eksklusif yang memuat aturan lokal, spesifikasi tumpukan teknologi, hasil pemindaian basis kode (*codebase scan*), dan konvensi arsitektur khusus node ini.

---

## 1. Parameter Fundamental Node

- **Preferensi Bahasa Kode & String:** English. Semua identifier Go (variable, function, struct, interface) wajib dalam Bahasa Inggris. Pesan log dalam Bahasa Inggris. Komentar kode dalam Bahasa Inggris.
- **Kepatuhan Anti-Slop:** Wajib mematuhi standar Anti-AI-Slop universal (`coding.md`, `ui-and-assets.md`, `testing.md`, `error-handling.md`).
- **Versi SDK & Dependensi Utama:**
  - Go: 1.26.7
  - Module: `github.com/rakafebriansy/radovid-bot`
  - DiscordGo: github.com/bwmarrin/discordgo (versi terkini saat inisialisasi)
  - SQLite driver: modernc.org/sqlite (pure Go, tanpa CGO) atau mattn/go-sqlite3 (CGO) — ditentukan saat implementasi pertama
  - Ollama: HTTP API lokal (tidak ada Go SDK resmi, gunakan net/http bawaan)
  - HTMX: CDN atau embedded dalam template (tidak ada build step frontend)
- **Path Codebase:** `/Users/raka/Developer/repositories/projects/radovid-bot-dir/radovid-bot-api`
- **Mode Operasional:** MODE 1 — Autonomous Planning
- **Testing Framework:** Built-in `go test` (tidak ada testify)
- **Linter:** golangci-lint
- **Graphify:** AKTIF — wajib jalankan `graphify build` di Path Codebase sebelum navigasi kode pertama kali.

---

## 2. Hasil Pemindaian Codebase (Initial Scan)

**Tanggal Pindai:** 2026-11-03
**Status:** Greenfield — hanya terdapat `go.mod`

```
radovid-bot-api/
└── go.mod   # module: github.com/rakafebriansy/radovid-bot, go 1.26.7
```

**Temuan:**
- Tidak ada source file Go yang ada saat ini.
- Tidak ada direktori struktur yang telah dibuat.
- Tidak ada dependensi yang telah diinstal (go.sum belum ada).
- Tidak ada konfigurasi, migration, atau asset yang ada.
- Codebase adalah greenfield murni.

---

## 3. Konvensi Lokal & Aturan Khusus

### 3.1 Konvensi Arsitektur
1. **Clean Architecture Wajib**
   - *Instruksi:* Seluruh kode WAJIB mengikuti lapisan: `delivery` → `usecase` → `domain` → `infrastructure`. Lapisan luar tidak boleh diimpor oleh lapisan dalam. Domain layer tidak boleh mengimpor library eksternal kecuali error types standar.

2. **Penamaan Package**
   - *Instruksi:* Package names wajib lowercase, satu kata, tanpa underscore. Contoh: `game`, `session`, `user`, `progress`, tidak boleh `game_session` atau `gameSession`.

3. **Interface-First Design**
   - *Instruksi:* Setiap dependency antar layer WAJIB melalui interface (khususnya repositories dan external clients). Implementasi konkret berada di `infrastructure/`. Wajib definisikan interface di sisi konsumen (use case layer), bukan di sisi implementor.

4. **Context Propagation**
   - *Instruksi:* Semua fungsi yang melakukan I/O (database query, HTTP call ke Ollama, Discord API) WAJIB menerima `context.Context` sebagai parameter pertama.

5. **Error Wrapping**
   - *Instruksi:* Gunakan `fmt.Errorf("operation: %w", err)` untuk membungkus error. Tidak boleh mengabaikan error (blank identifier `_` hanya boleh untuk nilai non-error).

### 3.2 Konvensi Database
6. **Datetime sebagai Epoch Milliseconds**
   - *Instruksi:* Semua kolom timestamp di SQLite WAJIB disimpan sebagai `INTEGER` (epoch milliseconds). Tidak boleh menggunakan `TEXT` ISO-8601 atau `DATETIME` SQLite.

7. **UUID sebagai Primary Key**
   - *Instruksi:* Semua primary key WAJIB berupa UUID (TEXT). Tidak boleh menggunakan integer auto-increment sebagai PK entitas domain.

8. **Migration File Terpisah**
   - *Instruksi:* Setiap perubahan skema database WAJIB direpresentasikan sebagai file migration numerik di `migrations/` (contoh: `001_initial_schema.sql`, `002_add_achievements.sql`). Tidak boleh menjalankan DDL langsung dari kode Go.

### 3.3 Konvensi Discord
9. **Slash Commands Only**
   - *Instruksi:* Bot hanya menggunakan Discord Application Commands (slash commands) dan Message Components (buttons, select menus). Tidak boleh merespons pesan prefixed (contoh: `!command`).

10. **Embed untuk Semua Output Terstruktur**
    - *Instruksi:* Semua output game yang memiliki lebih dari satu piece of information WAJIB menggunakan Discord Embed, bukan plain text. Pesan singkat konfirmasi boleh plain text.

11. **Ephemeral Response untuk Error**
    - *Instruksi:* Semua pesan error WAJIB dikirim sebagai ephemeral response (hanya terlihat oleh user yang bersangkutan).

### 3.4 Konvensi AI (Ollama)
12. **Timeout Wajib untuk Semua Ollama Calls**
    - *Instruksi:* Setiap HTTP request ke Ollama WAJIB menggunakan context dengan timeout (default 30 detik, dikonfigurasi via env var `OLLAMA_TIMEOUT_SECONDS`). Tidak boleh ada request tanpa timeout.

13. **Input Sanitization Sebelum Ollama**
    - *Instruksi:* Semua teks yang dikirim dari Discord user ke Ollama WAJIB melalui sanitization: hapus mention (@user, @role), strip markdown yang bisa memanipulasi prompt, batasi panjang input maksimum 2000 karakter.

14. **Structured Prompt Format**
    - *Instruksi:* Semua prompt ke Ollama WAJIB menggunakan format sistem yang konsisten. Prompt template disimpan sebagai konstanta atau embedded file di package `ollama`, bukan ditulis inline di use case.

### 3.5 Konvensi Testing
15. **Test Files Co-located**
    - *Instruksi:* File test (`_test.go`) WAJIB berada di direktori yang sama dengan file yang diuji. Tidak ada direktori `tests/` atau `test/` terpisah, kecuali untuk integration tests di `internal/integration/`.

16. **Table-Driven Tests**
    - *Instruksi:* Semua unit test WAJIB menggunakan pola table-driven test (`t.Run` dengan slice of test cases). Tidak boleh menulis multiple `TestX` functions untuk skenario berbeda dari fungsi yang sama.

---

## 4. Batasan Teknis & Keputusan Arsitektur

- **Tidak ada CGO jika memungkinkan:** Prioritaskan `modernc.org/sqlite` (pure Go) untuk portabilitas binary.
- **Single binary deployment:** Semua asset (HTML templates) di-embed menggunakan `//go:embed` agar binary mandiri.
- **No ORM:** Gunakan SQL raw query dengan `database/sql` standard library. Tidak boleh menggunakan GORM atau ORM lainnya.
- **No external job queue:** Background tasks (misal: reminder harian) menggunakan goroutine dengan ticker bawaan Go, bukan library queue eksternal.
- **Ollama lokal:** Tidak ada cloud AI API. Seluruh inferensi melalui Ollama di localhost:11434.
