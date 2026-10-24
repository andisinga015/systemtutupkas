# Pengujian V4.2.1 — 25 Oktober 2026 (Asia/Hovd)

Full backend **343 passed pada Python 3.12.14 (294,45 detik)**. **343 passed pada Python 3.10.22 (362,75 detik)**. Satu warning Starlette/httpx deprecation; dependency locks tetap V4.2. **283 test V4.2 lama tidak diubah/dihapus**, ditambah 60 kasus writer/proof/upgrade V4.2.1. Hasil V4.2 asli disimpan di [TEST_REPORT_V42.md](TEST_REPORT_V42.md).

| Pemeriksaan | Hasil |
|---|---|
| Full backend Python 3.12 | 343 passed |
| Full backend Python 3.10 | 343 passed |
| Chromium Playwright | 8 passed, 29,8 detik |
| TypeScript strict | Lulus |
| Frontend source dan semua prebuilt public assets | Byte-identik ZIP V4.2 |
| Alembic upgrade/current/check pada DB terisolasi | c92448ce233b head; no new upgrade operations |
| Syntax shell upgrade/rollback/e2e | Lulus |
| Compile Python | Lulus |
| Preservasi DB saat hold upgrade | Integrity/FK/schema/protected rows/history sama; deliberate ledger corruption terdeteksi |
| Actual V4.2 ZIP oracle vs V4.2.1 | 43 status dan 57 nilai/koordinat/Notes/bank/transaction sama persis |
| Live Google, systemd/Nginx/VPS pengguna | Pending; tidak ada akses/tulisan asli |

## Kasus patch yang diuji

Suite baru memakai READY mapping dan source authority sungguhan melalui API aplikasi, dengan transport Sheets wire fixture terisolasi. Termasuk financial logic lama melalui 8 snapshot dan 43 item; bukan mock return-success dari writer.

- Shared preflight/post proof BIAYA, grouping POST pada beberapa sheetId, expense_total saja setelah partial/cancel V4.2-style; tidak replay item sukses, tidak membaca ulang sumber dan tidak mengganti snapshot/mapping.
- Manual value/Note, formula, merge, missing/duplicate/late header, Google sheet identity, mapping tidak READY/revision berubah; grup independen tetap berjalan dan tidak ada false KLOP.
- Rencana atau snapshot corrupt, sumber berubah sebelum POST, dependency signed BIAYA berubah; tidak ada intent stale dikirim.
- Perubahan struktur tanggal/formula/merge di luar target, duplicate fee header, epoch berubah saat scan; proof global tetap wajib sebelum acknowledgement.
- Full/thin reader diferensial untuk header penuh/sparse/satu header penuh/padding multi-stripe, Note yang memperpanjang body, numeric DATE, merge; header nomor hari berwarna hitam di baris ketiga/keempat tetap diresolve sama.
- Full target/anchor fields tetap melindungi Note dan formula; fee header di bawah dua baris tetap diperiksa. Tidak ada duplicate physical cell dalam satu POST dan report ditolak sebagai output sebelum HTTP.
- NEW/KEEP seluruh grup, stale actual values, Note manual dipertahankan dan KEEP dengan BALANCE berbeda tetap NOT_BALANCED.
- Accepted POST lalu lost reply/timeout/crash worker; durable PREPARED dipulihkan tanpa POST ulang/slot/ownership tambahan. Amount/Note readback mismatch membuat seluruh grup itu tetap PREPARED; grup independen terbukti dapat VERIFIED.
- Stop saat persiapan, tepat setelah phase WRITING tetapi sebelum HTTP, dan setelah submit; tidak kirim bila belum submit, verifikasi grup yang sudah submit tetap dicoba.
- Tanpa token, token stale/hilang setelah POST/di tengah proof, metadata melebihi cap, final tab read gagal; bounded fallback dan independent errors. QRIS priority retry tidak mendahului DEPO transfer PENDING.
- Upgrade hold pada database populated dengan PREPARED dan terminal history, same READY mapping/8 snapshots/ledger/settings/users; verifier read-only menolak corruption.

Seluruh regression lama ikut berjalan: financial golden, fixed columns/carry-forward/fee/kiriman, source date indexes/snapshots/identities, retry claim/restart/concurrency lock, auth/CSRF, true KLOP/Clear, populated migrations dan ledger recovery. UI tetap sama; delapan browser tests lama lulus dan screenshot dokumentasi lama dipertahankan dari arsip asli.

## Benchmark rilis

Source oracle V4.2 benar-benar diekstrak dari ZIP SHA `35eb89eea6da7fc05e200ccd807b8607042b07ac8804e2c4ec65e11ddf36dc45`. Fixture BIAYA formatted/Note-heavy, 8 snapshot, 43 rencana, 57 sel, 13 kiriman:

| Ukuran | V4.2 | V4.2.1 |
|---|---:|---:|
| Destination GET | 660 | 77 |
| Financial POST | 36 | 1 |
| Response bytes | 100.971.103 | 117.986 |
| GET ≥2 MiB | 12 | 0 |
| Wall clock offline, ms | 3.024,637 | 1.184,567 |
| Peak RSS, MiB | 155,29 | 151,58 |

Output tokened persis sama, fingerprint `32c0ec33ef1164c5d971b66a16d6cd74ab842e6155053b5a34e48a59fc93f095`. Tanpa token: 114 GET, 7 POST, 221.906 byte, 1.275,149 ms, peak 152,09 MiB; himpunan sel lengkap sama walau urutan antar tab berbeda. Setiap probe memakai RLIMIT_AS 1536 MiB. Tidak ada MemoryMax/cap API/pacing diubah.

Detail per tahap dan JSON terdapat di [PERFORMANCE_V421.md](PERFORMANCE_V421.md). Source Google network, cold registration dan 7.936 target produksi tidak diukur. Waktu fixture menghilangkan jaringan/pacing; tidak boleh dianggap waktu live. **<60 detik belum terbukti**, dan default pacing 1,2 detik untuk 78 destination requests saja memerlukan sekitar 92,4 detik jarak start. Global scan yang tidak aman dieliminasi tetap dipertahankan. Sheets tidak menawarkan compare-and-set pada celah GET→POST; batas tersebut tetap ada.

## Distribusi dan penerimaan

ZIP source lengkap + public prebuilt, docs/test/migrations/dependency locks dan safe upgrade/rollback. Schema dan mapping READY dipertahankan; worker tetap STOPPED/disabled. Lihat [UPGRADE_V421.md](UPGRADE_V421.md), [CHANGED_FILES_V421.md](CHANGED_FILES_V421.md), [ROOT_CAUSE_V421.md](ROOT_CAUSE_V421.md) dan [INVARIANTS_V421.md](INVARIANTS_V421.md).

Final artifact diperiksa CRC, aset, scope terhadap ZIP V4.2 dan private-state exclusions. Uji extraction menjalankan subset writer/recovery/upgrade serta Alembic head/check dengan database terisolasi. Checksum rilis terpisah; publication memakai commit terpisah dan mempertahankan archive lama. Fresh Git fetch/HTTP download memverifikasi byte ZIP yang sama. Acceptance Google/VPS tetap manual dan belum dijalankan dari cloud ini.
