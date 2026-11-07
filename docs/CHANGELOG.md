# Changelog

## 2026-11-07 (keputusan 004)

- Identifier tabel, kolom, nilai status, action, dan izin di `cara-kerja.md` (2.41) dan `logika-isolir.md` (3.1) berbahasa Inggris. Padanan nama lama ada di `docs/glossary.md`. Keputusan 001–003 dan entri changelog lama tetap memakai nama lama.
- `is_irregular_payer` dipasang manual oleh Admin Global, Admin Wilayah, collector, atau `finance_auditor`, dengan catatan wajib. Tidak ada tanda otomatis.
- Invoice yang sudah menerima uang, `paid`, `pending_verification`, atau `cancelled` terkunci. Invoice yang belum dibayar boleh diubah nominalnya oleh Super Admin dan `finance_auditor`, dan jatuh temponya oleh Super Admin dan Admin Global. Status tidak diedit langsung.
- Collector mengajukan toleransi sementara lewat `deferral_requests`. Admin Global menyetujui.
- Harga paket dapat berbeda per desa atau dusun lewat `package_area_prices`. Kecepatan paket hanya catatan.
- `deferred_balances` tidak menyimpan saldo. Saldo dihitung dari jumlah baris saat dibutuhkan. Super Admin, Admin Global, dan `finance_auditor` dapat menambah, mengedit, dan menghapus record.

Masih TODO: konfirmasi asumsi di keputusan 003 dan 004, pilihan PostgreSQL atau MySQL, rekonsiliasi router (ditunda), dan sisa temuan review awal yang belum diputuskan.

## 2026-11-07 (keputusan 003)

- `next_isolir` menjadi hasil hitung `hitung_isolir_berikutnya`. Data masukannya `paid_through` (`prabayar`), `isolir_berlaku_mulai` dan tagihan (`tanggal_tetap`), dan tabel baru `JENDELA_PENUNDAAN`. `hari_isolir` dihapus.
- `prabayar` memakai satu rumus `max(paid_through, tanggal_bayar) + N x 30 hari - hari_nyala_belum_dibayar`. Membayar lebih lambat tidak lagi menghasilkan tanggal putus yang lebih jauh. Prorata tidak dipakai untuk `prabayar`.
- Tagihan `prabayar` terbit per periode 30 hari saat dibayar. Instalasi mengisi `paid_through` = `tanggal_instalasi` + 30 hari dan menerbitkan tagihan periode pertama.
- Transfer `tidak_wajib` pada `prabayar` langsung membeli periode. Penolakan membatalkan periode dan memutar ulang `paid_through` dari `paid_through_sebelum` pada `PEMBAYARAN`.
- `tanggal_tetap` memutus pada jatuh tempo tagihan ke-`batas_tunggakan`. Admin Global mengisi `isolir_berlaku_mulai` saat verifikasi.
- Toleransi, bypass, dan perpanjangan menulis `JENDELA_PENUNDAAN`. `RIWAYAT_ISOLIR` hanya mencatat putus nyata.
- Job harian dan pembayaran mengunci baris pelanggan. Job menghitung ulang `next_isolir` semua pelanggan sebelum memutus dan memberi alert bila hasilnya berbeda.
- `logika-isolir.md` versi 3.0 berisi daftar unit test wajib untuk `hitung_isolir_berikutnya` dan `alokasi_pembayaran`.

Masih TODO: konfirmasi asumsi di keputusan 003, pilihan PostgreSQL atau MySQL untuk kendala unik yang mengabaikan tagihan `batal`, tanda `tidak_tertib` untuk tagihan `biaya_pasang` yang baru terbit, serta temuan review nomor 3, 4, dan 7.

## 2026-11-07 (keputusan 002)

- Biaya pasang dan semua prorata terbit sebagai tagihan `biaya_pasang` dan `prorata`. Keduanya piutang yang ditagih kolektor, tetapi tidak dihitung tunggakan isolir, tidak menambah `N`, dan tidak mengubah `next_isolir`. Biaya aktivasi dan diskon tetap di `HUTANG_MENGENDAP`.
- Alokasi pembayaran melunasi tagihan `langganan` terlama dulu, lalu tagihan lain urut `jatuh_tempo`.
- Dusun boleh kosong selama `menunggu_verifikasi_pusat`. Persetujuan, tagihan, dan impor pelanggan aktif mewajibkan dusun. `CHECK` constraint menjaga aturan itu.
- Dying gasp dibaca Zabbix dari OLT, bukan dari ACS. Aplikasi tidak menyimpannya di tabel sendiri. Kejadian itu ada di log alert Zabbix dan di log tiket atau agenda yang terbuka.

