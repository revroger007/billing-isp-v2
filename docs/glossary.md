# Glosarium identifier

Semua identifier kode dan database memakai bahasa Inggris sesuai `AGENTS.md` dan keputusan 004. Bahasa Indonesia hanya untuk label UI, pesan ke pengguna, dan prosa dokumen. Tabel ini memetakan nama lama di keputusan 001–003 dan `CHANGELOG.md` ke nama yang berlaku. Dokumen lama tidak ditulis ulang.

Nama tabel memakai bentuk jamak bahasa Inggris sesuai konvensi Laravel. Kolom waktu memakai akhiran `_at` untuk timestamp dan `_on` untuk tanggal tanpa jam.

## Tabel

| Lama | Baru |
|---|---|
| `KECAMATAN` | `districts` |
| `DESA` | `villages` |
| `DUSUN` | `hamlets` |
| `ODP` | `odps` |
| `OLT` | `olts` |
| `PELANGGAN` | `customers` |
| `PERANGKAT_INVENTARIS` | `devices` |
| `RIWAYAT_SUSPEND` | `customer_status_periods` |
| `RIWAYAT_PENGGANTIAN_ALAT` | `device_replacements` |
| `REQUEST_PERUBAHAN_WIFI` | `wifi_change_requests` |
| `BERITA_ACARA_INSTALASI` | `installation_reports` |
| `DOMPET_PELANGGAN_MUTASI` | `customer_wallet_entries` |
| `POIN_MUTASI` | `point_entries` |
| `REQUEST_APPROVAL` | `approval_requests` |
| `PELANGGAN_MIKROTIK` | `customer_router` |
| `MIKROTIK` | `routers` |
| `PAKET` | `packages` |
| `HARGA_PAKET_WILAYAH` | `package_area_prices` |
| `TAGIHAN` | `invoices` |
| `BARIS_TAGIHAN` | `invoice_lines` |
| `HUTANG_MENGENDAP` | `deferred_balances` |
| `PEMBAYARAN` | `payments` |
| `ALOKASI_PEMBAYARAN` | `payment_allocations` |
| `KOLEKTOR` | `collectors` |
| `PERIODE_KAS` | `cash_periods` |
| `MUTASI_KAS` | `cash_entries` |
| `SETORAN` | `cash_deposits` |
| `RIWAYAT_ISOLIR` | `disconnections` |
| `JENDELA_PENUNDAAN` | `suspension_deferrals` |
| `PENGAJUAN_PENUNDAAN` | `deferral_requests` |
| `LOG_API_MIKROTIK` | `router_command_logs` |
| `AGENDA_GANGGUAN_MASSAL` | `outages` |
| `AGENDA_TEKNISI_ASSIGNMENT` | `outage_assignments` |
| `TIKET_KOMPLAIN` | `complaint_tickets` |
| `TIKET_TEKNISI_ASSIGNMENT` | `ticket_assignments` |
| `ABSENSI` | `attendances` |
| `PAYROLL_RECORD` | `payroll_records` |
| `USER_WILAYAH` | `user_areas` |
| `AUDIT_LOG` | `audit_logs` |
| `USERS` | `users` |
| `RESELLER` | `resellers` |
| `DOMPET_RESELLER_MUTASI` | `reseller_wallet_entries` |
| `PENCAIRAN_RESELLER` | `reseller_withdrawals` |

Tabel `package_area_prices`, `deferral_requests`, dan kolom `is_irregular_payer` baru di keputusan 004. Nama lamanya hanya muncul di commit perantara.

## Action dan izin

| Lama | Baru |
|---|---|
| `hitung_isolir_berikutnya` | `CalculateNextSuspensionDate` |
| `alokasi_pembayaran` | `AllocatePayment` |
| `putuskan_layanan` | `DisconnectService` |
| `nyalakan_layanan` | `ReconnectService` |
| `catat_pembayaran` | `RecordPayment` |
| `koreksi_keuangan` | `finance_correction` |
| `auditor_finansial` | `finance_auditor` |
| `wajib_verifikasi_transfer` | `transfer_verification_mode` |

## Kolom

