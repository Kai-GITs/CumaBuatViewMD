# Rencana Milestone — Yomu

**Proyek:** Yomu — Aplikasi Pembelajaran Literasi Informasi  
**Tech Stack:** Java · Spring Boot · Thymeleaf  
**Anggota:** Hasan (Auth) · Alya (Bacaan & Kuis) · Naufal (Achievements) · Kalfin (Interaksi & Liga)

---

## Ringkasan Modul

| # | Modul | PIC | Tanggung Jawab Utama |
|---|-------|-----|----------------------|
| 1 | Autentikasi | Hasan | Identitas pengguna, sesi, role, dan akses fitur |
| 2 | Bacaan & Kuis | Alya | Konten teks, alur kuis, dan evaluasi jawaban |
| 3 | Achievements | Naufal | Sistem penghargaan, daily mission, dan gamifikasi profil |
| 4 | Interaksi Sosial & Liga | Kalfin | Clan, leaderboard, sistem tier, buff/debuff |

---

## Prinsip Integrasi

- **Auth adalah pintu masuk.** Seluruh modul menggunakan `userId` dari Auth sebagai referensi data pengguna.
- **Bacaan & Kuis adalah sumber data aktivitas.** Setiap kuis yang diselesaikan menghasilkan event yang dikonsumsi oleh Achievements dan Liga.
- **Achievements dan Liga tidak menyimpan data mentah kuis.** Mereka hanya menerima notifikasi atau meminta statistik ringkasan dari Bacaan & Kuis.
- **Liga dapat mengirim sinyal balik ke Achievements** saat Clan berhasil promosi ke tier tertinggi (untuk membuka achievement khusus).
- Semua komunikasi antar-modul melalui kontrak API yang terdefinisi — tidak ada pemanggilan langsung antar komponen.

---

---

## Milestone 0 — Persiapan

> **Tujuan:** Semua anggota siap mulai coding. Repo terkonfigurasi, skema database disepakati, dan kontrak API awal terdokumentasi.

### Deliverables

**Setup Teknis (semua anggota, lead: Hasan)**
- Buat repository GitHub dan inisialisasi project Spring Boot dengan Thymeleaf
- Konfigurasi struktur package per modul (`auth`, `reading`, `achievement`, `social`)
- Setup database (PostgreSQL untuk production, H2 untuk development lokal)
- Konfigurasi `application.properties` untuk tiap environment (dev/prod)
- Setup CI dasar dengan GitHub Actions (build + jalankan test)
- Buat base layout Thymeleaf (navbar, footer, template halaman)

**Desain Database**
- ERD Modul Auth: entitas `User`, `Role`, skema akun — **Hasan**
- ERD Modul Bacaan & Kuis: entitas `Teks`, `Kategori`, `Soal`, `Jawaban`, `HasilKuis` — **Alya**
- ERD Modul Achievements: entitas `Achievement`, `UserAchievement`, `DailyMission`, `MisiProgress` — **Naufal**
- ERD Modul Liga: entitas `Clan`, `AnggotaClan`, `Tier`, `MusimLiga`, `Leaderboard` — **Kalfin**
- Review dan finalisasi ERD bersama sebelum mulai coding

**Kesepakatan Kontrak API**
- Tentukan format standar request/response (struktur JSON, format error)
- Buat dokumen draf endpoint untuk semua modul (URL, method, request body, response)
- Sepakati event/notifikasi antar-modul: apa yang dikirim, siapa yang mendengarkan

**Seed Data**
- Buat data awal: akun admin default, beberapa contoh teks bacaan, achievement awal — **Naufal**

### Definition of Done
- [ ] Project dapat di-clone dan dijalankan secara lokal oleh semua anggota
- [ ] ERD semua modul sudah direview dan disepakati bersama
- [ ] Dokumen kontrak API (draf) tersedia dan dapat diakses semua anggota
- [ ] CI pipeline berhasil (build tidak gagal)

---

---

## Milestone 25% — Skeleton Berjalan

> **Tujuan:** Setiap modul punya endpoint dan halaman minimal yang hidup. Demo tipis register → login → lihat bacaan → lihat Clan dapat dijalankan end-to-end.

### Deliverables

**Modul Auth — Hasan**
- Endpoint register akun baru (username, email/HP, display name, password)
- Endpoint login dengan email/password → mengembalikan session/token
- Endpoint GET profil pengguna yang sedang login
- Simpan role `PELAJAR` dan `ADMIN` di database
- Halaman Thymeleaf: form register, form login, halaman profil sederhana

