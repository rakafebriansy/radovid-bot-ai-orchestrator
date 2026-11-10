# Security & Environment Constraints

> **[PRINSIP UTAMA]** Standar keamanan mutlak untuk mencegah kebocoran kredensial, eksfiltrasi data, dan celah kerentanan dalam basis kode.

---

## 1. Tata Kelola Rahasia Lingkungan (Secrets Governance)

### 1.1 Larangan Hardcoded Secrets & Dummy Fallbacks
1. **[TERLARANG]** Menyisipkan kunci kriptografi, kredensial peladen, sandi koneksi basis data, token OAuth, atau API Key pihak ketiga (seperti OpenAI, Firebase, AWS, Stripe, Supabase, dll.) secara literal ke dalam berkas kode sumber.
2. **[TERLARANG]** Menyediakan string *fallback* semu atau dummy pada pembacaan kredensial lingkungan:
   ```typescript
   // [SALAH] Terlarang: Fallback dummy menutupi ketiadaan konfigurasi
   const apiKey = process.env.API_KEY || "dummy_secret";
   const secret = process.env.SECRET_KEY ?? "my_secret";
   ```
3. **[WAJIB]** Seluruh variabel rahasia dan konfigurasi lingkungan dikelola melalui *Environment Variables* dan divalidasi dengan pola *Fail-Fast* (melempar *runtime error* jika kunci tidak terdefinisi). Rujuk pola `getRequiredEnv()` pada [coding.md](./coding.md).

### 1.2 Isolasi Berkas Kredensial (.gitignore First)
1. **[WAJIB]** Sebelum membuat atau menginisialisasi berkas lingkungan lokal (`.env`, `.env.local`, `.env.*.local`), pastikan pola penamaan berkas tersebut telah tercatat dalam hirarki eksklusi `.gitignore`.
2. **[TERLARANG]** Melakukan commit berkas kredensial atau rahasia ke kontrol versi Git. Nol toleransi terhadap kebocoran secret.

### 1.3 Masking & Larangan Pencetakan Rahasia ke Log
1. **[TERLARANG]** Mencetak nilai *environment variables* rahasia secara mentah ke log terminal, error payload, URL redirect, atau log konsol runtime (`console.log(process.env)`, `print(os.environ)`). Rujuk [pipeline.md](./pipeline.md).
2. **[WAJIB]** Gunakan masking parsial jika identitas kredensial perlu dicatat untuk audit trail (contoh: `sk-...xxxx`).

### 1.4 Standar Primitif Kriptografi & Perbandingan Waktu Konstan
1. **Keacakan Kriptografis (*CSPRNG*):** Wajib menggunakan generator angka acak aman secara kriptografis (contoh: `crypto.randomBytes`, `crypto.randomUUID`, `secrets` pada Python) untuk pembuatan token, kunci, tantangan (*challenge*), dan *nonce*. Dilarang menggunakan `Math.random()` atau PRNG non-kriptografis.
2. **Kuat & Terautentikasi:** Gunakan algoritma enkripsi terautentikasi (seperti AES-GCM, ChaCha20-Poly1305) dan derivasi kunci baku (Argon2id, bcrypt, scrypt, PBKDF2). Dilarang menggunakan mode ECB, IV statis, atau *nonce reuse*.
3. **Perbandingan Waktu Konstan (*Constant-Time Comparison*):** Wajib menggunakan fungsi pembanding waktu konstan (contoh: `crypto.timingSafeEqual`) saat memverifikasi HMAC, tanda tangan webhook, API key, atau token rahasia guna mencegah *timing side-channel*.
4. **Kegagalan Tertutup (*Fail-Closed Crypto*):** Jika operasi dekripsi, parsing token, atau verifikasi tanda tangan gagal, jalur penanganan error wajib menolak akses secara mutlak (*fail-closed*). Dilarang jatuh ke mode tanpa enkripsi atau melewati verifikasi (*fail-open*).

---

## 2. Validasi Input, Penanganan Berkas, & Injeksi Lanjutan (Input & Sink Hardening)

