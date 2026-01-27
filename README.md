# Mobile App - Maestro Automation Tests

Repository ini berisi kumpulan **skrip end-to-end testing** untuk aplikasi mobile menggunakan **[Maestro](https://maestro.mobile.dev/)**.

Tujuan utama:
- Mempermudah QA / developer menjalankan **uji fungsi utama** aplikasi secara berulang.
- Menyediakan contoh alur test yang **sederhana, bisa dibaca non-programmer**, dan mudah dikembangkan.
- Menjadi dasar integrasi ke **CI/CD** di masa depan.

---

## 1. Tentang Maestro (Singkat)

**Maestro** adalah framework UI testing untuk mobile (Android & iOS) dengan ciri:

- Test ditulis dalam **file YAML** (bukan kode Java/Kotlin/Swift).
- Perintah-perintahnya sederhana: `launchApp`, `tapOn`, `inputText`, `assertVisible`, dll.
- Mendukung **Android emulator** maupun **device fisik**.
- Bisa dijalankan dari CLI:
  - `maestro test path/to/flow.yaml`

Dokumentasi lengkap: <https://maestro.mobile.dev>

---

## 2. Struktur Project

Struktur dasar repository ini:

- `env.example.yaml`
  - Contoh template environment. Salin menjadi `env.yaml` untuk penggunaan lokal setiap developer.

- `modules/`
  - Berisi **flow test per fitur** (satu file YAML per skenario/fitur).
  - Contoh:
    - `anterin_mobil_kecil.yaml`
    - `anterin_mobil_besar.yaml`
    - `anterin_motor_kecil.yaml`
    - `anterin_motor_besar.yaml`
    - `bengkelin_inspeksi.yaml`
    - `bengkelin_reparasi.yaml`
    - `bersihin.yaml`

> **Catatan**: Jika di kemudian hari ingin mengelompokkan beberapa flow menjadi satu "test suite", disarankan menambahkan folder `suites/` dan menyimpan file YAML yang hanya berisi `runFlow` ke modules yang sudah ada.

---
## 3. Konfigurasi `env.yaml`

Langkah awal yang disarankan:
1. Copy file `env.example.yaml` menjadi `env.yaml`.
2. Edit nilai di `env.yaml` sesuai environment lokal (DEV / STAGING / PROD QA).
3. Jalankan Maestro dengan opsi `-e env.yaml` agar variabel terbaca.

File `env.yaml` berisi variabel-variabel yang dapat dipanggil dari flow Maestro dengan sintaks `${NAMA_VARIABEL}`.

### 3.1. Mengubah nilai environment

1. Buka `env.yaml` di editor.
2. Sesuaikan nilai sesuai kebutuhan environment (DEV / STAGING / PROD QA), misalnya:
   - Ganti `APP_USER` dengan package name aplikasi Android yang benar.
   - Ganti `PHONE_TEST` dengan nomor telepon khusus testing yang valid.
   - Ganti `LOCATION_TARGET` dengan lokasi yang sering dipakai di skenario test.
3. Simpan file.

### 3.2. Menggunakan variabel di flow Maestro

Di file di dalam folder `modules/`, variabel dari `env.yaml` bisa digunakan dengan format:

- `appId: ${APP_USER}`
- `inputText: ${LOCATION_TARGET}`

Saat menjalankan test, Maestro akan mengganti `${APP_USER}`, `${LOCATION_TARGET}`, dll dengan nilai dari `env.yaml` (diberikan melalui opsi `-e env.yaml`).

---

## 4. Contoh Flow di `modules/`

Berikut konsep singkat beberapa flow yang sudah ada (deskripsi, **bukan isi YAML lengkap**):

### 4.1. `anterin_mobil_kecil.yaml`

- Buka aplikasi user (`appId: ${APP_USER}`).
- Menunggu animasi/loading selesai.
- Tap ikon **Anterin** (mobil kecil) di beranda.
- Tap "Pilih lewat peta".
- Tap bidang "Cari lokasi tujuan...".
- Input lokasi (saat ini hardcoded `"Stasiun Gawok"`).
- Pilih hasil pertama di list.
- Tap "Jemput aku di sini".
- Pilih **Mobil Kecil**.
- `takeScreenshot` hasil akhir.

**Saran pengembangan**:

- Ubah `inputText` lokasi agar menggunakan `${LOCATION_TARGET}` dari `env.yaml` untuk memudahkan penggantian lokasi.

### 4.2. `bengkelin_inspeksi.yaml`

- Buka aplikasi user (`appId: ${APP_USER}`).
- Menunggu animasi/loading selesai.
- Tap ikon **Bengkelin** di beranda (menggunakan koordinat `point`, misalnya `"61%,50%"`).
- Menunggu animasi.
- Tap "Lokasi Saat Ini" (juga via `point`, misalnya `"71%,37%"`).

Flow lain (`anterin_mobil_besar`, `anterin_motor_kecil`, `anterin_motor_besar`, `bengkelin_reparasi`, `bersihin`) memiliki pola serupa: mulai dari beranda aplikasi user, lalu menjalankan langkah UI sesuai fitur masing-masing.

---

## 5. Cara Menjalankan Test

### 5.1. Prasyarat

- **Maestro CLI** sudah ter-install di komputer (Windows):
  - Lihat panduan resmi: <https://maestro.mobile.dev/getting-started/installing-maestro>
- **Android emulator** atau **perangkat fisik** sudah menyala & terhubung.
- Aplikasi anda sudah ter-install di device, dan `APP_USER` / `APP_MITRA` di `env.yaml` sudah sesuai.

### 5.2. Menjalankan satu flow

Dari root project `d:\gercepinAutomationTest`:

```bash
maestro test -e env.yaml modules/anterin_mobil_kecil.yaml
```

Contoh lain:

```bash
maestro test -e env.yaml modules/bengkelin_inspeksi.yaml
```

Penjelasan:

- `maestro test` → perintah untuk menjalankan satu file flow Maestro.
- `-e env.yaml` → menyuntikkan variabel dari `env.yaml` agar `${APP_USER}`, dsb bisa digunakan.
- `modules/anterin_mobil_kecil.yaml` → path ke file flow yang mau dijalankan.

### 5.3. Output & artefak test

- Jika flow sukses, Maestro akan menampilkan status **PASSED** di terminal.
- Jika gagal, Maestro akan menampilkan step yang gagal dan biasanya menyimpan log/screenshot di folder artefak (lihat konfigurasi default Maestro di dokumentasi).

> Pastikan `.gitignore` sudah mengabaikan folder artefak (misalnya `artifacts/`, `reports/`, `screenshots/`) agar file besar tidak ikut di-commit.

---

## 6. Menambahkan Flow Baru

Untuk menambah skenario baru:

1. **Tentukan skenario** yang ingin di-otomasi, misalnya:
   - "Order anterin motor besar dengan tujuan tertentu".
   - "Buka fitur bersihin dan isi form dasar".

2. **Buat file baru** di folder `modules/`, misalnya:
   - `modules/anterin_motor_baru.yaml`

3. **Gunakan pola umum**:
   - `appId: ${APP_USER}` atau `appId: ${APP_MITRA}` sesuai aplikasi target.
   - `---` pemisah header dengan langkah.
   - Langkah-langkah berupa kombinasi perintah:
     - `launchApp`, `tapOn`, `inputText`, `waitForAnimationToEnd`, `assertVisible`, `takeScreenshot`, dll.

4. **Jalankan dan iterasi**:
   - Jalankan dengan `maestro test -e env.yaml modules/namafile_baru.yaml`.
   - Sesuaikan selector (`text`, `id`, atau `point`) sampai flow stabil.

> **Tips**: Sebaiknya hindari terlalu banyak penggunaan `point` (koordinat layar), lebih baik pakai `text` atau `id` jika memungkinkan, agar test lebih tahan terhadap perubahan UI.

---

## 7. Best Practice & Tips

- **Gunakan variabel environment** (di `env.yaml`) untuk data yang sering berubah:
  - Nomor HP testing, lokasi default, appId, dsb.
- **Hindari hard-coded teks** jika bisa diparameterisasi.
- **Gunakan `waitForAnimationToEnd` atau `extendedWaitUntil`** untuk menghindari flakiness karena loading.
- **Pisahkan flow per fitur** di `modules/` agar mudah dirawat.
- Pertimbangkan membuat **`suites/`** di masa depan jika ingin:
  - Menjalankan beberapa flow berurutan dengan `runFlow`.
  - Membangun smoke/regression suite.

---

## 8. Pengembangan Lanjutan

Beberapa ide pengembangan berikutnya:

- Tambah flow **login** (jika belum ada) yang bisa dipakai ulang oleh flow lain dengan `runFlow`.
- Buat **suites** untuk smoke test (misalnya `suites/smoke_user.yaml`) yang memanggil beberapa flow dari `modules/`.
- Integrasi dengan **CI/CD** (GitHub Actions, GitLab CI, Jenkins, dll) untuk menjalankan Maestro pada setiap commit.
- Standarisasi penamaan file dan penulisan komentar di YAML agar konsisten.

---

## 9. Referensi

- Dokumentasi resmi Maestro: <https://maestro.mobile.dev>
- Contoh perintah dan konfigurasi lain bisa dilihat di bagian **API Reference** dan **CLI** pada dokumentasi tersebut.
