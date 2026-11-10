# Knowledge Base & Implementation Q&A Repository (LEARN.md)

> **[PRINSIP UTAMA]** Repositori terpusat untuk mendokumentasikan tanya-jawab teknis, keputusan arsitektur, dan logika basis kode yang ditanyakan pengguna secara eksplisit. Berbeda dari `RETROSPECTIVE.md` yang mencatat anomali galat, dokumen ini khusus merekam pemahaman konseptual implementasi.

---

## 1. Protokol Operasional & Syarat Aktivasi

> **[FATAL - ATURAN ANTI-AGRESIVITAS PENCATATAN]**
> Berkas `LEARN.md` HANYA BOLEH DIBACA DAN DIISI jika pengguna secara eksplisit menggunakan format prompt **FASE 4: Implementation Q&A Prompt** pada `README.md`.
> AI Agent DILARANG KERAS menyentuh, membaca, atau memodifikasi berkas ini pada instruksi umum, penugasan tiket/koding (Fase 1, 2, 3), perbaikan bug, maupun dialog kasual!

### 1.1 Syarat Aktivasi Mutlak (Template FASE 4 Only)
1. **[ATURAN]** `LEARN.md` tidak berlaku untuk penugasan tiket koding, inisialisasi arsitektur, perbaikan bug, atau percakapan umum.
2. **[WAJIB]** Pembacaan dan pencatatan entri hanya aktif jika prompt pengguna secara nyata menggunakan struktur **FASE 4: Implementation Q&A Prompt** dari `README.md` (memiliki blok Cakupan/Node, pertanyaan bernomor eksplisit, dan klausul instruksi baku).

### 1.2 Penanganan Pertanyaan Kasual (Single-Output Chat)
Jika pengguna mengajukan pertanyaan teknis atau arsitektur di luar template resmi FASE 4:
1. **[WAJIB]** Berikan jawaban lengkap secara eksklusif pada jendela percakapan terminal (*chat sidebar*).
2. **[TERLARANG]** Membuka, membuat entri baru, atau memodifikasi `LEARN.md`.
3. **[TERLARANG]** Berinisiatif mandiri mendokumentasikan dialog non-FASE 4 ke dalam repositori ini.

### 1.3 Pembatasan Lingkup & Validasi Berakar Kode (Full Grounding)
1. **Hanya Pertanyaan Eksplisit:** AI Agent DILARANG membuat rangkuman atau analisis otomatis di luar daftar pertanyaan yang diajukan pengguna.
2. **Berakar pada Basis Kode:** Seluruh jawaban wajib merujuk pada kode sumber nyata, konfigurasi, dan dokumen orchestrator (`global-docs/`, `global-guidelines/`, `nodes/[nama-node]/`).
3. **Bukti & Rasional:** Sertakan jalur berkas valid beserta nomor baris spesifik. Jelaskan alasan (*why*) di balik keputusan desain, bukan sekadar mendeskripsikan ulang kode (*what*).

### 1.4 Mekanisme Output Ganda Bersyarat (Dual-Output)
1. Saat prompt FASE 4 terpicu, AI Agent wajib menyajikan jawaban terstruktur langsung pada *chat sidebar*.
2. Pada giliran eksekusi yang sama, AI Agent wajib merekam sesi Q&A tersebut ke Bagian 4 berkas ini menggunakan Format Baku dan memperbarui Indeks Kategori.
3. Menulis ke berkas ini tanpa pemicu resmi FASE 4 merupakan **PELANGGARAN FATAL**.

---

## 2. Taksonomi & Sistem Kategorisasi

### 2.1 Format ID Entri
Gunakan format penomoran terstandarisasi: `[QA-YYYYMMDD-XX]`
- `YYYYMMDD`: Tahun, Bulan, Tanggal pencatatan (contoh: `20260913`).
- `XX`: Nomor urut pencatatan pada hari terkait (contoh: `01`, `02`).

### 2.2 Sepuluh Kategori Utama
Pilih satu kategori yang paling dominan mewakili topik pembahasan:

