# Cakupan 79 persyaratan V4.2.2

Seluruh 343 kasus V4.2.1 dipertahankan; input fixture/API yang kini wajib period dibuat eksplisit tanpa mengubah financial golden/assertions. S = `backend/tests/test_v422_periods.py`; M = `backend/tests/test_v422_migration.py`; B = `frontend/e2e/v422.spec.ts`. Kasus lama engine/source/identity/writer/reconciliation tetap berjalan pada suite penuh. Beberapa assertion terkait sengaja berbagi skenario end-to-end.

| No. | Bukti otomatis |
|---:|---|
| 1 | B three URL cards: DOC A tanggal 1–15 |
| 2 | B three URL cards: DOC B dan actual last day |
| 3 | B three URL cards: satu QRIS bulanan |
| 4 | B save tiga URL nyata; S edit/validate/refresh parameter semua period |
| 5 | B/S edit A tidak mengubah B |
| 6 | B/S edit B tidak mengubah A |
| 7 | B/S edit QRIS tidak mengubah DEPO |
| 8 | S identical save menghasilkan doc/revision sama; B tombol unchanged disabled |
| 9 | B month/year switch, independent November dan leap-year February |
| 10 | B card history menuju doc ID tepat; S history metadata period |
| 11 | B selector A/B/QRIS ketiganya terlihat |
| 12 | B A memuat KHUSUS A saja |
| 13 | B B memuat KHUSUS B saja |
| 14 | B QRIS memuat E-POWER/QRIZZ saja |
| 15 | S/B nama SAMA pada A/B, identity/record berbeda |
| 16 | S activation scope; B actual server state A tetap |
| 17 | B bulk deactivate B, A tetap |
| 18 | B QRIS deactivate satu, DEPO tetap |
| 19 | B A→B clears checkboxes dan pilihan action |
| 20 | B authoritative states/counts setelah switch/mutation |
| 21 | B bulan/tahun membersihkan selection |
| 22 | B rapid A→B dan A→B→A |
| 23 | B delayed response, transport sengaja mengabaikan abort; request sequence tetap menolak stale |
| 24 | B missing DOC B menunjukkan status khusus |
| 25 | B missing B tidak menampilkan A; S missing-half menolak Run |
| 26 | S new tab inactive; kasus V4.2 identity/new tab lama |
| 27 | S November baru inactive; B November counts independen |
| 28 | S/B active states tidak disalin A/B |
| 29 | S parameter FIRST_HALF save/validate/refresh scoped; source/tab history guard |
| 30 | S parameter SECOND_HALF scoped |
| 31 | S parameter MONTHLY scoped |
| 32 | B/S bulk activation doc ID tepat |
| 33 | B/S bulk deactivation doc ID tepat |
| 34 | S cross-document ID 409; B crafted real API request 409 |
| 35 | S stale GET/toggle/refresh/validate 409; B old-revision tab IDs ditolak |
| 36 | S current-revision counts/dashboard; B counts setiap pilihan |
| 37 | Test V4.2 proven rename: lineage/history/account coordinates tetap; nama baru ditinjau/diaktifkan Admin |
| 38 | Test V4.2 deleted/recreated identity tetap blocked dan tidak aktif otomatis |
| 39 | S calendar day 1 → FIRST_HALF |
| 40 | S calendar day 15 → FIRST_HALF |
| 41 | S calendar day 16 → SECOND_HALF |
| 42 | S calendar day 18 → SECOND_HALF |
| 43 | S February 28 → SECOND_HALF |
| 44 | S leap February 29 → SECOND_HALF |
| 45 | S April 30 → SECOND_HALF |
| 46 | S October 31 → SECOND_HALF |
| 47 | S semua routing memilih MONTHLY yang sama |
| 48 | S required half missing; pesan A/B, tanpa Run/ledger |
| 49 | S no usable active/inactive/identity-conflict, regression engine inactive tabs |
| 50 | S financial Run A10/B18 dan performance dengan 10 unused active A tabs |
| 51 | S sources tepat satu DEPO sebelum snapshot/items |
| 52 | M populated legacy DEPO → UNASSIGNED |
| 53 | S explicit legacy assignment FIRST_HALF |
| 54 | S explicit legacy assignment SECOND_HALF |
| 55 | S/M source ID sama, FK graph utuh |
| 56 | M semua old columns/revisions tetap; S history dan assignment tidak membuat revision baru |
| 57 | S assignment mempertahankan state aktif maupun disabled; M preserved rows |
| 58 | S hanya satu original DEPO record setelah assignment, tidak ada clone tabs |
| 59 | M QRIS MONTHLY; S other source settings/history unchanged |
| 60 | S legacy JSON Run masih GET/readable tanpa mutation; M histories preserved |
| 61 | M ledger/ownership/lineage/Notes/snapshots/data exact, PREPARED tidak diakui |
| 62 | M consistent backup restore; failure transaction rollback, completed-schema resume, destructive downgrade refused |
| 63 | S financial Run tanggal 10 hanya A |
| 64 | S financial Run tanggal 18 hanya B |
| 65 | S kedua tanggal memakai monthly QRIS |
| 66 | S 18 item atau 43 item sesuai fixture terpilih; unused doc ID tidak punya item |
| 67 | S BIAYA 5.500 pada selected input saja; golden writer 43-item sama |
| 68 | S independent date index rows 290 vs 340 dengan token sama tetapi doc/identity berbeda |
| 69 | S selective fresh retry DEPO A10 |
| 70 | S selective fresh retry DEPO B18 |
| 71 | S selective fresh retry QRIS A10/B18 |
| 72 | S preserved successful result JSON; V4.2 retry regression 42 successful extras |
| 73 | S mapping IDs/state/evidence unchanged; byte audit destination registry |
| 74 | S performance gate 77 GET/1 POST/117.986 bytes, complete V4.2.1 batch writer byte-identik |
| 75 | Semua financial golden V4.2.1 lama + exact-output oracle benchmark |
| 76 | Suite V4.2.1 manual Note/formula/merge/NEW/KEEP/readback guards tetap |
| 77 | Suite V4.2.1 KET ownership/slot/DEPO priority/QRIS/crash-idempotency tetap |
| 78 | Suite report read-only/true KLOP/Clear dan KEEP mismatch tetap; engine/report byte-identik |
| 79 | S Crawl All: FANTASI missing B gagal konfigurasi, VELBETT memakai miliknya dan tetap diproses |

Tambahan: request metadata yang kembali sesudah URL/revision berubah ditolak; total BIAYA legacy tidak boleh melewati penetapan periode; source signature tidak stale karena edit half yang tidak digunakan; website navigation tidak mempertahankan selected IDs; database constraints menolak semua kombinasi invalid dan slot duplikat.

Hasil suite/runtime, build/browser, benchmark, schema dan batas live dicatat pada [TEST_REPORT.md](TEST_REPORT.md). Pemetaan ini bukan klaim pengujian Google/VPS asli.
