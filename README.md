# Tutorial 10 Advance Programming
Muhammad Albyarto Ghazali (2306241695)

### 1.2. Understanding how it works
![image](https://github.com/user-attachments/assets/ed2d2c62-c079-4d45-9d0a-84599b667a69)

Program ini merupakan contoh dari sebuah *async executor* sederhana yang dibuat dengan menggunakan crate `futures` di Rust. Ketika program dijalankan, output yang muncul di console adalah:

```
Alby's Komputer: hey hey
Alby's Komputer: howdy!
Alby's Komputer: done!
```

Urutan output ini menjelaskan bagaimana eksekusi tugas dilakukan secara bertahap dan tidak langsung (async). Pertama, teks **"Alby's Komputer: hey hey"** dicetak langsung oleh fungsi `main` sebelum executor dijalankan. Setelah itu, sebuah asynchronous task dimasukkan ke dalam queue melalui `spawner.spawn(...)`, yang kemudian dieksekusi oleh executor. Program ini langsung mencetak **"Alby's Komputer: howdy!"** sebelum menunggu selama dua detik menggunakan `TimerFuture`, yaitu sebuah *future* khusus yang mendelay penyelesaian hingga waktu tertentu.

Saat `TimerFuture` dipanggil dengan `.await`, eksekusi tugas akan dijeda sementara dan tidak dilanjutkan sampai timer selesai. Setelah dua detik, `TimerFuture` akan membangunkan kembali tugas yang tertunda dengan memanggil mekanisme *wake*, sehingga tugas kembali masuk ke queue executor dan dieksekusi ulang. Saat tugas dilanjutkan, bagian setelah `.await` akan dijalankan, mencetak **"Alby's Komputer: done!"** ke console.

Dengan demikian, program ini memperlihatkan bagaimana eksekusi asynchronous bekerja di Rust, tugas dapat dischedule, didelay, dan dilanjutkan kembali secara efisien tanpa memblokir thread utama. Executor yang dibuat di sini hanya berjalan di satu thread, namun sudah mampu menangani pola *awaiting* dan penjadwalan ulang dengan menggunakan `sync_channel` dan `ArcWake`.

---

