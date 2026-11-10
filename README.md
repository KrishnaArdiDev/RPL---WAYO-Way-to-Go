# WAYO — Way To Go

WAYO adalah aplikasi web pemetaan dan navigasi pejalan kaki untuk membantu pengunjung menemukan event, gedung, tempat, dan fasilitas di lingkungan Universitas Udayana, khususnya Kampus Sudirman dan Kampus Jimbaran.

Aplikasi dibangun menggunakan **Next.js 15 App Router, React 19, TypeScript, Tailwind CSS 4, Prisma ORM 6, dan SQLite**. Peta menggunakan Leaflet dan OpenStreetMap, sedangkan pencarian rute pejalan kaki menggunakan OSRM.

## 1. Prasyarat

Pastikan perangkat sudah memiliki:

- **Node.js** versi 20 atau lebih baru (disarankan versi LTS).
- **npm**, yang disertakan bersama Node.js.
- **Git**, apabila proyek diambil melalui GitHub.
- Browser modern seperti Google Chrome, Microsoft Edge, atau Firefox.

Periksa instalasi melalui terminal:

```bash
node -v
npm -v
git --version
```

## 2. Mengambil proyek

### Opsi A — Clone dari GitHub

```bash
git clone https://github.com/KrishnaArdiDev/RPL---WAYO-Way-to-Go.git
cd RPL---WAYO-Way-to-Go
```

Jika nama repository atau URL berubah, sesuaikan URL clone di atas.

### Opsi B — Menggunakan ZIP

1. Unduh ZIP proyek dari GitHub.
2. Ekstrak ZIP ke folder pilihan.
3. Buka terminal pada folder yang berisi `package.json`, `src/`, dan `prisma/`.

> Semua perintah pada bagian berikut harus dijalankan dari **root proyek**, yaitu folder yang berisi `package.json`.

## 3. Menyiapkan environment (`.env`)

Buat file `.env` berdasarkan `.env.example`.

**Windows PowerShell:**

```powershell
if (-not (Test-Path .env)) {
    Copy-Item .env.example .env
}
notepad .env
```

**Linux/macOS:**

```bash
[ -f .env ] || cp .env.example .env
nano .env
```

Konfigurasi minimal untuk menjalankan aplikasi secara lokal:

```env
DATABASE_URL="file:./dev.db"
BASE_URL="http://localhost:3000"
WAYO_DEMO="1"

WAYO_ADMIN_EMAIL="admin@wayo.local"
WAYO_ADMIN_PASSWORD="admin123"

GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
GOOGLE_REDIRECT_URI=""

SMTP_HOST=""
SMTP_PORT="587"
SMTP_USER=""
SMTP_PASS=""
SMTP_SECURE="false"
MAIL_FROM="WAYO <no-reply@wayo.local>"
```

Keterangan:

