# E-VOTING OSIS — Panduan Lengkap

**Sistem Pemilihan Ketua dan Wakil Ketua OSIS Secara Online**
SMP Negeri 2 Soyo Jaya — Desa Lembah Sumara, Kecamatan Soyo Jaya, Kabupaten Morowali Utara

---

## Daftar Isi

1. Cara kerja aplikasi
2. Isi repository
3. Struktur database spreadsheet
4. Langkah A — Memasang server di Google Apps Script
5. Langkah B — Memasang aplikasi di GitHub Pages
6. Memasukkan data pemilih (siswa, guru, staf)
7. Memasukkan kandidat
8. Cara update aplikasi
9. Penggunaan aplikasi
10. Pengujian keamanan dan pencegahan voting ganda
11. Akun admin
12. Catatan teknis dan pemecahan masalah

---

## 1. Cara Kerja Aplikasi

```
 Siswa / Panitia (HP atau laptop)
            │  membuka link
            ▼
 GitHub Pages  ──  https://r4hmadayat-dot.github.io/e-voting-osis/
 (tampilan: index.html + config.js)
            │  mengirim data
            ▼
 Google Apps Script  ──  Code.gs (server / "mesin")
            │  membaca & menyimpan
            ▼
 Google Spreadsheet  ──  database (data siswa, kandidat, suara)
```

- **Link untuk siswa dan panitia selalu link GitHub**, dan tidak berubah walaupun ada update.
- Google Apps Script hanya berjalan di belakang. Siswa tidak pernah melihat atau membuka spreadsheet.

---

## 2. Isi Repository

| File | Fungsi | Dipasang di |
|---|---|---|
| `index.html` | Seluruh tampilan aplikasi (halaman pemilih + dashboard panitia) | GitHub |
| `admin.html` | Jalan pintas ke dashboard panitia | GitHub |
| `config.js` | **Link server** (`GAS_URL`). Satu-satunya file yang perlu diisi | GitHub |
| `Code.gs` | Kode server: semua logika, keamanan, dan database | **Disalin ke Apps Script** |
| `appsscript.json` | Pengaturan server: zona waktu WITA, izin | **Disalin ke Apps Script** |
| `Template_Import_Pemilih.xlsx` | Template import siswa, guru, dan staf | — |
| `PANDUAN.md`, `README.md` | Dokumentasi | GitHub |

---

## 3. Struktur Database Spreadsheet

Semua sheet dibuat otomatis oleh `setupDatabase()`. Semua kolom diformat sebagai **teks** agar NIS/NISN yang diawali angka 0 tidak berubah.

### PENGATURAN (1 baris)
`ID | NAMA_SEKOLAH | TAHUN_PEMILIHAN | JUDUL_PEMILIHAN | TANGGAL_MULAI | TANGGAL_SELESAI | STATUS_VOTING | LOGO_SEKOLAH | FOTO_BANNER | PESAN_SELESAI`

- `STATUS_VOTING`: `BELUM DIBUKA` / `DIBUKA` / `DITUTUP`
- Format tanggal: `yyyy-MM-dd HH:mm` (WITA), contoh `2026-11-20 07:30`

### DATA_SISWA
`ID_SISWA | NIS | NISN | NAMA_SISWA | KELAS | PASSWORD/PIN | STATUS_AKUN | STATUS_MEMILIH | WAKTU_MEMILIH`

- `PASSWORD/PIN` disimpan sebagai **hash** dengan format `v1$iterasi$salt$hash`. PIN asli tidak pernah disimpan.
- `STATUS_AKUN`: `AKTIF` / `NONAKTIF`
- `STATUS_MEMILIH`: `BELUM MEMILIH` / `SUDAH MEMILIH`

### KANDIDAT
`ID_KANDIDAT | NOMOR_URUT | NAMA_KETUA | NAMA_WAKIL | KELAS | FOTO_KANDIDAT | VISI | MISI | PROGRAM_KERJA | STATUS_KANDIDAT`

- `MISI` dan `PROGRAM_KERJA` ditulis satu poin per baris.

### HASIL_VOTING
`ID_SUARA | ID_KANDIDAT | NOMOR_URUT | WAKTU_VOTING | TAHUN_PEMILIHAN`

