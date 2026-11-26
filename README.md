# Detektor Kebocoran Gas (Arduino UNO + MQ-5)

Proyek detektor kebocoran gas sederhana menggunakan Arduino UNO dan sensor MQ-5. Alat ini membaca kadar gas mudah terbakar di sekitar sensor (seperti LPG, gas alam, butana/gas korek api, serta uap alkohol), lalu menampilkan statusnya di LCD 16x2 dan menyalakan buzzer kalau kadar gas melonjak.

Untuk saat ini alat masih berjalan secara lokal (offline), belum terhubung ke internet. Kalau mau dikembangkan jadi versi IoT, catatan singkatnya ada di bagian bawah.

## Cara Kerja Singkat

- Saat baru dinyalakan, alat melakukan kalibrasi udara ruangan selama 10 detik. Nilai udara saat itu dipakai sebagai patokan normal supaya alat langsung mulai di `Status: AMAN`.
- Kalau sensor makin hangat dan angka udara bersihnya perlahan turun, nilai patokan normal akan ikut menyesuaikan otomatis.
- Pembacaan sensor diambil dari rata-rata 10 sampel tiap 500 ms biar angkanya tidak loncat-loncat.
- Perubahan status memakai ambang histeresis: masuk **BAHAYA** saat nilai naik `+80` dari patokan normal, dan baru kembali **AMAN** kalau sudah turun ke `+45`. Tujuannya supaya tampilan LCD, LED, dan buzzer tidak kedip-kedip di angka perbatasan.
- Buzzer berbunyi putus-putus tiap 300 ms menggunakan `millis()`, dan bisa dipakai untuk buzzer aktif maupun pasif.

## Komponen yang Dibutuhkan

- **Arduino UNO R3** + kabel USB data
- **Sensor Gas MQ-5** (modul 4 pin)
- **LCD 16x2 + modul I2C** (PCF8574)
- **LED Hijau 5 mm** + **Resistor 220 atau 330 Ohm** (wajib dipasang seri dengan LED)
- **Buzzer 5V** *(opsional, bisa pakai tipe aktif 2-kaki, pasif, atau modul 3-pin)*
- **Breadboard** dan kabel jumper (Male-to-Male & Male-to-Female)

> **Catatan daya:** Pakai kabel USB dari laptop atau adaptor 5V (minimal 1A). Jangan pakai baterai kotak 9V karena arusnya tidak kuat mengangkat pemanas MQ-5 dan LCD bersamaan, nanti Arduino bakal restart terus.

## Rangkaian (Wiring)

Sambungkan dulu pin `5V` Arduino ke jalur `+` breadboard dan `GND` ke jalur `-` breadboard sebagai jalur daya bersama. Kalau pakai breadboard panjang (830 titik), cek jalur `+` dan `-` di bagian tengah karena sering terputus. Sambungkan sisi kiri dan kanannya pakai jumper pendek, atau tancapkan semua komponen di sisi yang sama.

| Komponen | Pin Komponen | Pin Arduino / Breadboard | Keterangan |
| :--- | :--- | :--- | :--- |
| **LCD I2C 16x2** | `VCC` / `GND` | `5V` / `GND` breadboard | |
| | `SDA` | `A4` | Jangan tertukar dengan `SCL` |
| | `SCL` | `A5` | |
| **Sensor MQ-5** | `VCC` / `GND` | `5V` / `GND` breadboard | |
| | `AO` | `A0` | Pakai `AO` (analog), `DO` biarkan kosong |
| **LED Hijau** | Kaki panjang (`+`) | Pin `7` (lewat resistor 220-330 Ohm) | Wajib pakai resistor |
| | Kaki pendek (`-`) | `GND` breadboard | |
| **Buzzer 5V (2 pin)** | Kaki panjang (`+`) | Pin `8` | Jangan langsung ke `5V` |
| | Kaki pendek (`-`) | `GND` breadboard | |
| **Modul Buzzer (3 pin)** | `VCC` / `GND` | `5V` / `GND` breadboard | Khusus versi modul 3 pin |
| | `I/O` / `S` / `SIG` | Pin `8` | |

Urutan pemasangan LED di breadboard:

```text
Pin 7 Arduino ---> [ Resistor 220 / 330 Ohm ] ---> (+) LED Hijau (-) ---> GND
```

## Instalasi & Upload Kode

1. Buka **Arduino IDE**, lalu install library **`hd44780`** buatan **Bill Perry** lewat Library Manager (`Ctrl + Shift + I`).
   Di sini saya pakai `hd44780` dibanding `LiquidCrystal_I2C` biasa karena alamat I2C (`0x27` atau `0x3F`) dan pin backpack-nya terdeteksi otomatis, jadi lebih aman dari masalah huruf acak di layar.
2. Buka file kode di [`detektor_gas_mq5/detektor_gas_mq5.ino`](detektor_gas_mq5/detektor_gas_mq5.ino).
3. Pilih board **Arduino Uno** dan port COM yang sesuai. *(Untuk Arduino clone dengan chip CH340, pastikan driver CH340 sudah terpasang kalau port belum muncul.)*
4. Upload kode ke Arduino. Lebih enak upload sebelum semua kabel dipasang (atau cabut dulu kabel `5V` dan `GND` ke breadboard) biar proses upload tidak gagal.
5. Setelah selesai (`Done uploading`), cabut USB baru mulai rakit kabelnya. Supaya gampang mengecek kalau ada masalah, rakit bertahap saja: tes LCD dulu, lanjut MQ-5, lalu LED, terakhir buzzer.

