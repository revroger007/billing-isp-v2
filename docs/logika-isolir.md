# Spesifikasi Logika Isolir

> Versi: 3.0 | Tanggal: 7 November 2026
> Sistem: pra-bayar
> Mekanisme: IP statis dimasukkan ke address-list Mikrotik. Semua pelanggan memakai IP statis. Kolom `tipe_koneksi` hanya catatan teknis dan tidak mengubah cara isolir.

Aturan di dokumen ini mengalahkan diagram atau kalimat lama yang masih memakai `isolir_enabled`, `bypass_until`, `isolir_sementara`, `masa_aktif_hingga`, `prepaid_30hari`, atau `hari_isolir`. Keputusan di `docs/decisions/` mengalahkan dokumen ini. Bila dua keputusan berbeda, nomor yang lebih besar yang dipakai.

## Dua pengaturan

Isolir hanya memutus atau menyalakan layanan. Prorata dan biaya pasang terbit sebagai tagihan `prorata` dan `biaya_pasang`. Keduanya piutang, tetapi tidak dihitung tunggakan isolir dan tidak mengubah `next_isolir`. Diskon dan biaya aktivasi diatur di `docs/cara-kerja.md` sebagai hutang mengendap dan juga tidak dihitung tunggakan. Bulan tanpa tagihan tidak punya kuitansi dan tidak membuka isolir.

Pelanggan baru: `batas_tunggakan = 2`, `siklus_tagihan = tanggal_tetap`.

| Pengaturan | Nilai | Arti |
|---|---|---|
| `batas_tunggakan` | `null`, `1`, `2` | Hanya untuk `tanggal_tetap`. `null` berarti scheduler tidak pernah memutus. `1` memutus pada jatuh tempo tagihan langganan pertama yang belum dibayar. `2` memutus pada jatuh tempo tagihan kedua yang belum dibayar. Bawaan `2`. |
| `siklus_tagihan` | `tanggal_tetap`, `prabayar` | `tanggal_tetap`: tagihan terbit tiap tanggal 1, isolir dihitung dari jatuh tempo tagihan. `prabayar`: tagihan terbit per periode 30 hari saat dibayar, layanan berlaku sampai `paid_through`. |

`batas_tunggakan = null` tampil di layar sebagai Tidak pernah. Isolir manual tetap menunggu nyala manual.

## Data masukan dan next_isolir

`next_isolir` adalah tanggal kalender Asia/Jakarta saat scheduler akan memutus bila tidak ada perubahan. Kolom ini hasil hitung. Pembayaran, penundaan, impor, dan admin tidak menulisnya langsung. Yang mereka ubah adalah data masukan berikut, lalu `hitung_isolir_berikutnya` menulis ulang `next_isolir` di transaksi yang sama.

| Masukan | Siklus | Diubah oleh |
|---|---|---|
| `paid_through` | `prabayar` | Instalasi, pembayaran yang membeli periode, pembalikan pembayaran, admin saat pindah ke `prabayar` |
| `isolir_berlaku_mulai` | `tanggal_tetap` | Admin Global saat verifikasi, impor, admin saat pindah ke `tanggal_tetap` |
| Tagihan `langganan` penuh beserta `jatuh_tempo` | `tanggal_tetap` | Generate bulanan, pembayaran, pembatalan dan koreksi tagihan |
| `JENDELA_PENUNDAAN` | keduanya | Toleransi, bypass, perpanjangan satu bulan |
| `RIWAYAT_SUSPEND` cuti yang terbuka | keduanya | Mulai dan selesai cuti |
| `batas_tunggakan`, `siklus_tagihan`, `tanggal_instalasi` | keduanya | Admin |

`paid_through` adalah tanggal pertama yang tidak lagi dibayar. Job harian pada tanggal itu memutus. Contoh: bayar 15 September untuk satu periode menghasilkan `paid_through` 15 Oktober. Layanan yang dibayar adalah 15 September sampai 14 Oktober.

`isolir_berlaku_mulai` adalah tanggal paling awal scheduler boleh memutus otomatis pelanggan `tanggal_tetap`. Admin Global mengisinya saat verifikasi pelanggan baru. Persetujuan pelanggan `tanggal_tetap` ditolak bila kolom ini kosong. Impor pelanggan lama juga mengisinya, supaya malam pertama setelah migrasi tidak mengisolir massal.

`JENDELA_PENUNDAAN` menyimpan satu baris per penundaan dan tidak menimpa baris lama. Penundaan yang berlaku adalah baris terbaru yang `mulai`-nya sudah sampai dan `batas`-nya belum sampai. Pada tanggal `batas`, penundaan tidak berlaku lagi dan job harian boleh memutus.