Masih TODO: keputusan tentang `next_isolir`, tanda `tidak_tertib` untuk tagihan `biaya_pasang` yang baru terbit, serta temuan review nomor 3, 4, dan 7.

## 2026-11-07

- `onu_id` dan `onu_sn` ada di `PERANGKAT_INVENTARIS`. `olt_id` dan `pon_port` tetap di `PELANGGAN`. `serial_number` tetap nomor stok.
- `username_koneksi`, `password_koneksi`, `foto_instalasi`, dan `file_backup_config` ada di `PERANGKAT_INVENTARIS`. Merk memakai `tipe_merk` yang sudah ada.
- Saldo dompet diambil dari baris terakhir `DOMPET_PELANGGAN_MUTASI`. Saldo poin diambil dari baris terakhir `POIN_MUTASI`. `PELANGGAN` tidak menyimpan kedua saldo itu.
- Tunggakan isolir hanya tagihan penuh yang `terbayar = 0`. Biaya pasang masuk hutang mengendap. `status_langganan` menjadi `isolir` saat router benar-benar `putus`. Cuti hanya lewat satu baris `RIWAYAT_SUSPEND`.
- Data router pelanggan ada di `PERANGKAT_INVENTARIS`: SSID, password WiFi, IP perangkat, gateway, login standar, dan login superadmin. Tidak ada tabel `ROUTER_PELANGGAN`.
- Riwayat ping ada di Zabbix. Alert rumah digabung ke agenda induk yang sedang putus.
- Redaman tidak disimpan di database aplikasi. Cek live dan lapisan optik meminta angka saat dibuka. Riwayat naik-turun 7 hari dibuka dari profil pelanggan dan peta, dengan data diminta ke Zabbix. Webhook alert membuat tiket tanpa mengubah baris pelanggan.
- Riwayat suspend ada di `RIWAYAT_SUSPEND`. Snapshot terakhir ada di `last_suspend` dan `last_active`. Scheduler membaca `next_isolir`.
- Tagihan `konversi` dapat ditagih, tetapi tidak dihitung isolir dan tidak menambah `N`. Komisi reseller dan `N` transfer menunggu bukti `sudah_diverifikasi`.
- Pengeluaran gaji, komisi, dan operasional dari laci menulis baris keluar pada `MUTASI_KAS`. Konversi saldo mengendap tidak menulis kas.
- Super Admin dan `auditor_finansial` dapat menambah, mengedit, dan menghapus record saldo mengendap, serta membuat invoice dari record yang masih `mengendap`. Edit menyimpan nilai lama dan menghitung ulang saldo sesudahnya.
- Saldo mengendap dan tagihan dapat dikonversi dua arah. Catatan asal tetap ada. Konversi ke tagihan mengurangi saldo. Konversi dari tagihan menambah saldo.
- Bukti transfer bawaan langsung `lunas` dengan tanda `belum_diverifikasi`. Mode wajib, tanda pelanggan, atau kolektor level 2 menahan status `menunggu_verifikasi`.
- Prorata, diskon, biaya aktivasi, dan prorata aktivasi prabayar masuk `HUTANG_MENGENDAP`, bukan invoice. Saldo itu untuk evaluasi dan tidak menjadi piutang.
- Koreksi buku tagihan, kas, invoice, dan bukti transfer memakai izin `koreksi_keuangan`. Auditor finansial ditunjuk Super Admin. Admin Wilayah menerima izin itu dari pusat.
- Buku admin kantor memakai `MUTASI_KAS`. Kolektor, admin PC, admin ponsel, dan kios memakai satu aksi `catat_pembayaran`.
- `N` hanya dihitung dari tagihan yang menjadi lunas. Isolir tanggal 1 berjalan setelah autopay selesai.
- Satu `PEMBAYARAN` terbagi ke banyak `ALOKASI_PEMBAYARAN`.
- Perpanjangan menulis layanan nyala lagi ke `last_active` dan batas ke `next_isolir`. Hari aktif dihitung dari kedua kolom itu, tidak disimpan. Tidak ada `mulai_perpanjangan` atau `batas_perpanjangan`.
- `tidak_serempak` ikut nyala bila syarat putus gugur. Perpanjangan satu bulan menulis `next_isolir`.
- Tagihan unik per rentang layanan. Prorata dan komisi memakai integer. Potongan memakai `hari_mulai_putus`.
- `tanggal_daftar` dan `status_langganan` terpisah dari status isolir.
- Log API, kolektor, SNMP cadangan, webhook, WiFi, Google, dan batas wilayah diselaraskan. Primary key dan kolom rupiah memakai `bigint`.