- **Tidak ada** NIS, NISN, nama, atau PIN pemilih.
- `WAKTU_VOTING` hanya berisi **tanggal** (tanpa jam), dan setiap suara disisipkan pada **posisi baris acak**. Akibatnya urutan dan waktu suara tidak dapat dicocokkan dengan `WAKTU_MEMILIH` di DATA_SISWA, sehingga kerahasiaan pilihan tetap terjaga walaupun seseorang membuka spreadsheet.

### LOG_AKTIVITAS
`ID_LOG | WAKTU | USER | AKTIVITAS | KETERANGAN`

- Log **tidak pernah** mencatat kandidat yang dipilih siswa.

### ADMIN
`ID_ADMIN | USERNAME | PASSWORD_HASH | NAMA_ADMIN | ROLE | STATUS`

- `ROLE`: `SUPERADMIN` (akses penuh) atau `PANITIA` (data siswa, monitoring, hasil setelah ditutup, laporan).

---

## 4. Langkah A — Memasang Server di Google Apps Script

Dilakukan **satu kali saja**.

1. Buka Google Drive, buat **Google Spreadsheet** baru, misalnya `DATABASE E-VOTING OSIS 2026`.
2. Di spreadsheet, klik **Ekstensi → Apps Script**.
3. Klik **⚙ Setelan Project**, lalu centang **"Tampilkan file manifes 'appsscript.json' di editor"**.
4. Kembali ke **Editor**:
   - Buka `appsscript.json`, hapus isinya, lalu tempel isi `appsscript.json` dari repository.
   - Buka `Code.gs`, hapus isinya, lalu tempel isi `Code.gs` dari repository.
   - Klik **💾 Simpan**. Hanya 2 file ini yang dibutuhkan, tidak perlu file HTML.
5. Di bilah atas, pilih fungsi **`setupDatabase`**, lalu klik **▶ Jalankan**.
   - Saat diminta izin: **Tinjau izin → pilih akun → Lanjutan → Buka (tidak aman) → Izinkan**. Peringatan ini normal untuk skrip buatan sendiri.
6. **CATAT PASSWORD ADMIN.** Buka **Log eksekusi** di bagian bawah editor. Akan muncul:
   ```
   AKUN ADMIN DIBUAT — CATAT SEKARANG!
   Username: admin
   Password: Osis-XXXX-XXXX
   ```
   Password ini dibuat acak dan **hanya ditampilkan sekali**. Setelah login pertama, aplikasi akan meminta Anda menggantinya.
7. Deploy server:
   - Klik **Terapkan → Deployment baru**, klik ⚙ di samping "Pilih jenis", lalu pilih **Aplikasi web**.
   - **Jalankan sebagai:** **Saya**
   - **Yang memiliki akses:** **Siapa saja**
   - Klik **Terapkan**, lalu **salin URL aplikasi web** (berakhiran `/exec`).
8. Uji: buka URL `/exec` tadi di browser. Harus muncul tulisan *"API E-VOTING OSIS aktif"*.

> Spreadsheet **tidak perlu dan tidak boleh** dibagikan kepada siswa.

---

## 5. Langkah B — Memasang Aplikasi di GitHub Pages

1. Di repository `e-voting-osis`, buka file **`config.js`** dan klik ✏️ (Edit).
2. Ganti tulisan `https://script.google.com/macros/s/GANTI_DENGAN_ID_DEPLOYMENT/exec` dengan **URL `/exec`** dari Langkah A nomor 7. Biarkan tanda kutipnya.
3. Klik **Commit changes**.
4. Buka **Settings → Pages**. Pada **Source** pilih **Deploy from a branch**, lalu **Branch: main**, folder **/ (root)**. Klik **Save**.
   - GitHub Pages gratis hanya untuk repository **Public**. Jika repository masih Private, ubah di **Settings → General → Danger Zone → Change visibility → Public**.
   - Repository aman dibuat publik: tidak ada password di kode, dan data tersimpan di spreadsheet milik Anda.
