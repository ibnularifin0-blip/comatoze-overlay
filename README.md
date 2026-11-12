# Comatoze Overlay

Overlay FPS, CPU, GPU, RAM, suhu, dan baterai untuk Android. Tampil di atas game, bisa digeser.

## Gaya

- **Mini**: handle merah + satu bar horizontal, semua satu warna putih.
- **Standar**: kartu dengan FPS besar, frametime, dan metrik dalam grid.
- **Lengkap**: Standar + rata-rata FPS dan grafik riwayat FPS.

## Build APK lewat GitHub

1. Buat repo baru di GitHub, lalu push isi folder ini ke branch `main`.
2. Buka tab **Actions**. Workflow **Build APK** jalan otomatis.
3. Setelah hijau, buka run-nya dan unduh artifact **comatoze-overlay-debug**.
4. Ekstrak zip-nya, pasang `app-debug.apk`.

Bisa juga dijalankan manual lewat **Actions → Build APK → Run workflow**.

Build lokal (butuh JDK 17, Android SDK, dan Gradle 8.7):

```
gradle assembleDebug
```

APK ada di `app/build/outputs/apk/debug/`.

## Yang terbaca dan syaratnya

| Metrik | Tanpa root | Dengan root (KernelSU / Magisk) |
|---|---|---|
| RAM | ya | ya |
| Suhu | ya (suhu baterai) | ya |
| Baterai | ya | ya |
| CPU | tidak | ya, dari `/proc/stat` |
| GPU | tidak | ya, jika path sysfs dikenali |
| FPS game | tidak | ya, dari SurfaceFlinger |

Keterbatasan yang perlu diketahui:

- **CPU/GPU/FPS butuh root.** Android 8 ke atas memblokir akses aplikasi biasa ke `/proc/stat`, sysfs GPU, dan `dumpsys SurfaceFlinger`. Tanpa root nilainya tampil `--`.
- **GPU**: path sysfs berbeda tiap chipset. Yang dicoba: Adreno (`/sys/class/kgsl/kgsl-3d0/gpu_busy_percentage`, `gpubusy`), Mali, dan beberapa path umum lain. Kalau chipset kamu tidak ada di daftar, tambahkan path-nya di `StatsReader.kt` pada `gpuPaths`.
- **FPS**: dibaca dari layer game yang sedang fokus lewat `dumpsys SurfaceFlinger --latency`. Pada beberapa versi Android atau game tertentu, layer-nya tidak ditemukan dan FPS tampil `--`.
- Kode ini belum pernah dibangun atau dijalankan di HP. Bagian pembacaan sysfs dan FPS khususnya mungkin perlu penyesuaian per perangkat.

## Struktur

```
app/src/main/java/com/comatoze/overlay/
  MainActivity.kt     layar pengaturan
  OverlayService.kt   service foreground + tampilan overlay
  StatsReader.kt      pembaca CPU, GPU, RAM, suhu, baterai, FPS
  RootShell.kt        shell root yang dibuka sekali
  Prefs.kt            penyimpanan pengaturan
  Ui.kt               warna dan helper tampilan
.github/workflows/build.yml
```

## Izin

- **Tampil di atas aplikasi lain**: wajib, diminta saat pertama menekan Mulai.
- **Notifikasi**: untuk notifikasi service (ada tombol Hentikan).
- **Root**: diminta oleh KernelSU/Magisk saat overlay dijalankan.