| Lama | Baru |
|---|---|
| `alamat_detail` | `address_detail` |
| `dusun_id` | `hamlet_id` |
| `desa_id` | `village_id` |
| `kecamatan_id` | `district_id` |
| `tipe_sumber_koneksi` | `connection_source_type` |
| `catatan_sumber_koneksi` | `connection_source_note` |
| `parent_pelanggan_id` | `parent_customer_id` |
| `no_hp` | `phone` |
| `no_pelanggan` | `customer_number` |
| `paket_id` | `package_id` |
| `batas_tunggakan` | `arrears_limit` |
| `siklus_tagihan` | `billing_cycle` |
| `status_layanan` | `service_status` |
| `sumber_putus` | `disconnect_source` |
| `sumber_nyala` | `reconnect_source` |
| `isolir_berlaku_mulai` | `suspension_effective_from` |
| `next_isolir` | `next_suspension_date` |
| `tanggal_instalasi` | `installed_at` |
| `status_langganan` | `subscription_status` |
| `last_suspend` | `last_suspended_at` |
| `last_suspend_note` | `last_suspended_note` |
| `last_active` | `last_activated_at` |
| `last_active_note` | `last_activated_note` |
| `tanggal_daftar` | `registered_at` |
| `tanda_transfer` | `transfer_flag` |
| `tidak_tertib` | `is_irregular_payer` |
| `catatan_tidak_tertib` | `irregular_payer_note` |
| `pelanggan_id` | `customer_id` |
| `kode_odp` | `code` |
| `nama_odp` | `name` |
| `kapasitas_port` | `port_capacity` |
| `foto_tiang` | `pole_photo` |
| `diusulkan_oleh_user_id` | `proposed_by` |
| `approved_by_user_id` | `approved_by` |
| `nama_olt` | `name` |
| `tipe_merk` | `brand_model` |
| `tipe_merk_olt` | `olt_brand_model` |
| `jenis_perangkat` | `device_type` |
| `password_wifi` | `wifi_password` |
| `username_koneksi` | `connection_username` |
| `password_koneksi` | `connection_password` |
| `foto_instalasi` | `installation_photo` |
| `file_backup_config` | `config_backup_file` |
| `posisi_user_id` | `holder_user_id` |
| `username_biasa` | `standard_username` |
| `password_biasa` | `standard_password` |
| `catatan_mulai` | `start_note` |
| `catatan_selesai` | `end_note` |
| `dibuat_oleh` | `created_by` |
| `bukti_percakapan` | `conversation_proof` |
| `perangkat_lama_id` | `old_device_id` |
| `perangkat_baru_id` | `new_device_id` |
| `teknisi_user_id` | `technician_user_id` |
| `tiket_komplain_id` | `complaint_ticket_id` |
| `tanggal_penggantian` | `replaced_at` |
| `requested_by_user_id` | `requested_by` |
| `ssid_baru` | `new_ssid` |
| `password_baru` | `new_password` |
| `perangkat_id` | `device_id` |
| `nomor_dokumen` | `document_number` |
| `tanda_tangan_pelanggan` | `customer_signature` |
| `tanda_tangan_teknisi` | `technician_signature` |
| `file_pdf` | `pdf_file` |
| `disahkan_pada` | `signed_at` |
| `saldo_akhir` | `balance_after` |
| `referensi_id` | `reference_id` |
| `dibuat_pada` | `created_at` |
| `jumlah_poin` | `points` |
| `jenis_request` | `request_type` |
| `mikrotik_id` | `router_id` |
| `tipe_koneksi` | `connection_type` |
| `username_mk` | `router_username` |
| `nama_lokasi` | `location_name` |
| `port_api` | `api_port` |
| `username_api` | `api_username` |
| `password_api` | `api_password` |
| `nama_address_list` | `address_list_name` |
| `status_koneksi` | `connection_status` |
| `no_tagihan` | `invoice_number` |
| `tipe_tagihan` | `invoice_type` |
| `nominal_dipindah` | `moved_amount` |
| `jatuh_tempo` | `due_date` |
| `layanan_mulai` | `service_starts_on` |
| `layanan_selesai` | `service_ends_on` |
| `periode_penuh` | `is_full_period` |
| `kategori_tunggakan` | `arrears_category` |
| `status_penagihan` | `collection_status` |
| `catatan_penagihan` | `collection_note` |
| `terakhir_dikunjungi_at` | `last_visited_at` |
| `tagihan_id` | `invoice_id` |
| `tanggal_mulai` | `starts_on` |
| `tanggal_selesai` | `ends_on` |
| `ditutup_oleh` | `closed_by` |
| `ditutup_pada` | `closed_at` |
| `dibuka_kembali_oleh` | `reopened_by` |
| `dibuka_kembali_pada` | `reopened_at` |
| `diedit_oleh` | `edited_by` |
| `diedit_pada` | `edited_at` |
| `sumber_id` | `source_id` |
| `metode_bayar` | `payment_method` |
| `bukti_transfer` | `transfer_proof` |
| `status_verifikasi` | `verification_status` |
| `status_bukti` | `proof_status` |
| `jumlah_periode` | `period_count` |
| `paid_through_sebelum` | `paid_through_before` |
| `paid_through_sesudah` | `paid_through_after` |
| `tanggal_bayar` | `paid_at` |
| `pembayaran_id` | `payment_id` |
| `level_wewenang` | `authority_level` |
| `batas_di_tangan` | `cash_on_hand_limit` |
| `total_dari_pelanggan` | `total_from_customers` |
| `total_dari_setoran` | `total_from_deposits` |
| `total_disetor` | `total_deposited` |
| `sisa_di_tangan` | `cash_on_hand` |
| `disetor_pada` | `deposited_at` |
| `diterima_oleh` | `received_by` |
| `periode_kas_id` | `cash_period_id` |
| `setoran_id` | `cash_deposit_id` |
| `lawan_user_id` | `counterparty_user_id` |
| `lawan_pelanggan_id` | `counterparty_customer_id` |
| `diajukan_pada` | `submitted_at` |
| `dikonfirmasi_pada` | `confirmed_at` |
| `mulai_putus` | `disconnected_at` |
| `nyala_kembali` | `reconnected_at` |
| `jendela_penundaan_id` | `suspension_deferral_id` |
| `disetujui_oleh` | `approved_by` |
| `pengajuan_penundaan_id` | `deferral_request_id` |
| `batas_diminta` | `requested_ends_on` |
| `diajukan_oleh` | `requested_by` |
| `diproses_oleh` | `processed_by` |
| `alasan_tolak` | `rejection_reason` |
| `diproses_pada` | `processed_at` |
| `jumlah_tunggakan` | `arrears_count` |
| `trigger_by` | `triggered_by` |
| `status_api` | `api_status` |
| `jumlah_percobaan` | `attempt_count` |
| `hari_mulai_putus` | `disconnect_date` |
| `kode_agenda` | `code` |
| `nama_area` | `area_name` |
| `tipe_cakupan` | `scope_type` |
| `cakupan_id` | `scope_id` |
| `deskripsi_gangguan` | `description` |
| `dibuat_oleh_user_id` | `created_by` |
| `target_monitor_tipe` | `monitor_target_type` |
| `target_mikrotik_id` | `target_router_id` |
| `target_pelanggan_id` | `target_customer_id` |
| `waktu_mulai` | `started_at` |
| `waktu_selesai` | `ended_at` |
| `catatan_solusi` | `resolution_note` |
| `foto_bukti_selesai` | `resolution_photo` |
| `agenda_gangguan_id` | `outage_id` |
| `nama_tim` | `team_name` |
| `no_tiket` | `ticket_number` |
| `pelapor_user_id` | `reporter_user_id` |
| `kategori_gangguan` | `issue_category` |
| `foto_kendala` | `issue_photo` |
| `tipe_kasus` | `case_type` |
| `verified_by_user_id` | `verified_by` |
| `solusi_teknisi` | `technician_resolution` |
| `foto_penyelesaian` | `resolution_photo` |
| `is_penanggung_jawab` | `is_lead` |
| `jam_masuk` | `check_in_time` |
| `jam_pulang` | `check_out_time` |
| `latitude_masuk` | `check_in_latitude` |
| `longitude_masuk` | `check_in_longitude` |
| `foto_masuk` | `check_in_photo` |
| `gaji_pokok` | `base_salary` |
| `bonus_pekerjaan` | `work_bonus` |
| `uang_kehadiran` | `attendance_allowance` |
| `total_bersih` | `net_total` |
| `status_bayar` | `payment_status` |
| `dibayar_at` | `paid_at` |
| `nama_bank` | `bank_name` |
| `no_rekening` | `account_number` |
| `atas_nama` | `account_holder` |
| `saldo_dompet` | `wallet_balance` |
| `pencairan_id` | `withdrawal_id` |
| `basis_poin` | `commission_bps` |