### 2.1 Kebijakan Zero-Trust Payload
1. **[ATURAN]** AI DILARANG mempercayai struktur payload mentah dari sisi klien secara naif, baik pada *public-facing API* maupun interaksi form input antarmuka pengguna.
2. **[WAJIB]** Terapkan skema sanitasi data, filter regex ketat, atau pustaka validasi skema baku (seperti Zod, Yup, Joi, Pydantic) sebelum data diproses oleh logika bisnis. Validasi skema hanya membuktikan bentuk data, bukan otorisasi atau keamanan nilainya.

### 2.2 Mitigasi Injeksi Langsung & Tidak Langsung (Direct & Second-Order Injection)
1. **SQL & NoSQL Injection:** Wajib menggunakan *parameterized queries* atau ORM/query builder teruji; sanitasi operator kueri dinamis pada basis data dokumen. Dilarang konkatenasi string kueri mentah.
2. **Cross-Site Scripting (XSS) & Template Injection:** Lakukan sanitasi dan *escape* output data dinamis sebelum dirender ke HTML/DOM; dilarang menyisipkan input pengguna ke dalam evaluasi string (`eval()`, `new Function()`, template engine mentah).
3. **Command & Shell Injection:** Dilarang menyusun perintah shell melalui konkatenasi string input pengguna (`exec`, `system`). Gunakan pemanggilan proses berbasis array argumen tanpa shell interpolation.
4. **Second-Order Injection:** Waspadai data yang disimpan secara aman pada tahap awal namun diambil kembali dan digunakan pada konteks berbahaya (*dangerous sink*) oleh komponen hilir (contoh: nama field berubah menjadi JSON path, *slug* menjadi path berkas, atau teks tersimpan dieksekusi di dalam regex/template/policy). Periksa injeksi pada nama field, kunci objek, HTTP header, dan metadata—bukan hanya nilai body.

### 2.3 Keamanan Operasi Berkas, Path Traversal, & SSRF
1. **Path Traversal & Ekstraksi Arsip (*Zip Slip*):** Kanonikalisasi dan verifikasi setiap path berkas agar tetap berada di dalam direktori dasar yang diizinkan. Tolak sekuens `..`, karakter null byte, symlink tak tepercaya, serta validasi setiap path entri sebelum mengekstrak arsip.
2. **Server-Side Request Forgery (SSRF):** Saat peladen mengambil URL yang dipengaruhi pengguna (webhook, callback, pratinjau tautan, impor media), validasi skema dan resolusi IP tujuan terhadap daftar blokir jaringan internal, loopback (`127.0.0.0/8`, `::1`), link-local, dan layanan *cloud metadata* (`169.254.169.254`). Validasi ulang wajib dijalankan **setelah setiap HTTP redirect** untuk mencegah bypass via redirect dan *DNS rebinding*.
3. **Pencegahan Race Condition Berkas (*TOCTOU*):** Hindari celah *Time-of-Check to Time-of-Use* antara pemeriksaan eksistensi/izin berkas dengan pembukaan berkas; gunakan operasi berbasis *file descriptor* yang aman dari pergantian symlink.

---

## 3. Keamanan Protokol Web, Sesi, & Identitas (Web Protocol & Identity Security)

### 3.1 Keamanan Sesi Browser, Cookie, & CSRF
1. **Rotasi & Pencabutan Sesi (*Session Fixation & Invalidation*):** Wajib merotasi ID sesi atau token saat login, perpindahan akun, penyelesaian MFA, atau perubahan tingkat hak akses (*privilege escalation*). Cabut (*revoke*) sesi di sisi peladen, refresh token, dan state koneksi WebSocket saat pengguna logout, mengubah kata sandi, atau dinonaktifkan.
2. **Mitigasi CSRF:** Lindungi seluruh endpoint pengubah state (*state-changing*) yang menggunakan kredensial ambien browser (cookie) dengan token anti-CSRF, validasi ketat header `Origin`/`Referer`, serta atribut cookie `SameSite`, `HttpOnly`, dan `Secure` dengan cakupan `Domain`/`Path` paling sempit.

