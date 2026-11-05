---
id: PLAN-1762261030
ticket_id: TICKET-1762261030
title: Docker Containerization and Ollama Service Orchestration
status: Completed
created_at: 2026-11-04T18:15:00+07:00
---

# Rencana Implementasi Teknis: Docker Containerization and Ollama Service Orchestration

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261030](../tickets/TICKET-1762261030-docker-containerization-ollama.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Deployment Guidelines](../../../global-guidelines/deployment.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menyediakan kontainerisasi lengkap untuk Radovid Bot dan layanan AI lokal Ollama menggunakan Docker dan Docker Compose. Mengeliminasi prasyarat instalasi manual di host OS melalui multi-stage Docker build untuk aplikasi Go (menghasilkan binary berukuran ringkas berbasis Alpine) dan service composition yang secara otomatis mengunduh model offline Gemma 4b (`gemma3:4b`) saat inisialisasi awal.
- **Topologi Kontainer & Komunikasi Jaringan:**
  - Jaringan: `radovid-network` (bridge).
  - Service `ollama`: image `ollama/ollama:latest`, volume `ollama_storage:/root/.ollama`, port `11434:11434`.
  - Service `ollama-pull`: image `curlimages/curl:latest` atau container helper yang mengeksekusi `ollama pull gemma3:4b` ke daemon Ollama hingga model siap.
  - Service `radovid-bot`: build lokal dari `./Dockerfile`, environment variable `OLLAMA_BASE_URL=http://ollama:11434`, `DATABASE_PATH=/app/data/radovid.db`, volume `./data:/app/data`.
- **Potensi Efek Samping & Mitigasi:**
  - Unduhan model berukuran besar (~2.5GB): Diisolasi ke dalam volume persisten `ollama_storage` sehingga unduhan hanya dilakukan satu kali pada boot perdana.
  - SQLite Lock pada Container Mount: Volume `./data:/app/data` menggunakan driver direktori lokal standar yang mendukung PRAGMA WAL SQLite tanpa masalah file locking.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/.dockerignore` | Mengabaikan data lokal, berkas git, log, dan binary kompilasi |
| `CREATE` | `../../../../radovid-bot-api/Dockerfile` | Multi-stage Dockerfile untuk build biner Go dan packaging ke Alpine |
| `CREATE` | `../../../../radovid-bot-api/docker-compose.yml` | Orkesrasi layanan bot, daemon Ollama, auto-pull model, dan volume |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Penyusunan .dockerignore):**
   - Buat `.dockerignore` untuk memastikan context build Docker bersih dari direktori `.git`, file database lokal `*.db`, file `data/`, dan binary build lokal.
2. **Langkah 2 (Penyusunan Multi-Stage Dockerfile):**
   - Stage 1 (`builder`): Menggunakan `golang:1.26-alpine` (atau `golang:alpine`), install dependensi, compile dengan `CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /app/radovid-bot ./cmd/bot`.
   - Stage 2 (`runner`): Menggunakan `alpine:latest`, pasang `ca-certificates` dan `tzdata`, buat non-root user demi keamanan, copy binary `/app/radovid-bot` dan folder `migrations/`.
3. **Langkah 3 (Penyusunan docker-compose.yml):**
   - Definisikan service `ollama` dengan restart policy `unless-stopped`.
   - Definisikan service `ollama-pull` yang bergantung pada `ollama` dan menjalankan perintah penarikan model `gemma3:4b`.
   - Definisikan service `radovid-bot` dengan `env_file: .env`, `depends_on: [ollama, ollama-pull]`, dan mount volume data SQLite.
4. **Langkah 4 (Validasi Konfigurasi & Syntax):**
   - Jalankan `docker compose config` untuk memverifikasi sintaks file compose valid.
   - Uji build image Docker `docker build -t radovid-bot:local .` untuk memastikan proses kompilasi multi-stage sukses tanpa hambatan.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```dockerfile
# radovid-bot-api/Dockerfile
FROM golang:1.24-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /bin/radovid-bot ./cmd/bot

FROM alpine:3.20
RUN apk add --no-cache ca-certificates tzdata
WORKDIR /app
COPY --from=builder /bin/radovid-bot /app/radovid-bot
COPY --from=builder /src/migrations /app/migrations
VOLUME ["/app/data"]
CMD ["/app/radovid-bot"]
```

```yaml
# radovid-bot-api/docker-compose.yml
version: "3.8"

services:
  ollama:
    image: ollama/ollama:latest
    container_name: radovid-ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama_storage:/root/.ollama
    networks:
      - radovid-net

  ollama-pull:
    image: ollama/ollama:latest
    container_name: radovid-ollama-pull
    restart: "no"
    networks:
      - radovid-net
    depends_on:
      - ollama
    entrypoint: /bin/sh
    command:
      - "-c"
      - "sleep 3 && OLLAMA_HOST=ollama:11434 ollama pull gemma3:4b"

  radovid-bot:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: radovid-bot
    restart: unless-stopped
    depends_on:
      - ollama
      - ollama-pull
    env_file:
      - .env
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - DATABASE_PATH=/app/data/radovid.db
    volumes:
      - ./data:/app/data
    networks:
      - radovid-net

volumes:
  ollama_storage:
    name: radovid_ollama_storage

networks:
  radovid-net:
    name: radovid_network
    driver: bridge
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Model Download Concurrency:** Service `ollama-pull` menggunakan perintah idempotent (`ollama pull gemma3:4b`). Jika model sudah ada di volume `ollama_storage`, Ollama akan memverifikasi hash manifest dan selesai dalam 1-2 detik tanpa mengunduh ulang.
- **Database Persistence Across Container Lifecycle:** Direktori basis data SQLite `./data` di-mount ke `/app/data` di dalam container. Penghapusan atau rebuild container tidak akan menghilangkan data user atau riwayat migrasi.
- **Resource Constraints (RAM/GPU):** Pada macOS, Docker Desktop membatasi alokasi RAM VM. Model Gemma 4b membutuhkan minimal ~3.5GB RAM kosong, sehingga instruksi alokasi memori Docker Desktop akan disertakan dalam dokumentasi.