5. Tunggu 1–2 menit. Link aplikasi:
   - **Pemilih:** `https://r4hmadayat-dot.github.io/e-voting-osis/`
   - **Panitia:** `https://r4hmadayat-dot.github.io/e-voting-osis/admin.html`
6. Buka link panitia, login dengan `admin` + password dari Log eksekusi, lalu ganti password.

---

## 6. Memasukkan Data Pemilih (Siswa, Guru, Staf)

### Cara A — Import Excel/CSV (disarankan)
1. Gunakan file **`Template_Import_Pemilih.xlsx`** dari repository (atau dashboard: **Data Pemilih → Import Excel/CSV → Unduh Template Excel**).
2. Isi sheet `DATA` dengan kolom berikut. Siswa, guru, dan staf diisi dalam satu sheet. Guru: kolom NIS diisi NIP dan KELAS diisi `GURU`. Staf: NIS diisi NIP atau kode unik (misal `GTK-001`) dan KELAS diisi `STAF`.

   | NIS | NISN | NAMA_SISWA | KELAS | PIN |
   |---|---|---|---|---|
   | 2324001 | 0112345678 | Ahmad Fauzi | VII A | *(kosong)* |

   - **NIS** wajib diisi dan harus unik. **NISN** opsional. **PIN** opsional (4–8 digit). PIN yang kosong dibuat otomatis (6 digit).
   - Kolom NIS, NISN, dan PIN di template sudah berformat Teks. Jika memakai file sendiri, format kolom tersebut sebagai **Teks** agar angka 0 di depan NISN tidak hilang.
3. Pilih file, tentukan perlakuan untuk NIS yang sudah ada (*Lewati* atau *Perbarui*), lalu klik **Import**.
4. Setelah selesai, muncul daftar **PIN** untuk siswa baru. **PIN hanya ditampilkan sekali.** Langsung klik **Cetak Kartu PIN** dan/atau **Unduh Excel**.

### Cara B — Tambah satu per satu
Buka **Data Pemilih → Tambah Pemilih**, lalu pilih **Jenis Pemilih**:
- **Siswa:** isi NIS, NISN (opsional), nama, dan kelas.
- **Guru:** isi NIP dan nama. Kolom kelas otomatis diisi `GURU`. Jika tidak punya NIP, isi kode unik seperti `GURU-001`.
- **Staf / Tenaga Kependidikan:** isi NIP (atau kode unik seperti `STAF-001`) dan nama. Kolom kelas otomatis diisi `STAF`.

PIN baru ditampilkan sekali setelah data disimpan. Guru dan staf login di halaman yang sama dengan siswa, memakai **NIP + PIN**.

### Cara C — Ketik langsung di spreadsheet
Data juga dapat diketik di sheet DATA_SISWA. `PASSWORD/PIN` boleh diisi PIN biasa, misalnya `482913`. Isi `STATUS_AKUN` dengan `AKTIF` dan `STATUS_MEMILIH` dengan `BELUM MEMILIH`, serta beri setiap siswa `ID_SISWA` yang unik (misalnya `SIS-001`). Saat siswa pertama kali login, PIN tersebut otomatis diubah menjadi hash.

### Mencetak ulang kartu PIN
Karena PIN disimpan sebagai hash, PIN lama tidak bisa dilihat lagi. Untuk mencetak kartu, buka **Data Pemilih → Kartu PIN per Kelas** (khusus SUPERADMIN). Pilih kelas, dan sistem akan **membuat PIN baru** untuk siswa yang belum memilih, lalu menampilkan kartunya untuk dicetak. Untuk satu siswa yang lupa PIN, gunakan tombol **🔑 Reset PIN** di baris siswa tersebut.

> Ketentuan "PIN siswa tidak boleh ditampilkan dalam dashboard" dipenuhi: tabel siswa tidak pernah memuat PIN. PIN hanya muncul sekali, tepat saat dibuat, agar dapat dibagikan kepada siswa.

---

## 7. Memasukkan Kandidat

