# Praktik Mandiri Internet of Things

Sistem berbasis ESP32 untuk memantau suhu, kelembaban, dan jarak objek melalui LCD I2C serta dashboard Blynk. Buzzer memberikan peringatan berdasarkan jarak dan izin alarm yang dikendalikan dari tombol fisik maupun Blynk.

## Identitas

| Keterangan | Data |
| --- | --- |
| Nama | Jauza Ilham Mahardhika Putra |
| NIM | 123240174 |
| Kelas | IF-G |

## Komponen dan sambungan

Sensor suhu dan kelembaban pada kebutuhan tugas adalah **DHT11**. Simulasi Wokwi memakai **DHT22** sebagai pengganti, sehingga kode simulasi menggunakan `#define DHTTYPE DHT22`. Untuk praktik dengan DHT11, ubah definisi tersebut menjadi `DHT11`.

| Komponen | Sinyal | Pin ESP32 |
| --- | --- | --- |
| DHT22 pada simulasi | Data | GPIO 4 |
| HC-SR04 | TRIG | GPIO 16 |
| HC-SR04 | ECHO | GPIO 17 |
| Buzzer | Positif | GPIO 14 |
| Push button | Kontak sinyal | GPIO 15 |
| LCD I2C | SDA | GPIO 22 |
| LCD I2C | SCL | GPIO 23 |

Negatif buzzer dan kontak tombol lainnya terhubung ke GND. Semua komponen menggunakan GND bersama. Tombol menggunakan `INPUT_PULLUP`, sehingga rangkaian tombol simulasi tidak memerlukan resistor pull-up eksternal. Alamat LCD adalah `0x27`.

## Pengaturan template Blynk

Gunakan template dan device yang sesuai dengan identitas koneksi pada `src/main.cpp`.

| Pengaturan | Nilai |
| --- | --- |
| Template Name | LatihanResponsi |
| Hardware | ESP32 |
| Connection Type | WiFi |
| Jenis datastream | Virtual Pin |

Jika identitas koneksi belum diisi, masukkan Template ID, Template Name, dan Auth Token device pada bagian berikut di `src/main.cpp`:

```cpp
#define BLYNK_TEMPLATE_ID "ISI_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "LatihanResponsi"
#define BLYNK_AUTH_TOKEN "ISI_AUTH_TOKEN"
```

Gunakan token device milik sendiri. Nilai pada contoh adalah penanda pengisian; Auth Token asli tidak dicantumkan dalam README.

## Pengaturan datastream

Tabel berikut mengikuti konfigurasi Blynk terbaru yang digunakan pada proyek dan laporan.

| Pin | Nama | Tipe data | Min | Max | Default | Unit |
| --- | --- | --- | ---: | ---: | --- | --- |
| V0 | Buzzer | Integer | 0 | 1 | 1 | None |
| V1 | Temperature | Double | -41 | 90 | 0 | °C |
| V2 | Humidity | Double | 0 | 100 | 0 | % |
| V3 | Distance | Double | 0 | 500 | 0 | cm |
| V4 | Condition | String | — | — | Kosong | None |
| V5 | Buzzer Status | Integer | 0 | 1 | 0 | None |
| V6 | Objek Terdeteksi | Integer | 0 | 1 | 0 | None |
| V7 | Buzzer Dibisukan | Integer | 0 | 1 | 0 | None |

Atur **Semantics** sebagai berikut:

| Pin | Semantics |
| --- | --- |
| V1 | Temperature (Celsius) |
| V2 | Humidity |
| V3 | Distance / Proximity |
| V0, V4, V5, V6, V7 | Kosong / None |

Gunakan satu angka desimal untuk tampilan V1–V3. V1 adalah suhu ruangan dari sensor DHT, sehingga semantic yang dipakai adalah **Temperature (Celsius)**.

## Pengaturan widget dashboard

| Widget | Datastream | Judul dan pengaturan |
| --- | --- | --- |
| Switch | V0 | Buzzer / Izin Alarm; ON = 1, OFF = 0; label ON dan OFF |
| Gauge | V1 | Temperature; unit °C |
| Gauge | V2 | Humidity; unit % |
| Gauge | V3 | Distance; unit cm |
| Label | V4 | Condition; menampilkan teks kondisi dari program |
| LED | V5 | Buzzer Status; 1 = berbunyi, 0 = mati |
| Label | V6 | Objek Terdeteksi; 1 = objek dekat, 0 = tidak terdeteksi |
| LED | V7 | Buzzer Dibisukan; 1 = izin OFF, 0 = izin ON |

Jika menggunakan widget **Button** untuk V0, pilih mode **Switch** agar posisi ON/OFF tetap tersimpan setelah sekali klik.

Pengaturan tambahan yang sesuai dengan cara kerja kode:

- **Sync with latest server value on reconnect:** OFF. Kode mengirim status izin alarm lokal ke dashboard ketika tersambung kembali.
- **Wait for confirmation from device:** OFF.
- **Automations:** None untuk seluruh datastream. Aturan alarm dijalankan oleh ESP32.
- Setelah mengubah template atau dashboard, klik **Save and Apply**.

