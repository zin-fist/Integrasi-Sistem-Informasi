## Tabel Hasil Pengujian Benchmark

| **Jumlah Rekaman** | **Ukuran XML (Bytes)** | **Ukuran JSON (Bytes)** | **Efisiensi Ukuran (%)** |
|---:|---:|---:|---:|
| **10 Data** | 1438 | 1166 | 18.92% |
| **50 Data** | 7058 | 5826 | 17.46% |
| **100 Data** | 14084 | 11652 | 17.27% |
| **500 Data** | 70684 | 58652 | 17.02% |

Dari hasil pengujian, ukuran JSON selalu lebih kecil dibandingkan XML. Pengurangan ukuran berada di sekitar 17–19%. Semakin banyak data, perbedaan ukuran XML dan JSON juga semakin besar.

---

## Analisis Pertanyaan Kritis

### 1. Analisis Struktur Payload

Perbedaan ukuran XML dan JSON disebabkan oleh struktur sintaks masing-masing format.

- **Tag pembuka dan penutup**  
  XML membutuhkan tag seperti `<nama>...</nama>`, sedangkan JSON cukup menggunakan `"nama": value`.

- **Struktur yang berulang**  
  Elemen `<student>` serta nama field seperti `id`, `nama`, `jurusan`, dan `status` harus ditulis kembali pada setiap record XML.

- **Sintaks tambahan**  
  XML memiliki struktur seperti deklarasi, atribut, dan namespace yang dapat menambah ukuran payload. Pada benchmark ini struktur XML yang digunakan masih sederhana.

Jadi, banyaknya tag dan struktur XML membuat ukuran payload lebih besar dibandingkan JSON.

---

### 2. Analisis Deserialisasi / Parsing di Sisi Klien

XML dan JSON memiliki cara parsing yang berbeda.

**Pada XML:**

1. Response diproses sebagai dokumen XML.
2. Parser seperti `DOMParser` dapat digunakan untuk membaca struktur XML.
3. Data kemudian dicari berdasarkan tag dan diambil nilainya.

**Pada JSON:**

1. Response dapat diproses menggunakan `response.json()` atau `JSON.parse()`.
2. Hasilnya langsung berupa object dan array.
3. Data dapat diakses melalui property, misalnya `data.data[0].nama`.

JSON lebih banyak digunakan pada aplikasi web modern karena strukturnya lebih sederhana, mudah digunakan dengan JavaScript, dan ukuran payload-nya relatif lebih kecil. XML tetap berguna untuk kebutuhan tertentu seperti integrasi sistem lama.

---

### 3. Bottleneck Arsitektur RESTful Synchronous

Jika sistem legacy membutuhkan beberapa detik untuk menulis ke `data.txt`, request POST yang datang bersamaan dapat saling menunggu.

Beberapa dampaknya yaitu:

- **Request menjadi antre**, karena proses penulisan harus dilakukan secara terbatas.
- **Waktu respons meningkat**, karena client harus menunggu proses legacy selesai.
- **Resource server terbebani**, terutama jika banyak request menunggu.
- **Timeout dapat terjadi** jika waktu tunggu terlalu lama.

Artinya, API Gateway tetap bergantung pada kecepatan sistem legacy sehingga proses penulisan file dapat menjadi bottleneck.

**Synchronous dan gRPC**

Pada komunikasi synchronous, client harus menunggu response dari server. gRPC menggunakan HTTP/2 yang mendukung multiplexing sehingga beberapa komunikasi dapat berjalan melalui satu koneksi. Namun, jika penulisan ke `data.txt` tetap lambat, bottleneck pada sistem legacy masih tetap ada.

**gRPC (Minggu 5)**

Pada komunikasi synchronous, client harus menunggu response dari server. gRPC menggunakan HTTP/2 yang mendukung multiplexing sehingga beberapa komunikasi dapat berjalan melalui satu koneksi. Hal ini membuat komunikasi lebih efisien. Namun, jika penulisan ke `data.txt` tetap lambat, bottleneck pada sistem legacy masih ada.

**Asynchronous Message Broker (Minggu 6)**

Message Broker mengatasi masalah dengan memasukkan request ke dalam queue terlebih dahulu. API dapat menerima request tanpa harus menunggu proses penyimpanan selesai. Selanjutnya consumer memproses request ke sistem legacy secara bertahap, sehingga beban request tidak langsung menumpuk pada sistem legacy.