1. Login sebagai SUPERADMIN, lalu buka **Data Kandidat → Tambah Kandidat**.
2. Isi **nomor urut**, **nama calon ketua**, **nama calon wakil ketua**, **kelas**, **visi**, **misi** (satu poin per baris), dan **program kerja** (satu poin per baris).
3. **Foto:** pilih file JPG/PNG. Foto otomatis diperkecil, diunggah ke folder Google Drive **"E-VOTING OSIS - Media"**, lalu dibagikan dengan akses lihat. Pilihan lainnya: tempel tautan Google Drive atau URL gambar.
   - Rasio foto yang ideal adalah **4:3**, dengan wajah pasangan calon di bagian atas tengah.
4. Kandidat dapat **dinonaktifkan** (tidak tampil bagi siswa) atau **dihapus**. Penghapusan hanya bisa dilakukan sebelum voting dibuka.
5. Tombol **Detail** menampilkan kartu kandidat persis seperti yang dilihat siswa.

Pengaman lainnya: nomor urut tidak dapat diubah jika kandidat sudah menerima suara, dan kandidat tidak dapat dinonaktifkan selama voting berlangsung.

---

## 8. Cara Update Aplikasi

| Yang diubah | Yang dilakukan | Link berubah? |
|---|---|---|
| **Tampilan** (`index.html`) | Unggah file baru ke GitHub: **Add file → Upload files → Commit changes** | Tidak |
| **Server** (`Code.gs`) | Tempel kode baru di Apps Script, lalu **Terapkan → Kelola deployment → ✏️ Edit → Versi: Versi baru → Terapkan** | Tidak |
| Keduanya | Lakukan kedua langkah di atas | Tidak |

Hal yang perlu diingat:
- Untuk update server, **selalu** pakai **Kelola deployment → Edit**. **Jangan** pakai *Deployment baru*, karena itu membuat URL `/exec` baru.
- Jika URL `/exec` terpaksa berubah (misalnya pindah akun Google), cukup ubah `GAS_URL` di `config.js`. Link GitHub untuk siswa tetap sama.
- Jalankan ulang `setupDatabase` hanya jika catatan update memintanya.
- **Jangan update saat voting sedang berlangsung.** Setelah update, buka aplikasi di jendela Incognito untuk memastikan versi baru sudah tampil.
- Setiap update, tambahkan satu baris di tabel **Riwayat Versi** pada `README.md`.

---

## 9. Penggunaan Aplikasi

### Persiapan (H-7 s.d. H-1)
1. **Pengaturan:** isi nama sekolah, judul, tahun, tanggal/waktu mulai dan selesai, pesan selesai, **logo sekolah**, **logo OSIS**, dan banner. Klik **Simpan**. Kedua logo tampil berdampingan di halaman login, kartu pemilih, dan kop semua laporan. Gunakan file PNG berlatar transparan agar rapi.
2. **Data Pemilih:** import seluruh siswa, guru, dan staf, lalu cetak dan bagikan kartu PIN (bisa dimasukkan ke amplop per kelas).
3. **Data Kandidat:** masukkan semua pasangan calon beserta foto.
4. **Simulasi:** buka voting, lalu minta 3–5 siswa/guru mencoba memilih. Tutup voting, periksa hasilnya, kemudian jalankan **Data Pemilih → Reset Status Memilih** (ketik `RESET`). Langkah ini menghapus suara simulasi dan mengembalikan semua siswa ke status BELUM MEMILIH.
   - Jika PIN peserta simulasi sudah diketahui orang lain, reset PIN mereka.

### Hari pemilihan
1. SUPERADMIN membuka **Beranda → Buka Voting**.
   - Siswa hanya dapat memilih jika status **DIBUKA** **dan** waktu saat itu berada dalam jadwal (jika jadwal diisi).
2. Siswa membuka link aplikasi (GitHub Pages), memasukkan NIS/NISN + PIN, lalu tekan **MASUK**.
3. Siswa membaca kartu kandidat → **LIHAT DETAIL** → **PILIH PASANGAN INI** → konfirmasi **YA, KIRIM SUARA**.
4. Halaman terima kasih tampil dan otomatis kembali ke awal setelah 15 detik. Ini aman untuk perangkat bersama di bilik suara.
5. Panitia memantau melalui **Monitoring Voting**: jumlah suara masuk, partisipasi, siswa yang belum memilih, dan grafik per kelas, yang diperbarui otomatis setiap 15 detik. Perolehan suara **tidak** ditampilkan selama voting berlangsung.
6. Setelah selesai, SUPERADMIN membuka **Beranda → Tutup Voting**.