`hitung_isolir_berikutnya` menghasilkan:

- `null` bila `tanggal_instalasi` kosong.
- `prabayar`: `max(paid_through, batas penundaan yang berlaku)`.
- `tanggal_tetap` dengan `batas_tunggakan = null`, atau `isolir_berlaku_mulai` kosong: `null`.
- `tanggal_tetap` lainnya: `max(T, isolir_berlaku_mulai, batas penundaan yang berlaku)`. `T` adalah `jatuh_tempo` tagihan penuh belum dibayar yang ke-`batas_tunggakan`, urut `jatuh_tempo`, yaitu tanggal saat jumlah tunggakan mencapai batas. Bila tagihan seperti itu belum sebanyak batas, `T` adalah proyeksi jatuh tempo periode bulanan mendatang yang akan membuat jumlah mencapai batas, dengan anggapan tidak ada pembayaran.

Selama ada cuti terbuka, `next_isolir` tetap dihitung untuk tampilan, tetapi scheduler melewati pelanggan itu.

Fungsi ini dipanggil di transaksi yang sama dengan setiap perubahan masukan: pembayaran, pembalikan dan penolakan bukti, tagihan terbit, dibatalkan, atau diedit, penundaan, cuti mulai dan selesai, perubahan pengaturan, instalasi, dan impor. Job harian menghitung ulang semua pelanggan sebelum memutus. Bila hasilnya berbeda dengan `next_isolir` yang tersimpan, job menulis nilai baru dan mengirim alert Telegram internal, karena perbedaan itu berarti ada kejadian yang tidak memanggil fungsi ini.

## Job harian

Pada tanggal 1, urutannya: generate tagihan `tanggal_tetap` dan autopay mulai pukul 00:01, lalu autopay `prabayar`, lalu hitung ulang `next_isolir`, lalu evaluasi isolir. Hari lain, urutan yang sama tanpa generate dimulai pukul 00:05. Seluruh rantai memakai `withoutOverlapping()`. Langkah berikutnya baru mulai setelah langkah sebelumnya commit.

Job membandingkan tanggal kalender Jakarta, bukan tanggal UTC.

Evaluasi isolir memilih pelanggan yang `next_isolir`-nya sudah sampai. Untuk setiap pelanggan, job mengunci baris pelanggan dengan `lockForUpdate`, menghitung ulang syarat putus, lalu menulis baris antrean router di transaksi yang sama. Pembayaran juga mengunci baris pelanggan. Dengan begitu pembayaran yang commit lebih dulu selalu terbaca, dan pembayaran yang datang kemudian menunggu lalu membatalkan antrean putus lewat `nyalakan_layanan`.

Scheduler melewati pelanggan bila salah satu ini berlaku:

- `tanggal_instalasi` masih kosong
- ada `RIWAYAT_SUSPEND` berjenis `cuti` yang `selesai`-nya masih kosong. Pada saat itu `status_langganan` adalah `tunda_isolir`. Baris isolir yang belum selesai tidak membuat scheduler melewati pelanggan.

Pada tanggal `next_isolir` sendiri, job memutus bila syarat terpenuhi. Pembayaran yang masuk kemudian pada hari yang sama tetap diproses dan dapat menyalakan layanan lagi.

## Status langganan dan riwayat

`status_langganan` menjadi `isolir` hanya saat `status_layanan` menjadi `putus`. `menunggu_router`, `gagal_putus`, dan `tidak_serempak` meninggalkan `status_langganan` pada `aktif`. Pada saat `putus`, `last_suspend` dan `last_suspend_note` diisi `isolir`, satu baris `RIWAYAT_SUSPEND` berjenis `isolir` dibuat dengan `selesai` kosong, dan satu baris `RIWAYAT_ISOLIR` dibuat. `selesai` dan `nyala_kembali` diisi saat `status_layanan` menjadi `nyala`, bersama `last_active` dan catatannya. `last_suspend` dan `last_active` hanya salinan untuk tampilan. Tidak ada rumus yang membacanya.

Cuti membuat satu baris `RIWAYAT_SUSPEND` berjenis `cuti` saat cuti dimulai. `status_langganan` menjadi `tunda_isolir`. `last_suspend` diisi dari `mulai` baris itu. Catatan dapat `cuti sementara` atau teks lain. Permintaan pelanggan dapat menyertakan `bukti_percakapan`. Cuti tidak menulis `RIWAYAT_ISOLIR` dan tidak mengubah `status_layanan`. Saat cuti berakhir, `selesai` pada baris yang sama diisi. Bila layanan sedang `putus`, `status_langganan` kembali `isolir`. Bila layanan `nyala`, `status_langganan` menjadi `aktif` dan `last_active` diisi dengan catatan `selesai cuti`. Tidak ada kolom `cuti_mulai` atau `cuti_selesai`. Satu pelanggan hanya punya satu baris terbuka per `jenis`.

