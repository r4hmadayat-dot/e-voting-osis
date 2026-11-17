# E-VOTING OSIS

**Sistem Pemilihan Ketua dan Wakil Ketua OSIS Secara Online**
SMP Negeri 2 Soyo Jaya — Desa Lembah Sumara, Kecamatan Soyo Jaya, Kabupaten Morowali Utara

🔗 **Link pemilih:** https://r4hmadayat-dot.github.io/e-voting-osis/
🔐 **Link panitia:** https://r4hmadayat-dot.github.io/e-voting-osis/admin.html

Tampilan aplikasi di-hosting di **GitHub Pages**. Server berjalan di **Google Apps Script** dengan **Google Spreadsheet** sebagai database.

## Fitur Utama

- Login pemilih (siswa, guru, staf) dengan NIS/NISN/NIP + PIN
- Kartu pasangan calon lengkap dengan foto, visi, misi, dan program kerja
- Satu pemilih hanya bisa memilih satu kali (LockService + validasi server)
- Pilihan bersifat rahasia: identitas pemilih tidak disimpan bersama suaranya
- Dashboard panitia: beranda, data pemilih, kandidat, monitoring real-time, hasil, pengaturan, laporan
- Grafik hasil, cetak PDF (berita acara, rekapitulasi, partisipasi), dan unduh Excel
- Role SUPERADMIN dan PANITIA, password dan PIN di-hash, session bermasa berlaku, log aktivitas

## Isi Repository

| File | Fungsi |
|---|---|
| `index.html` | Tampilan aplikasi (pemilih + dashboard panitia) |
| `admin.html` | Jalan pintas ke dashboard panitia |
| `config.js` | Link server Apps Script (`GAS_URL`) |
| `Code.gs` | Kode server, disalin ke Google Apps Script |
| `appsscript.json` | Pengaturan server, disalin ke Google Apps Script |
| `Template_Import_Pemilih.xlsx` | Template import siswa, guru, dan staf |
| `PANDUAN.md` | Panduan lengkap pemasangan, penggunaan, update, dan uji keamanan |

## Pemasangan Singkat

1. **Server:** buat Google Spreadsheet → Ekstensi → Apps Script → tempel `Code.gs` dan `appsscript.json` → jalankan `setupDatabase` (catat password admin di Log eksekusi) → Terapkan sebagai Aplikasi web (*Saya*, *Siapa saja*) → salin URL `/exec`.
2. **Tampilan:** isi URL `/exec` ke `config.js` → Settings → Pages → Branch `main`, folder `/ (root)`.

Langkah lengkap ada di [PANDUAN.md](PANDUAN.md).

## Keamanan

- Tidak ada password di dalam kode. Password admin awal dibuat acak saat setup.
- Data pemilih, PIN, dan suara hanya tersimpan di Google Spreadsheet milik sekolah, tidak di GitHub.
- Jangan mengunggah file data asli pemilih, daftar PIN, atau laporan ke repository ini.

## Riwayat Versi

| Versi | Tanggal | Perubahan |
|---|---|---|
| v1.0 | 17 November 2026 | Rilis pertama: aplikasi di GitHub Pages, server di Google Apps Script |