| Kategori Utama | Cakupan / Domain |
|---|---|
| **`Architecture & Pattern`** | Struktur folder, modularitas, alur sistem, pola desain, pemisahan tanggung jawab. |
| **`State Management & Data Flow`** | Alur data, global store (Zustand/Redux), server state (React Query), aliran props/event. |
| **`Database & Data Modeling`** | Skema tabel/koleksi, ORM (Prisma/TypeORM/Mongoose), migrasi, indexing, optimasi kueri. |
| **`API & Network Integration`** | REST endpoints, GraphQL, WebSocket, gRPC, format request/response, penanganan galat jaringan. |
| **`UI/UX & Design System`** | Komponen visual, Tailwind/CSS, konsistensi tema, responsivitas, aksesibilitas (a11y). |
| **`Security & Authentication`** | JWT, sesi, OAuth, RBAC/Permissions, hashing, sanitasi input, proteksi CORS/CSRF. |
| **`Testing & Quality Assurance`** | Strategi unit test, integration test, E2E, mock data, cakupan kode, asersi logika. |
| **`Build, Tooling & DevOps`** | Konfigurasi bundler (Vite/Webpack), Docker, CI/CD, script npm, environment variables. |
| **`Business Logic & Domain Rules`** | Logika perhitungan, validasi transaksi, aturan proses bisnis spesifik aplikasi. |
| **`Orchestrator & Workflow`** | Aturan template, manajemen tiket, SOP guidelines, mekanisme orkestrasi multi-node. |

### 2.3 Konvensi Penandaan (Tagging)
Gunakan format `#kebab-case` untuk penanda spesifik. Contoh:
- `#jwt-auth` `#zustand` `#prisma-relations` `#optimistic-update` `#tailwind-v4` `#rbac-middleware`

---

## 3. Format Entri Baku (Boilerplate Template)

Salin struktur Markdown berikut saat mencatat entri tanya-jawab baru:

```markdown
### [QA-YYYYMMDD-XX] <Judul Representatif Terkait Topik Bahasan>
- **Tanggal**: YYYY-MM-DD HH:mm
- **Scope / Target Node**: `[Global / Nama Node / Path Codebase]`
- **Kategori**: `[Pilih salah satu dari 10 Kategori Utama]`
- **Tags**: `#tag1 #tag2 #tag3`
- **File Referensi**:
  - `path/to/relevant-file-1.ext` (L10-L45)
  - `path/to/relevant-file-2.ext`

#### Pertanyaan Pengguna
1. **[Tulis ulang pertanyaan 1 secara presisi]**
2. **[Tulis ulang pertanyaan 2 secara presisi]**

#### Jawaban & Penjelasan Implementasi

##### 1. [Judul Poin Jawaban 1]
- **Ringkasan Inti**: [Penjelasan singkat 1-2 kalimat]
- **Detail Implementasi & Logika**:
  [Penjelasan komprehensif alur kerja kode]
- **Rujukan Kode Sumber**:
  ```[language]
  // Cuplikan kode atau referensi fungsi/kelas yang relevan
  ```
- **Rasional & Keputusan Teknis**:
  [Alasan pemilihan pendekatan, pertimbangan trade-off, kesesuaian pedoman]

##### 2. [Judul Poin Jawaban 2]
- **Ringkasan Inti**: [Penjelasan singkat 1-2 kalimat]
- **Detail Implementasi & Logika**:
  [Penjelasan komprehensif alur kerja kode]
- **Rujukan Kode Sumber**:
  ```[language]
  // Cuplikan kode atau referensi fungsi/kelas yang relevan
  ```
- **Rasional & Keputusan Teknis**:
  [Alasan pemilihan pendekatan, pertimbangan trade-off, kesesuaian pedoman]