## Tiga fungsi

`hitung_isolir_berikutnya` hanya menghitung `next_isolir` dan jumlah tunggakan, dan satu-satunya fungsi yang menulis `next_isolir`. Fungsi ini tidak menghubungi Mikrotik. Tombol simulasi memanggilnya tanpa menyimpan hasil.

`alokasi_pembayaran` adalah satu-satunya fungsi yang membagi uang ke tagihan. Satu penerimaan uang membuat satu baris `PEMBAYARAN` dengan `idempotency_key` unik. `gateway_transaction_id` yang terisi juga unik. Baris `ALOKASI_PEMBAYARAN` mencatat nominal per tagihan. Tagihan, pembayaran, dan pelanggan dikunci `lockForUpdate` di transaksi yang sama. Petugas mengisi nominal. Sistem yang membagi.

Urutan pada `tanggal_tetap`:

1. Tagihan `langganan` yang belum lunas, urut `jatuh_tempo` terlama, lalu `layanan_mulai`.
2. Tagihan lain yang masih bersisa (`prorata`, `biaya_pasang`, `konversi`, `manual`), urut `jatuh_tempo`.

Sisa uang yang tidak menutup tagihan berikutnya tetap `sebagian` pada tagihan itu. Kelebihan di atas semua tagihan mengikuti aturan kelebihan bayar di kanal: kembalian atau dompet.

Urutan pada `prabayar`:

1. Tagihan `langganan` yang sudah terbit dan belum lunas, urut `jatuh_tempo` terlama. Ini tagihan periode pertama saat instalasi dan tagihan lama dari masa `tanggal_tetap`. Melunasinya tidak menambah hari.
2. Sisa uang membeli periode baru. `N` = sisa uang dibagi harga paket untuk lokasi pelanggan, dibulatkan ke bawah. Setiap periode menjadi satu tagihan `langganan` baru, `periode_penuh = true`, dan langsung lunas.
3. Sisa uang berikutnya melunasi tagihan lain yang masih bersisa, urut `jatuh_tempo`.
4. Sisa yang masih ada mengikuti aturan kelebihan bayar di kanal: kembalian atau dompet.

`N` hanya ada pada `prabayar` dan hanya dihitung oleh `alokasi_pembayaran`. Tidak ada pilihan `N` di layar. Pembayaran dengan `N = 0` tidak mengubah `paid_through`.

`putuskan_layanan` dan `nyalakan_layanan` adalah satu-satunya fungsi yang menulis address-list. Keduanya mencatat hasil per router. Status pelanggan menjadi `putus` atau `nyala` hanya setelah semua router yang dituju sukses. `nyalakan_layanan` membatalkan baris putus yang belum dijalankan untuk pelanggan dan router yang sama, supaya antrean lama tidak memutus lagi setelah pelanggan bayar.

## Status yang dihitung tunggakan

Bagian ini hanya untuk `tanggal_tetap`. Siklus `prabayar` tidak membaca jumlah tunggakan.

Hanya tagihan `langganan` dengan `periode_penuh = true` yang dihitung isolir. `periode_penuh` ditetapkan saat tagihan terbit, yaitu bila nominalnya utuh sebesar harga paket untuk lokasi pelanggan pada saat itu. Edit nominal sesudahnya tidak mengubah `periode_penuh`. Tagihan `konversi`, `prorata`, `biaya_pasang`, denda, dan invoice `manual` diperlakukan sama: piutang yang dapat ditagih dan tidak dihitung di `jumlah`. Saldo mengendap tidak dihitung. Bulan yang tidak diterbitkan tidak dihitung lunas dan tidak menyalakan layanan.

Satu tagihan penuh dihitung tunggakan hanya bila `terbayar = 0` dan statusnya `belum_bayar` atau `menunggu_verifikasi`. Tagihan penuh yang sudah dibayar sebagian tidak dihitung, walaupun `sisa` masih ada. Status invoice tetap `sebagian`. Bukti transfer pada mode `tidak_wajib` membuat tagihan `lunas` dan dapat menyalakan layanan. Pada `prabayar`, periode yang dibeli langsung masuk `N` walaupun tanda bukti masih `belum_diverifikasi`. Mode `wajib`, tanda pelanggan, atau kolektor level 2 membuat status `menunggu_verifikasi`, belum membeli periode, dan belum menyalakan layanan.

