# Scope dan release gate V4.1

Perbandingan dilakukan terhadap ZIP V4 yang telah diterbitkan, SHA256 `830a1db4beb31d3acca0d387843026fc77c47bcbe1b5a8a0b42b84f3225ed02e`. Full diff ditinjau dalam repository Git temporer terpisah; index/HEAD workspace pengguna tidak diganti.

| Gate | Bukti |
|---|---|
| Tidak menolak tujuan karena dimensi teralokasi | Fixture 1000×1000, 1000×1500, dan 1800 kolom lulus; guard lama hanya ada sebagai oracle offline pada tests |
| Tidak mematerialisasi tujuan/laporan penuh | Reader tidak mempunyai `rows`; indeks struktur di disk privat, page cache 512 KiB, payload biasa dibuang |
| Planning/pre-write/post-write/report | Semuanya memakai reader terbatas; legacy `SheetsClient.grid()` juga mendelegasikan ke reader |
| Koordinat absolut | Column 1421/row 74 diverifikasi BBR75; tidak ada translasi slice pada Write |
| Website/boundary/account/date | Differential V4, 40 blok, website dekat ujung, tetangga mirip, boundary tak terdaftar, marker/tanggal ambigu dan numeric dates |
| KET PENGIRIMAN | Calendar-date dan monthly side-total; bank pairs/merge/TOTAL/slots dan prioritas tetap; golden output/overflow lama tetap lulus |
| Notes/manual data/ownership | Target GET eksplisit, cache tidak menentukan blank; konflik Note/nilai/formula/merge dan ledger PREPARED/VERIFIED diuji |
| Struktur global | Signature JSON stream identik V4 pada fixture; merge di luar area aktif tetap diperiksa |
| Formula global | Semua koordinat/formula masuk canonical signature; perubahan di luar website saat write tetap menggagalkan verifikasi |
| Read-back/Stop | Nilai dan Note dicek ulang; POST yang telah diterima tetap diverifikasi walau cancel tiba |
| Laporan/KLOP/Clear | Read-only, marker vertikal/horizontal, exact zero dan repeated fresh reads tetap berlaku; missing BALANCE bukan nol |
| Memory safety | Cap byte 8 MiB tetap, range 8.000 sel, batch maksimal 64.000 sel/32 ranges dengan split bounded, indeks 64 MiB, cache 128 sel/4 MiB |
| Keuangan/sumber/UI/API/DB/worker | Source/parser, perhitungan, routes/auth, models, worker/queue, semua source frontend dan 59 public assets tidak diubah |
| Deployment | Backup dahulu; antrean lama diaudit dan ditahan; API dimulai, worker manual; rollback tersedia |
| Regresi | 198 backend passed di Python 3.10 dan 3.12; 5 browser passed |
| Live produksi | **Pending operator**; tidak ada akses Google/key/SSH VPS selama pengembangan |

Tidak ada kenaikan `max_destination_cells`, `max_sheet_cells`, cap respons atau MemoryMax, tidak ada perubahan struktur Google Sheets, dan tidak ada penghapusan history/ownership. SQLite pada reader adalah indeks temporer terpisah, bukan perubahan skema/jurnal DB finansial.

Pemindaian global masih diperlukan untuk mempertahankan invariant V4. Batching mengurangi request; tidak ada klaim semua tab produksi akan cepat atau semua ukuran workbook telah terbukti. Metadata merge Google tidak dapat dipaginasi; bila vektor koordinat melebihi cap HTTP, operasi gagal secara eksplisit. Lihat [PERFORMANCE_V41.md](PERFORMANCE_V41.md) dan [UPGRADE_V41.md](UPGRADE_V41.md).
