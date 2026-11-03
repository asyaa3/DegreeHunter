# AGENTS.md — Catatan untuk agen AI (Codex dll.) di proyek Degree Hunter

Baca file ini sampai habis sebelum mengubah apa pun.

## Ringkasan proyek
- Game: **Degree Hunter**, platformer aksi 2D edukatif tentang komponen IoT. Tugas akhir Kelompok 7, S1 Pendidikan Teknik Informatika, Universitas Negeri Malang.
- Tim: Nabigh Nailur Ridho (programmer), Rufi Nur Laili, Tasya Az Zahro. Dosen: Harits Ar Rosyid, S.T., M.T., Ph.D.
- Engine: **GDevelop 5.6**, resolusi tetap 1280 x 720, target akhir ekspor **Web (HTML5)** untuk PC dan HP.
- Bahasa UI dan komentar event: Bahasa Indonesia.

## ATURAN WAJIB
1. **Tutup proyek di GDevelop sebelum mengedit file JSON**, lalu minta pengguna membuka ulang. Kalau GDevelop masih terbuka dan menyimpan, perubahanmu bisa tertimpa.
2. Proyek memakai format **split JSON**:
   - `Degree hunter.json` adalah file utama. Isinya resource, variabel global, dan referensi `{"__REFERENCE_TO_SPLIT_OBJECT": true, "referenceTo": "/layouts/..."}`.
   - `layouts/*.json` berisi satu file per scene.
   - `externalEvents/*.json` berisi event bersama.
   - Jaga JSON tetap valid. Ikuti jumlah parameter instruksi GDevelop yang sudah ada; contoh paling aman adalah instruksi serupa di file yang sama.
3. **Nama variabel tidak boleh sama dengan nama objek atau grup.** GDevelop 5.6 memakai unified variables, jadi nama yang sama akan bentrok. Karena itu dipakai `HunterPilihan` (bukan `Hunter`) dan `ChipData` (bukan `Chip`).
4. Jangan menghapus file atau folder kecuali diminta pengguna. Jangan commit atau push; pengguna melakukannya sendiri lewat GitHub Desktop.
5. Skrip builder Python yang dulu dipakai untuk membuat proyek **tidak ada di repo ini**. Edit JSON langsung, atau minta pengguna mengubah lewat editor GDevelop.
6. Sesudah setiap perubahan, minta pengguna membuka proyek dan menekan Pratinjau untuk mengecek hasilnya.
7. **JANGAN mengubah gaya visual atau UI** (menu, tombol, HUD, panel, font, warna, posisi) kecuali pengguna meminta perubahan spesifik untuk satu bagian tertentu. Gaya game ini adalah **pixel art cyberpunk** (font VT323 / Press Start 2P, tombol gradasi pink-kuning, aset di `ui2/`) dan harus dipertahankan. Jangan membuat set UI baru dan jangan mengganti resource ke folder lain.
8. Kerjakan satu perubahan kecil per tugas. Jangan menggabungkan perbaikan logika dengan perubahan tampilan dalam satu tugas.

## Struktur scene (16)
| Scene | Fungsi |
|---|---|
| Scene Opening | Start screen: Mulai, Pengaturan, About, Keluar |
| Scene Selection | Pilih hunter (klik kartu, satu tombol SELECT) |
| Scene Komik | Semua komik. `KomikID`: `prolog`, `stage1`-`stage6` (kedatangan + kartu stage), `outro1`-`outro6` (setelah bos), `ending` |
| Scene Level Select | Peta 6 stage, Bengkel Lab, Ensiklopedia, Reset progres |
| Scene Memuat | Layar loading. Tujuan scene disimpan di variabel global `Tujuan` |
| Scene Game1 | Stage 1 (Pabrik Robot) |
| Scene Stage2 - Stage6 | Stage 2-6. Diawali area luar gedung (LatarLuar + TileMapLevel), lalu masuk ke dalam |
| Scene Bos | Satu arena bos untuk semua stage. Bos dipilih dari `StageNo` |
| Scene Edukasi | Kartu edukasi komponen + kuis 2 soal (`EduMode`, `EduNo`) |
| Scene Win | Ringkasan hadiah. Setelah Stage 6 masuk ke komik `ending` |
| Scene Bengkel | Upgrade Nyawa/Kapasitas/Damage memakai Chip Data |
| Scene About | Foto dan nama tim + dosen |

Alur: Opening → Selection → Komik prolog → Level Select → Komik stageN → Memuat → Stage → portal (setelah semua robot kalah) → Memuat → Bos → komponen jatuh, diambil → Komik outroN → Edukasi + Kuis → Win → Level Select (Stage 6: ending → About).

## External events (dipanggil lewat event Link)
- `Gameplay Inti` (`externalEvents/gameplay-inti.json`): seluruh logika stage. Berisi kamera, kontrol, ganti karakter, skill, animasi, peluru, item, energi/level, AI musuh, damage, HUD, dan portal.
- `Menu Jeda`: menu pause (Esc / P / tombol II). Memakai TimeScale 0.
- `Menu Pengaturan`: volume musik dan SFX, layar penuh, Kontrol Sentuh (0 Otomatis, 1 Aktif, 2 Mati).
- `Transisi`: tirai fade antar scene.
- Logika Scene Bos ada langsung di `layouts/scene-bos.json`, bukan di external event.

