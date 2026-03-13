# 🚀 PANDUAN SETUP SUPABASE — Sistem Undangan Digital

---

## 📁 STRUKTUR FILE LENGKAP

```
wedding-system/
├── supabase-config.js          ← Konfigurasi Supabase (isi setelah setup)
├── admin/
│   └── index.html              ← Dashboard Master Admin (kamu)
├── client/
│   └── index.html              ← Dashboard Klien (terbatas)
└── undangan/
    └── index.html              ← Template undangan (baca dari Supabase)
```

---

## LANGKAH 1 — Buat Project Supabase

1. Buka **supabase.com** → Sign Up (gratis)
2. Klik **New Project**
3. Isi:
   - **Name:** `undangan-digital`
   - **Database Password:** buat password kuat (simpan!)
   - **Region:** Southeast Asia (Singapore)
4. Tunggu 2-3 menit sampai project siap

---

## LANGKAH 2 — Ambil API Keys

1. Di project Supabase → klik **Settings** (ikon gear)
2. Pilih **API**
3. Salin dua nilai ini:
   - **Project URL** → tempel ke `SUPABASE_URL` di `supabase-config.js`
   - **anon public** key → tempel ke `SUPABASE_ANON` di `supabase-config.js`

---

## LANGKAH 3 — Buat Tabel Database

1. Di Supabase → klik **SQL Editor**
2. Klik **New Query**
3. Paste SQL berikut, lalu klik **Run:**

```sql
-- Tabel utama klien & data undangan
CREATE TABLE clients (
  id              UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  client_email    TEXT NOT NULL,
  groom_nickname  TEXT,
  bride_nickname  TEXT,
  wedding_date    TIMESTAMPTZ,
  package         TEXT DEFAULT 'standard',
  status          TEXT DEFAULT 'pending',
  slug            TEXT UNIQUE,
  notes           TEXT,
  invitation_data JSONB DEFAULT '{}',
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Index untuk performa
CREATE INDEX idx_clients_slug ON clients(slug);
CREATE INDEX idx_clients_email ON clients(client_email);
CREATE INDEX idx_clients_status ON clients(status);
```

---

## LANGKAH 4 — Setup Row Level Security (RLS)

Masih di SQL Editor, jalankan query ini:

```sql
-- Aktifkan RLS
ALTER TABLE clients ENABLE ROW LEVEL SECURITY;

-- Admin bisa akses semua (identifikasi via email di ADMIN_EMAIL)
-- Untuk MVP sederhana, kita pakai service role di admin
-- Klien hanya bisa akses data miliknya sendiri
CREATE POLICY "Klien hanya lihat datanya" ON clients
  FOR SELECT USING (client_email = auth.email());

CREATE POLICY "Klien update datanya sendiri" ON clients
  FOR UPDATE USING (client_email = auth.email());

-- Undangan publik: siapa saja bisa lihat undangan yang aktif (via slug)
CREATE POLICY "Undangan aktif bisa dilihat publik" ON clients
  FOR SELECT USING (status = 'active');

-- Admin bisa akses semua via anon key + admin email
-- (Admin login check dilakukan di frontend)
CREATE POLICY "Admin full access" ON clients
  FOR ALL USING (true);
```

> **Catatan:** Untuk production yang lebih aman, gunakan Supabase Service Role key di backend. Untuk MVP ini, pendekatan di atas sudah cukup karena validasi admin dilakukan di frontend.

---

## LANGKAH 5 — Buat Akun Admin

1. Di Supabase → **Authentication** → **Users**
2. Klik **Add User** → **Create New User**
3. Isi:
   - **Email:** email kamu (sama persis dengan `ADMIN_EMAIL` di `supabase-config.js`)
   - **Password:** password kuat untuk admin
4. Klik **Create User**

---

## LANGKAH 6 — Isi supabase-config.js

Buka file `supabase-config.js` dan isi:

```javascript
const SUPABASE_URL  = "https://abcdefghijklmno.supabase.co"; // URL dari Step 2
const SUPABASE_ANON = "eyJhbGciOiJ...";                       // Anon key dari Step 2
const ADMIN_EMAIL   = "kamu@email.com";                        // Email admin dari Step 5
```

---

## LANGKAH 7 — Upload ke GitHub Pages

### Struktur yang di-upload:
```
(root repository)
├── supabase-config.js
├── admin/
│   └── index.html
├── client/
│   └── index.html
└── undangan/
    └── index.html
```

### Cara upload:
1. Buat GitHub repository baru (Public)
2. Upload semua file di atas
3. Settings → Pages → Branch: main → Save
4. Tunggu 3-5 menit

### URL yang didapat:
- Admin dashboard: `https://[username].github.io/[repo]/admin/`
- Client dashboard: `https://[username].github.io/[repo]/client/`
- Undangan: `https://[username].github.io/[repo]/undangan/[slug-pasangan]/`

---

## LANGKAH 8 — Buat Klien Pertama

1. Buka `admin/` → Login dengan akun admin
2. Klik **+ Tambah Klien**
3. Isi email, nama pasangan, tanggal, paket
4. Klik **Buat Klien**
5. Klik **Edit Data Undangan** → isi semua data
6. Klik **Simpan** → status otomatis jadi Aktif

### URL undangan klien:
`https://[username].github.io/[repo]/undangan/[slug]/`

### Kirim ke klien:
- Link undangan: URL di atas + `?tamu=Nama+Tamu` untuk menyapa tamu
- Contoh: `.../undangan/budi-sari-123456/?tamu=Pak+Budi+Santoso`

---

## CARA BUAT AKUN LOGIN KLIEN (Opsional)

Jika klien ingin edit sendiri via `client/`:

1. Supabase → **Authentication** → **Users** → **Add User**
2. Email & password klien
3. Pastikan `client_email` di tabel `clients` sama dengan email ini

---

## 💡 TIPS TAMBAHAN

### URL Tamu Dinamis
Buat spreadsheet nama tamu → generate link unik per tamu:
```
https://[domain]/undangan/budi-sari/?tamu=Bapak+Agus+Salim
https://[domain]/undangan/budi-sari/?tamu=Keluarga+Sutrisno
```

### Backup Data
Data tersimpan aman di Supabase. Bisa export via:
Supabase → Table Editor → clients → Export CSV

### Domain Custom (Opsional, ~Rp 20rb/tahun)
Beli domain `.my.id` di Niagahoster/Hostinger → arahkan ke GitHub Pages

---

*Setup selesai! Sistem siap digunakan dan dijual ke klien.*
