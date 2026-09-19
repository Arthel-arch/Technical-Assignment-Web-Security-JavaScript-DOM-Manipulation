# Technical Assignment: Web Security, JavaScript, dan DOM Manipulation

## Tujuan

Assignment ini membahas manipulasi DOM menggunakan JavaScript dan keamanan saat menampilkan input dari pengguna. Demo pada [live-search.html](live-search.html) menerapkan pencarian secara real time tanpa menggunakan `innerHTML`.

## `innerHTML` vs. `textContent`

### `innerHTML`

`innerHTML` membaca atau mengubah isi elemen sebagai string HTML. Browser akan mem-parsing tag HTML yang terdapat di dalam string tersebut.

```js
output.innerHTML = "<strong>Welcome</strong>";
```

Gunakan `innerHTML` hanya untuk markup yang sudah dikontrol dan dipercaya, atau setelah input diproses dengan sanitizer yang sesuai. Jangan memasukkan input pengguna secara langsung karena dapat menyebabkan Cross-Site Scripting (XSS).

### `textContent`

`textContent` membaca atau mengubah isi elemen sebagai teks biasa. Tag HTML di dalam nilai tidak akan diproses sebagai markup.

```js
output.textContent = userInput;
```

Jika `userInput` berisi `<strong>Hello</strong>`, browser akan menampilkan teks tersebut apa adanya, bukan membuat teks tebal. Karena itu, `textContent` adalah pilihan yang tepat untuk nama, komentar, hasil pencarian, dan data lain yang berasal dari pengguna.

| API | Cara memproses nilai | Penggunaan yang sesuai | Risiko |
| --- | --- | --- | --- |
| `innerHTML` | Mem-parsing nilai sebagai HTML | Markup statis atau markup yang sudah disanitasi | XSS jika nilai tidak dipercaya |
| `textContent` | Memperlakukan nilai sebagai teks biasa | Input pengguna dan data dinamis | Tidak dapat membuat markup dari nilai |

## Contoh Skenario XSS

Bayangkan sebuah situs memiliki fitur komentar. Komentar disimpan ke database lalu ditampilkan kepada setiap pengunjung menggunakan kode berikut:

```js
commentBox.innerHTML += "<p>" + commentInput.value + "</p>";
```

Kode tersebut rentan karena nilai `commentInput.value` digabungkan langsung ke HTML. Seorang penyerang dapat mengirim komentar berisi payload JavaScript, misalnya:

```html
<img src="x" onerror="alert('XSS')">
```

Proses eksploitasinya:

1. Penyerang memasukkan payload ke formulir komentar.
2. Payload disimpan atau langsung ditampilkan oleh aplikasi.
3. Saat korban membuka halaman komentar, browser mem-parsing payload sebagai HTML.
4. Event handler berbahaya dijalankan dengan konteks sesi korban.

Dampaknya dapat berupa perubahan isi halaman, pencurian data sesi, pengiriman request atas nama korban, atau pengalihan pengguna ke halaman phishing. Perbaikan sederhananya adalah membuat elemen dengan `createElement()` dan memasukkan komentar menggunakan `textContent`:

```js
const paragraph = document.createElement("p");
paragraph.textContent = commentInput.value;
commentBox.append(paragraph);
```

## Demo Live Search

File [live-search.html](live-search.html) berisi katalog produk dan kolom pencarian. Setiap kali pengguna mengetik:

1. Event `input` dijalankan.
2. `array.filter()` mencari nama produk atau kategori yang cocok.
3. Fungsi render menghapus hasil lama.
4. Elemen baru dibuat menggunakan `createElement()`.
5. Nilai produk dimasukkan menggunakan `textContent`.

Implementasi ini sengaja tidak menggunakan `innerHTML`, sehingga data katalog diperlakukan sebagai teks dan tidak menjadi markup yang dapat dieksekusi.

Untuk menjalankan demo, buka [live-search.html](live-search.html) langsung di browser atau gunakan Live Server di VS Code.

## Verifikasi

- Pencarian memperbarui hasil secara real time.
- Pencarian berdasarkan nama produk maupun kategori berfungsi.
- Pesan ditampilkan ketika tidak ada hasil yang cocok.
- Implementasi menggunakan `input`, `filter`, `createElement`, dan `textContent`.
- File demo tidak menggunakan `innerHTML`.