**Modul Bacaan & Kuis — Alya**
- Endpoint GET daftar teks bacaan (tampilkan judul, kategori)
- Endpoint GET detail satu teks (tampilkan isi penuh)
- Endpoint POST buat teks baru (khusus Admin)
- Entitas `Teks`, `Kategori`, `Soal` tersimpan di database dengan relasi yang benar
- Halaman Thymeleaf: daftar bacaan, halaman baca teks

**Modul Achievements — Naufal**
- Endpoint GET daftar semua achievement yang ada di sistem
- Endpoint GET achievement milik user tertentu (berdasarkan `userId`)
- Endpoint POST buat achievement baru (khusus Admin)
- Entitas `Achievement` dan `UserAchievement` tersimpan di database
- Halaman Thymeleaf: daftar achievement (data boleh masih kosong)

**Modul Interaksi & Liga — Kalfin**
- Endpoint POST buat Clan baru
- Endpoint GET daftar semua Clan
- Entitas `Clan`, `AnggotaClan`, `Tier` tersimpan di database
- Halaman Thymeleaf: daftar Clan (boleh belum ada detail)

**Integrasi & Infrastruktur**
- Semua modul menggunakan `userId` dari Auth sebagai referensi — koordinasi Hasan dengan semua anggota
- README per modul: cara menjalankan dan daftar endpoint yang tersedia

### Demo Scenario
1. User register akun baru → login berhasil
2. User melihat daftar bacaan → klik satu bacaan → teks terbuka
3. User melihat daftar achievement (boleh kosong)
4. User membuat Clan baru → Clan muncul di daftar

### Definition of Done
- [ ] Semua modul dapat berjalan bersamaan tanpa konflik
- [ ] Endpoint minimal reachable dan mengembalikan response yang sesuai kontrak
- [ ] Skenario demo di atas dapat dijalankan end-to-end di lokal
- [ ] Minimal 1 unit test per modul (model atau service layer)

---

---

## Milestone 50% — Alur Inti Terintegrasi

> **Tujuan:** Pelajar dapat menyelesaikan satu siklus penuh: baca teks → kerjakan kuis → achievement terpicu → bergabung Clan → skor masuk leaderboard. Data antar-modul sudah terhubung.

### Deliverables

**Modul Auth — Hasan**
- Login via SSO Google (OAuth2 dengan Spring Security)
- Endpoint dan halaman update profil (username, display name, password)
- Tambah email atau HP sebagai metode login alternatif
- Guard berbasis role: halaman/endpoint admin tidak dapat diakses oleh pelajar
- Halaman Thymeleaf: halaman profil yang bisa diedit

**Modul Bacaan & Kuis — Alya**
- Alur kuis lengkap:
  1. Pelajar membuka teks
  2. Pelajar memilih "Mulai Kuis" → teks disembunyikan
  3. Pelajar menjawab soal satu per satu
  4. Sistem menghitung skor (jumlah benar / total soal) dan menyimpan hasilnya
- Validasi: pelajar yang sudah menyelesaikan kuis satu teks tidak dapat mengaksesnya lagi
- Admin dapat mengelola soal kuis (tambah, edit, hapus pertanyaan)
- Admin dapat menghapus teks bacaan
- **Event penyelesaian kuis:** kirim data `{userId, teksId, skor, akurasi, timestamp}` ke Achievements dan Liga setelah kuis selesai
- Halaman Thymeleaf: halaman kuis, halaman hasil kuis

**Modul Achievements — Naufal**
- Daily Mission: generate sekumpulan misi harian (contoh: "Selesaikan 3 bacaan hari ini")
- Tampilkan daftar misi harian yang aktif beserta progres pengguna (contoh: "2/3 selesai")
- Dengarkan event penyelesaian kuis dari Modul Bacaan → update progres misi dan achievement
- Achievement terbuka secara otomatis saat milestone tercapai (misal: selesaikan 10 bacaan)
- Pelajar dapat memilih achievement yang ingin ditampilkan di profilnya
- Admin dapat membuat, mengedit, dan menghapus daily mission
- Halaman Thymeleaf: daftar achievement dengan status, progress bar misi harian

**Modul Interaksi & Liga — Kalfin**
- Pelajar dapat bergabung ke Clan: kirim permintaan → Ketua Clan menerima atau menolak
- Leaderboard Clan dalam satu divisi (mulai dari Tier Bronze)
- Hitung skor Clan: penjumlahan total skor kuis semua anggota (algoritma Bronze)
- Dengarkan event penyelesaian kuis → update skor Clan di leaderboard
- Pelajar dapat melihat profil publik pelajar lain
- Halaman Thymeleaf: detail Clan, leaderboard, halaman profil publik

