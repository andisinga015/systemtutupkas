# Upgrade V4.2 / V4.2.1 → V4.2.2

Paket dibangun dari source V4.2.1 lengkap dan tetap menyertakan seluruh optimasi writer tersebut. V4.2 dan V4.2.1 sama-sama memakai schema `c92448ce233b`; skrip membaca README release terpasang dan **schema database nyata**, bukan nama folder staging. Target schema V4.2.2 adalah `d422a61e908f`.

Tidak ada akses VPS/Google produksi dari lingkungan pengembangan. Script systemd/Nginx diuji sintaks, sementara migration/hold/verifier/backup restore diuji dengan database berisi graph finansial. Penerimaan live dilakukan di VPS Anda. Jangan menjalankan install.sh untuk menimpa instalasi lama.

## Sebelum menjalankan

Jika ada Run aktif, gunakan Stop dan tunggu batas verifikasi aman bila memungkinkan. Jangan kill -9 saat HTTP POST. Simpan ZIP dan checksum ke staging lalu:

```bash
cd /tmp/tutupkas-upload
sha256sum -c tutupkas-lion-group-v4.2.2.zip.sha256
mkdir -p /tmp/tutupkas-staging-v422
unzip tutupkas-lion-group-v4.2.2.zip -d /tmp/tutupkas-staging-v422
cd /tmp/tutupkas-staging-v422/tutupkas-lion-group
less deploy/upgrade.sh
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh -p ActiveState
sudo bash deploy/upgrade.sh
```

Dependensi dengan hash lock dan public frontend tersedia dalam paket. Nilai MemoryMax/MemoryHigh, Nginx, HTTPS, key Google dan app.env tidak diganti.

| Komponen | Path tetap |
|---|---|
| Release aktif | `/opt/tutupkas/current` |
| SQLite | `/var/lib/tutupkas/tutupkas.sqlite3` atau path lama dalam direktori yang sama dari app.env |
| Environment | `/etc/tutupkas/app.env` |
| Key lama | `/home/aksesfinancebot/spreadsheet-robot/credentials/robot-key.json` |
| Backup baru | `/var/lib/tutupkas/backups/upgrade-v4.2.2-TIMESTAMP-PID/` |

## Apa yang dilakukan skrip

1. Memeriksa unit/user/working directory/environment/key/Nginx dan menampilkan release serta schema terpasang dan jumlah Run aktif. Menyiapkan release/dependensi baru sebelum stop. Mendukung c924 V4.2/V4.2.1 dan pemasangan ulang V4.2.2 yang valid; versi lain ditolak untuk mengikuti panduan upgrade sebelumnya.
2. Menghentikan API agar tidak ada job baru, membuat consistent **pre-stop.sqlite3**, lalu meminta cancel job aktif melalui **interpreter dan kode release yang masih terpasang**, protokol database worker lama, date lock dan audit. Ini menjaga cancellation batch recheck agar tidak memakai period-aware source queries sebelum kolom baru dimigrasikan. Worker lama dapat menyelesaikan verifikasi tulisan yang sudah dikirim.
3. Menghentikan dan men-disable worker saat boot. Membuat consistent stopped **database.sqlite3** beserta aplikasi/frontend/environment lama dan path pemulihannya. Backup stopped adalah pasangan rollback; pre-stop dipertahankan sebagai bukti sebelum proses penghentian.
4. Menjalankan Alembic upgrade/check. Migrasi source_documents mengganti uniqueness dan menambah source_period, mempertahankan semua ID/FK/kolom lama. Parent-table rebuild berlangsung dalam transaksi SQLite eksplisit, dengan FK check sebelum commit. DEPO lama menjadi **UNASSIGNED**, QRIS lama menjadi **MONTHLY**. URL, validation, revisions, SourceSheet states, lineage, indexes, snapshots, Runs, retries, ledger, transfer ownership, mappings dan audit lama tidak dihapus atau ditebak.
5. CLI hold lama `prepare-v42-upgrade --services-stopped` menahan pekerjaan aktif tersisa secara diaudit. Run terminal tetap histori; PREPARED tidak diakui otomatis. Histori canceled/partial tidak dilanjutkan otomatis.
6. `scripts/verify-v422-upgrade.py BACKUP CURRENT` membuka kedua SQLite read-only, mengecek integrity/FK/schema dan seluruh protected old columns. Hanya source_period yang ditambahkan serta field audited cancellation/append-only audit/reconciliation yang diizinkan berubah. Deliberate ledger corruption diuji dan ditolak. Tidak menghubungi Google.
7. Mengganti symlink dan frontend HTML setelah aset tersedia, memulai **API saja**, memeriksa health, dan mencetak backup path. **Worker tetap STOPPED dan disabled saat boot.**

Tidak ada mapping rebuild, refresh semua sumber, cloning tab, pengaktifan sheet atau financial Crawl oleh upgrade. Memasang dari V4.2 juga memasang kode optimasi writer V4.2.1 yang dipertahankan dalam V4.2.2.

## Verifikasi layanan dan schema

```bash
curl --fail http://127.0.0.1:8000/api/health
sudo systemctl status tutupkas-api --no-pager
sudo systemctl is-active tutupkas-worker
sudo systemctl is-enabled tutupkas-worker
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; ../.venv/bin/alembic current; ../.venv/bin/alembic check'
```

Worker harus inactive/disabled; exit nonzero dari kedua pemeriksaan itu sesuai. Schema harus d422a61e908f head tanpa operasi upgrade baru. Override RAM yang sudah ada, misalnya 1536M, tetap sama. Login memakai akun lama, hard refresh dan cocokkan history/ledger/settings sebelum meneruskan pekerjaan.