Tanda `tidak_tertib` dipasang manual oleh Admin Global, Admin Wilayah, kolektor, atau `auditor_finansial`, bukan dihitung dari tagihan. Tanda itu tidak dibaca `hitung_isolir_berikutnya` dan tidak mengubah jadwal isolir. Bila siklusnya masih `tanggal_tetap`, profil menampilkan rekomendasi untuk menjadikannya pelanggan `prabayar`. Rekomendasi tidak mengubah siklus.

## Siklus tanggal_tetap

Tagihan `langganan` terbit tanggal 1. Jatuh tempo bawaan tanggal 11. Bila jatuh tempo melewati akhir bulan, dipakai hari terakhir bulan itu. Isolir terjadi pada jatuh tempo tagihan yang membuat jumlah tunggakan mencapai batas. Tidak ada tanggal isolir terpisah dari jatuh tempo.

```
jumlah = jumlah tagihan langganan periode_penuh yang terbayar = 0 dan jatuh temponya sudah sampai

Jika batas_tunggakan berisi 1 atau 2 dan hari ini >= next_isolir:
    Jika jumlah >= batas dan status_layanan salah satu dari nyala, gagal_putus:
        putuskan_layanan
Jika jumlah < batas dan sumber_putus = scheduler dan status_layanan bukan nyala:
    nyalakan_layanan
```

`batas_tunggakan = null` membuat scheduler melewati putus otomatis. Bila putus sebelumnya berasal dari scheduler, jalan berikutnya menyalakan layanan. Putus manual tidak ikut terbuka.

Pembayaran tidak menggeser tanggal apa pun. Pembayaran hanya mengubah tagihan, lalu `next_isolir` dihitung ulang. Tagihan penuh yang sudah dibayar sebagian tidak menambah `jumlah`. Contoh tagihan Rp100.000 dibayar Rp50.000 tetap berstatus `sebagian`, tetapi untuk isolir dianggap tidak menunggak. Petugas dapat memasang tanda `tidak_tertib` secara manual.

Admin Global dapat mengedit `jatuh_tempo` tagihan yang belum dibayar sama sekali. Karena `T` dibaca dari `jatuh_tempo`, edit itu menggeser tanggal putus dan memanggil `hitung_isolir_berikutnya` di transaksi yang sama. Audit menyimpan nilai sebelum dan sesudah beserta alasan. Kolektor tidak mengedit jatuh tempo. Kolektor mengajukan penundaan.

Contoh `batas_tunggakan = 2`, jatuh tempo tanggal 11. Pelanggan biasa membayar sekitar tanggal 18. November dibayar, Desember ditunda, Januari tidak dibayar.

```
1 Des    tagihan Desember terbit
11 Des   jumlah = 1 (Desember)            < 2, tetap nyala
1 Jan    tagihan Januari terbit
11 Jan   jumlah = 2 (Desember, Januari)   >= 2, putus pukul 00:05
15 Jan   bayar Desember                   jumlah = 1, nyala
11 Feb   jumlah = 2 (Januari, Februari)   putus lagi bila belum ada pembayaran
```

Dengan `batas_tunggakan = 1`, pelanggan yang sama diputus setiap tanggal 11 dan dinyalakan setiap kali membayar.

Satu tagihan September yang belum dibayar sama sekali memutus pelanggan dengan batas 1, walaupun tagihan bulan ini sudah lunas.

Tombol simulasi hanya pratinjau. Simulasi tidak mengubah `status_layanan` dan tidak menyimpan `next_isolir`.

## Siklus prabayar

Pelanggan `prabayar` tidak mendapat tagihan bulanan tanggal 1. Tagihan `langganan` terbit saat pembayaran membeli periode 30 hari kalender. 30 hari kalender bukan satu bulan: 31 Januari ditambah 30 hari menjadi 2 Maret.

Saat `tanggal_instalasi` diisi, `paid_through` = `tanggal_instalasi` + 30 hari. Sistem menerbitkan tagihan `langganan` periode pertama untuk rentang itu, berstatus `belum_bayar`, dengan jatuh tempo tanggal instalasi. Pembayaran pertama melunasi tagihan ini lebih dulu dan tidak menambah hari.

Saat pembayaran membeli `N` periode:

```
paid_through_baru = max(paid_through_lama, tanggal_bayar) + (N x 30 hari) - hari_nyala_belum_dibayar
```

