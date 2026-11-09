# Daftar Fitur PuSaKap (Perpustakaan Sang Cakap)

## 1. Autentikasi
- Login admin (email + password), dengan opsi "remember me".
- Logout (invalidate session + regenerate token).
- Redirect otomatis: user yang sudah login diarahkan ke dashboard kalau coba akses halaman login lagi; halaman non-login dilindungi middleware `auth`.

## 2. Dashboard (Live Monitoring)
- Statistik ringkas real-time: total siswa, total buku, buku tersedia, buku dipinjam, total denda terkumpul.
- Riwayat 10 transaksi terakhir (nama siswa, NIS, judul buku, jenis transaksi, waktu pinjam/kembali, besar denda).
- Sesi tap kartu aktif: menampilkan siswa yang baru tap RFID dan sedang menunggu tap buku (auto-expire 60 detik).
- Polling otomatis setiap 1 detik (AJAX) untuk update data tanpa reload — cache metrik 2 detik di server biar nggak query terus-terusan.

## 3. Manajemen Buku
- List buku dengan pagination (15/halaman).
- Tambah, edit, hapus data buku (RFID UID, judul, pengarang, status).
- Validasi RFID UID unik.
- Export data buku ke CSV (format Excel-friendly, BOM UTF-8).

## 4. Manajemen Siswa/Anggota
- List siswa dengan pagination (15/halaman).
- Tambah, edit, hapus data siswa (RFID UID, NIS, nama, kelas/jurusan).
- Validasi RFID UID & NIS unik.
- Export data siswa ke CSV.

## 5. Manajemen Transaksi (Peminjaman/Pengembalian)
- List transaksi dengan pagination + filter status (dipinjam / dikembalikan).
- Input transaksi manual (pilih siswa + buku yang available).
- Tandai buku dikembalikan secara manual.
- Hapus catatan transaksi (otomatis mengembalikan status buku jadi available kalau masih dipinjam).
- Hitung denda otomatis: gratis 7 hari kalender, lewat itu dihitung dari hari kerja (weekday) dikurangi 5 hari gratis kerja, dikali Rp500/hari telat.
- Export data transaksi ke CSV (bisa difilter per status juga).

## 6. Integrasi RFID/IoT (ESP32 — API)
- `POST /api/rfid/tap` — endpoint utama dari alat ESP32: scan 1 kartu untuk pilih siswa (buka sesi), scan kartu lagi untuk tutup sesi tanpa pinjam buku, scan buku untuk pinjam/kembalikan otomatis.
- `POST /api/rfid-scan` — endpoint legacy versi sebelumnya (logika serupa, masih dipertahankan untuk kompatibilitas).
- `GET /api/session-status` — cek status sesi siswa aktif (dipakai alat untuk tampilan LCD/OLED).
- `GET /api/last-scan` & `POST /api/clear-scan` — menyimpan UID kartu terakhir yang discan, dipakai untuk auto-isi form registrasi siswa/buku baru dari web (tap kartu fisik → UID otomatis muncul di form).
- Anti double-tap: cooldown 5 detik dalam sesi yang sama.
- Deteksi kartu tidak dikenal → respons "AKSES DITOLAK".