### 3.2 Verifikasi Identitas Terfederasi (JWT, OAuth/OIDC, SAML)
1. **Pengikatan Klaim & Algoritma JWT:** Kunci algoritma verifikasi secara eksplisit di sisi peladen (*server-pinned algorithm*; tolak `alg: none` dan kebingungan kunci simetris/asimetris). Perlakukan header `kid`, `jku`, dan `x5u` sebagai input tak tepercaya. Wajib memverifikasi klaim `exp`, `nbf`, `iss`, dan `aud`—token yang sah untuk layanan lain wajib ditolak jika `aud` tidak cocok.
2. **Pengikatan OAuth/OIDC & SAML:** Validasi kepemilikan `redirect_uri` secara eksak, ikat parameter `state` dan PKCE ke sesi penginisiasi, serta verifikasi `nonce` dan audiens pada ID token. Pada SAML, pastikan elemen XML yang diverifikasi tanda tangannya adalah elemen yang sama yang digunakan sebagai identitas, dengan konfigurasi parser XML yang menonaktifkan DTD/XXE.

### 3.3 Pengikatan MFA, Passkey/WebAuthn, Pemulihan Akun, & API Key/mTLS
1. **Pengikatan Step-Up & MFA (*Assurance Binding*):** Tantangan MFA atau *step-up authentication* wajib diikat ke principal, sesi aktif saat ini, operasi/resource target, batas kedaluwarsa, dan status sekali pakai (*one-time completion*). Pendaftaran, penggantian, atau penonaktifan faktor kedua wajib mensyaratkan autentikasi segar.
2. **WebAuthn/Passkey & Penautan Akun (*Account Linking*):** Ikat `challenge`, `rpId`, `origin`, dan `userHandle` ke sesi penginisiasi. Penautan identitas eksternal (IdP, email, nomor telepon, passkey) ke akun eksisting wajib mensyaratkan sesi terautentikasi dan verifikasi kepemilikan atas identitas baru guna mencegah *pre-account takeover*.
3. **Keamanan Alur Pemulihan (*Password Reset*):** Token pemulihan wajib berentropi tinggi, terikat pada pengguna dan aksi spesifik, berumur pendek, *single-use*, serta langsung menggugurkan token sebelumnya dan sesi aktif begitu kata sandi berhasil direset.
4. **Cakupan API Key & Terminasi mTLS:** Pisahkan secara tegas *publishable key* dari *secret key*. Ikat API key pada cakupan tenant, resource, dan aksi di sisi peladen tanpa dapat di-override oleh parameter request. Jika proxy tepercaya melakukan terminasi mTLS, pastikan hanya proxy tersebut yang dapat terhubung dan seluruh header identitas sertifikat dari klien luar dibersihkan terlebih dahulu.

### 3.4 Keamanan HTTP Framing, Proxy Header, & Cache
1. **Kepercayaan Header Proxy (*Host & Forwarded Headers*):** Dilarang mempercayai header `Host`, `Forwarded`, `X-Forwarded-For`, atau `X-Forwarded-Host` dari klien untuk menyusun tautan reset kata sandi, routing tenant, URL callback, atau kunci rate-limit kecuali dijamin telah ditimpa oleh *trusted reverse proxy*.
2. **Pencegahan Web Cache Poisoning & Cache Deception:** Pastikan input yang tidak termasuk dalam *cache key* (*unkeyed headers/cookies*) tidak mengubah isi respons atau header keamanan yang disimpan di cache, serta pastikan rute dinamis privat memiliki direktif `Cache-Control: private, no-store` agar tidak disimpan oleh CDN sebagai aset statis publik.
3. **Pencegahan Open Redirect & Response-Header Injection:** Validasi parameter tujuan redirect (`redirect`, `next`, `return`, `continue`) terhadap *allowlist* path/origin internal, dan tolak karakter kontrol (`\r`, `\n`) pada nilai yang masuk ke header respons (`Location`, `Set-Cookie`).

---

## 4. Kontrol Akses, Isolasi Data Multi-Tenant, & Logika Bisnis