`hari_nyala_belum_dibayar` adalah jumlah tanggal mulai dari `paid_through_lama` sampai sehari sebelum `tanggal_bayar` yang bukan hari putus dan bukan hari cuti. Hari putus adalah tanggal `mulai_putus` sampai sehari sebelum `nyala_kembali` pada `RIWAYAT_ISOLIR`. Hari `menunggu_router`, `gagal_putus`, dan `tidak_serempak` dihitung nyala, karena layanan masih berjalan pada sedikitnya satu router. Bila `tanggal_bayar` belum melewati `paid_through_lama`, nilainya 0.

Periode yang dibeli dimulai dari `paid_through_baru - (N x 30 hari)` dan berurutan per 30 hari. Setiap periode menjadi satu tagihan dengan `layanan_mulai` dan `layanan_selesai` itu. Rentang tagihan adalah hari yang dibayar, bukan salinan tanggal nyala yang sebenarnya.

```
Jika hari ini >= next_isolir dan status_layanan salah satu dari nyala, gagal_putus:
    putuskan_layanan

Jika pembayaran membeli N >= 1, sumber_putus bukan manual, status_layanan bukan nyala, dan next_isolir baru sesudah hari ini:
    nyalakan_layanan
```

Contoh:

```
15 September bayar 1 periode saat putus
    paid_through = 15 Oktober
    layanan dinyalakan

10 Oktober bayar 1 periode saat masih nyala
    paid_through = 15 Oktober + 30 = 14 November

14 November, job harian
    hari ini >= 14 November -> putus

20 November bayar 1 periode saat putus
    hari_nyala_belum_dibayar = 0
    paid_through = 20 November + 30 = 20 Desember
    layanan dinyalakan
```

Autopay `prabayar` berjalan di job harian sebelum evaluasi isolir, untuk pelanggan yang `auto_pay_enabled` dan `next_isolir`-nya hari ini. Bila saldo dompet cukup untuk sisa tagihan `langganan` yang terbuka ditambah satu periode, autopay memanggil `alokasi_pembayaran` sebesar jumlah itu. Bila tidak cukup, dompet tidak dipotong.

Penolakan bukti transfer membalik pembayaran itu lewat `reverses_id` dan membatalkan tagihan periode yang dibelinya. `paid_through` dihitung ulang dengan memutar ulang pembayaran sah sesudahnya secara urut, mulai dari `paid_through_sebelum` yang tersimpan pada pembayaran yang ditolak. Tanggal periode pada tagihan pembayaran sesudahnya tidak diubah. Bila `next_isolir` baru sudah sampai, job harian berikutnya memutus.

Pelanggan `prabayar` tidak menumpuk tunggakan langganan baru. Yang tetap menjadi piutang adalah tagihan periode pertama yang belum dibayar, tagihan lama dari masa `tanggal_tetap`, serta tagihan `prorata`, `biaya_pasang`, `konversi`, dan `manual`.

## Banyak Mikrotik

Setiap router pelanggan punya satu baris log. IP selalu diambil dari `pelanggan_mikrotik.ip_address`. Perintah ke router idempotent. Nama address-list, misalnya `isolir`, harus sudah dipakai oleh aturan firewall di Mikrotik. Billing menyimpan nama itu di `nama_address_list` dan tidak membuat aturan firewall. Entri ditulis tanpa timeout. Hanya billing yang menghapusnya.

## Antrian perintah

Job isolir hanya menulis baris antrian di database, satu baris per pelanggan per Mikrotik, lalu status pelanggan menjadi `menunggu_router`. Setiap baris menyimpan `hari_mulai_putus`, yaitu tanggal Asia/Jakarta saat baris ditulis. Job itu tidak membuka koneksi API.

Setiap Mikrotik punya satu pekerja. Pekerja router yang berbeda boleh berjalan bersamaan. Pada satu router hanya ada satu koneksi API pada satu waktu. Perintah 1000 pelanggan di router yang sama tidak dibuka sebagai 1000 koneksi.

Pekerja mengambil 50 baris, membuka satu sesi ke IP utama, dan bila sesi itu gagal memakai IP cadangan untuk seluruh 50 baris itu. Entri ditulis di sesi yang sama. Entri yang sudah ada dihitung sukses. Sesi ditutup, lalu jeda 1 detik, lalu 50 baris berikutnya. Baris yang gagal kembali ke antrian dengan jeda 5 menit, paling banyak 10 percobaan termasuk percobaan pertama, tanpa mengulang baris yang sudah sukses.

