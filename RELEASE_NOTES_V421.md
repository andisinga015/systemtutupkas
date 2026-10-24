# V4.2.1 — 25 Oktober 2026

Patch destination writer untuk Run yang sudah memiliki snapshot/rencana dan monthly mapping READY. Preflight/readback sekarang dibagikan per window bounded, batchUpdate menggabungkan kelompok independen, dan proof global dapat menghindari Note/background body yang tidak berpartisipasi pada invariant. Koordinat, angka, deskripsi, Notes, konflik dan ownership V4.2 tetap sama. Global proof yang tidak aman dieliminasi tetap memakai fallback lengkap.

Fixture 8 snapshot/43 item/57 sel: GET tujuan **660 → 77**, POST **36 → 1**, byte respons **100.971.103 → 117.986**. Output persis sama dengan source V4.2 asli. Tidak ada OOM di fixture. Angka live Google, quota dan target <60 detik belum terbukti; pacing default 1,2 detik dipertahankan.

Upgrade mempertahankan schema `c92448ce233b` dan READY mapping tanpa revalidation/rebuild. Backup konsisten, cancellation diaudit, verifikator protected rows dan rollback disertakan. Worker tetap berhenti/disabled setelah upgrade. Tidak ada perubahan source indexing/snapshots/registration/auth/UI/concurrency, cap API atau MemoryMax.

Baca [UPGRADE_V421.md](UPGRADE_V421.md), [ROOT_CAUSE_V421.md](ROOT_CAUSE_V421.md), [INVARIANTS_V421.md](INVARIANTS_V421.md), [PERFORMANCE_V421.md](PERFORMANCE_V421.md) dan [TEST_REPORT.md](TEST_REPORT.md). Laporan rilis sebelumnya tetap tersimpan.
