# Acapella Studio

DAW mini berbasis web untuk menyusun acapella: piano roll, synth vokal, generator lagu 4 suara, impor/ekspor MIDI, dan ekspor WAV. Dibungkus jadi aplikasi Android (WebView) yang jalan offline.

> **Status:** belum diuji di perangkat Android sungguhan. Logika inti (parser/penulis MIDI, generator, encoder WAV) sudah dites lewat Node. Tampilan, sentuhan jari, dan hasil suara perlu dicek langsung.

## Fitur

- **Piano roll** dengan 4 alat: Geser, Gambar, Pindah (seret not, tarik ujung kanan untuk memanjangkan), Hapus.
- **Snap** 1/1 sampai 1/16, zoom, ketuk penggaris untuk titik mulai, ketuk tuts untuk mendengar nada.
- **Sampai 8 track**, tiap track punya vokal (A/E/I/O/U), volume, layer suara, detune, vibrato, dan napas.
- **Synth vokal formant:** sawtooth lewat 3 filter band-pass paralel, ditambah vibrato, layering, noise napas, dan reverb.
- **Generator acapella:** pilih nada dasar, mayor/minor, progresi, dan panjang (4/8/16 bar). Hasilnya lead, alto, tenor, dan bass.
- **Impor MIDI** (`.mid`): tiap track jadi satu suara, tempo ikut terbaca, drum (channel 10) dilewati.
- **Ekspor:** MIDI, WAV (stereo 44,1 kHz), proyek `.json`, dan daftar not dalam bentuk teks.
- **Urungkan**, simpan otomatis di perangkat, tema gelap/terang mengikuti sistem.
- **Tab Panduan** di dalam app (tabel formant, tips suara vokal, langkah Harmor di FL Studio).

## Batasan

- Bukan Vocaloid: hanya bunyi vokal panjang, tidak bisa menyanyikan lirik.
- Tidak ada resynthesis sample seperti Harmor. Untuk itu, ekspor MIDI lalu buka di FL Studio.
- Tidak ada playlist/pattern, mixer dengan insert effect, automation, atau rekam audio.
- Impor MIDI hanya memakai tempo pertama di file.
- Latency audio browser/WebView lebih tinggi daripada DAW desktop. Cocok untuk menyusun ide, bukan rekam langsung.

## Isi project

```
acapella-apk/
├── build.gradle, settings.gradle, gradle.properties
├── gradle/wrapper/gradle-wrapper.properties   (Gradle 8.7)
└── app/
    ├── build.gradle                            (AGP 8.5.2, minSdk 29, target 34)
    └── src/main/
        ├── AndroidManifest.xml
        ├── assets/index.html                   <- seluruh DAW (satu file HTML)
        ├── java/com/melvox/app/MainActivity.java
        └── res/                                (nama app, tema, ikon)
```

- `assets/index.html` berisi seluruh aplikasi (HTML, CSS, JavaScript). Bisa juga dibuka sendiri di browser.
- `MainActivity.java` membuka `index.html` lewat `WebViewAssetLoader` (domain virtual `appassets.androidplatform.net`), menangani izin mikrofon, pemilih file (untuk impor MIDI), dan menyimpan hasil ekspor ke folder **Download**.

## Cara build APK

Kebutuhan: **Android Studio** (versi terbaru), JDK 17 (bawaan Android Studio), dan koneksi internet untuk sinkronisasi Gradle pertama (mengunduh `androidx.webkit`).

1. Ekstrak zip, lalu buka foldernya di Android Studio (**File → Open**).
2. Tunggu **Gradle sync** selesai.
3. Colok HP (aktifkan USB debugging) lalu klik **Run**, atau pilih **Build → Build Bundle(s)/APK(s) → Build APK(s)**.
4. File APK ada di `app/build/outputs/apk/debug/app-debug.apk`. Kirim ke HP lalu install.

Syarat perangkat: **Android 10 (API 29) atau lebih baru**.

## Cara pakai singkat

1. Buka tab **MIDI**: muat contoh, impor file `.mid`, atau buat lagu lewat generator.
2. Buka tab **Piano roll**: pilih track lewat chip warna, lalu pakai alat Gambar/Pindah/Hapus. Pakai alat **Geser** untuk scroll dengan jari (di alat lain scroll dimatikan agar gambar tidak meleset).
3. Buka tab **Suara**: ganti vokal, vibrato, layer, napas, dan reverb.
4. Tekan **Putar**. Untuk menyimpan hasil, pakai tombol di tab **MIDI → Ekspor**.

## Mengganti ikon dan nama

- Nama app: `app/src/main/res/values/strings.xml`.
- Ikon: ganti `app/src/main/res/mipmap-xxxhdpi/ic_launcher.png` (saat ini placeholder).
- Package/applicationId masih `com.melvox.app`. Kalau ingin diganti, ubah di `app/build.gradle` dan pindahkan folder `java/com/melvox/app`.

## Pemecahan masalah

| Masalah | Coba ini |
|---|---|
| Gradle sync gagal | Pastikan online, lalu **File → Sync Project with Gradle Files**. Cek JDK 17 di pengaturan Gradle. |
| Layar putih saat dibuka | Pastikan `app/src/main/assets/index.html` ada, lalu build ulang (**Build → Clean Project**). |
| Tidak ada suara | Ketuk layar sekali dulu (browser butuh interaksi sebelum audio). Cek volume media HP dan volume track. |
| Tidak bisa pilih file MIDI | Pastikan file berekstensi `.mid` atau `.midi`. Coba buka dari penyimpanan internal. |
| WAV/MIDI tidak muncul di Download | Cek notifikasi di layar (app menampilkan toast saat tersimpan). Cari di folder **Download** lewat app File. |
| Not terlalu tinggi/rendah saat impor | Ubah dengan tombol **Naik oktaf / Turun oktaf** di tab Suara. |

## Catatan teknis

- Audio memakai Web Audio API: oscillator sawtooth, filter band-pass, convolver (reverb buatan), dan compressor di master.
- Ekspor WAV merender lewat `OfflineAudioContext`, lalu disimpan ke folder Download lewat jembatan JavaScript `AndroidBridge.save(...)`.
- Impor/ekspor MIDI ditulis sendiri (format 1, 480 PPQ), tanpa library tambahan.
- Lagu contoh dan hasil generator adalah komposisi orisinal untuk latihan.