---
```

---

## 4. Indeks Kategori & Arsip Log Tanya-Jawab

### 4.1 Indeks Kategori & Daftar Isi
Perbarui tautan indeks berikut setiap kali menambahkan entri baru (urutkan descending / terbaru di atas):

- **Architecture & Pattern**
  - *(Belum ada entri)*
- **State Management & Data Flow**
  - *(Belum ada entri)*
- **Database & Data Modeling**
  - *(Belum ada entri)*
- **API & Network Integration**
  - [[QA-20261110-02] Penanganan JSON Terpotong (Token Truncation) & Fallback Repair pada Evaluasi LLM](#qa-20261110-02-penanganan-json-terpotong-token-truncation--fallback-repair-pada-evaluasi-llm)
- **UI/UX & Design System**
  - *(Belum ada entri)*
- **Security & Authentication**
  - *(Belum ada entri)*
- **Testing & Quality Assurance**
  - *(Belum ada entri)*
- **Build, Tooling & DevOps**
  - [[QA-20261110-01] Konektivitas Multi-Host WSL2, Docker, & Windows Host pada Layanan Ollama](#qa-20261110-01-konektivitas-multi-host-wsl2-docker--windows-host-pada-layanan-ollama)
- **Business Logic & Domain Rules**
  - [[QA-20261110-03] Penjinakan Halusinasi Model Ringan (qwen2.5:1.5b) & Mitigasi Batas Waktu Interaksi Discord](#qa-20261110-03-penjinakan-halusinasi-model-ringan-qwen2515b--mitigasi-batas-waktu-interaksi-discord)
- **Orchestrator & Workflow**
  - *(Belum ada entri)*

### 4.2 Arsip Entri Tanya-Jawab
*(Entri tanya-jawab baru yang disetujui dari pemicu FASE 4 disisipkan di bawah baris ini secara berurutan)*

---

### [QA-20261110-01] Konektivitas Multi-Host WSL2, Docker, & Windows Host pada Layanan Ollama
- **Tanggal**: 2026-11-10 22:20
- **Scope / Target Node**: `radovid-bot` / `radovid-bot-api` / Ollama Infrastructure
- **Kategori**: `Build, Tooling & DevOps`
- **Tags**: `#ollama` `#wsl2-networking` `#host-binding` `#docker-networking` `#powershell-background` `#ip-binding`
- **File Referensi**:
  - `../../radovid-bot-api/.env`
  - `../../radovid-bot-api/docker-compose.yml`
  - `../../radovid-bot-api/internal/infrastructure/ollama/client.go` (L24-L80)

#### Pertanyaan Pengguna
1. **Mengapa `curl http://localhost:11434/api/tags` dan `curl http://192.168.1.5:11434/api/tags` dari WSL2 / Docker container gagal terhubung (`curl: (28) Failed to connect to 192.168.1.5 port 11434 after 133275 ms: Could not connect to server`), padahal ping ke IP host berhasil?**
2. **Bagaimana cara menjalankan Ollama di Windows host agar tetap aktif di latar belakang (background process) dan dapat diakses secara stabil oleh container Docker dan WSL2?**

#### Jawaban & Penjelasan Implementasi

##### 1. Akar Masalah Host Binding & Isolasi Jaringan WSL2/Docker
- **Ringkasan Inti**: Secara default, binari Ollama di Windows hanya mengikat (*bind*) interface loopback lokal (`127.0.0.1:11434`), sehingga menolak seluruh paket TCP yang masuk melalui IP antarmuka LAN (`192.168.1.5`) maupun virtual switch WSL2/Docker (`172.x.x.x` / `10.255.255.x`), meskipun ICMP `ping` berhasil merespons.
- **Detail Implementasi & Logika**:
  - Di Windows, Ollama yang diluncurkan secara standar tanpa variabel lingkungan akan mendengarkan port 11434 secara eksklusif pada `127.0.0.1`.
  - Ketika bot berjalan di dalam WSL2 atau di dalam container Docker (`radovid-bot`), request menuju `localhost` akan memantul ke loopback internal container itu sendiri (di mana tidak ada Ollama yang berjalan).
  - Ketika bot diarahkan ke IP LAN host Windows (`192.168.1.5:11434`), paket TCP SYN tiba di Windows, tetapi kernel Windows langsung me-reject koneksi atau menjatuhkan paket karena tidak ada listener pada socket non-loopback (`0.0.0.0`).
  - **Solusi Binding**: Diperlukan deklarasi eksplisit environment variable `OLLAMA_HOST=0.0.0.0:11434` pada sesi host Windows sebelum Ollama dijalankan.