### 4.1 Otorisasi Per-Resource & Konsistensi Jalur Paralel
1. **Verifikasi Tiga Dimensi Otorisasi:** Jangan hanya memeriksa apakah middleware autentikasi terpasang; pastikan sistem memeriksa **izin yang tepat**, untuk **resource yang tepat**, melalui **mekanisme yang tepat**.
2. **Konsistensi Jalur Alternatif (*Sibling Paths*):** Kebijakan keamanan efektif ditentukan oleh jalur paralel terlemah. Pastikan endpoint batch/bulk, ekspor/impor, GraphQL, webhook, dan *background job* menerapkan pemeriksaan izin per-item yang setara dengan endpoint UI utama.
3. **Pencegahan Mass Assignment:** Dilarang mengizinkan field pada body request (`role`, `permissions`, `tenantId`, `ownerId`, `status`) menimpa atribut kepemilikan atau otorisasi yang seharusnya berasal dari sesi terautentikasi.

### 4.2 Isolasi Data Multi-Tenant & Kunci Komposit (Data Lineage)
1. **Penegakan Filter Tenant Eksplisit:** Keberadaan kolom `tenant_id` atau `owner_id` pada tabel bukanlah isolasi otomatis. Setiap kueri baca, tulis, hapus, agregasi (`COUNT`), relasi bersarang, dan *raw query / unscoped client* wajib mengikat predikat tenant/pemilik dari principal yang terautentikasi.
2. **Namespace pada Kunci Komposit:** Sertakan identitas tenant dan environment pada setiap *cache key*, path objek/blob storage, ID dokumen indeks pencarian, berkas sementara, dan kunci deduplikasi guna mencegah tabrakan lintas-tenant (*composite-key collision*).
3. **Pembatasan Signed URLs:** Ikat URL bertanda tangan (*pre-signed URLs*) pada metode operasi spesifik (`GET` vs `PUT`), path objek/versi eksak, audiens/tenant, dan masa berlaku singkat.

### 4.3 Siklus Hidup Penghapusan, Data Turunan, & Pencegahan Oracle
1. **Sinkronisasi Data Turunan (*Derived Copies*):** Saat izin akses (*ACL*) dicabut atau data dihapus (*soft-delete* / *hard-delete*) pada penyimpanan utama, invalidasi segera salinan turunannya di *cache*, *search index*, pratinjau/thumbnail, dan sesi aktif.
2. **Kepatuhan Soft-Delete & Validasi Ulang saat Restore:** Pastikan kueri langsung, traversal relasi, dan pekerja latar belakang menghormati predikat *soft-delete*. Operasi pemulihan (*restore/undelete/rollback*) wajib memvalidasi ulang kepemilikan dan otorisasi terhadap kebijakan keamanan saat ini.
3. **Pencegahan Eksfiltrasi Ekspor & Enumeration Oracles:** Pastikan fitur ekspor/pencadangan memfilter izin per-item dan tidak menyertakan data *soft-deleted* atau rahasia lintas-tenant. Cegah kebocoran eksistensi data privat melalui filter pencarian, pengurutan pada kolom tersembunyi (*sort oracle*), atau perbedaan mencolok pada pesan error dan waktu respons.

### 4.4 Keamanan State Machine & Konkurensi Logika Bisnis
1. **Integritas Transisi State:** Cegah pelompatan tahapan alur kerja (*step skipping*), pengulangan alur yang telah selesai (*replay*), atau kegagalan parsial tanpa *rollback* pada transaksi multi-langkah.
2. **Keamanan Konkurensi (*Atomic Check-Then-Act*):** Gunakan transaksi atomik, penguncian baris (*row locking* `SELECT ... FOR UPDATE`), atau operasi pembaruan bersyarat untuk mencegah kondisi balapan berdampak bisnis (*double-spend*, *double-claim*, *race condition* kuota/saldo).

---

## 5. Keamanan Sisi Klien, RPC/Messaging, AI/LLM, Native IPC, & Cloud