### Daftar hadir dan pemanggilan pemilih
Ada dua daftar hadir yang memakai **nomor urut yang sama** (per kelas, urut nama):

| Daftar hadir | Dibuat | Isinya | Dipakai untuk |
|---|---|---|---|
| **Manual (Pemanggilan)** | Dicetak sebelum voting: **Laporan → Daftar Hadir Manual** | No. urut, NIS/NIP, nama, kotak Hadir, tanda tangan | Panitia memanggil pemilih sesuai nomor urut; pemilih mencentang Hadir dan tanda tangan |
| **Otomatis (Aplikasi)** | Terisi sendiri saat pemilih mengirim suara | No. urut, NIS/NIP, nama, status, jam memilih | Dipantau di menu **Daftar Hadir**; dicetak lewat **Laporan → Daftar Hadir Otomatis** untuk arsip |

Saat pemungutan suara:
1. Panitia pemanggil memegang Daftar Hadir Manual kelas yang sedang memilih.
2. Panitia lain membuka menu **Daftar Hadir** di dashboard dan memilih kelas yang sama. Kotak **Panggil berikutnya** menunjukkan nomor urut dan nama pemilih pertama yang belum memilih. Daftar ini diperbarui otomatis setiap 10 detik.
3. Setelah pemilih selesai mengirim suara, statusnya berubah menjadi **SUDAH MEMILIH** beserta jam memilih, dan kotak Panggil berikutnya pindah ke pemilih selanjutnya.
4. Setelah semua kelas selesai, cocokkan tanda tangan di daftar manual dengan cetakan Daftar Hadir Otomatis sebagai lampiran berita acara.

### Membuat SK Pengurus OSIS
Menu **SK OSIS** (khusus SUPERADMIN) membuat Surat Keputusan Kepala Sekolah tentang Pengangkatan Pengurus OSIS, lengkap dengan lampiran susunan pengurus.

1. **Data Surat Keputusan:** isi nomor SK, masa bakti, tempat dan tanggal ditetapkan, nama dan NIP **Kepala Sekolah**, nama dan NIP **Pembina OSIS**, serta ukuran kertas (A4 atau F4).
2. **Kop surat** (bisa dibuka-tutup): baris pemerintah daerah, dinas, nama sekolah, alamat, dan kontak. Logo sekolah dan logo OSIS diambil dari menu Pengaturan.
3. **Isi SK** (bisa dibuka-tutup): Menimbang, Mengingat, Memperhatikan, Tugas pengurus, dan Tembusan. Satu butir per baris. Dasar hukum bawaan:
    - UU Nomor 20 Tahun 2003 tentang Sistem Pendidikan Nasional
    - UU Nomor 23 Tahun 2002 tentang Perlindungan Anak, terakhir diubah dengan UU Nomor 17 Tahun 2016
    - PP Nomor 57 Tahun 2021 tentang Standar Nasional Pendidikan, diubah dengan PP Nomor 4 Tahun 2022
    - Permendiknas Nomor 39 Tahun 2008 tentang Pembinaan Kesiswaan
    - Permendikdasmen Nomor 6 Tahun 2026 tentang Budaya Sekolah Aman dan Nyaman

    Sesuaikan bila Dinas Pendidikan menetapkan format atau dasar hukum lain.
4. **Susunan Pengurus:** struktur bawaan berisi Pengurus Inti (Ketua, Wakil, Sekretaris I–II, Bendahara I–II) dan 10 Seksi Bidang sesuai Permendiknas 39/2008.
    - Klik **Isi Ketua & Wakil dari hasil pemilihan** (tersedia setelah voting ditutup).
    - Untuk jabatan lain, ketik nama siswa lalu pilih dari daftar yang muncul. Kelas terisi otomatis.
    - Nama kelompok dan jabatan bisa diubah. Tombol **Baris** menambah anggota; **Tambah kelompok** menambah seksi baru.
    - Pembina (Kepala Sekolah sebagai Penanggung Jawab, dan Pembina OSIS) otomatis masuk ke bagian A lampiran.