Setelah 10 percobaan habis, baris itu berstatus gagal dan tidak dicoba lagi oleh jeda 5 menit. Status pelanggan menjadi `gagal_putus`, `gagal_nyala`, atau `tidak_serempak`. Bila kemudian IP utama atau cadangan router itu terpantau menjawab, sistem menjadwalkan satu percobaan ulang otomatis 5 menit setelah pantauan nyala. Percobaan ini di luar hitungan 10. Hanya baris gagal pada router itu yang dijalankan. Bila percobaan ini gagal, percobaan otomatis berikutnya menunggu router terpantau mati lalu nyala lagi. Satu router hanya boleh punya satu jadwal ulang ini. Router yang flapping tidak menambah jadwal baru sebelum jadwal yang ada dijalankan.

Perintah menyalakan layanan dari pembayaran, penundaan, atau admin berprioritas di atas putus massal. Prioritas dicek di antara dua batch, sehingga pembayaran tidak menunggu 1000 pelanggan selesai. Pembayaran hanya menunggu batch 50 yang sedang berjalan.

## Komentar address-list

Komentar setiap entri address-list berbentuk `no_pelanggan` diikuti satu spasi dan nama pelanggan, contoh `CUST-00123 Budi Santoso`. Nomor pelanggan selalu utuh karena itu kunci audit. Nama memakai data terkini. Baris baru dan tanda kutip diganti spasi. Panjang komentar paling banyak 128 karakter. Bila tidak muat, hanya nama yang dipotong. Nomor telepon, alamat, dan NIK tidak ditulis. Bila nama berubah, entri yang sudah ada diperbarui komentarnya, bukan ditambah baru. Bila IP berubah, entri IP lama dihapus dan entri IP baru memakai komentar yang sama.

`status_layanan` pelanggan diturunkan dari log router yang dituju:

| Kondisi log | `status_layanan` |
|---|---|
| Masih ada router yang dicoba ulang | `menunggu_router` |
| Semua router sukses menambah IP | `putus` |
| Semua router sukses menghapus IP | `nyala` |
| Sebagian router sukses dan percobaan habis | `tidak_serempak` |
| Semua router gagal saat memutus dan percobaan habis | `gagal_putus` |
| Semua router gagal saat menyalakan dan percobaan habis | `gagal_nyala` |

Pelanggan tidak ditandai `putus` atau `nyala` sebelum semua router yang dituju sukses. Hari `tidak_serempak` bukan hari putus, sehingga tidak mengurangi tagihan dan dihitung nyala pada `prabayar`. Sepuluh percobaan berkala hanya untuk router yang gagal. Router yang sudah sukses tidak diubah lagi. Percobaan setelah Mikrotik nyala kembali mengikuti bagian antrian perintah.

## Penundaan isolir

Ada tiga jenis penundaan. Semuanya menulis satu baris `JENDELA_PENUNDAAN` dan tidak mengubah `next_isolir` secara langsung.

| `alasan` | Diajukan | Disetujui | `batas` |
|---|---|---|---|
| `toleransi` | Admin Global atau Admin Wilayah, langsung | Tanpa persetujuan kedua | Tanggal yang diisi admin |
| `toleransi` | Kolektor lewat pengajuan penundaan | Admin Global | Tanggal yang diminta kolektor |
| `bypass` | Admin Wilayah lewat pengajuan penundaan | Admin Global yang bukan pengaju | Tanggal yang diminta |
| `perpanjangan` | Kolektor lewat pengajuan perpanjangan satu bulan | Admin Global | 30 hari sejak tanggal setuju |

Penundaan baru menggantikan penundaan lama karena yang berlaku adalah baris terbaru. Baris lama tetap tersimpan sebagai audit. `disetujui_oleh` wajib untuk penundaan yang berasal dari pengajuan. Kolektor dan Admin Wilayah hanya mengajukan untuk pelanggan pada dusun di `user_wilayah`.

Bila pelanggan sedang putus oleh scheduler saat penundaan disetujui, `nyalakan_layanan` dipanggil. Saat semua router sukses, baris `RIWAYAT_ISOLIR` yang terbuka ditutup dengan `nyala_kembali` dan `sumber_nyala` sesuai `alasan`. Bila pada tanggal `batas` syarat putus masih terpenuhi, job harian memutus lagi dan membuat baris `RIWAYAT_ISOLIR` baru. Bila API gagal, penundaan tetap tersimpan dan `status_layanan` menjadi `menunggu_router` atau `gagal_nyala`. Layar tidak boleh mengaku layanan sudah nyala sebelum semua router sukses.

Pada `tanggal_tetap`, penundaan hanya menunda tanggal putus. Pembayaran selama penundaan mengurangi `jumlah` seperti biasa.

