---
id: PLAN-1762174715
ticket_id: TICKET-1762174715
title: SQLite Database Initialization & Migration System
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: SQLite Database Initialization & Migration System

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762174715](../tickets/TICKET-1762174715-sqlite-initialization-migration.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md) | [Database Guidelines](../../../global-guidelines/database.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menyediakan koneksi SQLite berkinerja tinggi menggunakan driver murni Go `modernc.org/sqlite`, mengonfigurasi PRAGMA optimal (WAL, busy timeout, foreign keys, synchronous normal), serta membangun sistem migrasi DDL otomatis berbasis file `.sql` numerik yang terisolasi dalam transaksi database per file untuk menjamin atomisitas skema.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: `cmd/bot/main.go` saat startup urutan ke-2 (setelah parsing konfigurasi).
  - Komponen Terdampak: Seluruh repository data (`internal/infrastructure/database/`), dimulai dari entitas `user` dan selanjutnya `game_sessions` pada fase berikutnya.
  - Potensi Efek Samping: Masalah database locking (`database is locked`) jika parameter concurrency atau file lock tidak diisolasi; kegagalan migrasi di tengah proses dapat meninggalkan state inkonsisten bila tidak dibungkus transaksi database.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/sqlite.go` | Inisialisasi koneksi `*sql.DB`, pengaturan PRAGMA, dan manajemen pool koneksi |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/migration.go` | Engine pembaca dan eksekutor migrasi SQL berurutan dengan tabel `schema_migrations` |
| `CREATE` | `../../../../radovid-bot-api/migrations/001_initial_schema.sql` | Berkas DDL awal pembuatan tabel `users` beserta indeks unik |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/user/repository.go` | Interface kontrak repository `UserRepository` pada domain layer |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/sqlite_test.go` | Pengujian unit inisialisasi SQLite in-memory dan verifikasi migrasi |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `internal/infrastructure/database/sqlite_test.go` yang membuka koneksi in-memory (`:memory:`) dan memanggil `RunMigrations()`.
   - Buktikan tes gagal sebelum implementasi runner dibuat (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Tulis `sqlite.go` untuk membuka koneksi dengan driver name `sqlite`, membuat direktori parent jika belum ada, serta menjalankan query PRAGMA penting:
     ```sql
     PRAGMA journal_mode = WAL;
     PRAGMA busy_timeout = 5000;
     PRAGMA foreign_keys = ON;
     PRAGMA synchronous = NORMAL;
     ```
   - Tulis `migration.go` dengan fungsi `RunMigrations(db *sql.DB, migrationsDir string) error`:
     - Buat tabel `schema_migrations` jika belum ada (`CREATE TABLE IF NOT EXISTS schema_migrations (version TEXT PRIMARY KEY, applied_at BIGINT NOT NULL)`).
     - Baca daftar file `.sql` dari direktori target, urutkan alfabetis/numerik.
     - Eksekusi setiap file yang belum tercatat di dalam transaksi database (`tx.BeginTx`), catat versi ke `schema_migrations`, lalu `tx.Commit()`.
   - Buat berkas `migrations/001_initial_schema.sql` yang mendefinisikan tabel `users`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Definisikan kontrak port `UserRepository` di `internal/domain/user/repository.go`.
   - Pastikan migrasi bersifat idempotent: mengeksekusi `RunMigrations` dua kali berturut-turut tidak menimbulkan error.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan `go test -v ./internal/infrastructure/database/...` dan pastikan coverage memenuhi standar.
   - Verifikasi rollback transaksi otomatis saat terdapat syntax SQL yang sengaja dibuat keliru.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/infrastructure/database/sqlite.go
package database

import (
	"database/sql"
	"fmt"
	"os"
	"path/filepath"

	_ "modernc.org/sqlite"
)

func NewSQLiteDB(dbPath string) (*sql.DB, error) {
	if dir := filepath.Dir(dbPath); dir != "." && dir != "/" {
		if err := os.MkdirAll(dir, 0755); err != nil {
			return nil, fmt.Errorf("failed to create db directory: %w", err)
		}
	}

	db, err := sql.Open("sqlite", dbPath)
	if err != nil {
		return nil, fmt.Errorf("failed to open sqlite database: %w", err)
	}

	db.SetMaxOpenConns(1) // Single writer safe for SQLite

	pragmas := []string{
		"PRAGMA journal_mode = WAL;",
		"PRAGMA busy_timeout = 5000;",
		"PRAGMA foreign_keys = ON;",
		"PRAGMA synchronous = NORMAL;",
	}
	for _, pragma := range pragmas {
		if _, err := db.Exec(pragma); err != nil {
			db.Close()
			return nil, fmt.Errorf("failed to execute pragma (%s): %w", pragma, err)
		}
	}

	return db, nil
}
```

```sql
-- migrations/001_initial_schema.sql
CREATE TABLE IF NOT EXISTS users (
    id          TEXT PRIMARY KEY,
    discord_id  TEXT UNIQUE NOT NULL,
    username    TEXT NOT NULL,
    created_at  BIGINT NOT NULL,
    updated_at  BIGINT NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_users_discord_id ON users(discord_id);
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Jika path database berada di direktori yang belum dibuat (misalnya `./data/radovid.db`), runner otomatis membuat direktori dengan izin `0755`. Jika terdapat query migrasi gagal, seluruh perubahan dalam berkas tersebut di-rollback secara otomatis tanpa menyisakan modifikasi parsial.
- **Strategi Rollback:** File migrasi yang gagal tidak dicatat ke tabel `schema_migrations`, memungkinkan perbaikan berkas migrasi sebelum server dinyalakan kembali.