| Variabel | Fungsi |
|---|---|
| `DATABASE_URL` | Lokasi database SQLite. Dengan konfigurasi contoh, file database berada di `prisma/dev.db`. |
| `BASE_URL` | Alamat utama aplikasi. Secara lokal gunakan `http://localhost:3000`. |
| `WAYO_DEMO` | Isi `1` untuk mengaktifkan endpoint simulasi QR `/api/dev/qr-sample`. Matikan dengan `0` untuk penggunaan publik. |
| `WAYO_ADMIN_EMAIL` | Email akun Admin yang dibuat melalui seed. |
| `WAYO_ADMIN_PASSWORD` | Password akun Admin yang dibuat melalui seed. Gunakan password kuat untuk penggunaan publik. |
| `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET` | Kredensial OAuth 2.0 dari Google Cloud. Boleh dikosongkan untuk penggunaan lokal tanpa Google Login. |
| `GOOGLE_REDIRECT_URI` | Callback OAuth Google. Jika kosong, aplikasi menggunakan `${BASE_URL}/api/auth/google/callback`. |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_SECURE`, `MAIL_FROM` | Pengaturan server email untuk mengirim tautan reset password. |
| `OSRM_URL` | Alamat server OSRM. Jika tidak diisi, aplikasi menggunakan server OSRM publik bawaan. |

**Google Login dan email bersifat opsional untuk menjalankan aplikasi secara lokal.** Jika kredensial Google belum diisi, tombol Google akan menampilkan informasi bahwa Google belum dikonfigurasi. Jika `SMTP_HOST` kosong, tautan reset password tidak dikirim melalui email; informasi tautannya dicetak pada log terminal server dalam mode pengembangan.

Jangan mengunggah file `.env` ke GitHub. File tersebut berisi konfigurasi dan dapat berisi rahasia. Gunakan `.env.example` untuk membagikan nama variabel tanpa kredensial.

## 4. Instal dependensi

Dari root proyek, jalankan:

```bash
npm ci
```

Perintah ini memasang dependensi sesuai `package-lock.json`. Skrip `postinstall` proyek juga menjalankan `prisma generate`.

Jika instalasi gagal karena masalah jaringan, periksa koneksi internet dan coba jalankan kembali perintah tersebut.

## 5. Menyiapkan database dan data awal

Untuk instalasi pertama atau membuat ulang database lokal beserta data contoh, jalankan:

```bash
npm run db:fresh
```

Perintah tersebut akan:

1. Menjalankan migrasi database yang tersedia (`prisma migrate deploy`).
2. Mengosongkan data pada tabel aplikasi.
3. Memasukkan kembali data contoh melalui `prisma/seed.ts`.

Database SQLite bawaan berada di `prisma/dev.db`.

> **Peringatan:** `npm run db:fresh` menghapus data pada database yang ditunjuk oleh `DATABASE_URL` sebelum melakukan seed ulang. Gunakan untuk database lokal/pengujian, bukan untuk database produksi yang berisi data penting.

## 6. Menjalankan aplikasi

### Mode pengembangan

```bash
npm run dev
```

Buka browser dan akses:

```text
http://localhost:3000
```

Biarkan terminal tetap berjalan selama aplikasi digunakan. Perubahan source code akan dipantau oleh Next.js dalam mode pengembangan.

### Mode production lokal

Bangun aplikasi terlebih dahulu:

```bash
npm run build
```

Jika build berhasil, jalankan:

```bash
npm run start
```

Secara default aplikasi berjalan pada `http://localhost:3000`.

Untuk menghentikan server, tekan `Ctrl + C` pada terminal tempat server dijalankan.

## 7. Akun demo

Setelah `npm run db:fresh` atau `npm run db:seed` berhasil, akun demo berikut dibuat:

| Peran | Email | Password |
|---|---|---|
| Pengunjung (`VISITOR`) | `pengunjung@contoh.id` | `demo123` |
| Panitia (`ORGANIZER`) | `panitia@contoh.id` | `demo123` |
| Admin (`ADMIN`) | `admin@wayo.local` | Nilai `WAYO_ADMIN_PASSWORD` di `.env` (default `admin123`) |

Akun Admin dibuat melalui proses seed, bukan melalui formulir pendaftaran publik. Jika `WAYO_ADMIN_EMAIL` atau `WAYO_ADMIN_PASSWORD` diubah, jalankan kembali `npm run db:seed` agar akun Admin pada database diperbarui.

> Akun tersebut hanya untuk demonstrasi/pengembangan. Ganti password dan data akun sebelum aplikasi dibuka untuk penggunaan publik. Data event dan tempat bawaan juga merupakan data contoh.

## 8. Perintah database

| Perintah | Fungsi | Catatan |
|---|---|---|
| `npm run db:migrate` | Menjalankan `prisma migrate dev` untuk pengembangan skema. | Gunakan saat membuat/menerapkan migrasi baru dalam pengembangan. |
| `npm run db:seed` | Menjalankan seed berbasis data JSON dengan operasi upsert. | Dapat dijalankan ulang; akun Admin mengikuti variabel environment. |
| `npm run db:fresh` | Menerapkan migrasi, mengosongkan tabel, lalu seed ulang. | **Menghapus data database yang sedang digunakan.** Umumnya dipakai untuk menyiapkan database tes. |
| `npm run db:reset` | Menjalankan `prisma migrate reset` lalu seed. | **Destruktif:** database di-reset dan data dihapus. |