5. Klik **Simpan**. Data tersimpan di sheet `PENGURUS_OSIS` dan bisa dibuka lagi kapan saja.
6. Klik **Unduh Word (.docx)** untuk file yang masih bisa diedit di Microsoft Word, atau **Cetak / Simpan PDF** lalu pilih *Simpan sebagai PDF*.

Dokumen berisi dua bagian: halaman SK (ditandatangani Pembina OSIS dan Kepala Sekolah) dan Lampiran susunan pengurus (ditandatangani keduanya juga).

### Berita Acara Serah Terima Jabatan dan Susunan Acara Pelantikan
Di menu **SK OSIS** ada tiga tab: **SK Pengurus**, **Serah Terima Jabatan**, dan **Susunan Acara**. Tombol *Unduh Word* dan *Cetak / PDF* di bawah halaman selalu membuat dokumen dari tab yang sedang dibuka. Semua isian tersimpan bersama data SK lewat tombol **Simpan**.

**Serah Terima Jabatan** (Berita Acara Serah Terima Jabatan Ketua OSIS):
1. Isi tanggal, pukul, dan tempat serah terima. Masa bakti pengurus lama terisi otomatis (satu tahun sebelum masa bakti baru) dan bisa diubah.
2. **Pihak Pertama** (Ketua OSIS lama): ketik nama lalu pilih dari daftar siswa. Kelas terisi otomatis dan bisa diubah.
3. **Pihak Kedua** (Ketua OSIS baru) diambil otomatis dari Susunan Pengurus, jabatan "Ketua OSIS".
4. Isi **Saksi 1** (bawaan: Wakil Kepala Sekolah Bidang Kesiswaan). Saksi 2 adalah Pembina OSIS, dan *Mengetahui* adalah Kepala Sekolah; keduanya diambil dari tab SK Pengurus.

**Susunan Acara** (upacara pelantikan):
1. Isi nama acara, tanggal, pukul, dan tempat.
2. **Butir Acara:** satu butir per baris; nomor ditambahkan otomatis. Awali baris dengan `*` untuk membuat sub-butir. Butir bawaan mencakup: menyiapkan peserta, penghormatan, laporan pemimpin upacara, doa pembukaan, pembacaan SK, serah terima jabatan, penandatanganan berita acara, janji/ikrar pengurus, pelantikan oleh Kepala Sekolah, sambutan Ketua OSIS terpilih, amanat Pembina Upacara, doa penutup, dan pembubaran.
3. Tombol **Kembalikan butir acara bawaan** mengisi ulang daftar bawaan.

### Setelah voting ditutup
- **Hasil Voting:** jumlah dan persentase suara setiap kandidat, diagram batang, diagram lingkaran, total suara sah, serta partisipasi.
- **Laporan:** isi dulu data penandatangan, lalu cetak dokumen yang dibutuhkan:
  - Cetak Hasil Voting (PDF)
  - Berita Acara Pemilihan
  - Rekapitulasi Suara
  - Hasil Perolehan Setiap Kandidat
  - Daftar Hadir Manual (Pemanggilan) dan Daftar Hadir Otomatis (Aplikasi). Lihat bagian *Daftar hadir dan pemanggilan pemilih* di bawah
  - Daftar Partisipasi Pemilih (sudah tersedia sejak voting berlangsung)
  - Download Excel
- Untuk menyimpan sebagai PDF, pilih **"Simpan sebagai PDF"** di jendela cetak.

### Hak akses per role

| Menu | SUPERADMIN | PANITIA |
|---|:-:|:-:|
| Beranda, Monitoring | ✔ | ✔ |
| Data Pemilih (tambah, edit, import, reset PIN) | ✔ | ✔ |
| Hapus siswa, Kartu PIN massal, Reset Status Memilih | ✔ | ✘ |
| Data Kandidat (kelola) | ✔ | lihat saja |
| Buka/Tutup Voting, Pengaturan | ✔ | ✘ |
| Hasil Voting, Laporan | ✔ (setelah ditutup) | ✔ (setelah ditutup) |
| Akun & Log | ✔ | ✘ |

