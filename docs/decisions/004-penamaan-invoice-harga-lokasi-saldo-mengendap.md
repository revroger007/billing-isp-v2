# 004 — Identifier bahasa Inggris, tanda tidak tertib manual, kunci invoice, harga per lokasi, saldo mengendap tanpa kolom saldo

## Konteks

Review 7 November 2026 menyisakan beberapa temuan:

- Tanda tidak tertib dihitung otomatis dari sisa tagihan nonbulanan. Sejak keputusan 002, setiap pelanggan baru langsung bertanda pada hari pemasangan karena tagihan `installation_fee` berjatuh tempo tanggal terbit.
- `AGENTS.md` mewajibkan identifier berbahasa Inggris, tetapi ERD berbahasa Indonesia dan sebagian identifier mencampur dua bahasa.
- Invoice dapat diedit dan dihapus, termasuk statusnya. Sejak keputusan 003, isolir `fixed_date` dibaca dari `due_date`, sehingga edit jatuh tempo menggeser isolir.
- Harga paket hanya satu per paket. Kecepatan paket tidak disinkronkan dengan queue Mikrotik.
- Saldo mengendap memakai `balance_after` per baris yang harus dihitung ulang setiap kali record diedit atau dihapus.
- Rekonsiliasi address-list router dengan database belum ada.

## Keputusan

Keputusan ini menggantikan aturan 002 dan 003 tentang tanda tidak tertib otomatis, serta aturan dokumen lama tentang edit invoice dan `balance_after` saldo mengendap.

- **Penamaan.** Semua identifier tabel, kolom, nilai status, action, peran, dan izin memakai bahasa Inggris. Bahasa Indonesia hanya untuk label UI, pesan ke pengguna, dan prosa dokumen. Padanan nama lama ada di `docs/glossary.md`. Keputusan 001–003 dan `CHANGELOG.md` tidak ditulis ulang.
- **Tanda tidak tertib.** `customers.is_irregular_payer` dipasang dan dicabut manual oleh Admin Global, Admin Wilayah, collector, atau `finance_auditor`, dengan `irregular_payer_note` wajib. Admin Wilayah dan collector hanya pada dusun di `user_areas`. Sistem tidak memasang tanda ini. Tanda tidak mempengaruhi isolir.
- **Invoice terkunci.** Invoice dengan `paid_amount > 0`, `moved_amount > 0`, atau status `paid`, `pending_verification`, atau `cancelled` tidak dapat diedit, dibatalkan, atau dihapus.
- **Invoice yang belum dibayar.** `amount` boleh diubah (lewat `invoice_lines`) oleh Super Admin dan `finance_auditor`. `due_date` boleh diubah oleh Super Admin dan Admin Global, dan pada `fixed_date` perubahan itu menggeser isolir lewat `CalculateNextSuspensionDate`.
- **Pengajuan penundaan.** Collector tidak mengubah jatuh tempo. Collector mengajukan toleransi sementara lewat `deferral_requests`, disetujui Admin Global, dan persetujuan menulis `suspension_deferrals` dengan `reason = tolerance`. Pengajuan perpanjangan satu bulan dan bypass memakai tabel yang sama.
- **Harga per lokasi.** `package_area_prices` menyimpan harga per paket untuk satu desa atau satu dusun. Harga untuk dusun pelanggan dipakai lebih dulu, lalu desanya, lalu `packages.price`. Harga yang berlaku disalin ke `invoice_lines` saat tagihan terbit.
- **Kecepatan paket.** `packages.speed` hanya catatan. Billing tidak membaca atau mengubah queue Mikrotik.
- **Saldo mengendap.** `deferred_balances` tidak punya kolom saldo. Saldo dihitung saat dibutuhkan: jumlah `amount` baris `in` dikurangi jumlah `amount` baris `out`. Super Admin, Admin Global, dan `finance_auditor` dapat menambah, mengedit, dan menghapus record. Audit menyimpan nilai sebelum dan sesudah.
- **Rekonsiliasi router** ditunda. Belum ada perubahan rancangan.

## Asumsi

Diambil tanpa konfirmasi, dan perlu diperiksa:

- "Admin" yang boleh memasang tanda tidak tertib adalah Admin Global dan Admin Wilayah. Super Admin tetap boleh karena memegang akses penuh.
- Status `pending_verification` ikut mengunci invoice, karena ada pembayaran yang sedang diperiksa.
- Status invoice tidak diedit langsung. Status hanya berubah lewat `AllocatePayment`, pembayaran balik, verifikasi bukti, dan aksi membatalkan.
- Invoice yang belum dibayar tetap boleh dibatalkan dan dihapus oleh Super Admin dan `finance_auditor`, seperti aturan sebelumnya.
- `finance_auditor` tidak mengubah `due_date`, karena peran itu tidak mengelola isolir.
- `is_full_period` ditetapkan saat tagihan terbit dan tidak berubah oleh edit nominal.
- "Lokasi" untuk harga adalah desa atau dusun, bukan lokasi router.
- Membuat invoice dari saldo mengendap dan memindahkan sisa invoice ke saldo mengendap tetap hanya untuk Super Admin dan `finance_auditor`.
- Record saldo mengendap yang terkait konversi dengan tagihan yang belum `cancelled` tidak dapat diedit atau dihapus.
- Nilai enum juga berbahasa Inggris. Nama address-list di Mikrotik, contohnya `isolir`, adalah konfigurasi router dan tidak diganti.

## Konsekuensi

- `docs/cara-kerja.md` (versi 2.41) dan `docs/logika-isolir.md` (versi 3.1) memakai identifier bahasa Inggris. `docs/glossary.md` baru.
- Kolom baru: `customers.is_irregular_payer`, `customers.irregular_payer_note`. Tabel baru: `package_area_prices`, `deferral_requests`. Kolom baru `suspension_deferrals.deferral_request_id`. Kolom `deferred_balances.balance_after` dihapus.
- `package_area_prices` dijaga `CHECK` agar tepat satu dari `village_id` dan `hamlet_id` terisi, dengan kunci unik `(package_id, village_id)` dan `(package_id, hamlet_id)`.
- Laporan bulan lalu tetap dapat berubah bila invoice yang belum dibayar diedit atau dihapus, atau bila record saldo mengendap diedit. Audit menjadi satu-satunya jejak perubahan itu.
- Saldo mengendap dibaca dengan `SUM` per pelanggan. Untuk laporan seluruh pelanggan, query perlu index pada `(customer_id, direction)`.
- Setiap model Eloquent memakai nama tabel bawaan Laravel. Model `Customer` memakai `customers` tanpa `$table` eksplisit.
- Rekonsiliasi router tetap menjadi TODO.