### 5.1 Keamanan Sisi Klien & Browser (Client-Side Security)
1. **DOM XSS, DOM Clobbering, & Prototype Pollution:** Hindari mengalirkan sumber browser (`location`, `window.name`, `document.referrer`, `postMessage`, storage) ke sink eksekusi (`innerHTML`, `document.write`, URL `javascript:`/`data:`). Cegah *DOM Clobbering* pada atribut `id`/`name` dari markup pengguna dan blokir kunci `__proto__`, `constructor`, serta `prototype` pada penggabungan objek rekursif (*deep merge*).
2. **Keamanan Lintas-Origin (`postMessage`, CORS, WebSocket, Clickjacking):**
   - Verifikasi `event.origin` dengan *allowlist* eksak (dilarang menggunakan pencocokan substring/regex longgar) serta `event.source` pada penerima `postMessage`, dan tentukan `targetOrigin` eksplisit (bukan `'*'`) saat mengirim data sensitif.
   - Dilarang memantulkan (`reflect`) header `Origin` sembarangan bersamaan dengan `Access-Control-Allow-Credentials: true`.
   - Verifikasi header `Origin` atau token kanal saat *WebSocket upgrade*, serta terapkan `Content-Security-Policy: frame-ancestors` / `X-Frame-Options` pada aksi sensitif untuk mencegah *Clickjacking*.
3. **Pembersihan Storage & Service Worker:** Jangan menyimpan respons terpersonalisasi pada *Service Worker* atau Cache Storage tanpa mengikatnya ke identitas akun/tenant, dan bersihkan seluruh token serta data sensitif di `localStorage`, `sessionStorage`, `IndexedDB`, dan `Cache Storage` saat logout atau perpindahan akun.

### 5.2 Keamanan Protokol RPC, Message Broker, & Ketahanan Sumber Daya
1. **Pengikatan Identitas Kanal & Interceptor RPC:** Ikat identitas kanal (*peer identity* mTLS/workload) secara deterministik ke principal aplikasi; pastikan klien tidak dapat mengganti tenant/user di dalam payload pesan. Terapkan interceptor autentikasi/otorisasi secara merata pada metode unary, *streaming RPC*, batch GraphQL, dan validasi ulang izin per-item pada aliran pesan (*stream*).
2. **Isolasi Broker, DLQ, & Idempotensi:** Isolasi topik dan *routing key* broker per-tenant, redaksi data rahasia sebelum pesan gagal dikirim ke *Dead-Letter Queue (DLQ)*, verifikasi tanda tangan serta *replay state* pada setiap webhook/event produsen, dan terapkan kunci idempotensi yang terikat per-tenant.
3. **Pencegahan Eksploitasi Sumber Daya (*Anti-Amplification & Availability*):** Cegah regex dengan *catastrophic backtracking* (ReDoS), batasi kedalaman bersarang (*nesting depth*) serta ukuran hasil dekompresi (*decompression bomb*), tolak komputasi berat sebelum gerbang autentikasi terawal, teruskan sinyal pembatalan (*cancellation/deadline*) ke database/worker saat klien terputus, dan sediakan mekanisme *quarantine* untuk *poison records* agar tidak memblokir antrean bersama.

### 5.3 Keamanan Integrasi AI, LLM, RAG, & Agentic Tool-Calling
1. **Perlakuan Konteks & Output Model sebagai Untrusted Input:** *System prompt* atau *guardrail prompt* bukanlah batas keamanan deterministik. Perlakukan seluruh keluaran model, dokumen hasil *retrieval* (RAG), memori percakapan, dan respons MCP sebagai input tak tepercaya yang wajib divalidasi oleh kode deterministik sebelum menyentuh sink (HTML, SQL, shell, URL, atau file).
2. **Isolasi Konteks RAG & Pencegahan Memory Poisoning:** Terapkan filter ACL dan tenant langsung di dalam kueri *vector database* serta *prompt cache*. Cegah konten dari sumber berkepercayaan rendah menyuntikkan instruksi permanen ke dalam memori lintas-sesi atau memalsukan atribut peran pesan (`system` / `tool`).
3. **Otorisasi Handler Tool & Pengikatan Konfirmasi Aksi (*Action Binding*):** Setiap handler tool wajib memverifikasi otorisasi berdasarkan identitas *requesting principal* (bukan kredensial servis penuh milik agen) guna mencegah *confused deputy*. Untuk aksi sensitif yang memerlukan persetujuan pengguna, ikat konfirmasi tersebut pada nama tool yang telah dinormalisasi, seluruh objek argumen, principal, target resource, dan batas waktu kedaluwarsa, serta terapkan batas iterasi/anggaran per-request pada loop agen dan sub-agen/MCP.