### Demo Scenario
1. Pelajar login via Google → update profil
2. Pilih teks bacaan → kerjakan kuis → lihat hasil skor
3. Achievement terbuka otomatis → progres daily mission terupdate
4. Bergabung ke Clan → skor kuis masuk leaderboard Clan

### Definition of Done
- [ ] Alur bacaan → kuis → achievement → leaderboard berjalan end-to-end
- [ ] Event penyelesaian kuis dari Modul Bacaan diterima dengan benar oleh Modul Achievements dan Liga
- [ ] Minimal 1 integration test untuk alur kuis lengkap
- [ ] Daily mission terupdate otomatis setelah kuis selesai

---

---

## Milestone 75% — Sistem Kompetitif & Fitur Admin Lengkap

> **Tujuan:** Seluruh fitur wajib berjalan. Sistem Liga multi-tier dengan buff/debuff aktif, admin dapat mengelola semua aspek sistem, dan profil publik lengkap tersedia.

### Deliverables

**Modul Auth — Hasan**
- Pelajar dapat menghapus akun mereka sendiri
- Halaman admin: daftar pengguna (dapat di-filter dan dicari)
- Profil publik pelajar: halaman yang dapat dilihat oleh pelajar lain (tampilkan display name, achievement pilihan)
- Keamanan tambahan: rate limiting untuk endpoint login, proteksi CSRF

**Modul Bacaan & Kuis — Alya**
- Riwayat kuis pelajar: halaman yang menampilkan semua kuis yang pernah dikerjakan beserta skor
- API statistik pelajar (dikonsumsi oleh Modul Liga):
  - Total teks yang diselesaikan
  - Rata-rata akurasi kuis
  - Skor total akumulasi
- Dashboard admin: daftar semua teks, filter berdasarkan kategori, status (dipublikasikan / draf)
- Validasi konten: teks tidak dapat dipublikasikan jika belum memiliki soal kuis

**Modul Achievements — Naufal**
- Rotasi daily mission otomatis setiap hari (menggunakan Spring `@Scheduled`)
- Notifikasi di UI saat achievement baru terbuka (misal: banner atau badge)
- Achievement pilihan tampil di halaman profil publik pelajar
- Admin dapat melihat distribusi achievement (berapa pelajar yang telah membuka tiap achievement)

**Modul Interaksi & Liga — Kalfin**
- Sistem tier lengkap dengan algoritma skor yang berbeda per divisi:
  - **Bronze:** penjumlahan total skor semua anggota
  - **Silver & Gold:** rata-rata tertimbang (anggota aktif diberi bobot lebih)
  - **Diamond:** rata-rata tertimbang dengan faktor frekuensi aktivitas mingguan
- Sistem Buff & Debuff yang dihitung secara dinamis:
  - *Productivity Buff (×1.2):* aktif jika ≥50% anggota menyelesaikan daily mission hari ini
  - *Low Accuracy Penalty (×0.8):* aktif jika rata-rata akurasi kuis anggota <50%
  - Buff dan debuff dapat ditumpuk (stackable)
- Buff/debuff yang sedang aktif ditampilkan di halaman leaderboard
- Admin dapat memicu pergantian musim (end of season): sistem secara otomatis memproses promosi dan degradasi Clan berdasarkan posisi akhir klasemen
- Ketua Clan dapat menghapus Clan yang mereka buat

### Demo Scenario
1. Admin buat teks baru + soal kuis → publish
2. Beberapa pelajar mengerjakan kuis → skor masuk ke Clan
3. 50% anggota Clan selesaikan daily mission → Productivity Buff aktif → skor Clan naik
4. Admin trigger end of season → Clan promosi dari Bronze ke Silver
5. Pelajar membuka profil publik temannya dan melihat achievement yang ditampilkan

### Definition of Done
- [ ] Algoritma skor berjalan sesuai tier yang aktif
- [ ] Buff dan debuff diterapkan secara otomatis dan tercermin di leaderboard
- [ ] End of season yang dipicu admin memproses promosi/degradasi dengan benar
- [ ] Rotasi daily mission berjalan otomatis (dapat diverifikasi via log atau test)
- [ ] Semua halaman Thymeleaf tidak ada broken UI atau data yang tidak muncul
- [ ] Test suite mencakup: alur kuis selesai, achievement terbuka, buff aktif, promosi Clan