---

## 10. Pengujian Keamanan dan Pencegahan Voting Ganda

Lakukan pengujian berikut **saat simulasi**. Setelah selesai, jalankan Reset Status Memilih.

| # | Skenario | Cara menguji | Hasil yang benar |
|---|---|---|---|
| 1 | Memilih dua kali | Siswa A memilih, lalu login lagi | Pesan "Hak suara Anda sudah digunakan pada …" |
| 2 | Dua perangkat bersamaan | Login Siswa A di HP 1 dan HP 2 (keduanya sampai halaman kandidat). Kirim suara di HP 1, lalu di HP 2 | HP 2 ditolak: "Hak suara Anda sudah digunakan". Jumlah suara hanya bertambah 1 |
| 3 | Klik ganda tombol kirim | Tekan **YA, KIRIM SUARA** berkali-kali dengan cepat | Tombol terkunci setelah klik pertama. Hanya 1 suara tersimpan |
| 4 | Token dipakai ulang | Setelah memilih, tekan tombol Back di browser lalu coba kirim lagi | Ditolak: sesi berakhir |
| 5 | Voting belum dibuka | Status BELUM DIBUKA, siswa mencoba login | Muncul informasi jadwal, siswa tidak bisa masuk |
| 6 | Voting ditutup / lewat jadwal | Tutup voting atau tunggu waktu selesai | Login dan pengiriman suara ditolak di server |
| 7 | PIN salah berulang | Masukkan PIN salah 5 kali | Akun terkunci 10 menit. Panitia dapat membuka kuncinya dengan Reset PIN |
| 8 | Hasil sementara | Buka menu Hasil Voting saat voting DIBUKA | Terkunci: "Hasil hanya dapat dibuka setelah voting ditutup" |
| 9 | Kerahasiaan pilihan | Buka sheet HASIL_VOTING | Tidak ada NIS/NISN/nama/PIN, waktu hanya tanggal, urutan baris acak |
| 10 | PIN tidak terlihat | Buka sheet DATA_SISWA dan tabel Data Pemilih | Spreadsheet hanya berisi hash `v1$…`, tabel dashboard tanpa kolom PIN |
| 11 | Pembatasan role | Login sebagai PANITIA, coba buka Pengaturan | Menu tidak tersedia. Pemanggilan langsung ke server ditolak "Akses ditolak" |
| 12 | Sesi kedaluwarsa | Diamkan halaman siswa > 15 menit, lalu pilih | "Sesi berakhir, silakan masuk kembali" |
| 13 | Manipulasi browser | Buka DevTools → Console, jalankan `api('submitVote','token-palsu','KAN-xxx')` | Ditolak (SESSION_EXPIRED). Identitas pemilih selalu diambil dari sesi server |
| 14 | Fungsi setup dari luar | Dari Console jalankan `api('setupAdmin')` | Ditolak: "Fungsi tidak dikenal". Server hanya menerima fungsi yang ada di daftar `API_FUNCTIONS` |
| 15 | Integritas data | Setelah simulasi, lihat kartu Ringkasan di Beranda | "Integritas data baik": jumlah suara = jumlah siswa sudah memilih |

### Mekanisme yang diterapkan
- **LockService (ScriptLock):** semua penulisan suara dan data siswa berjalan bergiliran. Di dalam lock, status siswa **dibaca ulang dari spreadsheet** sebelum suara disimpan.
- **Validasi server penuh:** browser tidak pernah mengirim identitas siswa. Server memakai token sesi (64 karakter acak, disimpan di CacheService) untuk menentukan siapa yang memilih.
- **Token sekali pakai:** sesi siswa langsung dihapus setelah suara terkirim.
- **Rollback:** jika penandaan "SUDAH MEMILIH" gagal, suara yang baru ditulis dibatalkan.
- **Daftar fungsi API:** server hanya menjalankan fungsi yang terdaftar di `API_FUNCTIONS` (`Code.gs`). Fungsi setup dan fungsi internal tidak bisa dipanggil dari internet.
- **Hash:** password admin (500 iterasi SHA-256) dan PIN siswa memakai *salt* per akun dan *pepper* rahasia di Script Properties (tidak ada di spreadsheet).
- **Session bermasa berlaku:** siswa 15 menit, admin 120 menit, dan diperpanjang otomatis selama aktif. Admin yang dinonaktifkan langsung terputus.
- **Pembatasan percobaan login:** 5 kali gagal → dikunci 10 menit (siswa dan admin).
- **Proteksi sheet:** setiap sheet dilindungi sehingga hanya pemilik yang dapat mengedit.
- **Pencegahan formula injection** dan **escape HTML** pada semua data yang ditampilkan.