### 5.4 Keamanan Aplikasi Desktop, Mobile, WebView, & Kode Native
1. **Keamanan Deep-Link & WebView Bridge:** Ikat callback autentikasi *deep-link* / *custom scheme* ke sesi aktif. Batasi akses *JavaScript-to-Native Bridge* pada WebView hanya untuk origin tepercaya yang divalidasi ulang pada **setiap navigasi, redirect, dan subframe** (bukan hanya saat muat awal), serta matikan akses berkas lokal (`file://`) dari konten remote.
2. **Autentikasi Peer Local IPC & Komponen Terekspos:** Pada komunikasi antar-proses lokal (Unix socket, Named Pipes, XPC, Binder, D-Bus) atau *privileged helper*, verifikasi kredensial peer tingkat OS (*UID/PID/code signature*), bukan sekadar nama proses atau klaim payload. Pastikan komponen mobile (`Activity`, `Service`, `BroadcastReceiver`, `ContentProvider`) tidak diekspor tanpa izin ketat.
3. **Keamanan Memori & Batas FFI (*Memory Safety*):** Pada kode native/unsafe (C/C++, Rust `unsafe`, FFI, parser biner), hitung ulang batas buffer dan potensi *integer overflow/underflow/truncation* sebelum alokasi atau penyalinan memori, selaraskan satuan ukuran (byte vs elemen) di kedua sisi FFI, cegah *use-after-free* pada jalur error/pembatalan, dan muat pustaka/plugin hanya dari path tepercaya yang terverifikasi tanda tangannya.

### 5.5 Keamanan Cloud, Kontainer, & Rantai Pasok (Supply Chain & CI/CD)
1. **Least-Privilege Workload & Konfigurasi Deployment:** Batasi izin IAM workload/pod/fungsi hanya pada resource spesifik yang dibutuhkan. Pastikan *overlay* konfigurasi produksi tidak menonaktifkan kontrol keamanan akibat nilai default development (*security-control precedence drift*), dan jangan mengekspos soket runtime kontainer, mode `privileged`, atau port manajemen internal.
2. **Integritas Rantai Pasok & Isolasi Workflow CI/CD:** Kunci sumber registri paket untuk mencegah *dependency confusion*, gunakan *lockfile* dan digest imutabel pada proses build/promosi rilis, jangan pernah menjalankan kode PR dari sumber tak tepercaya dengan akses ke *secrets* atau token tulis produksi, serta dilarang menyisipkan input yang dapat dikontrol pengguna (nama branch, judul PR, pesan komit) secara mentah ke dalam ekspresi shell workflow CI.

---

## 6. Protokol Audit Keamanan Terstruktur (Defensive Security Audit Protocol)

### 6.1 Dua Mode Operasional Keamanan
1. **Guidance Mode (Default):** Berlaku pada pengembangan fitur rutin, tinjauan kode terfokus, atau tanya-jawab arsitektur. AI Agent menerapkan aturan pada Seksi 1–5 secara langsung tanpa membuat direktori laporan audit terpisah.
2. **Full Audit Mode:** Diaktifkan hanya ketika pengguna secara eksplisit meminta audit keamanan menyeluruh (*security audit*, *vulnerability review*, atau *pen-test* basis kode). Hasil temuan didokumentasikan secara terstruktur ke direktori `nodes/[nama-node]/hunt/` atau dokumen laporan yang diminta.

