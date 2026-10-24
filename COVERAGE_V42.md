# Bukti cakupan 80 persyaratan regresi V4.2

Nomor mengikuti §21 spesifikasi. Setiap baris menunjuk test nyata; parameter test mencakup beberapa variasi. Semua 198 kasus V4.1 dipertahankan; perubahan assertion pada transport timeout/resync dijelaskan dalam TEST_REPORT.md. Pengujian Google nyata tetap pending.

| No. | Persyaratan | Bukti test (backend/tests/) |
|---:|---|---|
| 1 | One selected date produces all applicable metrics from one source snapshot. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_one_complete_snapshot_serves_every_normal_metric` |
| 2 | Correct fixed DOC DEPO columns. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_snapshot_keeps_confirmed_fixed_source_columns` |
| 3 | Correct fixed DOC QRIS columns. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_snapshot_keeps_confirmed_fixed_source_columns` |
| 4 | Empty date with carried-forward balance. | [test_parser.py](../backend/tests/test_parser.py) · `test_empty_day_and_carry_forward` |
| 5 | Selected-date and following-date separator rules. | [test_v4_access.py](../backend/tests/test_v4_access.py) · `test_bounded_reads_match_v3_financial_parser` |
| 6 | Forward index continuation from a verified separator. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_forward_and_backward_date_index_uses_verified_boundaries` |
| 7 | Backward-date selection. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_forward_and_backward_date_index_uses_verified_boundaries` |
| 8 | Appended source rows. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_changed_revision_invalidates_unsafe_rows_then_rebuilds` |
| 9 | Inserted historical rows invalidate unsafe index assumptions. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_changed_revision_invalidates_unsafe_rows_then_rebuilds` |
| 10 | Missing and duplicate separators. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_index_does_not_guess_missing_duplicate_or_invalid_separators` |
| 11 | No whole-grid source materialization. | [test_v4_access.py](../backend/tests/test_v4_access.py) · `test_bounded_reads_match_v3_financial_parser` |
| 12 | Bounded Google API requests. | [test_v4_access.py](../backend/tests/test_v4_access.py) · `test_bounded_reads_match_v3_financial_parser` |
| 13 | Newly discovered inactive tab. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_new_source_tab_is_inactive_and_does_not_rebuild_destination` |
| 14 | Explicitly activated new tab. | [test_api.py](../backend/tests/test_api.py) · `test_months_revisions_validation_tabs` |
| 15 | Confirmed tab rename. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_proven_rename_preserves_account_coordinates_and_transaction_lineage` |
| 16 | Deleted and recreated tab with the same name. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_deleted_and_recreated_same_name_requires_revalidation_and_new_identity` |
| 17 | Same account but different source identity. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_deleted_and_recreated_same_name_requires_revalidation_and_new_identity` |
| 18 | Preserved transaction ownership across a proven rename. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_proven_rename_preserves_account_coordinates_and_transaction_lineage` |
| 19 | No duplicate writes caused by rename. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_proven_rename_preserves_account_coordinates_and_transaction_lineage` |
| 20 | New monthly URL triggers mapping. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_priority_retry_during_active_run_at_safe_boundaries` |
| 21 | Full horizontal website discovery. | [test_v41_destination.py](../backend/tests/test_v41_destination.py) · `test_numeric_dates_similar_neighbors_and_merged_headers_cross_chunk` |
| 22 | Far-right website blocks. | [test_v41_destination.py](../backend/tests/test_v41_destination.py) · `test_numeric_dates_similar_neighbors_and_merged_headers_cross_chunk` |
| 23 | Multiple websites with similar account headers. | [test_v41_destination.py](../backend/tests/test_v41_destination.py) · `test_numeric_dates_similar_neighbors_and_merged_headers_cross_chunk` |
| 24 | Absolute coordinates preserved. | [test_v41_destination.py](../backend/tests/test_v41_destination.py) · `test_all_22_golden_writes_on_large_sparse_destination_and_reconciliation` |
| 25 | Source tab rename does not change destination website/date mapping. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_proven_rename_preserves_account_coordinates_and_transaction_lineage` |
| 26 | Destination URL changed after mapping. | [test_reconciliation.py](../backend/tests/test_reconciliation.py) · `test_url_change_preserves_history_requires_explicit_resync` |
| 27 | Old Run remains pinned to the old destination. | [test_reconciliation.py](../backend/tests/test_reconciliation.py) · `test_url_change_preserves_history_requires_explicit_resync` |
| 28 | Missing/ambiguous destination headers. | [test_v41_destination.py](../backend/tests/test_v41_destination.py) · `test_old_ambiguity_and_unregistered_boundaries_match` |
| 29 | Stale destination mapping blocks unsafe writes. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_stale_destination_registry_blocks_and_never_remaps` |
| 30 | Revalidation produces a new mapping version without deleting historical ownership. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_mapping_revalidation_preserves_all_historical_ledgers` |
| 31 | Valid FEE QRIS DEPO destination header. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_fee_headers_are_scoped_and_merged_origin_is_unique` |
| 32 | Valid FEE QRIS WD destination header. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_fee_headers_are_scoped_and_merged_origin_is_unique` |
| 33 | Merged KETERANGAN header. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_fee_headers_are_scoped_and_merged_origin_is_unique` |
| 34 | Website-boundary protection. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_fee_headers_are_scoped_and_merged_origin_is_unique` |
| 35 | Multiple fee entries. | [test_parser.py](../backend/tests/test_parser.py) · `test_multiple_fee_descriptions_preserve_newlines_and_sum_recorded_only` |
| 36 | No fee in source: valid no-op. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_no_fee_is_noop_or_explicit_old_fee_conflict` |
| 37 | Source read failure is not interpreted as no fee. | [test_v42_sources.py](../backend/tests/test_v42_sources.py) · `test_incomplete_snapshot_is_not_consumed_and_retry_fetches_raw` |
| 38 | Existing old fee with no new fee creates an actionable conflict. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_no_fee_is_noop_or_explicit_old_fee_conflict` |
| 39 | Admin-authorized fee correction. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_no_fee_is_noop_or_explicit_old_fee_conflict` |
| 40 | Formula and Notes remain protected. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_manual_note_is_preserved_when_only_value_is_authorized` |
| 41 | One failed item retried while 42 others exist. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_single_failed_item_among_42_successes_uses_fresh_source_only` |
| 42 | Other successful items remain unchanged. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_single_failed_item_among_42_successes_uses_fresh_source_only` |
| 43 | Retry reads current source raw data. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_single_failed_item_among_42_successes_uses_fresh_source_only` |
| 44 | Retry does not reuse the previous snapshot as financial truth. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_single_failed_item_among_42_successes_uses_fresh_source_only` |
| 45 | Retry does not rebuild destination mapping. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_priority_retry_during_active_run_at_safe_boundaries` |
| 46 | Retry while original Run is active. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_priority_retry_during_active_run_at_safe_boundaries` |
| 47 | Retry added while another item is writing. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_priority_retry_during_active_run_at_safe_boundaries` |
| 48 | Retry double-click is idempotent. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_durable_retry_queue_lifecycle` |
| 49 | Retry queue survives worker restart. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_durable_retry_queue_lifecycle` |
| 50 | No concurrent duplicate execution. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_durable_retry_queue_lifecycle` |
| 51 | Existing dependent aggregate is handled safely. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_successful_expense_total_requires_explicit_dependency_retry` |
| 52 | Existing prepared writes remain protected. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_prepared_recovery_never_accepts_changed_global_formula` |
| 53 | Old value differs from proposed value. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_exact_google_values_and_provenance_are_exposed` |
| 54 | Display exact existing and proposed values. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_exact_google_values_and_provenance_are_exposed` |
| 55 | `Masukkan Data Baru` after revalidation. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_new_value_requires_decision_and_clears_only_after_verified` |
| 56 | `Tetapkan Data Lama` preserves the destination. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_keep_old_never_writes_or_forces_balance_zero` |
| 57 | Conflict resolution clears only the corresponding active error. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_new_value_requires_decision_and_clears_only_after_verified` |
| 58 | Historical audit remains. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_new_value_requires_decision_and_clears_only_after_verified` |
| 59 | Keeping old data does not force KLOP. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_keep_old_never_writes_or_forces_balance_zero` |
| 60 | Formula cannot be overwritten merely through conflict approval. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_formula_cannot_be_overwritten_even_by_admin_decision` |
| 61 | Conflict changes between display and approval. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_stale_approval_is_rejected_and_conflict_refreshed` |
| 62 | Multi-cell fee conflicts. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_fee_description_and_amount_are_one_authorized_group` |
| 63 | Multi-cell transfer and Note conflicts. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_changed_transfers_correct_existing_group_without_appending` |
| 64 | Timeout after an actual Google write. | [test_v42_conflicts.py](../backend/tests/test_v42_conflicts.py) · `test_timeout_after_google_accepts_group_is_verified_without_second_post` |
| 65 | DOC DEPO transfer priority. | [test_engine.py](../backend/tests/test_engine.py) · `test_qris_overflow_uses_mandiri_after_depo_priority` |
| 66 | QRIS bank priority. | [test_engine.py](../backend/tests/test_engine.py) · `test_qris_overflow_uses_mandiri_after_depo_priority` |
| 67 | Brown and blue classification. | [test_parser.py](../backend/tests/test_parser.py) · `test_blue_is_excluded_and_unapproved_fails` |
| 68 | Transfer slot occupancy. | [test_engine.py](../backend/tests/test_engine.py) · `test_qris_overflow_uses_mandiri_after_depo_priority` |
| 69 | Changed transaction identity. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_changed_transfers_correct_existing_group_without_appending` |
| 70 | Duplicate transfer prevention. | [test_engine.py](../backend/tests/test_engine.py) · `test_duplicate_start_and_repeated_run_do_not_append` |
| 71 | Read-only reconciliation. | [test_v41_destination.py](../backend/tests/test_v41_destination.py) · `test_wide_readonly_report_and_stable_zero` |
| 72 | Individual and consolidated BALANCE checks. | [test_reconciliation.py](../backend/tests/test_reconciliation.py) · `test_nonzero_consolidated_prevents_clear` |
| 73 | Cancel while reading. | [test_cancellation.py](../backend/tests/test_cancellation.py) · `test_stop_during_read_starts_no_further_items` |
| 74 | Cancel while retry is pending. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_durable_retry_queue_lifecycle` |
| 75 | Worker interrupted after planned write. | [test_cancellation.py](../backend/tests/test_cancellation.py) · `test_uncertain_write_keeps_prepared_intent_and_can_be_retried` |
| 76 | Worker interrupted after Google accepted write. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_worker_death_after_accepted_write_recovers_exact_intent_without_repost` |
| 77 | API restart. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_api_process_restart_preserves_retry_queue_and_authenticated_session` |
| 78 | Database migration and rollback. | [test_migration.py](../backend/tests/test_migration.py) · `test_v41_to_v42_is_additive_and_backup_rollback_keeps_audit` |
| 79 | Memory-bounded large-source tests. | [test_v4_access.py](../backend/tests/test_v4_access.py) · `test_complete_website_memory_with_three_large_tabs` |
| 80 | Accurate full-website performance instrumentation. | [test_v42_workflow.py](../backend/tests/test_v42_workflow.py) · `test_attempt_telemetry_is_bounded_and_separates_stages` |

Tambahan penting di luar pemetaan satu-bukti: readonly Preview/CSRF/idempotency, worker subprocess kedua ditolak flock, source checksum/incomplete, fallback tanpa Drive, alias priority setelah rename, legacy lineage eksplisit, restored retired transfer slot, KEEP tidak mengubah PREPARED asing/Preview, serta delapan browser cases. Beberapa jaminan seperti formula/Notes diuji lagi di beberapa modul agar tidak hanya bergantung pada satu happy path.
