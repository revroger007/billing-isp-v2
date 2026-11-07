# Changelog

## 2026-11-07

- Isolir hanya menghitung tagihan bulanan `periode_penuh` yang belum dibayar sama sekali. Pembayaran sebagian dan tagihan nonbulanan memberi tanda `tidak_tertib` serta rekomendasi `prabayar`.
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
- `tidak_serempak` ikut nyala bila syarat putus gugur. Perpanjangan satu bulan menulis `isolir_ditunda_hingga`.
- Tagihan unik per rentang layanan. Prorata dan komisi memakai integer. Potongan memakai `hari_mulai_putus`.
- `tanggal_daftar`, `penundaan_catatan`, dan `status_langganan` terpisah dari status isolir.
- Log API, kolektor, SNMP cadangan, webhook, WiFi, Google, dan batas wilayah diselaraskan. Primary key dan kolom rupiah memakai `bigint`.

## 2026-11-06

Spesifikasi diselaraskan dengan review rancangan.

- Isolir memakai `batas_tunggakan` (`null`, `1`, `2`) dan `siklus_tagihan` (`tanggal_tetap`, `prabayar`). Scheduler hanya membaca `isolir_berikutnya`.
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