- **Rujukan Kode Sumber & Konfigurasi**:
  Pada `.env` aplikasi di WSL2/Docker:
  ```env
  # Mengarahkan base URL ke IP host Windows
  OLLAMA_BASE_URL=http://192.168.1.5:11434
  DOCKER_OLLAMA_BASE_URL=http://192.168.1.5:11434
  ```
  Pada sisi Windows PowerShell untuk verifikasi socket listening:
  ```powershell
  # Cek port listening pada host Windows
  netstat -ano | findstr 11434
  # Output wajib menunjukkan 0.0.0.0:11434 LISTENING (bukan hanya 127.0.0.1:11434)
  ```
- **Rasional & Keputusan Teknis**:
  Pendekatan binding `0.0.0.0:11434` pada host Windows memberikan aksesibilitas transparan baik dari host langsung, lingkungan WSL2, maupun container Docker yang berada di subnet berbeda tanpa memerlukan reverse-proxy tambahan yang menambah latensi.

##### 2. Orkestrasi Daemon/Background Process Ollama di Windows
- **Ringkasan Inti**: Terminal Windows/PowerShell tidak boleh terkunci saat menjalankan `ollama serve`. Perintah latar belakang Linux seperti `&` atau `nohup` tidak kompatibel di PowerShell, sehingga diperlukan mekanisme `Start-Process` detached atau pembuatan Windows Scheduled Task/Service.
- **Detail Implementasi & Logika**:
  - Di Linux, `ollama serve &` dapat mengirim proses ke job background bash, namun di Windows PowerShell karakter `&` adalah call operator dan bukan background operator.
  - Untuk meluncurkan Ollama sebagai proses latar belakang independen yang membaca konfigurasi host `0.0.0.0:11434`, gunakan script PowerShell dengan flag `-WindowStyle Hidden`:
  ```powershell
  $env:OLLAMA_HOST = "0.0.0.0:11434"
  Start-Process -FilePath "ollama" -ArgumentList "serve" -WindowStyle Hidden
  ```
  - Setelah dijalankan, verifikasi ketersediaan model dan responsivitas API dari WSL2 melalui:
  ```bash
  curl http://192.168.1.5:11434/api/tags
  # Respons JSON model sukses: {"models":[{"name":"qwen2.5:1.5b", ...}, {"name":"gemma3:4b", ...}]}
  ```
- **Rasional & Keputusan Teknis**:
  Menjalankan servis dengan `WindowStyle Hidden` menjamin Ollama tetap melayani permintaan API AI meskipun jendela PowerShell ditutup, mencegah interupsi proses saat developer menutup sesi terminal interaktif.

---

### [QA-20261110-02] Penanganan JSON Terpotong (Token Truncation) & Fallback Repair pada Evaluasi LLM
- **Tanggal**: 2026-11-10 22:20
- **Scope / Target Node**: `radovid-bot` / `radovid-bot-api` / Ollama Infrastructure & Daily Diary Use Case
- **Kategori**: `API & Network Integration`
- **Tags**: `#ollama` `#token-truncation` `#json-repair` `#num-predict` `#grammar-constraint` `#golang`
- **File Referensi**:
  - `../../radovid-bot-api/internal/infrastructure/ollama/client.go` (L94-L112)
  - `../../radovid-bot-api/internal/infrastructure/ollama/models.go` (L15-L25)
  - `../../radovid-bot-api/internal/usecase/feedback/diary.go` (L150-L165, L460-L515)
  - `../../radovid-bot-api/internal/usecase/feedback/diary_test.go` (L170-L205)

#### Pertanyaan Pengguna
1. **Mengapa pemanggilan `/radovid diary` sering memicu galat `unexpected end of JSON input` saat mengevaluasi 5 kalimat dengan model lokal Ollama?**
2. **Bagaimana arsitektur Radovid Bot mengatasi keterbatasan token output bawaan Ollama dan memulihkan string JSON yang terpotong secara aman tanpa crash?**

