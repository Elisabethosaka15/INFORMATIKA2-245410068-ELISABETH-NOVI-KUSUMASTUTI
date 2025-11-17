# UJIAN TENGAN SEMESTER 
# SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI 
## Nama  : Elisabeth Novi Kusumastuti
## NIM   : 245410068 
## Kelas : INFORMATIKA 2
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
 Prinsip BASE (yang merupakan akronim dari Basically Available, Soft state, Eventual consistency) adalah pendekatan desain yang menjadi filosofi di balik sistem yang memilih  A/P dalam teorema CAP. BASE menukar konsistensi real-time dengan ketersediaan tinggi.
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
   GraphQL pada dasarnya adalah bahasa kueri untuk API. Dalam sistem terdistribusi, di mana terdapat banyak microservice, GraphQL sering diimplementasikan sebagai GraphQL Gateway atau Layer Agregasi Data. Peran ini sangat terkait dengan cara komunikasi antar proses (IPC) diatur.
Keterkaitan dengan IPC
- Mengurangi Round-Trips: Secara tradisional, klien yang membutuhkan data dari tiga microservice yang berbeda harus melakukan tiga panggilan IPC terpisah (misalnya, tiga panggilan REST). GraphQL memungkinkan klien mengirimkan satu kueri tunggal ke GraphQL Gateway.
- Orkestrasi Internal: GraphQL Gateway kemudian mengambil alih tugas IPC internal. Ia menggunakan resolver untuk melakukan panggilan IPC (misalnya, menggunakan REST, gRPC, atau message queue) ke setiap microservice yang diperlukan (User Service, Order Service, Product Service).
- Efisiensi Jaringan: Karena klien hanya meminta bidang data yang mereka butuhkan (no over-fetching), ukuran payload respons dari Gateway ke klien menjadi sangat kecil. Ini membuat IPC antara klien dan gateway lebih efisien.
  
- Gambar Diagramnya
  ![Diagram menggunakan aplikasi DIA](DiagramUTSSTDT.jpeg)

3. Dengan menggunakan Docker / Docker Compose, buatlah streaming replication di PostgreSQL yang bisa menjelaskan sinkronisasi tulislah langkah - langkah pengerjaanya dan buat penjelasan secukupnya.

Judul: Streaming Replication di PostgreSQL.
Langkah - langkah:
1. Persiapan Awal:
   - Aktifkan Docker Dekstop
   - Struktur Direktori dan File
   - Menentukkan variabelnya seperti nama database, dan kata sandi
   - Mmebuat jaringan dan Volume Docker
2. Jalankan Node Primary: Menjalankan kontainer Primary (pg-primary) menggunakan docker run dengan versi Postgres 16, menghubungkannya ke jaringan replikasi, dan mem-mount volume data.
Konfigurasi Replikasi:
- Membuat Role khusus untuk replikasi (repl_user) pada Primary.
- Mengubah konfigurasi di file postgresql.conf (misalnya: wal_level = replica, max_wal_senders, hot_standby = on).
- Mengubah konfigurasi di file pg_hba.conf untuk mengizinkan koneksi replikasi dari REPL_USER.
- Restart Node Primary agar perubahan konfigurasi diterapkan.
- Slot Replikasi: Membuat Physical Replication Slot bernama standby1 di Primary untuk memastikan Write-Ahead Log (WAL) tidak dihapus sebelum dikirim ke Standby.
3. Konfigurasi Node Standby
- Cloning Database: Melakukan base backup atau cloning database dari Primary ke volume data Standby menggunakan pg_basebackup.
- Jalankan Node Standby: Menjalankan kontainer Standby (pg-standby) menggunakan docker run, menghubungkannya ke jaringan, dan mem-mount volume data yang sudah di-clone.
- Verifikasi Log Standby: Log Standby menunjukkan bahwa database system sudah dalam standby mode dan mulai melakukan recovery serta streaming WAL dari primary.
4. Uji Coba dan Verifikasi
- Cek Status di Primary: Menggunakan pg_stat_replication untuk memastikan koneksi replikasi ke Standby berstatus streaming.
- Tes Replikasi Data:
- Membuat tabel (test_repl) dan menyisipkan data ("Halo dari Primary!") di Primary.
- Memverifikasi bahwa data tersebut berhasil dibaca di Standby.
- Tes Gagal Tulis di Standby: Mencoba menyisipkan data langsung di Standby dan menghasilkan ERROR karena transaksi bersifat read-only (cannot execute INSERT in a read-only transaction), yang merupakan perilaku yang benar untuk Standby.

