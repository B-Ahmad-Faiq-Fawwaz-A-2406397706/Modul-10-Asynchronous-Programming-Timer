## Experiment 1.2 – Understanding How It Works

Setelah `spawner.spawn(...)`, ditambahkan:
```rust
println!("Faiq Computer: hey hey!");
```

**Output yang terjadi:**
![](images/Screenshot%202026-05-15%20191945.png)

`spawner.spawn(...)` hanya **mendaftarkan** future ke dalam antrian task, future belum dieksekusi sama sekali. Eksekusi baru terjadi ketika `executor.run()` dipanggil. Karena `println!("hey hey!")` berada **sebelum** `drop(spawner)` dan `executor.run()`, maka ia tercetak **lebih dulu** daripada `howdy!` dan `done!`. Ini membuktikan bahwa async Rust bersifat *lazy*: future tidak melakukan apa-apa sampai di-*poll* oleh executor.

## Experiment 1.3 – Multiple Spawn and Removing `drop`

### Tanpa `drop`
![](images/Screenshot%202026-05-15%20192745.png)

Jika `drop(spawner)` dihapus, program **hang selamanya** (tidak pernah selesai). Ini karena `executor.run()` menunggu channel receiver hingga semua sender di-drop. Selama `spawner` masih hidup (ada SyncSender aktif), executor tidak tahu apakah masih akan ada task baru, sehingga terus menunggu tanpa berhenti.

### Dengan `drop`
![](images/Screenshot%202026-05-15%20192910.png)

Semua `howdy` muncul hampir bersamaan, lalu setelah ~2 detik semua `done` muncul. Ini menunjukkan bahwa executor menjalankan task secara **concurrent** yang artinya tidak menunggu satu task selesai sebelum memulai yang lain. Saat TimerFuture menunggu, executor beralih ke task lain.