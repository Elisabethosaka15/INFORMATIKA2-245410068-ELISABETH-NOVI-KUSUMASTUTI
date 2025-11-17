# UJIAN TENGAN SEMESTER 
# SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI 
## Nama  : Elisabeth Novi Kusumastuti
## NIM   : 245410068 
## Kelas : INFORMATIKA 
## JAWABAN 
1. Jelaskan teorema CAP dan BASE dan keterkaitan keduanya. Jelaskan menggunakan contoh yang pernah anda gunakan. 
Jawaban: Teorema CAP adalah konsep fundamental dalam sistem terdistribusi yang menyatakan bahwa sistem data hanya dapat menjamin dua dari tiga properti berikut secara bersamaan: Consistency (Konsistensi), Availability (Ketersediaan), dan Partition Tolerance (Toleransi Partisi).
- Konsistensi (C): Berarti setiap operasi baca akan mengembalikan data terbaru. Semua node sistem harus melihat data yang sama pada waktu yang sama.
- Ketersediaan (A): Berarti setiap permintaan non-gagal akan menerima respons (non-error) dari sistem, dan sistem selalu up dan merespons.
- Toleransi Partisi (P): Berarti sistem tetap beroperasi meskipun terjadi kegagalan komunikasi (partisi jaringan) antara node-nya.
Ketika terjadi partisi jaringan (P), sistem harus membuat keputusan:
 a. Pilih C dan P (C/P): Jika konsistensi diutamakan, node yang berada di sisi minoritas partisi akan menolak permintaan untuk mencegah data yang tidak konsisten.
 b. Pilih A dan P (A/P): Jika ketersediaan diutamakan, semua node tetap menerima permintaan dan merespons, namun mungkin mengembalikan data yang stale (tidak up-to-date).

 Prinsip BASE
 Prinsip BASE (yang merupakan akronim dari Basically Available, Soft state, Eventual consistency) adalah pendekatan desain yang menjadi filosofi di balik sistem yang memilih A/P dalam teorema CAP. BASE menukar konsistensi real-time dengan ketersediaan tinggi.
 - Basically Available: Sistem menjamin ketersediaan.
 - Soft State: Status sistem dapat berubah seiring waktu meskipun tidak ada input eksternal, karena konsistensi masih menyebar.
 - Eventual Consistency (Konsistensi Akhir): Jika tidak ada input baru, data di semua node akhirnya akan menjadi konsisten.

Keterkaitan dan contoh
Prinsip BASE adalah implementasi dari keputusan A/P pada Teorema CAP. Sistem yang menganut BASE (seperti Cassandra atau DynamoDB) dirancang untuk selalu tersedia dan tahan terhadap partisi, dengan mengorbankan konsistensi seketika.
Contoh yang digunakan : Dalam sistem e-commerce yang saya kelola, saya memilih A/P (atau prinsip BASE) untuk layanan keranjang belanja. 
Saya menggunakan database yang didesain untuk ketersediaan tinggi. Jika terjadi partisi, pengguna tetap bisa memasukkan barang ke keranjang (Available), 
meskipun data keranjang di server lain mungkin sedikit tertinggal. Sistem menjamin bahwa setelah partisi selesai, 
semua data keranjang akan konsisten pada akhirnya (Eventual Consistency). Ini jauh lebih baik daripada menolak pesanan karena server tidak bisa berkomunikasi (Unavailable).

2. Jelaskan keterkaitan antara GraphQL dengan komunikasi antar proses pada sistem terdistribusi buat diagramnya
Jawaban: GraphQL pada dasarnya adalah bahasa kueri untuk API. Dalam sistem terdistribusi, di mana terdapat banyak microservice, GraphQL sering diimplementasikan sebagai GraphQL Gateway atau Layer Agregasi Data. Peran ini sangat terkait dengan cara komunikasi antar proses (IPC) diatur.
Keterkaitan dengan IPC
- Mengurangi Round-Trips: Secara tradisional, klien yang membutuhkan data dari tiga microservice yang berbeda harus melakukan tiga panggilan IPC terpisah (misalnya, tiga panggilan REST). GraphQL memungkinkan klien mengirimkan satu kueri tunggal ke GraphQL Gateway.
- Orkestrasi Internal: GraphQL Gateway kemudian mengambil alih tugas IPC internal. Ia menggunakan resolver untuk melakukan panggilan IPC (misalnya, menggunakan REST, gRPC, atau message queue) ke setiap microservice yang diperlukan (User Service, Order Service, Product Service).
- Efisiensi Jaringan: Karena klien hanya meminta bidang data yang mereka butuhkan (no over-fetching), ukuran payload respons dari Gateway ke klien menjadi sangat kecil. Ini membuat IPC antara klien dan gateway lebih efisien.
  
