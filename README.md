## Experiment 1.2 - Understanding How It Works

Setelah `spawner.spawn(...)`, ditambahkan:
```rust
println!("Faiq Computer: hey hey!");
```

**Output yang terjadi:**
![](images/Screenshot%202026-05-15%20191945.png)

`spawner.spawn(...)` hanya mendaftarkan future ke dalam antrian task, sehingga future belum langsung dieksekusi pada saat baris tersebut dipanggil. Eksekusi sebenarnya baru terjadi ketika `executor.run()` mulai berjalan dan melakukan polling terhadap task yang sudah masuk ke antrian. Karena `println!("Faiq Computer: hey hey!")` berada sebelum `drop(spawner)` dan sebelum `executor.run()`, maka pesan tersebut tercetak lebih dulu dibandingkan `howdy!` dan `done!`. Urutan output ini menunjukkan bahwa proses spawn tidak sama dengan menjalankan isi future secara langsung. Future yang dibuat masih berada dalam keadaan menunggu sampai executor mengambil dan mem-poll future tersebut. Hal ini membuktikan bahwa async Rust bersifat lazy, karena future tidak melakukan pekerjaan apa pun sebelum diproses oleh executor. Dengan begitu, hasil percobaan ini sesuai dengan konsep dasar async Rust dan penjelasannya terdengar masuk akal.

## Experiment 1.3 - Multiple Spawn and Removing `drop`

### Tanpa `drop`
![](images/Screenshot%202026-05-15%20192745.png)

Jika `drop(spawner)` dihapus, program akan hang selamanya dan tidak pernah selesai dengan sendirinya. Hal ini terjadi karena `executor.run()` masih menunggu channel receiver sampai semua sender benar-benar ditutup. Selama variabel `spawner` masih hidup, masih ada `SyncSender` aktif yang dianggap dapat mengirim task baru ke executor. Executor tidak bisa memastikan bahwa tidak akan ada task tambahan lagi, sehingga proses menunggu tetap berlangsung. Walaupun task yang sudah di-spawn telah selesai dijalankan, channel belum dianggap tertutup karena sender utamanya masih ada. Kondisi tersebut membuat executor terus berada dalam loop dan tidak mencapai akhir program. Jadi, `drop(spawner)` diperlukan agar executor tahu bahwa tidak ada lagi task baru yang akan masuk.

### Dengan `drop`
![](images/Screenshot%202026-05-15%20192910.png)

Dengan `drop(spawner)`, semua task yang sudah didaftarkan tetap dapat dijalankan oleh executor sampai selesai. Semua pesan `howdy` muncul hampir bersamaan karena setiap task mulai diproses tanpa harus menunggu task sebelumnya selesai sepenuhnya. Setelah sekitar dua detik, semua pesan `done` muncul karena masing-masing task memiliki `TimerFuture` dengan durasi tunggu yang serupa. Hasil ini menunjukkan bahwa executor menjalankan task secara concurrent, bukan secara sequential satu per satu sampai tuntas. Ketika sebuah `TimerFuture` sedang menunggu waktu selesai, executor dapat berpindah untuk memproses task lain yang juga sudah ada di antrian. Setelah timer selesai, task yang bersangkutan dibangunkan kembali agar bisa melanjutkan eksekusi sampai mencetak `done`. Dengan demikian, percobaan ini menjelaskan bahwa penggunaan `drop(spawner)` membuat program bisa berhenti dengan benar setelah seluruh task selesai.