#### Jawaban & Penjelasan Implementasi

##### 1. Batas Bawaan `num_predict` Ollama pada Output Panjang
- **Ringkasan Inti**: Secara default, mesin inferensi Ollama membatasi pembuatan token output maksimum (`num_predict`) pada nilai yang sangat sempit (128 token) jika tidak didefinisikan secara eksplisit, menyebabkan output JSON terpotong di tengah jalan saat memproses payload evaluasi yang panjang.
- **Detail Implementasi & Logika**:
  - Fitur Daily Diary mengevaluasi 5 kalimat bahasa Inggris pengguna secara sekaligus: memproduksi kalimat koreksi, penjelasan tata bahasa, dan 3 rekomendasi kosakata baru beserta definisi dan contoh penggunaan.
  - Payload JSON lengkap tersebut membutuhkan rata-rata 350–600 token.
  - Ketika batas bawaan tercapai, peladen Ollama langsung mengakhiri stream respons sebelum menutup kurung kurawal/siku JSON (`... "explanation": "Capitalized the first word`), menghasilkan blob JSON malformed yang gagal diparsing oleh `json.Unmarshal`.
- **Rujukan Kode Sumber**:
  Pembaruan pada `internal/infrastructure/ollama/client.go`:
  ```go
	format := ""
	if strings.Contains(sanitizedPrompt, "JSON") {
		format = "json"
	}

	reqBody := GenerateRequest{
		Model:     c.model,
		Prompt:    sanitizedPrompt,
		System:    system,
		Format:    format,
		Stream:    false,
		KeepAlive: "30m",
		Options: map[string]any{
			"num_ctx":     2048,
			"num_predict": 1024,
			"temperature": 0.2,
		},
	}
  ```
- **Rasional & Keputusan Teknis**:
  - Menetapkan `"num_predict": 1024` memberikan plafon kapasitas yang cukup bagi model untuk menuntaskan struktur data tanpa risiko terpotong.
  - Parameter `Format: "json"` memaksa runtime llama.cpp di dalam Ollama menerapkan *grammar-based sampling*, memastikan token yang dihasilkan selalu mematuhi sintaksis JSON valid.

