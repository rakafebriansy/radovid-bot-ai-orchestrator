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
1. **[TERLARANG]** Mencetak nilai *environment variables* rahasia secara mentah ke log terminal atau log konsol runtime (`console.log(process.env)`, `print(os.environ)`). Rujuk [pipeline.md](./pipeline.md).
2. **[WAJIB]** Gunakan masking parsial jika identitas kredensial perlu dicatat untuk audit trail (contoh: `sk-...xxxx`).

---

## 2. Validasi & Sanitasi Input (Input Hardening)

### 2.1 Kebijakan Zero-Trust Payload
1. **[ATURAN]** AI DILARANG mempercayai struktur payload mentah dari sisi klien secara naif, baik pada *public-facing API* maupun interaksi form input antarmuka pengguna.
2. **[WAJIB]** Terapkan skema sanitasi data, filter regex ketat, atau pustaka validasi skema baku (seperti Zod, Yup, Joi, Pydantic) sebelum data diproses oleh logika bisnis.

### 2.2 Mitigasi Vektor Kerentanan Baku
1. **SQL Injection:** Wajib menggunakan *parameterized queries* atau ORM/query builder teruji. Dilarang konkatenasi string SQL mentah.
2. **NoSQL Manipulation:** Sanitasi operator kueri dinamis sebelum dieksekusi oleh mesin basis data dokumen.
3. **Cross-Site Scripting (XSS):** Lakukan sanitasi dan *escape* output data dinamis pengguna sebelum dirender pada antarmuka web; cegah injeksi HTML atau skrip tak terverifikasi.

---

## 3. Protokol Audit Keamanan Terstruktur (Security Audit Skill)

### 3.1 Integrasi Keahlian Audit Resmi
1. **[SKILL RUJUKAN AUDIT]** Untuk penugasan evaluasi keamanan menyeluruh, tinjauan kerentanan (*vulnerability review*), penetrasi defensif (*pen-test*), atau pertanyaan keamanan arsitektur, AI Agent WAJIB memuat dan mematuhi [security-audit SKILL.md](../skills/security-audit/SKILL.md).
2. **Kriteria Bukti Konkret:** Temuan kerentanan WAJIB melanggar batas kepercayaan (*trust boundary*) nyata dengan principal terdampak, sumber bukti (*source evidence*), dan langkah reproduksi lokal yang aman. Dilarang mengangkat asumsi tanpa dampak nyata menjadi temuan keamanan.
3. **Katalog Kelas Serangan (Attack Classes):** Rujuk taksonomi serangan pada [skills/security-audit/ATTACK-CLASSES.md](../skills/security-audit/ATTACK-CLASSES.md) (Auth, Web Protocol, Client-Side, Cloud, RPC/Messaging, AI/LLM, Supply Chain) sebagai acuan verifikasi keamanan sistem.
