# Sistem Kasir & Pemesanan — Kafe Titik Temu Kopi

Aplikasi Point of Sales (POS) berbasis web untuk operasional **Dine-In** dan **Takeaway**, manajemen stok menu, pemantauan dashboard eksekutif, serta pelaporan penjualan berkala dengan ekspor CSV/Excel dan PDF.

---

## 1. Tech Stack & Arsitektur

- **Backend**: PHP 8.x Native (Clean Modular Architecture, PDO Prepared Statements, ACID Database Transactions)
- **Database**: MySQL / MariaDB (`db_pos_kafe`)
- **Frontend**: HTML5, CSS3, Bootstrap 5.3, Bootstrap Icons, Vanilla JavaScript, Chart.js 4.4
- **Keamanan**:
  - Hashing password menggunakan `password_hash(..., PASSWORD_BCRYPT)` & `password_verify()`
  - Proteksi Route / Middleware (`includes/auth_guard.php`) untuk memisahkan hak akses `admin` dan `kasir`
  - Proteksi CSRF token pada seluruh formulir `POST` & sanitasi XSS (`htmlspecialchars`)
  - Transaksi atomik (`beginTransaction` / `commit` / `rollBack`) dengan row-level locking (`SELECT ... FOR UPDATE`) untuk mencegah selisih stok

---

## 2. Kredensial Akun Uji Coba

| Role | Username | Password | Hak Akses Utama |
| :--- | :--- | :--- | :--- |
| **Administrator / Manager** | `admin` | `admin123` | Akses penuh: Dashboard, POS, Riwayat Semua Transaksi, CRUD Menu & Stok, Kategori, Kelola Pengguna, Laporan & Ekspor CSV/PDF |
| **Kasir / Waiter** | `kasiraulia` | `kasir123` | Akses operasional: Dashboard, Transaksi POS (Dine-In/Takeaway), Pembayaran, Cetak Struk, dan Riwayat Transaksi Pribadi |
| **Kasir / Waiter** | `kasirmiftah` | `kasir123` | Akun kasir kedua untuk pengujian pemisahan riwayat transaksi per kasir |

---

## 3. Langkah Instalasi Lokal (XAMPP / Laragon)

1. **Ekstrak Proyek**
   - Salin atau ekstrak folder `pos-kafe-titik-temu` ke dalam direktori web server Anda, misalnya:
     `C:\xampp\htdocs\pos-kafe-titik-temu`
2. **Jalankan Apache & MySQL**
   - Buka **XAMPP Control Panel**, lalu klik **Start** pada modul **Apache** dan **MySQL**.
3. **Impor Database & Seeder**
   - Buka `http://localhost/phpmyadmin` di browser.
   - Klik tab **Import** (atau buat database baru bernama `db_pos_kafe`), lalu pilih file `database.sql` yang ada di root folder proyek dan klik **Go / Import**.
   - File `database.sql` sudah otomatis membuat 5 tabel (`users`, `categories`, `products`, `orders`, `order_details`) beserta data akun uji coba, 12 menu kafe, peringatan stok rendah, dan 10 transaksi contoh.
4. **Konfigurasi Koneksi (Opsional)**
   - Buka `config/database.php` jika username/password MySQL lokal Anda berbeda dari default XAMPP (`root` tanpa password).
5. **Buka Aplikasi di Browser**
   - Akses `http://localhost/pos-kafe-titik-temu/` dan login menggunakan akun uji coba di atas.

---

## 4. Struktur Folder Proyek

```text
pos-kafe-titik-temu/
├── index.php                  # Entry point & pengarah sesi
├── database.sql               # Skema 5 tabel MySQL & data seeder lengkap
├── README.md                  # Dokumentasi instalasi & akun uji coba
├── config/
│   └── database.php           # Koneksi PDO MySQL, helper rupiah, & generator nomor faktur
├── includes/
│   ├── auth_guard.php         # Middleware autentikasi, RBAC (admin/kasir), & CSRF guard
│   ├── header.php             # Layout navigasi sidebar & topbar responsif
│   └── footer.php             # Penutup layout & script global
├── auth/
│   ├── login.php              # Halaman login dengan validasi & Bcrypt verification
│   └── logout.php             # Penghancuran sesi & cookie bersih
├── dashboard/
│   └── index.php              # 4 KPI Utama, Grafik Tren 7 Hari & Kategori, 8 Transaksi Terbaru
├── pos/
│   ├── index.php              # Form POS Master-Detail dinamis + Katalog Menu Cepat
│   ├── store.php              # Proses simpan transaksi ACID (validasi stok, PB1 10%, tunai)
│   ├── history.php            # Riwayat transaksi (terfilter per role) & pembatalan restore stok
│   └── receipt.php            # Struk nota siap cetak (Thermal 80mm, 58mm, & A4)
├── products/
│   └── index.php              # CRUD Master Menu, HPP, Harga Jual, & Restock cepat (Admin)
├── categories/
│   └── index.php              # CRUD Kategori Menu (Admin)
├── users/
│   └── index.php              # CRUD Akun Pengguna dengan Bcrypt (Admin)
├── reports/
│   ├── index.php              # Laporan Penjualan, Filter Dinamis, & 5 Summary Metrics (Admin)
│   ├── export_csv.php         # Ekspor laporan ke CSV / Microsoft Excel
│   └── export_pdf.php         # Ekspor / cetak laporan formal format PDF A4
└── assets/
    ├── css/style.css          # Tema antarmuka modern Kafe Titik Temu Kopi
    └── js/pos.js              # Logika kalkulasi POS dinamis & validasi client-side
```