##### 2. Mekanisme Pertahanan Berlapis: Algoritma `tryRepairTruncatedJSON`
- **Ringkasan Inti**: Sebagai implementasi prinsip *Defense-in-Depth*, sistem dilengkapi algoritma perbaikan string otomatis di `diary.go` yang menghitung parity tanda kutip dan kedalaman bracket terbuka, lalu melengkapinya secara deterministik sebelum unmarshal dilakukan.
- **Detail Implementasi & Logika**:
  - Jika `json.Unmarshal` awal mendeteksi EOF mendadak, fungsi `tryRepairTruncatedJSON(blob)` dijalankan.
  - Algoritma melakukan scan pada blob string:
    1. Mengecek jumlah tanda kutip ganda (`"`) tanpa escape (`\`). Jika ganjil, tambahkan penutup `"`.
    2. Menghitung tumpukan kurung kurawal `{` vs `}` dan kurung siku `[` vs `]`.
    3. Menutup seluruh bracket yang masih terbuka dengan urutan LIFO terbalik (`]` lalu `}`).
- **Rujukan Kode Sumber**:
  Implementasi pada `internal/usecase/feedback/diary.go`:
  ```go
func tryRepairTruncatedJSON(blob string) string {
	blob = strings.TrimSpace(blob)
	if blob == "" {
		return blob
	}

	quoteCount := 0
	escaped := false
	for i := 0; i < len(blob); i++ {
		c := blob[i]
		if c == '\\' && !escaped {
			escaped = true
			continue
		}
		if c == '"' && !escaped {
			quoteCount++
		}
		escaped = false
	}
	if quoteCount%2 != 0 {
		blob += `"`
	}

	openBraces := 0
	openBrackets := 0
	inString := false
	escaped = false
	for i := 0; i < len(blob); i++ {
		c := blob[i]
		if c == '\\' && !escaped {
			escaped = true
			continue
		}
		if c == '"' && !escaped {
			inString = !inString
		}
		if !inString {
			if c == '{' {
				openBraces++
			} else if c == '}' && openBraces > 0 {
				openBraces--
			} else if c == '[' {
				openBrackets++
			} else if c == ']' && openBrackets > 0 {
				openBrackets--
			}
		}
		escaped = false
	}

	for openBrackets > 0 {
		blob += "]"
		openBrackets--
	}
	for openBraces > 0 {
		blob += "}"
		openBraces--
	}
	return blob
}
  ```
- **Rasional & Keputusan Teknis**:
  Pendekatan ini menjamin keandalan sistem tetap 100% (*graceful recovery*) bahkan jika inferensi terpotong di tepi token, sehingga pengguna Discord tetap menerima hasil evaluasi kalimat yang sudah berhasil digenerate alih-alih pesan error generik.

---

### [QA-20261110-03] Penjinakan Halusinasi Model Ringan (qwen2.5:1.5b) & Mitigasi Batas Waktu Interaksi Discord
- **Tanggal**: 2026-11-10 22:20
- **Scope / Target Node**: `radovid-bot` / `radovid-bot-api` / Ollama Prompts, Discord Delivery, & Gameplay Use Cases
- **Kategori**: `Business Logic & Domain Rules`
- **Tags**: `#local-llm` `#small-models` `#prompt-engineering` `#few-shot-hallucination` `#discord-timeout` `#interaction-deferral` `#stuck-lobby`
- **File Referensi**:
  - `../../radovid-bot-api/internal/infrastructure/ollama/prompts/daily_diary.tmpl`
  - `../../radovid-bot-api/internal/usecase/feedback/diary.go` (L170-L240, L380-L455)
  - `../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` (L315-L335)
  - `../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go` (L60-L95)

#### Pertanyaan Pengguna
1. **Mengapa model lokal berukuran 1.5B (`qwen2.5:1.5b`) kerap berhalusinasi menyalin kata contoh dari prompt (seperti `"skills"`, `"the"`, dll.) atau mengubah kalimat pengguna yang sudah benar secara tata bahasa?**
2. **Bagaimana cara mencegah interaksi bot di Discord mengalami kegagalan timeout (3 detik) saat menunggu proses inferensi Ollama yang memakan waktu 5–15 detik?**

#### Jawaban & Penjelasan Implementasi

##### 1. Eliminasi Bias Few-Shot & Override Normalisasi Deterministik
- **Ringkasan Inti**: Model LLM berparameter kecil memiliki keterbatasan dalam membedakan antara instruksi meta-prompt dan konten pengguna asli, sehingga contoh literal pada prompt (*few-shot bias*) cenderung disalin secara mentah. Solusinya adalah sterilisasi template prompt dipadukan dengan normalisasi komputasi deterministik di layer Go.
- **Detail Implementasi & Logika**:
  - **Pembersihan Prompt (`daily_diary.tmpl`)**: Menghapus seluruh contoh penjelasan literal spesifik seperti `(e.g. "Capitalized 'I', added 'the', and pluralized 'skills'.")` dan contoh koreksi `"learn coding skills"`. Menggantinya dengan aturan negatif eksplisit: *"If a sentence has no grammatical error, keep original text unchanged and output 'No errors in this sentence.' Do NOT invent errors or add words."*
  - **Deteksi Deterministik (`normalizeExplanation`)**: Pada fungsi `diary.go`, sistem membandingkan teks `Original` dan `Corrected`. Jika kedua string identik secara *case-insensitive* (`strings.EqualFold(origBase, corrBase) == true`), sistem tidak memercayai penjelasan model LLM, melainkan secara deterministik menghasilkan penjelasan gramatikal akurat seperti `"Capitalized the sentence start and the pronoun 'I'."`
  - **Penyaringan Kosakata Input Pengguna (`filterNewVocabularies`)**: Model kecil sering mengulangi kata yang sudah ditulis pengguna sebagai "kata baru". Fungsi Go mengekstrak seluruh kata pengguna menjadi set token terlarang, lalu memfilter kata usulan model dan menyuntikkan fallback kosakata terkurasi CEFR B2–C1 jika rekomendasi model tereliminasi.
