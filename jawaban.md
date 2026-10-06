# Laporan Analisis pertemuan 3

## 1. Analisis Pola Arsitektur
Rancangan ini dikategorikan sebagai Adapter Pattern dalam arsitektur EAI karena program berbasis C hanya mampu membaca dan menulis file teks lokal tanpa memiliki antarmuka jaringan. Server `adapter_service.py` bertindak sebagai adapter yang mengubah permintaan HTTP/XML dari klien modern menjadi operasi yang dapat diproses oleh sistem legacy, tanpa perlu mengubah kode sumber C.

Keuntungan bagi klien web adalah klien dapat mengakses sistem legacy melalui HTTP dan XML tanpa perlu mengetahui cara kerja internal sistem tersebut. Hal ini juga membuat sistem lebih fleksibel dan memungkinkan integrasi dengan aplikasi lain.

---

## 2. Analisis Overhead Serialisasi Data

Respons XML aktual dari endpoint `GET /students/2415354001`:

```xml
<?xml version='1.0' encoding='utf-8'?><StudentResponse><NIM>2415354001</NIM><Nama>I Made Sujana</Nama><Jurusan>Teknologi Informasi</Jurusan><Status>ACTIVE</Status></StudentResponse>
```

### Kalkulasi Data Aktual (Murni)

- `2415354001` = 10 byte
- `I Made Sujana` = 13 byte
- `Teknologi Informasi` = 19 byte
- `ACTIVE` = 6 byte
- **Total Data Asli:** 10 + 13 + 19 + 6 = **48 byte**

### Kalkulasi Ukuran Respons XML

- Total ukuran seluruh karakter dokumen XML, termasuk deklarasi XML dan tag pembungkus, adalah **181 byte**.

### Kalkulasi Rasio Overhead

- **Ukuran Overhead:** 181 − 48 = **133 byte**
- **Rasio Overhead (vs. Data Murni):** (133 ÷ 48) × 100% = **277,08%**
- **Proporsi Overhead (vs. Total Payload):** (133 ÷ 181) × 100% = **73,48%**

### Kesimpulan :
serialisasi XML menambahkan **133 byte overhead**, atau sekitar **277,08%** dibandingkan ukuran data murninya.

## 3. Analisis Integritas Data dan Message Broker

Jika 500 request POST diterima dalam waktu yang hampir bersamaan, proses penulisan ke `students_db.txt` dapat menjadi masalah karena semua data diarahkan ke file yang sama. Semakin banyak request yang masuk, semakin besar kemungkinan terjadi antrean atau gangguan saat proses penyimpanan.

Beberapa masalah yang mungkin terjadi antara lain:

- proses penulisan menjadi lebih lambat karena request harus diproses bergantian
- data dapat mengalami konflik apabila terdapat beberapa proses yang mengakses file secara bersamaan
- request yang gagal dapat menyebabkan data tidak tersimpan dengan sempurna
- file teks kurang cocok digunakan sebagai media penyimpanan ketika jumlah akses semakin besar

Untuk kondisi tersebut, **Message Broker** dapat digunakan sebagai perantara antara adapter dan sistem legacy. Request yang masuk tidak langsung semuanya menulis ke file, tetapi dapat ditampung terlebih dahulu dan kemudian diproses oleh sistem legacy secara bertahap.

Dengan pendekatan tersebut, beban sistem dapat lebih terkontrol, kemungkinan konflik penulisan dapat dikurangi, dan integrasi dengan sistem legacy akan lebih mudah dikembangkan ketika jumlah pengguna bertambah.


