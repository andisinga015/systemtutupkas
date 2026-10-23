# Upgrade V4 → V4.1 pada VPS yang sudah berjalan

Patch ini mengganti akses tujuan/laporan. Tidak memindahkan kolom/baris Google Sheets, mengganti akun/URL/key, menaikkan MemoryMax, atau mengubah UI. Source reader V4 tetap sama. Worker tetap satu proses. **Pengembang belum mengakses VPS/workbook asli; penerimaan live masih pending.**

## 1. Unggah paket ke staging

Unduh `tutupkas-lion-group-v4.1.zip` beserta `.zip.sha256`, unggah lewat FileZilla/SFTP ke `/tmp/tutupkas-upload/`, lalu:

```bash
cd /tmp/tutupkas-upload
sha256sum -c tutupkas-lion-group-v4.1.zip.sha256
mkdir -p /tmp/tutupkas-staging-v41
unzip tutupkas-lion-group-v4.1.zip -d /tmp/tutupkas-staging-v41
cd /tmp/tutupkas-staging-v41/tutupkas-lion-group
less deploy/upgrade.sh
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh -p ActiveState
```

Pertahankan override `MemoryMax=1536M`. Jangan memakai `deploy/install.sh`, mengekstrak langsung menimpa `/opt/tutupkas/current`, atau mengubah struktur workbook untuk menghindari error. Jika ada job berjalan, gunakan Stop pada aplikasi dan tunggu batas aman sebelum menghentikan layanan. Jangan memakai kill -9; intent PREPARED dapat berarti Google sudah menerima tulisan.

## 2. Backup dan upgrade

```bash
sudo bash deploy/upgrade.sh
```

Skrip memeriksa instalasi/unit/environment lama, menyiapkan rilis `/opt/tutupkas/releases/v4.1-...` dengan dependensi terkunci, menghentikan API/worker, dan menonaktifkan auto-start worker saat boot selama penerimaan. Setelah layanan berhenti, backup dibuat di `/var/lib/tutupkas/backups/upgrade-v4.1-...` (0700): SQLite backup API dengan integrity check, aplikasi lama, frontend lama, environment privat, serta path pemulihan.

Skema persisten tetap `b7c31ef05a92`; Alembic upgrade/check tidak menambah tabel atau kolom. `prepare-v41-upgrade --services-stopped` memfinalkan antrean lama yang belum selesai sebagai CANCELED secara diaudit **sesudah backup**, tanpa menghapus hasil, histori, akun, ledger PREPARED/VERIFIED atau transfer ownership. Run #7 tidak otomatis aktif kembali. Run terminal yang sudah FAILED/PARTIAL/SUCCESS dan cancellation sebelumnya dipertahankan.

API baru dimulai; **worker tetap berhenti**. Key `/home/aksesfinancebot/spreadsheet-robot/credentials/robot-key.json`, `/etc/tutupkas/app.env`, unit/override memori, Nginx, SSL dan domain tidak ditulis ulang. Frontend/aset identik dengan V4; footer tetap v3.0 sesuai batas tidak mengubah label UI.

```bash
curl --fail http://127.0.0.1:8000/api/health
sudo systemctl status tutupkas-api --no-pager
sudo systemctl is-active tutupkas-worker
readlink -f /opt/tutupkas/current
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; exec ../.venv/bin/python -c "import app.destination_reader; print(app.destination_reader.__file__)"'
```

Periksa rilis `v4.1-...`, API sehat, worker inactive, serta akun/WEB/URL bulanan/sheet aktif dan Run #7 tetap tersedia. Jika upgrade gagal, jangan memulai worker; lihat lokasi backup dan rollback di bawah.

## 3. Tinjau Run #7 dan intent yang belum pasti

Di riwayat aplikasi, periksa status, item sukses/gagal, target sel, dan error Run #7. Jangan mengedit DB untuk mengubahnya menjadi QUEUED/SUCCESS atau menghapus ownership. Pemeriksaan DB baca saja bila diperlukan:

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; exec ../.venv/bin/python -' <<'PY'
from sqlalchemy import select
from app.db import SessionLocal
from app.models import Run, RunItem, WriteLedger
with SessionLocal() as db:
    run=db.get(Run,7)
    print('Run 7:',run.status if run else 'tidak ditemukan')
    for ledger in db.scalars(select(WriteLedger).join(RunItem,WriteLedger.run_item_id==RunItem.id).where(RunItem.run_id==7)):
        print(ledger.destination_tab,ledger.target_range,ledger.verification_status)