## Kolom satu kata

| Lama | Baru |
|---|---|
| `nama` | `name` |
| `harga` | `price` |
| `kecepatan` | `speed` |
| `periode` | `period` |
| `nominal` | `amount` |
| `terbayar` | `paid_amount` |
| `sisa` | `outstanding_amount` |
| `jenis` | `type` |
| `tipe` | `type` |
| `arah` | `direction` |
| `hari` | `days` |
| `alasan` | `reason` |
| `keterangan` | `description` |
| `catatan` | `notes` |
| `sumber` | `source` |
| `poin` | `points` |
| `judul` | `title` |
| `deskripsi` | `description` |
| `tanggal` | `date` |
| `mulai` | `started_at` |
| `selesai` | `ended_at` |
| `batas` | `ends_on` |
| `tujuan` | `destination` |
| `waktu` | `logged_at` |
| `aksi` | `action` |
| `potongan` | `deductions` |
| `kanal` | `channel` |
| `jumlah` (jumlah tunggakan isolir) | `arrears_count` |

Pengecualian per tabel: `suspension_deferrals.mulai` menjadi `starts_on` dan `batas` menjadi `ends_on`. `customer_status_periods.mulai` dan `selesai` menjadi `started_at` dan `ended_at`. `attendances.tanggal` menjadi `attendance_date`.