### Pengaturan di Dalam Kode

Beberapa variabel di bagian atas file `.ino` bisa diubah sesuai kebutuhan:

| Variabel | Default | Fungsi |
| :--- | :---: | :--- |
| `PAKAI_BUZZER` | `true` | Ubah ke `false` kalau rangkaian tidak memakai buzzer |
| `BUZZER_AKTIF` | `true` | Ubah ke `false` kalau memakai buzzer pasif |
| `SELISIH_BAHAYA` | `80` | Ambang kenaikan dari udara normal untuk memicu bahaya (turunkan ke ~50 biar lebih sensitif) |
| `SELISIH_AMAN` | `45` | Ambang kembali aman, pastikan selalu lebih kecil dari `SELISIH_BAHAYA` |
| `DETIK_KALIBRASI` | `10` | Durasi kalibrasi awal dalam detik (bisa dinaikkan ke 20-30 detik) |

## Cara Pakai & Pengujian

1. Taruh rangkaian di tempat berudara bersih (jauh dari gas, alkohol, parfum, atau asap solder), lalu colokkan daya 5V.
2. Tunggu hitungan mundur `Kalibrasi Udara` selama 10 detik. Kalau sensor MQ-5 baru pertama kali dinyalakan, biarkan hangat sekitar 3-5 menit dulu biar pembacaannya stabil.
3. Kondisi indikator alat:

| Kondisi | Tampilan LCD | LED Hijau | Buzzer |
| :--- | :--- | :---: | :---: |
| **Kalibrasi** | `Kalibrasi Udara` / `Tunggu: 10 dtk` | Menyala | Mati |
| **Aman** | `Gas MQ5: 250` / `Status: AMAN` | Menyala | Mati |
| **Bahaya** | `Gas MQ5: 620` / `AWAS! GAS BOCOR!` | Mati | Bunyi bip-bip |

4. **Cara tes sensor:**
   - **Pakai korek api gas:** dekatkan ujung korek sekitar 2 cm dari jaring sensor, lalu tekan tuas gasnya 1-2 detik **tanpa menyalakan api**.
   - **Alternatif tanpa korek:** teteskan sedikit parfum atau *hand sanitizer* ke tisu, atau buka tutup spidol *boardmarker*, lalu dekatkan 1-2 cm ke depan sensor. Jangan semprot cairan langsung ke jaring logamnya.
   - Nilai bacaan dan batas dinamisnya juga bisa dipantau lewat **Serial Monitor** di `9600 baud`.

## Troubleshooting

- **Gagal upload (`programmer is not responding` / `not in sync`):** Cabut sementara kabel `5V` dan `GND` yang ke breadboard, tutup Serial Monitor, pastikan port COM sudah benar (dan driver CH340 terpasang jika pakai clone), atau coba ganti kabel USB.
- **LCD menyala tapi kosong / hanya kotak putih:** Putar potensiometer biru di belakang modul I2C pelan-pelan pakai obeng kecil sampai tulisannya muncul.
- **LCD kedip-kedip atau muncul karakter aneh:** Pastikan sudah pakai library `hd44780` (bukan `LiquidCrystal_I2C`), cek kabel `SDA`, `SCL`, dan `GND` jangan kendor, dan pastikan sumber daya 5V stabil.
- **Arduino restart terus (hitungan kalibrasi mengulang):** Biasanya tegangan drop atau ada yang korslet. Pastikan LED sudah pakai resistor 220-330 Ohm dan kedua kakinya tidak menancap di baris breadboard yang sama, kaki `+` buzzer masuk ke Pin 8 (bukan `5V`), dan jangan pakai baterai 9V.
- **Angka `Gas MQ5` mentok di 850-1023:** Cek kabel `GND` MQ-5 (kalau daya masuk normal, jaring besinya terasa hangat) dan pastikan pin yang disambung ke `A0` adalah `AO`, bukan `DO`.
- **Status sudah `AWAS! GAS BOCOR!` tapi buzzer diam atau cuma bunyi "tik-tik":** Berarti buzzernya tipe pasif. Ubah `BUZZER_AKTIF` di kode menjadi `false`, lalu upload ulang.
- **Sensor MQ-5 terasa hangat:** Normal, karena ada elemen pemanas di dalamnya. Hindari menyentuh jaring logamnya langsung.

## Ide Pengembangan

Kalau mau dikembangkan lebih lanjut jadi versi IoT:
- Ganti mikrokontroler ke **ESP8266** atau **ESP32** supaya bisa kirim notifikasi ke Telegram, WhatsApp, atau Blynk. Perlu dicatat, output `AO` MQ-5 bisa mencapai 5V sedangkan pin analog ESP maksimal 3.3V (1V di ADC mentah ESP8266), jadi wajib tambah pembagi tegangan (*voltage divider*, misalnya resistor 10k dan 20k Ohm).
- Tambah modul **Relay 5V** untuk menyalakan kipas *exhaust* otomatis saat kebocoran gas terdeteksi.

---

*Catatan: Proyek ini dibuat untuk keperluan belajar dan prototipe, bukan pengganti detektor gas bersertifikat untuk keamanan rumah/industri. Jangan pernah menguji sensor dengan api terbuka.*