---

---

## Milestone 100% — Final

> **Tujuan:** Sistem stabil, fitur lengkap sesuai spesifikasi, siap dipresentasikan. Demo end-to-end berjalan mulus tanpa langkah manual.

### Deliverables

**Modul Auth — Hasan**
- Hardening sesi: session timeout, penanganan token kedaluwarsa
- Edge case SSO: pelajar login via Google dengan email yang sudah terdaftar secara manual → akun digabung, bukan duplikat
- Pastikan tidak ada route yang dapat diakses tanpa autentikasi yang sesuai (final security review)

**Modul Bacaan & Kuis — Alya**
- Edge case: admin menghapus teks yang sedang dikerjakan oleh pelajar → tangani dengan graceful (pelajar mendapat pesan yang jelas, data tidak corrupt)
- Pagination pada daftar bacaan
- Pesan error yang informatif di seluruh halaman kuis (koneksi gagal, soal tidak ditemukan, dsb.)
- Full unit test dan integration test untuk modul ini

**Modul Achievements — Naufal**
- Idempotency: achievement tidak dapat terbuka dua kali untuk satu pelajar yang sama (cek duplikasi sebelum insert)
- Edge case: admin membuat daily mission baru di tengah hari → pelajar yang sudah aktif hari itu mendapat misi baru tanpa kehilangan progres yang sudah ada
- Test untuk scheduler rotasi misi harian
- UI polish: tampilan achievement dan profil bersih dan konsisten

**Modul Interaksi & Liga — Kalfin**
- Edge case: Clan dihapus saat musim liga sedang berjalan → data historis tetap tersimpan, tidak ada error di leaderboard
- Pagination pada leaderboard
- Pastikan kalkulasi skor + buff/debuff konsisten meskipun banyak event datang bersamaan
- UI polish: badge tier, indikator buff/debuff yang jelas, halaman Clan detail yang informatif

**Semua Anggota**
- Diagram arsitektur sistem (minimal: diagram modul + alur integrasi antar-modul)
- Dokumentasi API ringkas per modul (endpoint, contoh request, contoh response)
- Deploy ke satu environment staging yang dapat diakses saat presentasi
- Siapkan data demo: akun admin, minimal 2 akun pelajar, beberapa teks bacaan, dan Clan aktif
- README final: langkah instalasi, konfigurasi environment, dan cara menjalankan project
- Lakukan dry run demo minimal satu kali sebelum hari presentasi

### Demo Scenario (Final)
1. Admin login → buat teks baru + soal kuis → publikasikan
2. Pelajar A register via Google → baca teks → kerjakan kuis → achievement terbuka
3. Pelajar A buat Clan → Pelajar B bergabung (di-approve Ketua)
4. Keduanya mengerjakan kuis → daily mission tercapai → Productivity Buff aktif
5. Admin trigger end of season → Clan promosi → achievement promosi terbuka
6. Lihat profil publik Pelajar A: tampil achievement pilihan dan statistik

### Definition of Done
- [ ] Semua fitur wajib sesuai spesifikasi berjalan tanpa error kritis
- [ ] Demo scenario di atas dapat dijalankan end-to-end tanpa workaround
- [ ] Tidak ada inkonsistensi data antar-modul
- [ ] Dokumentasi cukup untuk orang lain menjalankan project dari awal
- [ ] Staging environment aktif dan dapat diakses

---

---

## Ringkasan Tugas per Milestone

| Milestone | Hasan (Auth) | Alya (Bacaan & Kuis) | Naufal (Achievements) | Kalfin (Liga) |
|-----------|--------------|----------------------|----------------------|---------------|
| **Prep** | Setup repo & CI, ERD Auth | ERD Bacaan & Kuis, base layout | ERD Achievements, seed data | ERD Liga & Clan |
| **25%** | Register, login, halaman profil | List & detail bacaan, buat teks (admin) | List achievement, buat achievement (admin) | Buat & list Clan |
| **50%** | SSO Google, update profil, role guard | Alur kuis end-to-end, event notif, CRUD soal | Daily mission, listener event kuis | Join Clan, leaderboard Bronze |
| **75%** | Hapus akun, profil publik, user management | Riwayat kuis, API statistik, dashboard admin | Rotasi misi (scheduler), notif UI | Multi-tier, buff/debuff, end of season |
| **100%** | Security review, edge case SSO, hardening sesi | Edge cases, pagination, full test | Idempotency, edge cases misi, UI polish | Edge cases Clan, pagination leaderboard, UI polish |
