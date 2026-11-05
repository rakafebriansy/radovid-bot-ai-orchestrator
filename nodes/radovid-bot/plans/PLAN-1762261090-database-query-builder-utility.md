---
id: PLAN-1762261090
ticket_id: TICKET-1762261090
title: Database Query Builder Utility
status: Completed
created_at: 2026-11-05T07:45:00+07:00
---

# Rencana Implementasi Teknis: Database Query Builder Utility

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261090](../tickets/TICKET-1762261090-database-query-builder-utility.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Security Guidelines](../../../global-guidelines/security.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menyediakan pustaka utilitas query builder SQL berparameter (*parameterized query builder*) ringan di `internal/infrastructure/database/query_builder.go`. Utilitas ini mengabstraksi perakitan string SQL untuk operasi batch (Bulk Insert, Batch Update WHERE IN, Bulk Case Update, Batch Delete WHERE IN, dan Batch Select WHERE IN) tanpa mengorbankan performa (zero reflection, pure string builder) dan menjamin proteksi terhadap SQL injection pada nama identifier (tabel/kolom).
- **Hasil Pemetaan Ketergantungan (Graphify / Architecture):**
  - Modul Pemanggil: `SessionRepository`, serta repositori masa depan (`GameRoundRepository`, `UserScoreRepository`, `VocabularyRepository`).
  - Komponen Terdampak: `internal/infrastructure/database`.
  - Potensi Efek Samping: Nol regresi fungsional karena fungsi bersifat murni (*pure functions*) yang hanya memanipulasi string SQL berparameter dan divalidasi dengan unit test table-driven ketat.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/query_builder.go` | Implementasi utilitas builder query batch SQL berparameter |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/query_builder_test.go` | Unit test table-driven komprehensif untuk seluruh builder fungsi |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/database/session_repository.go` | Refactor `CreateSession` untuk mengonsumsi `BuildBulkInsertQuery` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/database/session_repository_test.go` | Memastikan kompatibilitas dan integrasi repo tetap 100% PASS |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)

### Tahap 1: Definisi Error & Validasi Identifier
1. Buat custom errors untuk input query builder:
   - `ErrEmptyTableName = errors.New("table name cannot be empty")`
   - `ErrEmptyColumns = errors.New("columns cannot be empty")`
   - `ErrInvalidRowCount = errors.New("row count must be greater than zero")`
   - `ErrInvalidIdentifier = errors.New("invalid SQL identifier")`
2. Implementasikan fungsi validasi regex/rune `ValidateIdentifier(name string) error` untuk memastikan nama tabel dan kolom hanya mengandung karakter `^[a-zA-Z0-9_]+$`.

### Tahap 2: Implementasi Fungsi Query Builder
1. **Bulk Insert:**
   - `BuildBulkInsertQuery(tableName string, columns []string, rowCount int) (string, error)`
   - Format: `INSERT INTO <table> (<col1>, <col2>) VALUES (?, ?), (?, ?)`
2. **Batch Update (WHERE IN):**
   - `BuildBatchUpdateQuery(tableName string, setColumns []string, inColumn string, inCount int) (string, error)`
   - Format: `UPDATE <table> SET <col1> = ?, <col2> = ? WHERE <inCol> IN (?, ?)`
3. **Bulk Case Update (Update nilai berbeda per baris berdasarkan Primary Key):**
   - `BuildCaseUpdateQuery(tableName string, primaryKey string, updateColumn string, rowCount int) (string, error)`
   - Format: `UPDATE <table> SET <updateCol> = CASE <pk> WHEN ? THEN ? WHEN ? THEN ? END WHERE <pk> IN (?, ?)`
4. **Batch Delete (WHERE IN):**
   - `BuildBatchDeleteQuery(tableName string, inColumn string, inCount int) (string, error)`
   - Format: `DELETE FROM <table> WHERE <inCol> IN (?, ?)`
5. **Batch Select (WHERE IN):**
   - `BuildSelectInQuery(tableName string, selectColumns []string, inColumn string, inCount int) (string, error)`
   - Format: `SELECT <col1>, <col2> FROM <table> WHERE <inCol> IN (?, ?)`

### Tahap 3: Pengujian Unit TDD (Red-Green-Refactor)
1. Tulis `query_builder_test.go` yang mencakup:
   - Validasi nama tabel dan kolom kosong/ilegal (termasuk karakter spasi, petik, titik koma).
   - Pengujian 1 baris, 2 baris, dan banyak baris.
   - Verifikasi ketepatan jumlah tanda tanya `?` sesuai formula `rowCount * len(columns)`.
2. Pastikan test suite lulus 100%.

### Tahap 4: Refactor `session_repository.go`
1. Ubah blok penyimpanan partisipan di `CreateSession` untuk menggunakan `BuildBulkInsertQuery`.
2. Jalankan `go test -v ./internal/infrastructure/database/...` untuk memverifikasi fungsionalitas CRUD tetap terjaga utuh.

---

## 5. Pratinjau Desain API & Signature

```go
package database

import (
	"errors"
	"fmt"
	"strings"
	"unicode"
)

var (
	ErrEmptyTableName     = errors.New("table name cannot be empty")
	ErrEmptyColumns       = errors.New("columns cannot be empty")
	ErrInvalidRowCount    = errors.New("row count must be greater than zero")
	ErrInvalidIdentifier   = errors.New("invalid SQL identifier")
)

func ValidateIdentifier(name string) error
func BuildBulkInsertQuery(tableName string, columns []string, rowCount int) (string, error)
func BuildBatchUpdateQuery(tableName string, setColumns []string, inColumn string, inCount int) (string, error)
func BuildCaseUpdateQuery(tableName string, primaryKey string, updateColumn string, rowCount int) (string, error)
func BuildBatchDeleteQuery(tableName string, inColumn string, inCount int) (string, error)
func BuildSelectInQuery(tableName string, selectColumns []string, inColumn string, inCount int) (string, error)
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **SQL Injection via Identifiers:** Identifier SQL tidak bisa menggunakan parameter berikatan `?`. Oleh karena itu, seluruh string identifier (nama tabel, kolom) wajib melalui `ValidateIdentifier` untuk mencegah injeksi seperti `users; DROP TABLE users;--`.
- **Zero Rows / Empty Parameters:** Fungsi builder langsung mengembalikan error spesifik jika `rowCount <= 0` atau `columns` kosong, mencegah pembentukan query malformed.
- **Efisiensi Alokasi Memori:** Menggunakan perkiraan kapasitas awal pada `strings.Builder.Grow()` untuk meminimalkan alokasi memori berulang saat membentuk query berukuran besar.

