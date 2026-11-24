---
id: PLAN-1795452322
ticket_id: TICKET-1795452322
title: "[BUG] Game Generation Fails with Total Blackout When Ollama Unreachable"
status: Completed
created_at: 2026-11-23
---

# Rencana Perbaikan Bug: Game Generation Total Blackout (Ollama Unreachable)

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1795452322](../tickets/TICKET-1795452322-bug-game-generation-fails-when-ollama-unreachable.md)
- **Tiket Asal Implementasi:** [TICKET-1795430610](../tickets/TICKET-1795430610-gemini-primary-llm-with-ollama-fallback.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Error Handling](../../../global-guidelines/error-handling.md)

---

## 2. Analisis Akar Masalah (Root Cause Analysis)

### Jejak Error Produksi
```
Failed to start game round: failed to generate game content:
AI game generation service is currently unavailable:
Post "http://192.168.1.5:11434/api/generate": dial tcp 192.168.1.5:11434:
connect: no route to host
```

### Alur Keputusan di `cmd/bot/main.go` (Saat Bug Terjadi)
```
GEMINI_API_KEY = "" (kosong)
        ↓
llmClient = ollamaClient   ← sole provider, single point of failure
        ↓
Ollama di 192.168.1.5:11434 unreachable
        ↓
Semua game generation → ERROR → "service is currently unavailable"
```

### Alur yang Seharusnya (Setelah Perbaikan)
```
GEMINI_API_KEY = "AIzaSy..." (terisi)
        ↓
llmClient = FallbackClient(primary=geminiClient, fallback=ollamaClient)
        ↓
Gemini Flash → SUCCESS (latency < 1 detik)
        ↓ (jika Gemini gagal)
Ollama (opsional) → dipanggil sebagai fallback
```

### Komponen yang Terdampak
| Komponen | Lokasi | Status |
| :--- | :--- | :--- |
| `cmd/bot/main.go` | Line 86-102 | ✅ Logika sudah benar — butuh konfigurasi env |
| `llm/fallback_client.go` | Line 32-50 | ✅ Implementasi sudah benar |
| `.env` | Root `radovid-bot-api/` | ❌ `GEMINI_API_KEY` belum diisi |
| `fallback_client_test.go` | `internal/infrastructure/llm/` | ⚠️ Belum ada test case untuk network-level error (dial tcp) |

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/.env` | Tambah `GEMINI_API_KEY` dan `GEMINI_MODEL=gemini-2.5-flash` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/llm/fallback_client_test.go` | Tambah test case: primary gagal dengan network error → fallback berhasil |

---

## 4. Langkah Eksekusi Perbaikan (Step-by-Step)

### Fase 1: Aktifkan Gemini di `.env`
1. Buka `radovid-bot-api/.env`.
2. Tambahkan dua baris:
   ```env
   GEMINI_API_KEY=<API_KEY_DARI_AI_STUDIO>
   GEMINI_MODEL=gemini-2.5-flash
   ```
3. Dapatkan API key gratis dari [Google AI Studio](https://aistudio.google.com/apikey).

### Fase 2: Tambah Unit Test Regresi (O-07)
Tambahkan test case di `fallback_client_test.go` yang mensimulasikan primary gagal dengan network-level error (misalnya `dial tcp ... connect: no route to host`), memastikan `FallbackClient` tetap memanggil fallback dan berhasil:

```go
func TestFallbackClient_Generate_PrimaryNetworkError_FallbackSucceeds(t *testing.T) {
    networkErr := errors.New("Post \"http://192.168.1.5:11434/api/generate\": dial tcp 192.168.1.5:11434: connect: no route to host")
    primary := &mockLLMClient{err: networkErr}
    fallback := &mockLLMClient{resp: "fallback response"}
    client, _ := NewFallbackClient(primary, fallback)
    resp, err := client.Generate(context.Background(), "prompt", "system")
    if err != nil {
        t.Fatalf("expected fallback to succeed, got: %v", err)
    }
    if resp != "fallback response" {
        t.Fatalf("expected 'fallback response', got: %q", resp)
    }
}
```

### Fase 3: Verifikasi
1. Jalankan: `cd radovid-bot-api && go test -race ./internal/infrastructure/llm/...`
2. Pastikan 100% PASS.
3. Jalankan full suite: `go test -race ./... && go vet ./... && go build ./cmd/bot && rm -f bot`

---

## 5. Catatan Arsitektur
- Konfigurasi `.env` adalah satu-satunya perbaikan operasional yang diperlukan untuk menyelesaikan bug ini di production — kode infrastruktur sudah benar.
- Test case O-07 ditambahkan untuk memastikan `FallbackClient` mencakup network-level error (bukan hanya application-level error) secara eksplisit dalam test coverage.
- `GEMINI_API_KEY` **TIDAK BOLEH** di-commit ke repositori `radovid-bot-api`; file `.env` sudah tercantum di `.gitignore`.