### 6.2 Syarat Sah Temuan & Gerbang Bukti (Concrete Boundary & Invariant Gate)
Setiap kandidat temuan kerentanan WAJIB membuktikan pelanggaran batas kepercayaan (*trust boundary*) nyata dengan menyebutkan 6 elemen secara eksplisit:
1. **Lower-Trust Principal:** Aktor penyerang dan kapabilitas awalnya yang realistis.
2. **Accepted Input / Action:** Input, aksi, transisi state, atau selektor resource yang diterima sistem.
3. **Intended Control:** Kontrol keamanan pada kode sumber yang seharusnya menolak, mengikat, atau mengisolasi aksi tersebut.
4. **Crossed Boundary:** Batas kepercayaan yang dilanggar (jejak kode lengkap dari `entrypoint` $\rightarrow$ `propagation` $\rightarrow$ `sink`).
5. **Affected Principal / Resource:** Korban, tenant lain, atau resource bersama yang terdampak.
6. **Concrete Observable Result:** Dampak nyata yang teramati. Ketiadaan *best practice* tanpa jalur eksploitasi nyata, crash parser tanpa dampak lintas-batas, atau dampak terhadap diri sendiri (*self-impact* / *self-XSS*) bukanlah temuan kerentanan dan hanya dicatat sebagai catatan *hardening*.

### 6.3 Keselamatan Eksekusi Verifikasi (Bounded & Sandboxed Local Verification)
1. **Read-Only & Sandboxed:** Inspeksi kode bersifat *read-only*. Pembuktian perilaku kerentanan wajib menggunakan pengujian lokal terbatas (*unit test*, *function harness* kecil, *dummy tenant*, atau *isolated loopback*) di dalam lingkungan terisolasi tanpa jaringan eksternal.
2. **[TERLARANG KERAS] Eksploitasi Layanan Live & Ekspansi Dampak:** Dilarang mem-probe endpoint produksi/staging bersama, menggunakan data atau kredensial pengguna asli, menjalankan uji beban/saturasi (*stress/DoS*), menghabiskan kuota API berbayar, atau melanjutkan eksploitasi melebihi bukti minimum (*minimum boundary effect*).

### 6.4 Klasifikasi Status Verifikasi & Kalibrasi Keparahan (Severity Calibration)
1. **Tiga Status Verifikasi (*Verdicts*):**
   - `confirmed`: Jejak kode lengkap dan dampak lintas-batas terbukti melalui verifikasi lokal terbatas. Hanya temuan berstatus `confirmed` yang boleh diberikan tingkat keparahan (*severity*).
   - `needs_validation`: Hipotesis berbasis kode nyata yang kepastiannya bergantung pada konfigurasi di luar repositori (aturan proxy/CDN, kebijakan IAM cloud, IdP, atau header peladen produksi). Wajib mencantumkan pemblokir spesifik beserta rencana verifikasi aman (*owner-observed check*), dan **DILARANG** diberi label *severity*.
   - `rejected`: Klaim yang terbantahkan oleh kontrol keamanan di kode sumber atau hasil pengujian lokal.
2. **Jangkar Kalibrasi Severity (Khusus Status `confirmed`):**
   - **`Critical`:** Aktor tanpa autentikasi (*unauthenticated*) memperoleh eksekusi kode (RCE), akses penuh penyimpanan data, atau pengambilalihan akun arbitrer.
   - **`High`:** Aktor sepenuhnya mematahkan kontrol keamanan eksplisit dengan konsekuensi nyata: bypass autentikasi, baca/tulis data lintas-tenant, *stored XSS* yang memengaruhi pengguna lain, eksekusi kode terautentikasi, atau penghentian layanan bersama dari jarak jauh.
   - **`Medium`:** Pelanggaran batas kepercayaan nyata namun dengan *blast radius* terbatas, prasyarat tidak umum, atau dampak terbatas pada sebagian kecil resource.
   - **`Low`:** Kebocoran informasi internal non-rahasia atau dampak minimal yang membutuhkan upaya berkelanjutan tinggi.
   - **`Informational`:** Observasi terkonfirmasi berdampak minimal yang relevan sebagai prasyarat dalam rantai temuan lain.
   - *Aturan Diskriminator:* Tingkat *overall severity* tidak boleh melebihi dampak nyata yang berhasil dibuktikan.

### 6.5 Prinsip Perbaikan Terkecil (Smallest Effective Source Fix)
Untuk setiap temuan `confirmed`, rumuskan *security invariant* yang wajib ditegakkan dan terapkan perubahan kode paling sempit tepat pada **titik keputusan tepercaya terakhir (*last trusted decision point*)** sebelum *sink*, disertai pengujian regresi otomatis sesuai [testing.md](./testing.md).