## 9. Pengujian aplikasi

### Pemeriksaan dasar

```bash
npx prisma validate
npm run lint
npm run build
```

### Pengujian API

Pengujian API membutuhkan server yang sudah berjalan dan database yang sudah disiapkan. Untuk hasil yang konsisten, gunakan database lokal khusus pengujian karena tes membuat dan mengubah beberapa data.

**Terminal 1 — siapkan database dan jalankan server:**

```bash
npm run db:fresh
npm run build
npm run start
```

**Terminal 2 — jalankan tes API:**

Windows PowerShell:

```powershell
$env:WAYO_URL = "http://localhost:3000"
npm run test:api
```

Linux/macOS:

```bash
WAYO_URL=http://localhost:3000 npm run test:api
```

Tes API memeriksa endpoint aplikasi dan, jika Google belum dikonfigurasi, memeriksa bahwa aplikasi menampilkan status Google belum dikonfigurasi. Tes juga membaca database untuk memeriksa data seed, kecuali variabel `WAYO_SKIP_DB=1` diatur.

### Pengujian end-to-end (Playwright)

Pasang browser Chromium satu kali:

```bash
npx playwright install chromium
```

Pastikan build berhasil terlebih dahulu, lalu jalankan:

```bash
npm run build
npm run test:e2e
```

Konfigurasi Playwright menjalankan server production pada port `3100` dan menjalankan `npm run db:fresh` sebelum tes. Karena itu, **tes E2E mengosongkan dan mengisi ulang database yang ditunjuk oleh `DATABASE_URL`**. Gunakan database tes terpisah dan jangan arahkan tes ini ke database yang berisi data penting.

Playwright menggunakan viewport 440 × 956 dan koordinat geolokasi simulasi untuk skenario E2E. Pengujian tersebut tidak menggantikan pengujian GPS pada perangkat nyata.

### Mengambil screenshot E2E

Pada Windows PowerShell:

```powershell
npm run build
$env:SHOTS = "1"
npx playwright test shots
Remove-Item Env:SHOTS
```

Pada Linux/macOS:

```bash
npm run build
SHOTS=1 npx playwright test shots
```

Hasil pengujian dan screenshot disimpan oleh Playwright pada direktori hasil tes yang dikonfigurasi.

## 10. Google Login (opsional)

Untuk mengaktifkan Google Login:

1. Buat OAuth 2.0 Client ID bertipe **Web application** di Google Cloud Console.
2. Tambahkan alamat callback berikut pada **Authorized redirect URIs**:

   ```text
   http://localhost:3000/api/auth/google/callback
   ```

3. Isi `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET` pada `.env`.
4. Isi `BASE_URL` dengan origin aplikasi yang digunakan.
5. Jika memakai callback khusus, isi `GOOGLE_REDIRECT_URI` dengan alamat callback yang sama persis seperti di Google Cloud Console.
6. Restart server Next.js.

Untuk deployment, gunakan domain HTTPS sebenarnya, misalnya `https://nama-domain.example/api/auth/google/callback`, dan daftarkan URL tersebut di Google Cloud Console. Jangan gunakan callback localhost untuk domain production.

## 11. Email reset password (opsional)

Untuk mengirim email reset password sungguhan, isi konfigurasi SMTP pada `.env`:

```env
SMTP_HOST="alamat-server-smtp"
SMTP_PORT="587"
SMTP_USER="akun-smtp"
SMTP_PASS="password-atau-app-password"
SMTP_SECURE="false"
MAIL_FROM="WAYO <alamat-pengirim@example.com>"
```

Gunakan nilai SMTP dari penyedia email yang dipakai. Port `465` umumnya menggunakan koneksi secure (`SMTP_SECURE="true"`), sedangkan port `587` umumnya menggunakan STARTTLS (`SMTP_SECURE="false"`). Sesuaikan dengan dokumentasi penyedia SMTP.