Pada `prabayar`, hari nyala selama penundaan dibayar oleh pembelian periode berikutnya lewat `hari_nyala_belum_dibayar`. Contoh: `paid_through` 5 September, putus 5 September. Toleransi disetujui 7 September dengan batas 10 September. Putus lagi 10 September. Hari nyala yang belum dibayar adalah 7, 8, dan 9 September.

```
Bayar 1 periode 9 September    9 Sep + 30 - 2 = 7 Oktober
Bayar 1 periode 20 September   20 Sep + 30 - 3 = 17 Oktober
Bayar 2 periode 20 September   20 Sep + 60 - 3 = 16 November
Bayar 1 periode 8 Oktober      8 Okt + 30 - 3 = 4 November
```

Bila ada toleransi kedua 12 sampai 14 September dan bayar 1 periode 20 September, hari nyala yang belum dibayar adalah 5 hari, sehingga `paid_through` = 15 Oktober. Membayar lebih lambat tidak pernah menghasilkan `paid_through` yang lebih jauh dari membayar lebih cepat dengan `N` yang sama.

## Isolir manual

Admin Global dan Admin Wilayah boleh memutus manual walaupun `batas_tunggakan` kosong. Aksi ini menulis `sumber_putus = manual` dan wajib berisi keterangan. Modal konfirmasi menampilkan nama pelanggan, IP, dan setiap Mikrotik yang dituju. Pembayaran tidak menghapus putus manual. Pada `prabayar`, pembayaran selama putus manual tetap membeli periode dan memajukan `paid_through`. Yang menyalakan kembali adalah Admin Global, atau penundaan yang sudah disetujui.

## Pindah pengaturan

Pengaturan baru berlaku pada penyimpanan. Perubahan menulis audit, lalu `next_isolir` dihitung ulang.

- Ke `prabayar`: admin wajib mengisi `paid_through`. Generate bulanan berhenti untuk pelanggan itu. Tagihan `langganan` yang masih terbuka tetap piutang dan dilunasi lebih dulu oleh pembayaran berikutnya.
- Ke `tanggal_tetap`: admin wajib mengisi `isolir_berlaku_mulai`. Hari dari `paid_through` sampai sebelum periode bulanan berikutnya ditagih sebagai `prorata`. Generate bulanan mulai pada periode berikutnya.
- Status yang sedang berjalan tidak dihapus diam-diam. Job harian berikutnya yang memutuskan.

## State

```mermaid
stateDiagram-v2
    [*] --> Nyala: Pelanggan terpasang

    Nyala --> MenungguRouter: Syarat putus terpenuhi
    MenungguRouter --> Putus: Semua router sukses
    MenungguRouter --> TidakSerempak: Percobaan habis, hasil router campuran
    MenungguRouter --> GagalPutus: Semua router gagal memutus
    MenungguRouter --> Nyala: Syarat putus gugur sebelum router selesai

    Putus --> MenungguRouter: Syarat nyala terpenuhi
    TidakSerempak --> MenungguRouter: Ulang manual
    TidakSerempak --> MenungguRouter: Syarat putus gugur
    GagalPutus --> MenungguRouter: Ulang manual
    GagalPutus --> MenungguRouter: Syarat putus gugur

    MenungguRouter --> Nyala: Semua router sukses menyalakan
    MenungguRouter --> GagalNyala: Semua router gagal menyalakan
    GagalNyala --> MenungguRouter: Ulang manual
```

## Data

Tambahan pada `pelanggan`:

| Field | Tipe | Keterangan |
|---|---|---|
| `batas_tunggakan` | integer atau null | `null`, `1`, atau `2`. Default `2`. Hanya untuk `tanggal_tetap`. |
| `siklus_tagihan` | string | `tanggal_tetap` atau `prabayar`. Default `tanggal_tetap`. |
| `status_layanan` | string | `nyala`, `menunggu_router`, `putus`, `tidak_serempak`, `gagal_putus`, `gagal_nyala` |
| `sumber_putus` | string atau null | `scheduler` atau `manual`. Null bila layanan nyala. |
| `paid_through` | date atau null | Hanya `prabayar`. Tanggal pertama yang tidak lagi dibayar. |
| `isolir_berlaku_mulai` | date atau null | Hanya `tanggal_tetap`. Kosong berarti scheduler tidak memutus otomatis. |
| `next_isolir` | date atau null | Hasil `hitung_isolir_berikutnya`. Tidak ditulis oleh fungsi lain. |
| `tanggal_daftar` | datetime | Waktu pendaftaran. Terpisah dari `tanggal_instalasi`. |

