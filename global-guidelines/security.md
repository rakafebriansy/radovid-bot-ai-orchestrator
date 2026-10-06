# Security & Environment Constraints

Standar keamanan mutlak ini diperuntukkan guna mencegah kecerobohan kebocoran privasi (*credentials leakage*) atau penyalahgunaan penulisan kode oleh AI Agent.

## 1. Zero Hardcoded Secrets & No Dummy Fallbacks
Anda **DILARANG SECARA MUTLAK** untuk:
1. Menyisipkan kunci kriptografi, kredensial peladen (*server credentials*), sandi koneksi basis data (*database passwords*), token OAuth, atau *API Key* pihak ketiga (seperti OpenAI, Firebase, AWS, Stripe, Supabase, dll.) secara literal (sebagai teks tertanam) ke dalam struktur berkas kode sumber.
2. Menyediakan string *fallback* semu/dummy pada pemanggilan kredensial (contoh TERLARANG: `process.env.API_KEY || "dummy_secret"` atau `process.env.SECRET_KEY ?? "my_secret"`).

Seluruh variabel rahasia dan konfigurasi lingkungan **WAJIB** dikelola melalui *Environment Variables* dan divalidasi dengan pola *Fail-Fast* (melempar *runtime error* jika kunci tidak terdefinisi). Rujuk detail implementasi pada [coding.md](./coding.md).

## 2. Kewajiban Pengabaian Git (Strict .gitignore)
Sebelum sistem orkestrasi Anda menginjeksikan atau menginisialisasi skema pengumpulan kredensial *Environment Variables* lokal (seperti penciptaan file `.env`), tugas pertama dan paling esensial Anda adalah menjamin bahwa rute penamaan file kredensial tersebut telah tercatat teguh di dalam daftar hirarki eksklusi kontrol versi `.gitignore`. Tidak ada toleransi bagi kebocoran kunci rahasia (*secret leak*) menuju repositori Git.

## 3. Kewajiban Sanitasi Input Dasar (Input Validation)
Saat membangun fitur antarmuka pemrograman terbuka (*public-facing API*) maupun interaksi *form inputs* dari sisi antarmuka klien pengguna:
- AI **DILARANG** mempercayai struktur *payload* mentah dari sisi klien secara naif.
- Anda wajib mensisipkan skema penyaringan (*sanitization*), filter regulasi regex, atau memanfaatkan perpustakaan *Data Validation Schema* baku untuk mencegat upaya penyusupan manipulatif seperti SQL Injection, manipulasi struktur NoSQL, hingga pancingan XSS (Cross-Site Scripting).