## Objek dan konsep penting
- **Hunter = satu objek `Pemain`** dengan grup `Hunter` yang hanya berisi `Pemain`.
  - Animasi 0-9 untuk Nabigh, 10-19 untuk Rufi (nama internal "Rupii"), 20-29 untuk Tasya.
  - Urutan per hunter: 0 jalan kiri, 1 jalan, 2 tembak kiri, 3 tembak, 4 lompat kiri, 5 lompat, 6 idle kiri, 7 idle, 8 mati kiri, 9 mati.
  - Animasi dipasang dengan `angka + Variable(WarnaIdx)*10`.
  - `WarnaIdx` (variabel scene): 0 Nabigh, 1 Rufi, 2 Tasya. Disinkronkan dengan global `HunterPilihan` ("Nabigh"/"Rupii"/"Tasya").
  - **Jangan kembali memakai pola hapus lalu buat objek saat ganti karakter.** Pola itu dulu membuat lag dan gerakan kacau.
- Ganti karakter: tombol C atau tombol sentuh C, jeda 0,8 detik. Efeknya memakai objek `GantiFX` (pilar cahaya, animasi per warna) dan `LabelGanti` (nama hunter). Nyawa dan Energi dipakai bersama.
- Peluru `PeluruHunter` punya 3 animasi warna (pakai `WarnaIdx`). `Muzzle` juga per warna. Posisi peluru: kanan `Hunter.X()+100`, kiri `Hunter.X()-10`, tinggi `Hunter.Y()+62`.
- Platformer behavior bernama `PlatformerObject` dengan `ignoreDefaultControls` = true. Kontrol disimulasikan lewat event.
- Variabel object hunter: `arah` (1 kanan, -1 kiri). Timer object: `kebal`, `cd`, `animtembak`.

## Kontrol sentuh (HP)
- Layer `Sentuh` berisi tombol TbKiri, TbKanan, TbTurun, TbLompat, TbTembak, TbSkill, TbGantiSkill, TbGanti, TbJeda (animasi 0 lepas, 1 tekan).
- Ada event **JavaScript** di grup "Kontrol sentuh" yang membaca multi-touch dan klik mouse, lalu mengisi variabel scene `TKiri`, `TKanan`, `TLompat`, `TTembak`, `TTurun`, `TSkill`, `TGantiSkill`, `TGanti`, `TJeda` (1 = ditekan) serta versi `...Tekan` (1 hanya di frame pertama ditekan).
- Mode Otomatis hanya menampilkan tombol di perangkat sentuh. Di PC, pilih Pengaturan → Kontrol Sentuh → Aktif untuk mengetes dengan mouse.
- Skill memakai sistem skill terpilih supaya tombol tidak menumpuk: X / TbGantiSkill = ganti skill, F / TbSkill = pakai skill. Di keyboard, Q/E/M/R tetap langsung.

## Variabel global utama
HunterPilihan, StageTerbuka, StageSelesai, StageNo, ChipData, XP, LevelHunter, NyawaMax, EnergiMax, Energi, Damage, UpNyawa, UpKapasitas, UpDamage, SudahProlog, KomikID, KepinganTotal, CatatanLab, SesiChip, SesiKeping, KuisBenar, EduMode, EduNo, Tamat, VolMusik, VolSFX, LayarPenuh, KontrolSentuh, TutorialSelesai, Tujuan, Tip* (petunjuk sekali tampil).
Progres disimpan di storage grup `DegreeHunter`. Pengaturan disimpan terpisah (`DegreeHunterSet`).

## Pelajaran dari bug yang pernah terjadi
- **Urutan event saudara penting.** Kalau event A mengubah variabel X lalu pindah scene, dan event B sesudahnya mengecek X lalu pindah scene lain, maka B menimpa A di frame yang sama. Contohnya bug ending Stage 6 yang dulu tertimpa. Cek urutan seperti ini setiap kali menambah logika perpindahan scene.
- Di Scene Bos, kontrol tetap aktif setelah bos kalah (setelah 1,4 detik) supaya hunter bisa mengambil `KomponenDrop`. Jangan dikunci lagi.
- Tabrakan dengan bos hanya melukai saat `Intro` = 0.
- Layer `Transisi`, `Pause`, `Pengaturan`, dan `Sentuh` disembunyikan secara default dan baru ditampilkan lewat event.

## Status saat ini
Sudah diuji di pratinjau:
- Menu, pengaturan, About, Level Select, tutorial, dan ganti karakter (keyboard dan sentuh).
- Keenam bos sampai komponen diambil.
- Alur Stage 1 dan Stage 6 sampai Win dan ending.
- Stage 4: area luar, portal, ruang bos.

Belum diuji:
- Main penuh Stage 1-6 tanpa jalan pintas, termasuk menyelesaikan Stage 2, 3, 5, 6 dari awal sampai portal.
- Penyimpanan progres setelah game ditutup lalu dibuka.
- Main di HP sungguhan setelah ekspor web.
- Penyeimbangan kesulitan.

Belum dikerjakan:
- Ekspor web.
- Musuh baru (Drone Sensor, Robot Perisai).
- Satu crate 1 tile di pintu masuk Stage 1 yang menghalangi peluru.
- Nama di HUD masih gambar "RUPII", sedangkan label lain memakai "RUFI".

## Cara mengetes
Pengguna menekan Pratinjau di GDevelop. Untuk menguji satu scene langsung, buka tab scene tersebut lalu Pratinjau.
Kalau perlu jalan pintas uji, tambahkan event debug **sementara** (misalnya tombol untuk menghapus semua `Musuh` atau mengisi `Bos.Variable(hp)` = 1) dan **hapus lagi sebelum selesai**.
