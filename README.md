## Experiment 1.2 – Understanding How It Works

Setelah `spawner.spawn(...)`, ditambahkan:
```rust
println!("Faiq Computer: hey hey!");
```

**Output yang terjadi:**
![](images/Screenshot%202026-05-15%20191945.png)

`spawner.spawn(...)` hanya **mendaftarkan** future ke dalam antrian task, future belum dieksekusi sama sekali. Eksekusi baru terjadi ketika `executor.run()` dipanggil. Karena `println!("hey hey!")` berada **sebelum** `drop(spawner)` dan `executor.run()`, maka ia tercetak **lebih dulu** daripada `howdy!` dan `done!`. Ini membuktikan bahwa async Rust bersifat *lazy*: future tidak melakukan apa-apa sampai di-*poll* oleh executor.