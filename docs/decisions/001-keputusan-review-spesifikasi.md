# 001 — Keputusan hasil review spesifikasi

## Konteks

Versi 0.3 cara kerja dan versi 1.2 logika isolir memakai tiga default isolir, satuan uang yang berbeda, dan reaktivasi yang berhenti bila `isolir_enabled` false. Review 6 November 2026 mengunci pilihan di bawah.

## Keputusan

- Pelanggan baru: `batas_tunggakan = 2`, `siklus_tagihan = tanggal_tetap`.
- Isolir tidak punya empat mode dan tidak punya `isolir_enabled`. `batas_tunggakan` berisi `null`, `1`, atau `2`. `siklus_tagihan` berisi `tanggal_tetap` atau `prabayar`.
- Scheduler hanya membaca `next_isolir`. Penundaan hanya lewat `isolir_ditunda_hingga`.
- Isolir otomatis menunggu `tanggal_instalasi`. Biaya pasang dicatat sebagai hutang mengendap, bukan baris invoice, dan tidak dihitung tunggakan.
- Tunggakan isolir hanya tagihan `langganan` `periode_penuh` yang `terbayar = 0` dan statusnya `belum_bayar` atau `menunggu_verifikasi`. Status `sebagian` tidak dihitung.
- Uang disimpan sebagai integer rupiah. Harga paket sudah termasuk PPN. Prorata membagi harga dengan jumlah hari periode, lalu dibulatkan ke rupiah terdekat.
- Status isolir pelanggan final hanya setelah setiap Mikrotik yang dituju selesai. Antrian isolir memakai database queue.
- Admin Pusat pada dokumen lama adalah Admin Global. Wilayah admin disimpan di `user_wilayah`.
- Pesan pelanggan memakai tombol `wa.me`. Alert internal memakai Telegram.
- GPS dan cetak thermal dari browser hanya saat halaman terbuka. iOS mencetak PDF.
- Bonus teknisi tidak timbul dari tiket yang tertutup karena agenda massal.
- Failover server, replikasi database, dan restore SQL adalah prosedur infrastruktur, bukan modul aplikasi.
- Impor pelanggan lama memakai `isolir_mulai_evaluasi`. Mutasi uang yang dibatalkan menulis jurnal balik.
- Cuti tidak menagih tanggal cuti dan tidak membatalkan invoice. Prorata, diskon, biaya aktivasi, dan prorata aktivasi prabayar dicatat sebagai hutang mengendap, bukan invoice dan bukan piutang. Memindahkan invoice menjadi saldo mengendap hanya manual oleh Super Admin atau `auditor_finansial`.
- Tagihan langganan hanya menghitung hari layanan. Hari cuti dan hari `putus` tidak ditagih, sehingga bulan kosong tidak punya kuitansi.
- `isolir_ditunda_hingga` menggantikan `isolir_sementara` dan `bypass_until`. Alasan berisi `toleransi` atau `bypass`. Pembayaran satu bulan saat toleransi mengembalikan jadwal ke `hari_isolir` pada semua siklus.
- Satu tagihan langganan yang sudah lewat jatuh tempo memutus pelanggan dengan `batas_tunggakan = 1`, termasuk tagihan bulan lain.
- Bulan tanpa hari layanan tidak punya tagihan dan tidak punya kuitansi. Aktivasi di tengah periode menambah hutang mengendap, bukan tagihan. Tidak adanya tagihan tidak membuka isolir.
- Tombol simulasi admin menjumlahkan sisa tunggakan dan prorata aktivasi tanpa menyimpan apa pun.
- Tanggal 1: generate tagihan dan autopay mulai pukul 00:01. Job isolir tanggal 1 berjalan setelah autopay commit. Hari lain, isolir pukul 00:05. `N` hanya jumlah tagihan langganan yang menjadi lunas.
- Invoice periode yang tidak lagi penuh tidak dibatalkan oleh cuti. Nilai evaluasi masuk hutang mengendap tanpa mengubah invoice. Nyala layanan memakai antrean satu sesi per router dengan prioritas tinggi.
- `riwayat_isolir` menyimpan periode putus sampai nyala per pelanggan. `log_api_mikrotik` tetap menyimpan percobaan API per router.
- Komentar address-list isolir adalah nomor pelanggan lalu nama. Nomor pelanggan tidak dipotong. Nama yang tidak muat dipotong pada batas 128 karakter.
- Seribu isolir pada hari yang sama tidak membuka seribu koneksi. Satu Mikrotik satu sesi API, 50 entri, jeda 1 detik. Nyala layanan dari pembayaran didahulukan.
- Perintah yang gagal karena Mikrotik mati dicoba 10 kali, jeda 5 menit. Setelah batas itu, satu percobaan ulang dijalankan 5 menit setelah router terpantau nyala, hanya untuk baris yang gagal.
- Menyalakan layanan membatalkan baris putus yang belum dijalankan. `hari_isolir` yang melewati akhir bulan menjadi hari terakhir bulan itu. Entri address-list tidak punya timeout.
- `status_layanan` memakai `tidak_serempak` untuk router yang tidak selesai bersama. `sebagian` tetap status pembayaran.
- Uang tunai kasir, admin, dan kolektor masuk periode kas per tanggal Asia/Jakarta. Sisa menjadi 0 hanya bila user lain mengonfirmasi setoran sebesar sisa. Total periode tetap tersimpan. Transfer, dompet, gateway, dan komisi reseller tidak masuk saldo di tangan.
- Buku laci admin adalah `MUTASI_KAS`. Kolektor, admin PC, admin ponsel, dan kios kantor memanggil `catat_pembayaran`. `kanal` hanya penanda asal.

