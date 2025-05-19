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

### 1.3. Multiple Spawn and removing drop

**Dengan drop(spawner) :**
![image](https://github.com/user-attachments/assets/a5d48e77-ff7c-4a53-af36-228fed97d700)

**Tanpa drop(spawner) :**
![image](https://github.com/user-attachments/assets/57c68935-ba93-48e2-a735-1bb2b18bd0c8)

Pada eksperimen ini, program dimodifikasi dengan menambahkan tiga buah *task* yang masing-masing mencetak pesan sebelum dan sesudah menunggu selama dua detik menggunakan `TimerFuture`. Ketika program dijalankan **dengan `drop(spawner)`**, urutan output di konsol adalah sebagai berikut:

```
Alby's Komputer: hey hey  
Alby's Komputer: howdy!  
Alby's Komputer: howdy2!  
Alby's Komputer: howdy3!  
Alby's Komputer: done!  
Alby's Komputer: done2!  
Alby's Komputer: done3!
```

Urutan ini menjelaskan bahwa semua tugas berhasil dijalankan secara bersamaan (asinkron), masing-masing mencetak "howdy" di awal, kemudian menunggu selama dua detik, dan mencetak "done" setelah timer selesai. Karena `drop(spawner)` dipanggil, executor tahu bahwa tidak akan ada tugas baru yang dikirim, sehingga ia akan berhenti dengan benar setelah seluruh queue task kosong.

Sebaliknya, saat program dijalankan **tanpa `drop(spawner)`**, output awal masih sama, namun setelah seluruh tugas selesai, program **tidak pernah berhenti** dan tetap berjalan tanpa akhir. Ini karena channel sinkron (`sync_channel`) yang digunakan antara executor dan spawner tidak pernah ditutup secara eksplisit. Tanpa `drop(spawner)`, channel dianggap masih terbuka, sehingga executor terus menunggu kemungkinan ada tugas baru masuk, padahal tidak ada lagi tugas yang akan dikirim. Hal ini menyebabkan program *hang* atau tidak selesai secara otomatis.

Dengan demikian, `drop(spawner)` berperan penting sebagai sinyal bagi executor bahwa tidak ada lagi tugas baru yang akan dikirim, dan bahwa ia boleh menghentikan prosesnya ketika task queue habis. Ini merupakan bagian penting dari manajemen *lifecycle* pada sistem asynchronous sederhana di Rust.

---