PY
```

Untuk PREPARED, bandingkan nilai/Note/formula tujuan aktual dengan intent tersimpan sebelum memberi izin retry. Retry eksplisit akan membaca ulang tujuan, menerima hanya nilai yang diperbolehkan ownership/intent, dan menolak perubahan manual atau sumber berubah pada uncertain write. Hasil VERIFIED dan item sukses tetap disimpan. Jangan membuat URL/revisi baru untuk menghindari konflik ownership.

## 4. PREVIEW live lebih dahulu

Pastikan tidak ada antrean tulis lain yang belum ditinjau. Dalam UI yang sama, pilih tanggal individual **18 Oktober 2026**, WEB **VELBETT**, lalu buat **Pratinjau**. Pastikan run baru `preview=true`. Jangan menekan Mulai atau retry finansial dulu.

```bash
sudo systemctl start tutupkas-worker
sudo systemctl show tutupkas-worker -p MemoryCurrent -p MemoryPeak -p MemoryMax -p NRestarts
sudo journalctl -u tutupkas-worker -f -n 50
```

Pengamatan berkala di terminal lain:

```bash
watch -n 2 'systemctl show tutupkas-worker -p MemoryCurrent -p MemoryPeak -p MemoryMax -p NRestarts'
```

Pada systemd lama tanpa MemoryPeak, lihat RSS/high-water dari log. Log `destination_scan` menampilkan tab, fase, durasi, jumlah sel struktur yang diindeks dan memori; tidak menampilkan transaksi/Notes/key. Log privat lama `/var/lib/tutupkas/worker-performance.log` tetap berotasi 5 MiB dengan tiga backup.

Pada PREVIEW, cocokkan seluruh rencana dengan workbook aktual: sumber contoh `WD DANA 1 / FATMAWATI ( PP )`, sel sumber `L24`, target **SALDO BANK** di blok VELBETT dan tanggal yang benar; TOTAL DEPO/WD, BIAYA, fee dan kiriman tab aktif lainnya juga harus benar. Angka FATMAWATI harus mengikuti sumber aktual—jangan memaksakan saldo contoh rekening IVAN. Pastikan preview tidak membuat POST tulis atau ledger finansial baru, tidak menulis blok tetangga dan tidak mengubah `Laporan deposit harian`.

V4.1 tidak menolak sheet karena `rowCount × columnCount`. Tidak perlu menghapus kolom, memindah website, mengecilkan atau memecah workbook. Jika terjadi RESPONSE_TOO_LARGE pada satu rentang, MERGE_METADATA_LIMIT atau DESTINATION_STRUCTURE_LIMIT, hentikan penerimaan dan simpan diagnostik; jangan menaikkan MemoryMax/byte cap atau melewati proteksi. Batas ini menyangkut respons/struktur aktual, bukan ukuran alokasi kosong.

Global formula/struktur tetap dipindai sebelum dan sesudah setiap item tulis. Batching mengurangi request tetapi pekerjaan pada sheet lebar masih dapat memerlukan banyak menit; angka benchmark offline tidak memasukkan throttle Google. Stop saat POST sudah dikirim tetap menunggu read-back aman. Lihat [PERFORMANCE_V41.md](PERFORMANCE_V41.md) untuk jumlah request dan batas pengukuran.

## 5. Otorisasi tulis dan aktivasi permanen

Hanya setelah operator memeriksa PREVIEW dan memberi izin finansial, gunakan **Ulangi Item Tersisa** pada Run #7 jika konfigurasinya masih cocok, atau alur resync yang sudah ada bila URL/revisi memang berubah. Retry memperbarui otorisasi engine dan tetap memeriksa ownership/intent; item sukses tidak diulang. Tidak ada replay otomatis saat upgrade.

Periksa hasil tulis/read-back, saldo WEB dan konsolidasi dengan aturan Clear yang sama. Skrip lama `scripts/verify-real-run.py RUN_ID` tetap baca saja dan sekarang mendapat reader terbatas juga. Setelah penerimaan selesai, aktifkan start saat boot hanya jika Anda memang menginginkannya:

```bash
sudo systemctl enable tutupkas-worker
```

## 6. Batas operasional

| Batas | Nilai |
|---|---:|
| `TK_GRID_CHUNK_CELLS` | 8.000 per rentang; batas rentang V4 dipertahankan |
| `TK_DESTINATION_BATCH_CELLS` | 64.000 total sel per request, maksimal 32 rentang |
| `TK_MAX_API_RESPONSE_BYTES` | 8.388.608 byte, tidak dinaikkan |
| `TK_DESTINATION_INDEX_BYTES` | 64 MiB indeks struktur temporer per view |
| Cache sel aktual | Maksimal 128 sel dan estimasi konservatif 4 MiB per view |
| Cache halaman SQLite temporer | 512 KiB per view, mmap nonaktif |

Batch yang melebihi cap byte dipecah menjadi batch lebih kecil dengan rentang asli yang sama. Jika satu rentang saja masih terlalu besar, operasi gagal secara eksplisit. Nilai/formula/Note biasa dari scan langsung dibuang; hanya struktur yang diperlukan diindeks pada direktori temporer 0700, file 0600. Indeks ini terpisah dari DB persisten, ditutup/dihapus setelah pemakaian; tidak mengubah jurnal/PRAGMA database finansial. Unit PrivateTmp yang sudah ada tetap berlaku.

`TK_MAX_DESTINATION_CELLS` dan `TK_MAX_SHEET_CELLS` lama tetap diterima untuk kompatibilitas environment, tetapi tidak menjadi guard ukuran alokasi tujuan. Default baru berlaku tanpa menulis ulang app.env. Source reader dan batas sumber V4 tetap berlaku.

## 7. Rollback

```bash
sudo bash /tmp/tutupkas-staging-v41/tutupkas-lion-group/deploy/rollback.sh /var/lib/tutupkas/backups/upgrade-v4.1-TANGGAL-ID
```

Rollback menghentikan layanan, menyimpan backup DB setelah upgrade, memulihkan DB/rilis/frontend sebelum upgrade, menahan antrean snapshot lama dan memulai API saja. Worker tetap berhenti. Perubahan DB setelah backup tidak masuk DB aktif hasil rollback; salinannya disimpan. Tulisan Google yang sudah terjadi tidak dibatalkan oleh rollback DB, sehingga intent/sel aktual perlu ditinjau sebelum operasi dilanjutkan. V4 lama masih mempunyai guard SHEET_TOO_LARGE; jangan mengaktifkan ulang worker lama untuk memaksa run yang sama.

Panduan instalasi/rollback ini diperiksa lokal, belum dijalankan pada VPS pengguna. Backup memuat environment privat dan tidak boleh masuk paket GitHub.