## Nilai status dan enum

| Lama | Baru |
|---|---|
| `menunggu_verifikasi_pusat` | `pending_approval` |
| `siap_pasang` | `ready_to_install` |
| `tunda_isolir` | `on_hold` |
| `menunggu_router` | `pending_router` |
| `gagal_putus` | `disconnect_failed` |
| `gagal_nyala` | `reconnect_failed` |
| `tidak_serempak` | `out_of_sync` |
| `tanggal_tetap` | `fixed_date` |
| `belum_bayar` | `unpaid` |
| `menunggu_verifikasi` | `pending_verification` |
| `biaya_pasang` | `installation_fee` |
| `biaya_aktivasi` | `activation_fee` |
| `diskon_isolir` | `suspension_discount` |
| `konversi_ke_tagihan` | `conversion_to_invoice` |
| `belum_diverifikasi` | `unverified` |
| `sudah_diverifikasi` | `verified` |
| `tidak_wajib` | `not_required` |
| `sering_salah_bukti` | `frequent_invalid_proof` |
| `bukti_palsu` | `fake_proof` |
| `di_tangan` | `on_hand` |
| `belum_ditagih` | `not_collected` |
| `didatangi_belum_bayar` | `visited_unpaid` |
| `didatangi_belum_ketemu` | `visited_not_met` |
| `perpanjangan_tempo_1_bulan` | `one_month_extension` |
| `vip_siklus_non_reguler` | `vip_irregular_cycle` |
| `gudang_utama` | `in_warehouse` |
| `dibawa_teknisi` | `with_technician` |
| `terpasang_pelanggan` | `installed` |
| `ditarik_bekas` | `retrieved` |
| `rusak_servis` | `under_repair` |
| `afkir_musnah` | `scrapped` |
| `menunggu_verifikasi_noc` | `pending_noc_verification` |
| `menunggu_approval_noc` | `pending_noc_approval` |
| `kolektor_mobile` | `collector_mobile` |
| `terima_tunai_pelanggan` | `customer_cash_in` |
| `terima_setoran_kolektor` | `collector_deposit_in` |
| `terima_serah_admin` | `admin_handover_in` |
| `keluar_serah_admin` | `admin_handover_out` |
| `keluar_setoran_bank` | `bank_deposit_out` |
| `keluar_gaji` | `salary_out` |
| `keluar_komisi` | `commission_out` |
| `keluar_operasional` | `operational_out` |
| `aktif` | `active` |
| `isolir` | `suspended` |
| `berhenti` | `terminated` |
| `cuti` | `hold` |
| `nyala` | `connected` |
| `putus` | `disconnected` |
| `lunas` | `paid` |
| `sebagian` | `partial` |
| `batal` | `cancelled` |
| `langganan` | `subscription` |
| `konversi` | `conversion` |
| `toleransi` | `tolerance` |
| `perpanjangan` | `extension` |
| `pembayaran` | `payment` |
| `prabayar` | `prepaid` |
| `masuk` | `in` |
| `keluar` | `out` |
| `mengendap` | `deferred` |
| `disetor` | `deposited` |
| `menunggu` | `pending` |
| `disetujui` | `approved` |
| `ditolak` | `rejected` |
| `sukses` | `success` |
| `gagal` | `failed` |
| `wajib` | `required` |
| `kasir` | `cashier` |
| `kolektor` | `collector` |
| `kios` | `kiosk` |
| `dikerjakan` | `in_progress` |
| `koreksi` | `correction` |

Nilai `isolir` pada `customer_status_periods.type` dan `subscription_status` menjadi `suspended`. Nama address-list di Mikrotik, contohnya `isolir`, adalah konfigurasi router dan tidak diganti.

## Variabel rumus dan kolom impor

| Lama | Baru |
|---|---|
| `harga_paket` | `package_price` |
| `jumlah_hari_periode` | `period_days` |
| `hari_nyala_belum_dibayar` | `unpaid_connected_days` |
| `paid_through_baru` | `new_paid_through` |
| `paid_through_lama` | `old_paid_through` |
| `no_tagihan_lama` | `legacy_invoice_number` |
| `periode_bulan` | `period_month` |
| `tanggal_bayar_lama` | `legacy_paid_at` |
| `link_portal` | `portal_link` |

## Nama yang tidak dipakai lagi

`isolir_enabled`, `bypass_until`, `isolir_sementara`, `masa_aktif_hingga`, `prepaid_30hari`, `hari_isolir`, `cuti_mulai`, dan `cuti_selesai` sudah dihapus dari rancangan dan tidak punya padanan.