## Konsekuensi

- Satu `PEMBAYARAN` memiliki banyak `ALOKASI_PEMBAYARAN`. `idempotency_key` unik per penerimaan.
- `tidak_serempak` menyalakan layanan bila syarat putus gugur dan sumber putus bukan manual.
- Persetujuan perpanjangan satu bulan menulis `isolir_ditunda_hingga`. Label VIP tidak menunda isolir.
- `hari_mulai_putus` menentukan catatan diskon isolir. Prorata dan komisi memakai pembagian integer. Komisi memakai `basis_poin`. Hutang mengendap bukan piutang.
- `PELANGGAN` menyimpan `tanggal_daftar` dan `penundaan_catatan`. `status_langganan` terpisah dari `status_layanan`.
- `LOG_API_MIKROTIK` menyimpan `status_api`, `jumlah_percobaan`, `jumlah_tunggakan`, dan salinan `next_isolir`.
- SNMP OLT adalah cadangan aktif bila `snmp_supported` benar. Callback gateway dan webhook pemantauan ditolak sebelum transaksi bila tanda atau sumber tidak sah.
- Perubahan WiFi dan reboot modem menunggu NOC. Password ajuan terenkripsi dan tidak masuk teks audit. Google tidak membuat akun pelanggan. Admin wilayah hanya menyetujui pembayaran pada dusunnya bila memegang `koreksi_keuangan`.
- Koreksi tagihan, kas, invoice, dan bukti transfer pelanggan hanya untuk Super Admin, Admin Global, `auditor_finansial` yang ditunjuk Super Admin, dan Admin Wilayah yang diberi izin oleh pusat.
- Verifikasi bukti transfer bawaan `tidak_wajib`: tagihan langsung `lunas`, pembayaran bertanda `belum_diverifikasi`. Komisi reseller dan `N` menunggu `sudah_diverifikasi`. Mode `wajib`, `tanda_transfer` pada pelanggan, atau kolektor level 2 menahan tagihan di `menunggu_verifikasi`.
- Satu pelanggan dapat memiliki banyak `ROUTER_PELANGGAN`. Login standar terlihat teknisi. Login superadmin hanya untuk izin `router_superadmin`.
- Redaman tidak disimpan di database aplikasi. Riwayat naik-turun dibuka dari profil pelanggan dan peta. Aplikasi meminta deret 7 hari ke Zabbix saat grafik dibuka. Webhook alert membuat tiket dan tidak menulis baris pelanggan.
- `next_isolir` adalah tanggal yang dibaca scheduler. `status_langganan` menjadi `isolir` hanya saat `status_layanan` menjadi `putus`. `menunggu_router`, `gagal_putus`, dan `tidak_serempak` tetap `aktif`. Cuti memakai satu baris `RIWAYAT_SUSPEND` berjenis `cuti` yang `selesai`-nya kosong. Tidak ada `cuti_mulai` atau `cuti_selesai`.
- `RIWAYAT_ISOLIR` mencatat periode hasil API, dibuat saat status menjadi `putus`. `RIWAYAT_SUSPEND` untuk isolir dibuat pada saat yang sama, satu baris, dan `selesai` diisi saat layanan kembali. Cuti tidak menulis `RIWAYAT_ISOLIR`.
- Riwayat ping ada di Zabbix. Mikrotik tidak menjalankan netwatch atau skrip naik-turun. Trigger rumah bergantung pada induknya. Selama induk putus, billing menempelkan pelanggan ke satu agenda dan tidak membuka tiket per rumah.
- Login perangkat yang punya `perangkat_id` dibaca dari `PERANGKAT_INVENTARIS`. Password tidak disalin ke `ROUTER_PELANGGAN`. Login superadmin di kedua layar memakai izin `router_superadmin`.
- Hanya Super Admin dan `auditor_finansial` yang dapat menambah, mengedit, dan menghapus record saldo mengendap serta invoice. Hapus menghilangkan record. Bila record harus tetap ada, gunakan status `closed` atau edit. Hapus invoice ditolak bila `terbayar` masih ada, dan nomor yang sudah terbit tidak dipakai ulang. Status `closed` tidak mengurangi saldo mengendap dan tidak dapat dijadikan invoice.

`docs/cara-kerja.md` versi 2.29 dan `docs/logika-isolir.md` versi 2.18 mengikuti keputusan ini. Perubahan berikutnya dicatat sebagai keputusan baru, bukan dengan menghidupkan kalimat lama.