---

## 11. Akun Admin

### Login pertama
Password admin awal dibuat **acak** oleh `setupDatabase` dan ditampilkan sekali di **Log eksekusi** (lihat Langkah A nomor 6). Password tidak pernah ditulis di kode, sehingga kode aman disimpan di GitHub publik.

### Menambah admin/panitia lain
1. Login sebagai SUPERADMIN, buka **Akun & Log → Tambah Admin**.
2. Isi username, nama, role (`PANITIA` atau `SUPERADMIN`), dan password sementara (minimal 8 karakter, huruf dan angka).
3. Admin baru akan diminta mengganti password saat login pertama.

### Lupa password admin utama
Di editor Apps Script, pilih fungsi **`resetPasswordAdminUtama`**, lalu **▶ Jalankan**. Password acak baru muncul di **Log eksekusi**.

---

## 12. Catatan Teknis dan Pemecahan Masalah

**Kapasitas.** Setiap pengiriman suara memegang lock selama ±1–2 detik. Untuk sekolah dengan ratusan siswa, pengaturan bergiliran per kelas (misalnya 30–40 siswa sekaligus) berjalan lancar. Jika server sibuk, siswa mendapat pesan "Server sedang sibuk… coba lagi" tanpa kehilangan hak suara.

**Foto tidak tampil.**
- Pastikan file di Google Drive diatur *"Siapa saja yang memiliki link dapat melihat"*.
- Akun Google Workspace sekolah kadang melarang berbagi publik. Jika terjadi, gunakan akun Gmail pribadi sebagai pemilik aplikasi, atau tempel URL gambar dari tempat lain.

**Halaman muncul tetapi tidak bisa login / "Tidak dapat terhubung ke server".**
- Pastikan `GAS_URL` di `config.js` sudah benar dan berakhiran `/exec`.
- Buka URL `/exec` itu di browser. Harus muncul tulisan *"API E-VOTING OSIS aktif"*. Jika muncul halaman login Google atau error, deployment belum diatur ke **Akses: Siapa saja**.

**"Respons server tidak valid".** Deployment Apps Script belum benar. Ulangi langkah 5 dan pastikan *Jalankan sebagai: Saya* dan *Akses: Siapa saja*.

**Grafik tidak muncul.** Grafik dimuat dari internet (Chart.js). Jika koneksi lambat, data tetap tampil tanpa grafik. Muat ulang halaman setelah koneksi membaik.

**Zona waktu.** `appsscript.json` memakai `Asia/Makassar` (WITA). Jika diubah, sesuaikan juga `APP.TZ_LABEL` di `Code.gs`.

**Mengubah konfigurasi.** Semua angka penting ada di objek `APP` (`Code.gs`): durasi sesi, batas percobaan login, lama blokir, dan iterasi hash.

**Mengganti tahun pemilihan.** Ubah tahun di Pengaturan. Suara disimpan per `TAHUN_PEMILIHAN`, sehingga data tahun lalu tetap tersimpan dan tidak tercampur. Status memilih siswa perlu direset dengan **Reset Status Memilih** sebelum pemilihan baru.

**"Sheet belum tersedia".** Jalankan ulang `setupDatabase()` dari editor Apps Script.

**Mengubah data langsung di spreadsheet.** Hindari mengubah HASIL_VOTING atau STATUS_MEMILIH secara manual. Kartu "Integritas data" di Beranda akan memberi peringatan jika jumlah suara tidak cocok dengan jumlah siswa yang sudah memilih.