Jika `SMTP_HOST` kosong, email tidak dikirim. Dalam mode tersebut, tautan reset dicetak pada log terminal server.

## 12. File yang diunggah dan penyimpanan

Aplikasi menyimpan file unggahan pada direktori lokal proyek:

- Poster event: `data/uploads/` (JPG, PNG, atau WEBP, maksimal 2 MB).
- Surat pengajuan event: `data/proposals/` (PDF, maksimal 5 MB; akses dibatasi kepada panitia pemilik event dan Admin).

Direktori tersebut diabaikan oleh Git melalui `.gitignore`. File unggahan tidak ikut ter-commit ke repository.

**Penting untuk hosting:** gunakan server dengan filesystem persisten. Database SQLite, `data/uploads/`, dan `data/proposals/` harus berada pada penyimpanan yang tetap tersedia setelah restart atau redeploy. Hosting serverless dengan filesystem sementara tidak cocok tanpa penyesuaian penyimpanan.

## 13. Fitur dan peran pengguna

WAYO memiliki tiga peran:

- **Visitor (`VISITOR`)**: melihat dan mencari event, mendaftar event, mengakses peta, mencari tempat, menggunakan navigasi pejalan kaki, memindai QR Peta, serta mengelola profil dan notifikasi.
- **Organizer (`ORGANIZER`)**: mengelola event miliknya, poster, surat pengajuan, jadwal kegiatan, pin lokasi, dan QR Peta.
- **Admin (`ADMIN`)**: mengelola pengguna, seluruh event, master tempat, fasilitas, pendaftaran, pin, dan notifikasi melalui area `/admin`.

Seluruh aktor harus login untuk menggunakan area aplikasi yang dilindungi. Tidak ada mode Guest.

## 14. Troubleshooting

### `next is not recognized` atau `next: command not found`

Dependensi belum terpasang atau instalasi tidak selesai. Jalankan kembali:

```bash
npm ci
```

### Prisma Client belum ditemukan atau belum ter-generate

Jalankan:

```bash
npx prisma generate
```

### Database belum memiliki tabel

Pastikan `DATABASE_URL` di `.env` benar, lalu jalankan:

```bash
npm run db:fresh
```

Ingat bahwa perintah tersebut menghapus data pada database yang dipilih.

### Port 3000 sudah digunakan

Hentikan aplikasi lain yang menggunakan port tersebut atau jalankan Next.js pada port lain:

```bash
npm run dev -- -p 3001
```

Kemudian buka `http://localhost:3001` dan sesuaikan `BASE_URL` apabila fitur yang membangun tautan absolut memerlukannya.

### Google Login menampilkan pesan belum dikonfigurasi

Pastikan `GOOGLE_CLIENT_ID` dan `GOOGLE_CLIENT_SECRET` sudah diisi. Periksa juga kesesuaian redirect URI di Google Cloud Console dan `.env`, kemudian restart server.

### Email reset password tidak masuk

Periksa `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_SECURE`, dan `MAIL_FROM`. Jika SMTP belum dikonfigurasi, periksa log terminal server untuk tautan reset dalam mode pengembangan.

### File poster atau proposal tidak ditemukan setelah restart/redeploy

Pastikan direktori `data/uploads/` dan `data/proposals/` berada pada penyimpanan persisten dan proses server memiliki izin baca/tulis.

## 15. Catatan data dan penggunaan

- Data bawaan pada `data/*.json` merupakan data contoh/dummy, bukan data resmi Universitas Udayana.
- Peta menggunakan data OpenStreetMap. Ketersediaan dan kelengkapan jalur bergantung pada data peta dan layanan routing yang digunakan.
- Posisi pengguna bergantung pada izin browser dan kemampuan geolokasi perangkat. Pengujian dengan koordinat simulasi tidak membuktikan akurasi GPS lapangan.
- `WAYO_DEMO=1` hanya untuk kebutuhan demonstrasi/pengembangan. Gunakan `WAYO_DEMO=0` pada deployment publik.
- Jangan gunakan akun demo atau password bawaan untuk deployment publik.