Jika memakai aplikasi seluler Blynk, susun widget pada dashboard seluler dengan datastream V0–V7 yang sama.

## Cara kerja kontrol buzzer

**V0 adalah izin alarm, sedangkan V5 adalah status bunyi buzzer yang sebenarnya.**

| Izin alarm V0 | Pembacaan objek | Buzzer V5 | Dibisukan V7 |
| --- | --- | --- | --- |
| ON / 1 | Jarak valid kurang dari 80 cm | ON / 1 | 0 |
| ON / 1 | Jarak 80 cm atau lebih | OFF / 0 | 0 |
| OFF / 0 | Objek dekat maupun jauh | OFF / 0 | 1 |
| ON / 1 | Tidak ada echo | OFF / 0 | 0 |

- Saat perangkat mulai menyala, izin alarm berada pada posisi **ON**.
- Satu kali menekan tombol fisik mengubah izin **ON menjadi OFF**, atau **OFF menjadi ON**. Tombol tidak perlu ditahan.
- Perubahan tombol fisik memperbarui V0 pada dashboard. Perubahan switch Blynk memperbarui izin alarm yang sama di program.
- Jika izin diaktifkan kembali saat objek masih kurang dari 80 cm, buzzer kembali berbunyi tanpa perlu mengubah jarak.
- Mengaktifkan V0 ketika objek berjarak lebih dari atau sama dengan 80 cm membuat buzzer tetap mati.
- Ketika koneksi Blynk tersambung kembali, dashboard mengikuti status izin lokal terakhir. Setelah perangkat di-reset, izin kembali ON.

## Aturan kondisi ruangan

- **Normal:** suhu 18–25 °C, kelembaban lebih dari 75%, dan tidak ada objek pada jarak kurang dari 80 cm.
- **Suhu Abnormal:** suhu kurang dari 18 °C atau lebih dari 25 °C.
- **Bahaya:** suhu dan kelembaban abnormal serta terdapat objek pada jarak kurang dari 80 cm. Kondisi ini diperiksa sebelum Suhu Abnormal.
- **Kelemb Abnormal:** kelembaban tidak memenuhi syarat normal pada kombinasi yang tidak termasuk Bahaya atau Suhu Abnormal.
- **Objek Terdeteksi:** suhu dan kelembaban normal, tetapi objek berada kurang dari 80 cm.
- **Menunggu DHT / DHT Error:** pembacaan DHT belum tersedia atau gagal.

Suhu tepat 18 °C dan 25 °C masih normal. Kelembaban tepat 75% belum normal. Jarak tepat 80 cm tidak memicu buzzer.

## Pembacaan tanpa data

- **Jarak 0 cm** adalah penanda tidak ada echo, bukan objek pada jarak nol. Program menetapkan objek tidak terdeteksi dan mematikan buzzer.
- Jika pembacaan DHT belum valid, kode mengirim suhu **-41** ke V1 dan kelembaban **-1** ke V2, disertai kondisi dari V4.
- Rentang V1 sudah menampung -41. Pada konfigurasi V2 saat ini, minimum 0 membuat penanda -1 dibatasi menjadi 0. Gunakan teks **DHT Error** pada V4 untuk mengenali kegagalan pembacaan; angka kelembaban 0 saat error tidak dianggap hasil pengukuran yang sah.
- Nilai default datastream adalah nilai awal, sehingga nilai sensor perlu dibaca bersama kondisi V4 dan status koneksi device.

## Menjalankan simulasi Wokwi local

1. Buka folder proyek di Visual Studio Code dengan PlatformIO dan Wokwi.
2. Pastikan identitas Blynk pada `src/main.cpp` sesuai dengan template dan device.
3. Untuk simulasi, gunakan SSID `Wokwi-GUEST` dengan password kosong. Untuk perangkat fisik, isi jaringan WiFi yang digunakan.
4. Jalankan **PlatformIO: Build** menggunakan environment `esp32dev`.
5. Jalankan **Wokwi: Start Simulator**. Konfigurasi `wokwi.toml` memakai `.pio/build/esp32dev/firmware.bin` dan `.pio/build/esp32dev/firmware.elf`.
6. Buka dashboard device Blynk dan pastikan statusnya **Online**.
7. Ubah suhu, kelembaban, serta jarak pada simulator. Uji tombol fisik dan switch Blynk sambil mengamati LCD, Serial Monitor, dan indikator dashboard.

Baudrate Serial Monitor adalah **115200**. Jarak dibaca setiap **200 ms**, DHT setiap **2 detik**, dan data dashboard dikirim setiap **2 detik**.

Contoh hasil yang sesuai kode: **6,5 °C**, **80%**, **135,9 cm**, dan izin alarm **ON** menghasilkan **Suhu Abnormal**, Objek Terdeteksi **0**, Buzzer Status **0**, serta Buzzer Dibisukan **0**.

## Dokumentasi Blynk

- [Pengaturan datastream](https://docs.blynk.io/en/blynk.console/templates/datastreams/datastreams-common-settings)
- [Pengaturan widget Switch](https://docs.blynk.io/en/blynk.console/widgets-console/switch)