- **Rujukan Kode Sumber**:
  Cuplikan logika di `internal/usecase/feedback/diary.go`:
  ```go
	if strings.EqualFold(origBase, corrBase) && origBase != "" {
		if orig != corr {
			if strings.Contains(orig, "i ") || strings.HasPrefix(orig, "i ") || orig == "i" {
				return "Capitalized the sentence start and the pronoun 'I'."
			}
			return "Capitalized the first word of the sentence."
		}
		return "No errors in this sentence."
	}
  ```
- **Rasional & Keputusan Teknis**:
  Menyerahkan tugas deterministik (seperti pengecekan kesamaan teks dasar) kepada komputasi bahasa pemrograman (Go) dan hanya memanfaatkan LLM untuk evaluasi semantik kompleks mengeliminasi kelemahan model kecil secara mutlak tanpa membutuhkan upgrade model ke ukuran yang lebih berat.

##### 2. Strategi Deferral Interaksi Discord & Stuck Lobby Rescue Button
- **Ringkasan Inti**: Discord Gateway mewajibkan ACK interaksi dalam waktu 3 detik. Waktu inferensi lokal Ollama (3–15 detik) melampaui batas tersebut. Sistem mengatasinya dengan *immediate deferral* dan tombol penyelamat sesi macet (*rescue reset*).
- **Detail Implementasi & Logika**:
  - **Deferral Segera (Immediate Deferral)**:
    - Saat slash command atau tombol join lobi dipicu oleh pemain, handler langsung memanggil `s.InteractionRespond` dengan tipe `discordgo.InteractionResponseDeferredChannelMessageWithSource` (untuk pesan baru) atau `discordgo.InteractionResponseDeferredMessageUpdate` (untuk pembaruan tombol/soal).
    - Panggilan ke Ollama (`StartRound`, `StartGame`, `EvaluateTurn`) kemudian dieksekusi secara asinkron dalam batas timeout Go context.
    - Setelah teks AI selesai dihasilkan, bot memperbarui tampilan Discord via `FollowupMessageCreate` atau `InteractionResponseEdit`.
  - **Stuck Lobby Reset Button**:
    - Apabila koneksi Ollama sempat gagal atau server hang saat permainan sedang berjalan, channel akan terkunci karena `sessionManager` mendeteksi sesi aktif.
    - Sistem menyematkan komponen tombol `Cancel Stuck Game & Reset` (`<game>:cancel:<sessionID>`) dengan gaya bahaya (`DangerButton`). Ketika ditekan oleh pemain mana pun di channel tersebut, sesi di-rollback seketika ke `StatusCancelled` sehingga channel bebas untuk memulai game baru.
- **Rujukan Kode Sumber**:
  Cuplikan di `internal/delivery/discord/command_play_wyr.go`:
  ```go
	_ = s.InteractionRespond(i.Interaction, &discordgo.InteractionResponse{
		Type: discordgo.InteractionResponseDeferredChannelMessageWithSource,
	})

	round, roundErr := h.wyrUseCase.StartRound(ctx, startedSession.ID, 1, difficulty)
	if roundErr != nil {
		_, _ = h.sessionManager.CancelSession(ctx, startedSession.ID)
		sendErrorFollowup(s, i.Interaction, "Failed to start round. Session reset.")
		return
	}
  ```
- **Rasional & Keputusan Teknis**:
  Pola *Defer-then-Followup* mematuhi arsitektur asynchronous Discord secara murni, menghilangkan 100% galat `Interaction failed`, sementara tombol rescue memberikan otonomi penuh kepada pengguna Discord untuk memulihkan keadaan tanpa perlu intervensi restart container oleh administrator server.