`JENDELA_PENUNDAAN` satu baris per penundaan: pelanggan, `alasan` (`toleransi`, `bypass`, `perpanjangan`), `mulai`, `batas`, `dibuat_oleh`, `disetujui_oleh`, `pengajuan_penundaan_id` (kosong untuk toleransi langsung), `keterangan`, dan waktu dibuat. Baris tidak diedit setelah disetujui.

`RIWAYAT_ISOLIR` satu baris per periode putus nyata seorang pelanggan, bukan per router dan bukan per percobaan. Baris dibuat saat `status_layanan` menjadi `putus` dan ditutup saat menjadi `nyala`. `sumber_putus` berisi `scheduler` atau `manual`. `sumber_nyala` berisi `scheduler`, `pembayaran`, `admin`, `toleransi`, `bypass`, atau `perpanjangan`. `jendela_penundaan_id` terisi bila nyala berasal dari penundaan. Hanya boleh ada satu baris terbuka, yaitu yang `nyala_kembali`-nya masih kosong. `tidak_serempak` dan percobaan gagal tidak membuat baris baru. Cuti tidak menulis tabel ini. Rentang pada tabel ini adalah sumber hari putus untuk `hari_nyala_belum_dibayar`.

Contoh pelanggan A:

```
mulai_putus 5 November,  nyala_kembali 10 November
mulai_putus 5 Desember,  nyala_kembali 15 Desember
```

`PEMBAYARAN` pelanggan `prabayar` menyimpan `jumlah_periode` (`N`), `paid_through_sebelum`, dan `paid_through_sesudah`. Kolom ini dipakai untuk memutar ulang saat pembayaran ditolak.

`log_api_mikrotik` satu baris per percobaan per Mikrotik: siklus saat itu, jumlah tunggakan, salinan `next_isolir`, IP yang ditulis, `status_api` (`pending`, `sukses`, `gagal`), dan `jumlah_percobaan`.

`PENGAJUAN_PENUNDAAN` menyimpan pengajuan toleransi, bypass, dan perpanjangan: pelanggan, `alasan`, pengaju, tanggal yang diminta, keterangan, status `menunggu` / `disetujui` / `ditolak`, pemroses, dan alasan tolak. Persetujuan menulis satu baris `JENDELA_PENUNDAAN` yang merujuk pengajuan itu.

## Kasus uji wajib

Unit test `hitung_isolir_berikutnya` dan `alokasi_pembayaran` minimal mencakup:

1. `tanggal_tetap`, batas 2: satu tagihan belum dibayar, `next_isolir` adalah proyeksi jatuh tempo bulan berikutnya.
2. `tanggal_tetap`, batas 2: dua tagihan belum dibayar, `next_isolir` adalah jatuh tempo tagihan kedua.
3. `tanggal_tetap`, batas 1: tagihan lama belum dibayar dan tagihan bulan ini lunas tetap memutus.
4. `tanggal_tetap`: `isolir_berlaku_mulai` lebih lambat dari jatuh tempo menunda putus sampai tanggal itu. `isolir_berlaku_mulai` kosong menghasilkan `null`.
5. `tanggal_tetap`: tagihan dibayar sebagian tidak dihitung tunggakan.
6. `prabayar`: instalasi menghasilkan `paid_through` = instalasi + 30 dan satu tagihan periode pertama `belum_bayar`. Pembayaran pertama melunasi tagihan itu tanpa menambah hari.
7. `prabayar`: bayar saat nyala sebelum `paid_through` menambah tepat `N x 30` dari `paid_through`.
8. `prabayar`: semua baris contoh pada bagian penundaan, termasuk toleransi kedua.
9. `prabayar`: untuk `N` yang sama, `paid_through` tidak pernah lebih jauh bila tanggal bayar lebih lambat.
10. `prabayar`: hari `tidak_serempak` dan `gagal_putus` dihitung nyala. Hari cuti tidak.
11. `prabayar`: uang di bawah harga paket tanpa tagihan terbuka tidak membeli periode.
12. `prabayar`: penolakan bukti pada pembayaran yang bukan terakhir menghasilkan `paid_through` yang sama dengan pemutaran ulang pembayaran sah.
13. Penundaan baru menggantikan penundaan lama. Batas penundaan lebih lambat dari `paid_through` atau `T` menjadi `next_isolir`.
14. Pembayaran yang commit saat job harian sedang mengevaluasi pelanggan yang sama tidak berakhir dengan pelanggan lunas yang diputus.
15. Bulan 31 hari dan Februari: jatuh tempo tanggal 31 menjadi hari terakhir bulan, dan 30 hari kalender dihitung lintas bulan.
