# 003 — next_isolir sebagai hasil hitung dari paid_through dan jendela penundaan

## Konteks

Review 7 November 2026 atas `next_isolir` menemukan bahwa kolom itu ditulis langsung oleh pembayaran, toleransi, perpanjangan, impor, dan admin. Akibatnya:

- Pelanggan `prabayar` bisa terkunci putus, karena menyala butuh `N >= 1` sedangkan bulan tanpa tagihan dan tagihan prorata tidak menambah `N`.
- Hitungan 30 hari bergeser terhadap tagihan bulanan. Pasang 1 Januari diputus 31 Januari sebelum tagihan Februari terbit.
- Transfer `tidak_wajib` menyalakan layanan, tetapi `N` menunggu verifikasi.
- Membayar lebih lambat menghasilkan tanggal putus yang lebih jauh: bayar 6 Oktober menjadi 2 November, bayar 8 Oktober menjadi 7 November.
- Toleransi kedua menghapus toleransi pertama dan bentrok dengan aturan satu baris `RIWAYAT_ISOLIR` terbuka.
- Pembalikan pembayaran tidak dapat mengembalikan tanggal, karena tanggal lama tidak tersimpan.
- Hubungan `hari_isolir` dengan jatuh tempo tidak jelas.
- Pembayaran bisa kalah balapan dengan scheduler.
- Nilai awal saat instalasi tidak didefinisikan.

## Keputusan

Keputusan ini menggantikan kalimat 001 dan 002 tentang penulisan langsung `next_isolir`, `hari_isolir`, `N` sebagai jumlah tagihan yang menjadi lunas, prorata aktivasi `prabayar`, dan `N` yang menunggu `sudah_diverifikasi`.

- `next_isolir` hanya ditulis oleh `hitung_isolir_berikutnya`. Data masukannya disimpan: `paid_through` untuk `prabayar`, `isolir_berlaku_mulai` dan tagihan untuk `tanggal_tetap`, dan tabel `JENDELA_PENUNDAAN` untuk keduanya. Fungsi itu dipanggil di transaksi yang sama dengan setiap perubahan masukan dan dihitung ulang oleh job harian sebelum memutus.
- `prabayar` memakai satu rumus: `paid_through_baru = max(paid_through_lama, tanggal_bayar) + N x 30 hari - hari_nyala_belum_dibayar`. Hari putus diambil dari `RIWAYAT_ISOLIR`. Hari cuti tidak dihitung nyala. Hari `tidak_serempak`, `gagal_putus`, dan `menunggu_router` dihitung nyala.
- Tagihan `prabayar` terbit per periode 30 hari saat dibayar, sebagai tagihan `langganan` penuh yang langsung lunas. Tidak ada generate bulanan dan tidak ada prorata untuk `prabayar`.
- Transfer `tidak_wajib` pada `prabayar` langsung membeli periode saat tagihan `lunas`. Penolakan membatalkan tagihan periode itu dan memutar ulang pembayaran sah mulai dari `paid_through_sebelum`.
- `tanggal_tetap` memutus pada `jatuh_tempo` tagihan penuh belum dibayar yang ke-`batas_tunggakan`. `hari_isolir` dihapus. Pembayaran tidak menggeser tanggal apa pun.
- Nilai awal: `prabayar` memakai `tanggal_instalasi + 30 hari`. `tanggal_tetap` memakai `isolir_berlaku_mulai` yang diisi Admin Global saat verifikasi. Persetujuan pelanggan `tanggal_tetap` ditolak bila kolom itu kosong.
- Toleransi, bypass, dan perpanjangan satu bulan menulis satu baris `JENDELA_PENUNDAAN`. Penundaan yang berlaku adalah baris terbaru. `RIWAYAT_ISOLIR` hanya mencatat interval putus yang benar-benar terjadi.
- Job harian dan pembayaran sama-sama mengunci baris pelanggan dengan `lockForUpdate`. Job menghitung ulang syarat putus dan menulis antrean router di transaksi yang sama.

## Asumsi

Diambil tanpa konfirmasi, dan perlu diperiksa:

- Saat instalasi `prabayar`, tagihan periode pertama terbit `belum_bayar` dengan jatuh tempo tanggal instalasi. Pembayaran pertama melunasinya tanpa menambah hari.
- Urutan alokasi `prabayar`: tagihan `langganan` terbuka, lalu periode baru, lalu tagihan lain. Pelanggan yang membayar selalu mendapat layanan lebih dulu daripada pelunasan biaya pasang atau prorata lama.
- Uang `prabayar` di bawah harga paket setelah tagihan terbuka lunas tidak membeli periode dan mengikuti aturan kelebihan bayar kanal.
- Autopay `prabayar` berjalan pada tanggal `next_isolir` dan hanya memotong bila saldo cukup untuk satu periode beserta tagihan `langganan` yang terbuka.
- `tanggal_tetap` memutus tepat pada jatuh tempo, tanpa tambahan hari.
- Jatuh tempo yang melewati akhir bulan memakai hari terakhir bulan itu.

## Konsekuensi

- Kolom baru di `PELANGGAN`: `paid_through`, `isolir_berlaku_mulai`. Kolom `hari_isolir` dihapus dari rancangan.
- Tabel baru `JENDELA_PENUNDAAN`. `RIWAYAT_ISOLIR` kehilangan `alasan` dan `disetujui_oleh`, dan mendapat `jendela_penundaan_id`. `sumber_nyala` bertambah `toleransi`, `bypass`, dan `perpanjangan`.
- `PEMBAYARAN` mendapat `jumlah_periode`, `paid_through_sebelum`, dan `paid_through_sesudah`.
- Kendala unik `(pelanggan_id, tipe_tagihan, layanan_mulai)` pada `TAGIHAN` harus mengabaikan tagihan `batal`, karena pemutaran ulang setelah penolakan dapat membuat periode dengan `layanan_mulai` yang sama. PostgreSQL memakai partial unique index. MySQL tidak punya partial index dan perlu kolom generated yang bernilai null untuk tagihan `batal`. Pilihan database belum diputuskan.
- Pelanggan `tanggal_tetap` dengan `batas_tunggakan = 2` dapat memakai layanan sekitar 41 hari tanpa membayar sebelum diputus. Piutang berjalannya kira-kira satu kali harga paket per pelanggan.
- Pindah siklus wajib mengisi `paid_through` atau `isolir_berlaku_mulai`.
- `hitung_isolir_berikutnya` dan `alokasi_pembayaran` wajib punya unit test sesuai daftar di `docs/logika-isolir.md`.
