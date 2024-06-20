# LARAVEL REDIS

## POINT UTAMA

### 1. Instalasi

-   Minimal PHP versi 8 atau lebih,

-   Composer versi 2 atau lebih,

-   Lalu pada cmd ketikan `composer create-project laravel/laravel=v10.2.9 belajar-laravel-redis`.

-   Link instalasi redis di OSwin11, [Redis](https://youtu.be/DLKzd3bvgt8?si=mSOQv7e8gh6M7lKT)

-   Perintah menjalankan redis `redis-cli` lalu `ping`.

---

### 2. Pengenalan

-   Redis adalah salah satu database In Memory yang paling populer di dunia,

-   Laravel, sejak awal bisa diintegrasikan dengan baik dengan database Redis, bahkan banyak fitur di Laravel bisa menggunakan Redis, seperti _Cache_, _Session_ dan _Rate Limiting_,

-   Secara default, Redis menggunakan library phpredis (native menggunakan C)

-   Namun, di beberapa kasus, kadang sulit menginstall library phpredis karena membutuhkan
    kompilasi bahasa C,

-   Oleh karena itu, di kelas ini kita akan mengunakan library Predis, yang hanya menggunakan kode PHP,

-   [Dokumentasi](https://github.com/phpredis/phpredis)

-   [Dokumentasi](https://github.com/predis/predis)
  
-   Perintah instalasi `composer require predis/predis`

---

### 3. Redis Facade

-   Secara default, semua konfigurasi untuk Redis di aplikasi Laravel terdapat di file `config/database.php`,

-   Bisa di ubah konfigurasi redis di file tersebut, ubah yang defaultnya menggunakan `phpredis` menjadi `predis` pada konfigurasi file nya.

    ```PHP
    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'), // default `phpredis` ubah menjadi `predis`
    ]
    ```

-   Untuk menggunakan Redis di Laravel, kita bisa menggunakan Redis _Facade_ yang secara otomatis membaca konfigurasi dari konfigurasi redis di Laravel.

-   [Dokumentasi](https://laravel.com/api/10.x/Illuminate/Support/Facades/Redis.html)

-   Untuk mengirim perintah ke Redis, kita bisa menggunakan _method_ `command()` di Redis _Facade_,

-   Atau bisa langsung menggunakan nama _method_ sesuai dengan command di Redis, Redis _Facade_ akan menggunakan Magic _Method_ untuk mengubah nama method secara otomatis menjadi nama perintah yang dikirim ke Redis.

-   Kode Ping Redish

    ```PHP
    public function testPing()
    {
        $response = Redis::command("ping");
        self::assertEquals("PONG", $response);

        $response = Redis::ping();
        self::assertEquals("PONG", $response);
    }
    ```

-   Ubah configurasi redis berada di file `.env`.

---

### 4. String

-   Struktur data yang sering digunakan di Redis adalah _String_,

-   Command yang sering kita gunakan adalah menggunakan `set()`, `setEx()`, `get()`, `mGet()`, dan lain-lain.

-   Kode String

    ```PHP
    public function testString()
    {
        Redis::setex("name", 2, "Gusti");
        $response = Redis::get("name");
        self::assertEquals("Gusti", $response);

        sleep(5); // berhenti selama 5 detik agar expired

        $response = Redis::get("name");
        self::assertNull($response);
    }
    ```

---

### 5. Struktur Data List

-   Kode list

    ```PHP
    public function testList()
    {
        Redis::del("names");

        Redis::rpush("names", "Gusti");
        Redis::rpush("names", "Alifiraqsha");
        Redis::rpush("names", "Akbar");

        $response = Redis::lrange("names", 0, -1);
        self::assertEquals(["Gusti", "Alifiraqsha", "Akbar"], $response);

        self::assertEquals("Gusti", Redis::lpop("names"));
        self::assertEquals("Alifraqsha", Redis::lpop("names"));
        self::assertEquals("Akbar", Redis::lpop("names"));
    }
    ```

---

### 6. Struktur Data Set

-   Kode set

    ```PHP
    public function testSet()
    {
        Redis::del("names");

        Redis::sadd("names", "Gusti");
        Redis::sadd("names", "Gusti");
        Redis::sadd("names", "Alifiraqsha");
        Redis::sadd("names", "Alifiraqsha");
        Redis::sadd("names", "Akbar");
        Redis::sadd("names", "Akbar");

        $response = Redis::smembers("names"); // smembers untuk mendapatkan data unik
        self::assertEquals(["Gusti", "Alifiraqsha", "Akbar"], $response);
    }
    ```

---

### 7. Struktur Data Sorted Set

-   Kode sorted set

    ```PHP
     public function testSortedSet()
    {

        Redis::del("names");

        Redis::zadd("names", 100, "Gusti");
        Redis::zadd("names", 100, "Gusti");
        Redis::zadd("names", 85, "Alifiraqsha");
        Redis::zadd("names", 85, "Alifiraqsha");
        Redis::zadd("names", 95, "Akbar");
        Redis::zadd("names", 95, "Akbar");

        $response = Redis::zrange("names", 0, -1); // zrange mengurutkan index data dari yang terkecil->terbesar
        self::assertEquals(["Alifiraqsha", "Akbar", "Gusti"], $response);
    }
    ```

---

### 8. Struktur Data Hash

-   Kode hash

    ```PHP
    public function testHash()
    {
        Redis::del("user:1");

        Redis::hset("user:1", "name", "Gusti");
        Redis::hset("user:1", "email", "gusti@localhost");
        Redis::hset("user:1", "age", 30);

        $response = Redis::hgetall("user:1");
        self::assertEquals([
            "name" => "Gusti",
            "email" => "gusti@localhost",
            "age" => "30"
        ], $response);
    }
    ```

---

### 9. Struktur Data Geo Point

-   Kode Geo Point

    ```PHP
    public function testGeoPoint()
    {
        Redis::del("sellers");

        Redis::geoadd("sellers", 106.820990, -6.174704, "Toko A");
        Redis::geoadd("sellers", 106.822696, -6.176870, "Toko B");

        // Menghitung jarak secara manual
        $result = Redis::geodist("sellers", "Toko A", "Toko B", "km");

        // Membandingkan jarak dalam toleransi tertentu (dalam kasus ini, hingga dua tempat desimal)
        self::assertEquals(0.31, round($result, 2));

        // Menggunakan GEORADIUS untuk mencari toko-toko dalam radius tertentu
        $radiusResult = Redis::georadius("sellers", 106.819875, -6.176091, 5, "km", "WITHDIST");
        $stores = [];
        foreach ($radiusResult as $storeInfo) {
            $stores[] = $storeInfo[0]; // Menyimpan nama toko
        }

        // Memeriksa apakah hasilnya sama dengan yang diharapkan
        self::assertEquals(["Toko B", "Toko A"], $stores);
    }
    ```

---

### 10. Hyper Log Log

-   Kode Hyper Log Log / menghitung jumlah data unik

    ```PHP
    public function testHyperLogLog()
    {
        Redis::pfadd("visitors", "gusti", "alifiraqsha", "akbar");
        Redis::pfadd("visitors", "gusti", "kiana", "elaina");
        Redis::pfadd("visitors", "yuna", "kiana", "elaina");

        $result = Redis::pfcount("visitors");
        self::assertEquals(6, $result);

    }
    ```

---

## PERTANYAAN & CATATAN TAMBAHAN

-   Tidak ada.

---

### KESIMPULAN

-   Redis menawarkan berbagai keuntungan termasuk kecepatan akses data yang sangat tinggi, dukungan untuk berbagai tipe data (seperti strings, hashes, lists, sets, dan sorted sets), serta fitur-fitur tambahan seperti pub/sub, Lua scripting, dan geospatial indexing.