- Gambar Diagramnya
  ![Diagram menggunakan aplikasi DIA](DiagramUTSSTDT.jpeg)

3. Dengan menggunakan Docker / Docker Compose, buatlah streaming replication di PostgreSQL yang bisa menjelaskan sinkronisasi tulislah langkah - langkah pengerjaanya dan buat penjelasan secukupnya.

Judul: Streaming Replication di  PostgreSQL Windows. 
Langkah - langkah dan penjelasannya sebagai berikut :
1. Persiapan Awal:
   - Aktifkan Docker Dekstop
   - Struktur Direktori dan File
   - Menentukkan variabelnya seperti nama database, dan kata sandi
   - Mmebuat jaringan dan Volume Docker
2. Jalankan Node Primary. penjelasan:  Jalankan container Docker yang akan bertindak sebagai server PostgreSQL primary. Ini melibatkan penggunaan Docker Compose atau perintah  docker run  dengan konfigurasi yang sesuai.
3. Konfigurasi Node Primary untuk Replikasi:
   - Ubah Konfigurasi  postgresql.conf, dengan mengedit file postgresql.conf di dalam container primary. Ubah parameter berikut:
      .  wal_level = replica  atau  logical : Mengatur level WAL (Write-Ahead Logging) untuk mendukung replikasi.  replica  sudah cukup untuk streaming replication.
      . listen_addresses = '*' : Mengizinkan koneksi dari semua alamat IP (gunakan dengan hati-hati dan pertimbangkan keamanan).
      . max_wal_senders : Menentukan jumlah maksimum koneksi WAL sender yang diizinkan (harus lebih besar dari jumlah standby).
      . wal_keep_size : Menentukan ukuran minimum WAL yang dipertahankan server.
   - Ubah Konfigurasi  pg_hba.conf, dengan mengedit file  pg_hba.conf  untuk mengizinkan koneksi replikasi dari node standby. Tambahkan baris yang mengizinkan koneksi dengan peran  replication  dari alamat IP node standby.
   - Restart Node Primary.  penjelasannya Setelah mengubah  postgresql.conf  dan  pg_hba.conf , restart container PostgreSQL primary agar perubahan diterapkan.
4. Buat Slot Replikasi.Penjelasan: Buat slot replikasi pada server primary. Ini memastikan bahwa WAL yang diperlukan oleh standby tidak dibersihkan sebelum diterima oleh standby. dapat membuat slot replikasi menggunakan perintah SQL:SELECT pg_create_physical_replication_slot('stand by_slot');5.  Cloning Database ke Node Standby. Penjelasannya, Gunakan utilitas  pg_basebackup  untuk meng-clone database dari server primary ke server standby. Ini membuat salinan awal database. Anda harus menjalankan ini dari container standby. dengan perintah:
     pg_basebackup -h <ip_node_primary> -p 5432 -U replication -D /var/lib/postgresql/data -P -X stream -R
    keterangan:
    . -h : host primary
    . -p : Port PosterSQL
    . -u : User replikasi
    . -D : Direktori data stndby
    . -P : Menampilkan progres
    . -X : Membuat file recorvery.conf (atau postgresql.auto.conf)
6.  Jalankan Node Standby (Database Cadangan). Penjelasan: Jalankan container Docker yang akan berfungsi sebagai server standby.  Pastikan file  recovery.conf  (atau  postgresql.auto.conf ) telah dibuat dengan benar oleh  pg_basebackup . File ini berisi informasi koneksi ke primary.
7.  Uji Coba dan Verifikasi Replikasi. Cek Status di Primary. Penjelasan: Periksa status replikasi di server primary untuk memastikan bahwa streaming replication aktif dan berjalan dengan benar. Anda bisa menggunakan query SQL, yaitu: SELECT * FROM pg_stat_replication;. Ini akan menunjukkan informasi tentang koneksi standby.
     Buat Data di Primary. Penjelasan, Buat beberapa data di server primary (misalnya, membuat tabel baru atau memasukkan data ke dalam tabel yang ada).
     Baca Data di Standby. Penjelasan,  Periksa server standby untuk memastikan bahwa data yang dibuat di server primary telah direplikasi ke server standby.
     Tes Gagal Tulis di Standby. Penjelasan, Cobalah untuk menulis data langsung ke server standby.  Replikasi yang berhasil harus mencegah penulisan langsung ke standby (read-only).


