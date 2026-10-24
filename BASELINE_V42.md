# Baseline V4.1 sebelum implementasi V4.2 — 24 Oktober 2026

Baseline: ZIP V4.1 `637e62b2f6af46fa8f1fde9f86c28e3c7455d29d3eec863d1b227daa81823344`, commit publikasi `b38ea19f7cba0550f0fe6c93feccac175fbc60e3`. Full-suite sebelum perubahan: **198 passed**, satu warning Starlette/httpx, Python 3.12, 62,07 detik. Skema `b7c31ef05a92`.

## Temuan kode dan pemanggil

- `source_reader.SourceReader.discover/read`: discovery values-only 200 baris per band, seluruh histori ditelusuri untuk membuktikan uniqueness; tidak ada token revisi isi atau indeks tanggal persisten. Snapshot Day sparse hanya hidup dalam satu proses. Parser Day tetap sumber aturan kolom/tanggal/carry-forward/fee/warna.
- `destination_reader.DestinationReader`: V4.1 sudah bounded, disk-index dan cleanup; tidak ada registry bulanan persisten. Metadata/header/global formula/struktur dibaca lagi pada setiap view. `resolver.regular/labeled/fees/transfer_slots` dipanggil ulang saat planning.
- `sheets.SheetsClient.request`: min interval 1,2 detik per instance, streaming 8 MiB, retry/backoff; POST transport dapat dicoba ulang sebelum inspeksi hasil. Sheets metadata tidak memberikan token revisi isi yang dapat dipakai sebagai jaminan cache.
- `runner._process_run`: source_cache dibersihkan ketika berpindah tab; planning seluruh item kemudian writing; tidak ada scheduler retry internal. BIAYA total bergantung input tersimpan dan urutan DEPO sebelum QRIS dipertahankan.
- `writer.owner_key`: memakai nama tab; fingerprint transfer juga memakai nama dan row. Rename bisa kehilangan lineage. PREPARED/VERIFIED, Notes/formula/merge/manual-data/read-back sudah ada dan harus dipertahankan.
- `operations.queue_retry`: menolak Run aktif lewat assert_available lalu mengubah Run ke QUEUED; tidak ada record claim retry persisten. Input expense gagal juga otomatis mengantrikan total.
- `api.refresh_tabs`: hanya nama; belum membedakan rename dan delete/recreate lewat sheetId. Tab baru nonaktif. Save monthly URL mengubah revision dan validation tetapi tidak membangun mapping.
- `resolver.fees/labeled`: hanya dua baris header global, sel inherited merged dihitung sebagai beberapa kecocokan; header fee yang mulai pada baris lain terlewat. `qris_fee` tetap memetakan dan menulis deskripsi kosong/nol ketika source fee tidak ada.
- `job_control/worker`: satu proses dengan flock; restart memfinalkan Run terputus tanpa auto financial replay. `workflow/batches/reconciliation`: pin revisi, date gate, active-run uniqueness, laporan read-only, exact-zero dan Clear sudah tersedia.
- `models/db/migrations`: SQLite WAL, busy_timeout 30 detik; tiga migrasi sampai b7c31ef05a92. Tidak ada tabel snapshot/index/map/retry/decision. Frontend RunDetail menonaktifkan retry saat aktif dan belum mempunyai tindakan konflik.

## Rencana keselamatan yang ditetapkan sebelum perubahan

Migrasi hanya menambah tabel/kolom nullable. Registry absolute dipin ke destination revision dan tidak memakai source tab sebagai key. Satu coordinator tetap menulis; retry diklaim atomic di batas aman. Snapshot lengkap tersimpan atomik dan retry selalu membuat snapshot baru. Keputusan konflik menyimpan seluruh logical group, approval stale ditolak, Preview tidak boleh menulis. Rekonsiliasi tetap berdasarkan nilai Google.

Indeks tanggal memerlukan bukti revisi isi untuk melewati histori dengan aman: posisi dua anchor saja tidak membuktikan tidak ada duplicate lain setelah edit. Gunakan metadata versi Drive baca saja dengan Service Account yang sama bila tersedia; bila tidak tersedia, lakukan pembuktian bounded/fail eksplisit, bukan menganggap revisi konfigurasi sebagai revisi isi. Key dan file kredensial tidak diganti. Akses metadata Drive perlu diuji di VPS; warm-index tidak diklaim bila token tidak tersedia.

Cache registry tidak menggantikan proteksi global formula/struktur. Jalur fast memerlukan bukti versi tidak berubah; fallback memvalidasi struktur global bounded tanpa otomatis memetakan ulang koordinat. Setelah POST, pembuktian global dan read-back tetap dilakukan. Target Preview <60 detik adalah sasaran pengukuran, bukan janji.

## Batas baseline

Belum ada akses Google/key/SSH VPS pengguna. Benchmark V4.1 offline sebelumnya: 1.319 request gabungan pada satu website, peak 129,41 MiB; tujuan blank terformat peak 172,79 MiB. Tidak ada klaim angka ini membuktikan produksi. Permintaan dokumentasi resmi batchGet, quota, batchUpdate dan Drive files melalui HTTPS pada lingkungan ini ditolak proxy; URL referensi akan dicantumkan, retrieval live belum terverifikasi. Pacing 1,2 detik tidak akan dihapus tanpa bukti quota.