## Tetapkan DEPO lama secara eksplisit

Pada Admin Setting WEB dan bulan yang benar, panel **DOC DEPO lama — perlu penetapan periode** menunjukkan URL lama. Buka dan tinjau dokumen, lalu pilih salah satu:

- **Tetapkan sebagai DOC A — tanggal 1–15**, atau
- **Tetapkan sebagai DOC B — tanggal 16–akhir bulan**.

Tindakan mempertahankan record ID, revisi dan status aktif/nonaktif tab yang sama. Jika ditetapkan sebagai B, tab lama ikut **record yang sama di B**, tidak disalin ke A. Tidak ada penebakan dari tanggal Run lama, judul atau nama tab. Slot periode yang sudah berisi URL tidak ditimpa; UI men-disable pilihan itu dan backend menolak konflik. QRIS lama sudah menjadi bulanan dengan setting yang sama.

Isi URL pasangan A/B yang memang diperlukan dan validasi dokumen tersebut saja. Buka **Daftar Sheet**, pilih A, B atau QRIS, dan tinjau status masing-masing. Tab baru default nonaktif; aktifkan hanya yang sudah diverifikasi. Nama sama pada A dan B tetap dua identitas. Periksa counts aktif/nonaktif/unavailable. Jangan menyalin aktivasi antar periode/bulan.

DOC A wajib untuk tanggal 1–15, DOC B untuk 16–akhir bulan. Missing/invalid/no usable active tabs pada periode relevan menjadi configuration error, bukan fallback ke periode lain. QRIS absent mempertahankan sifat optional lama; QRIS configured invalid tetap error. Jika sumber lama belum memiliki verified identity yang cukup, validasi/refresh **dokumen terkait** dan tinjau statusnya; jangan mengaktifkan semua tab otomatis.

Tidak perlu revalidasi URL Kas Kecil hanya karena source period berubah. Pertahankan mapping READY dan revision tujuan yang sudah benar. Source save/validate/assignment tidak membangun ulang destination mapping.

## Penerimaan sebelum mode tulis

Tinjau semua antrean dan PREPARED intents; jangan replay seluruh Run historis. Untuk menggunakan data lama pada retry, record DEPO asal harus ditetapkan ke periode yang cocok dengan tanggal Run. Item tidak dipindahkan A↔B berdasarkan nama. Jika hanya satu item gagal, retry item tersebut saja; successful items dan snapshot lama tidak dihitung ulang.

Antrekan Preview terotorisasi untuk tanggal contoh 10 dan 18 pada salinan aman setelah konteks sumber/tujuan ditinjau. Mulai worker manual:

```bash
sudo systemctl start tutupkas-worker
sudo journalctl -u tutupkas-worker -n 150 --no-pager
```

Tanggal 10 hanya boleh memakai tab aktif A + QRIS bulanan; tanggal 18 hanya B + QRIS yang sama. Preview tetap nol financial POST. Tidak ada akses index/snapshot ke periode yang tidak dipilih. Tinjau tab inactive/unavailable, berikutnya separator date, carry-forward, BIAYA/fee/kiriman/Notes dan target absolut. Missing next-date marker tetap error; tidak ada boundary buatan pada hari 15/16/akhir bulan.

Setelah Run tulis yang diotorisasi SUCCESS, verifikator lama hanya GET:

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; ../.venv/bin/python ../scripts/verify-real-run.py RUN_ID'
```

Ganti RUN_ID. KEEP menggunakan bukti konflik dan BALANCE aktual; tidak memaksa KLOP. Jika penerimaan selesai dan antrean aman telah ditinjau, aktifkan boot secara eksplisit dengan `sudo systemctl enable tutupkas-worker`. Live Google/performa <60 detik tetap belum terverifikasi.

## Rollback dan gangguan migrasi

Rollback harus memulihkan **release/frontend lama dan stopped database.sqlite3 bersama**, bukan menghapus tabel/history baru:

```bash
sudo bash /tmp/tutupkas-staging-v422/tutupkas-lion-group/deploy/rollback.sh /var/lib/tutupkas/backups/upgrade-v4.2.2-TIMESTAMP-PID
```

Gunakan folder tepat yang dicetak upgrade. Skrip menyimpan database setelah upgrade sebagai post-upgrade-*.sqlite3, memulihkan backup/symlink/frontend lama, menahan antrean restored dan memulai API saja. Worker tetap berhenti. app.env/key/Nginx tidak diganti. Simpan kedua database untuk pemeriksaan finansial.

Jika migrasi parent-table gagal, transaksi mengembalikan schema/row lama. Jika terhenti setelah commit schema tetapi sebelum stamp Alembic, retry hanya mengenali bentuk completed migration yang tepat dan period hasil migrasi yang belum ditetapkan; schema parsial/tidak dikenal ditolak. Jangan melakukan manual stamp atau menghapus intent untuk meloloskan error.

Downgrade Alembic ditolak bila DOC A/B sudah ditetapkan karena informasi periode dapat hilang/tidak cocok dengan constraint lama. Jalur rollback resmi adalah backup pasangan app+DB. **Rollback SQLite tidak membatalkan tulisan Google setelah backup.** Cocokkan actual cells, Notes, intent dan ownership sebelum mengizinkan worker berjalan lagi. Jika upgrade berhenti sebelum switch, layanan mungkin tetap berhenti; ikuti log/backup alih-alih menyalakan worker tanpa tinjauan.