## 2026-11-06

Bagian ini riwayat penulisan. Aturan yang berlaku ada di bagian 2026-11-07 dan di `docs/decisions/001-keputusan-review-spesifikasi.md`.

Spesifikasi diselaraskan dengan review rancangan.

- Isolir memakai `batas_tunggakan` (`null`, `1`, `2`) dan `siklus_tagihan` (`tanggal_tetap`, `prabayar`). Scheduler hanya membaca `next_isolir`.
- Cuti memotong tagihan bulan berjalan secara prorata sampai sehari sebelum cuti. Aktif kembali di tengah bulan menerbitkan prorata baru sampai akhir bulan. Keduanya tetap invoice tunggakan.
- Tagihan langganan hanya menghitung hari layanan. Hari cuti dan hari putus tidak ditagih.
- Penundaan isolir hanya lewat `isolir_ditunda_hingga`, dengan alasan `toleransi` atau `bypass`. Pembayaran satu bulan saat toleransi mengembalikan jadwal ke `hari_isolir`.
- Komentar address-list isolir berisi nomor pelanggan dan nama, paling banyak 128 karakter. Nomor pelanggan tidak dipotong.
- Perintah isolir masuk antrian database. Satu Mikrotik satu koneksi, 50 entri per sesi, jeda 1 detik. Pembayaran mengantri di depan putus massal.
- Gagal ke Mikrotik dicoba 10 kali dengan jeda 5 menit. Setelah habis, satu percobaan ulang otomatis dijadwalkan 5 menit setelah router terpantau nyala lagi.
- Baris putus yang belum jalan dibatalkan saat layanan dinyalakan. `hari_isolir` pada bulan pendek memakai hari terakhir. Entri address-list tidak memakai timeout.
- Nama kolom diselaraskan: `status_layanan`, `log_api_mikrotik`, `nama_address_list`, `tipe_tagihan`, `jumlah_percobaan`. Status router yang tidak selesai serempak adalah `tidak_serempak`.
- `riwayat_isolir` menyimpan satu periode putus sampai nyala per pelanggan. Percobaan API tetap di `log_api_mikrotik`.
- Perangkat WiFi menyimpan `username_biasa` dan `password_biasa` di samping `superadmin_username` dan `superadmin_password`. Keduanya terenkripsi. Teknisi hanya boleh melihat login biasa.
- Tanggal 1: tagihan dan autopay pukul 00:01, isolir pukul 00:05. Pembayaran melunasi tagihan terlama lebih dulu. `N` adalah jumlah tagihan yang menjadi lunas. Nominal dipotong hanya saat status menjadi `putus`. Nyala memakai antrean yang sama dengan prioritas tinggi.
- Bulan yang terisolir penuh tidak diterbitkan, sehingga tidak ada kuitansi bulan itu. Aktivasi di tengah bulan berikutnya menerbitkan prorata seperti pasang baru.
- Admin punya tombol simulasi biaya aktivasi dan tunggakan. Hasilnya pratinjau, tidak menulis tagihan atau isolir.
- Uang, PPN, prorata, baris tagihan, dan jurnal balik dikunci sebagai integer rupiah.
- Kasir, admin, dan kolektor punya periode kas harian. Sisa di tangan menjadi 0 hanya setelah setoran utuh dikonfirmasi user lain. Riwayat total tetap tersimpan.
- Peran Admin Pusat disatukan ke Admin Global. WhatsApp pelanggan hanya lewat tombol. Failover database dikeluarkan dari modul aplikasi.

Belum dikerjakan: kode aplikasi. Tahap berikut adalah proyek Laravel, migrasi, serta autentikasi delapan peran.
