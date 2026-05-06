## 1. setup server
- tentukan folder `static` (html, css, js, assets)
- mount ke `/static`
- route:
  - `/` → kirim `index.html`
  - `/{file_name}` → kirim file static lain
- api:
  - `/api/calculate?kmh=...`

---

## 2. api `/api/calculate`
input:
- kmh (query string)

proses:
- mph = kmh * 0.621371
- light_percent = (kmh / 1_079_252_848) * 100

output:
```json
{
  "kmh": number,
  "mph": number,
  "light_percent": number
}
```

---

## 3. ui (index.html)
komponen:
- canvas (bola visualisasi)
- slider (range speed)
- input angka + tombol `+ / -`
- tab satuan:
  - km/h
  - mph
  - % cahaya
- tombol preset:
  - diam
  - kota
  - pesawat
  - roket
  - cahaya

---

## 4. event handler – input / slider
trigger: value berubah

- stop animasi (kalau aktif)
- update `currentSpeed`
- call `updateDisplay(kmh)`

---

## 5. event handler – preset
trigger: tombol ditekan

- set `targetSpeed`
- aktifkan animasi spring
- tandai tombol aktif

---

## 6. function `updateDisplay(kmh, fromAnimation = false)`

- set `currentSpeed = kmh`

- hitung:
  ```
  visualRatio = (kmh / C) ^ 0.15
  ```
  (C = kecepatan cahaya)

- jika bukan dari animasi:
  - update slider
  - update input
  - update label

- update slider-fill width

- fetch `/api/calculate`
  - cancel request sebelumnya jika ada
  - tampilkan:
    - mph
    - light_percent

---

## 7. function `animationLoop()`

loop tiap frame (`requestAnimationFrame`)

### a. spring animation
jika aktif:
- diff = targetSpeed - currentSpeed
- currentSpeed += diff * springFactor
- updateDisplay(currentSpeed, true)

- jika diff kecil:
  - stop animasi

---

### b. vibrasi bola
```
vib = baseRadius * 0.18 * (visualRatio ^ 1.1)
offsetX = sin(time) * vib
offsetY = cos(time) * vib
```

---

### c. stretch horizontal
```
scaleX = 1 + (visualRatio ^ 1.4) * 0.7
```

---

### d. render

- clear canvas

- gambar streaks (garis putih)
  - jumlah & speed ∝ visualRatio

- jika visualRatio > 0.02:
  - gambar trail (jejak cahaya)

- gambar bola:
  - posisi: center + offset
  - bentuk: ellipse (scaleX)

- tambah:
  - shadow
  - highlight (biar 3d-ish)

---

### e. next frame
```
requestAnimationFrame(animationLoop)
```

---

## 8. inisialisasi

- resize canvas sesuai layar
- pasang `ResizeObserver`
- call:
  ```
  updateDisplay(0)
  ```
- start:
  ```
  animationLoop()
  ```

---

## end
