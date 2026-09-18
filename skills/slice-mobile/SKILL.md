---
name: slice-mobile
description: >
  Slicing UI aplikasi mobile 1:1 (Flutter, React Native / Expo, Android
  native, iOS native) dari sumber desain: URL mockup, file HTML, gambar, atau
  MCP desain (mis. Figma). Hasil dicek lewat screenshot emulator/simulator dan
  overlay sampai cocok. Use when user says "slice mobile", "slicing app",
  "slicing flutter", "slicing react native", "1:1 mobile", or gives a mobile
  screen design that must be implemented exactly. For websites (including
  mobile web / responsive) use slice-web.
user-invocable: true
argument-hint: "[sumber: url | path .html | path gambar | mcp] [target: screen | path file]"
---

# Slicing Mobile 1:1

Argumen: $ARGUMENTS

## Input

| Field    | Wajib | Isi / default |
|----------|-------|---------------|
| Sumber   | ya    | URL, path .html / folder mockup, path gambar, atau MCP + link frame |
| Target   | ya    | nama screen, route, atau path file (boleh belum ada) |
| Scope    | tidak | satu screen penuh |
| Device   | tidak | ukuran logis sumber (mis. 390x844 / 360x800); gambar @2x/@3x dibagi skalanya |
| Platform | tidak | yang tersedia di mesin ini (iOS simulator hanya di macOS) |
| Catatan  | tidak | komponen wajib, cara login, dll |

- Sumber atau Target kosong: tanya keduanya dalam satu pesan, sebut jenis sumber, MCP desain yang tersambung (`mcp__*`), dan default field opsional. Tunggu.
- MCP tidak tersambung: tawarkan sambungkan atau ganti sumber.
- Gambar di-paste tanpa path: minta path-nya (sampling pixel butuh file).
- Lengkap: tulis ulang Input dalam satu blok, lanjut tanpa konfirmasi.

## Aturan

1. Tiru persis. Tanpa redesign, ganti teks, atau tambah/kurang elemen.
2. Semua nilai hasil ukur (computed style / data MCP / sampling pixel), bukan tebakan. Satuan logis (dp / pt), bukan pixel fisik.
3. Sumber kode/MCP: nilai presisi, jangan dibulatkan ke token; di luar token pakai nilai persis dan catat. Sumber gambar: bulatkan ke token terdekat, catat kalau selisih > 2dp.
4. Pakai komponen, theme, dan token project. Komponen shared beda ukuran: jangan diubah, override lokal, catat.
5. Jangan tambah dependency. Playwright (ukur sumber dan overlay): `$CLAUDE_PLUGIN_DATA/node_modules/playwright` ada, pakai itu; belum ada, pasang di folder temp di luar repo. Font sumber boleh di-bundle dengan cara standar framework (pubspec fonts, expo-font, res/font, Info.plist), sebut di laporan.
6. Data dummy sama persis dengan sumber. Perubahan sementara (data dummy, initial route, deep link) dicatat di laporan.
7. Hanya ukuran device dan orientasi sumber: tanpa layout tablet, landscape, atau dark mode yang tidak ada di sumber.
8. Status bar / safe area jangan di-hardcode: pakai API platform (SafeArea, useSafeAreaInsets, WindowInsets, safeAreaInset), hasil visual tetap sama dengan sumber.
9. Jangan ubah setting HP fisik. Ukuran/density hanya diubah di emulator, reset setelah selesai.
10. Kode tanpa komentar `ponytail:`, banner/pemisah section, atau komentar yang mengulang kode. Komentar hanya untuk alasan yang tidak kelihatan dari kode.
11. Berhenti tanya hanya kalau sumber tidak bisa diambil, tidak ada emulator/simulator yang bisa jalan, atau screen target butuh login tanpa cara mock.

## Hemat token: graphify

- Ada `graphify-out/graph.json`: cari screen, widget/komponen, theme/token, dan navigasi pakai `graphify query "<apa>"`, `graphify explain "<Simbol>"`, `graphify path "<A>" "<B>"`. Baca file mentah hanya yang akan diubah. Hasil `[!] TRUNCATED`: sempitkan ke nama simbol.
- Belum ada graph: build hanya kalau `command -v graphify` dan `git check-ignore -q graphify-out/graph.json` lolos → `graphify extract . --code-only` (0 token). Kalau tidak, jangan build (bisa ke-commit); pakai grep.
- Sumber folder mockup, atau URL mockup lokal (mis. `localhost`) yang foldernya punya graph: cari kode screen-nya pakai `graphify query "<screen>" --graph <folder>/graphify-out/graph.json`. Angka tetap dari ukur Playwright.
- Setelah semua section selesai (bukan tiap putaran): `graphify update .`.

## Langkah

### 0. Kenali project dan device
- Framework (Flutter / React Native / Expo / Compose atau XML / SwiftUI atau UIKit), theme/token, komponen yang ada, navigasi (cara buka screen target langsung), cara run.
- Device: `adb devices`, `emulator -list-avds`, `flutter devices`, atau `xcrun simctl list devices`. Belum ada yang jalan: nyalakan sendiri.
- Ukuran logis beda dari Device (Android emulator): `adb shell wm size <px>` + `adb shell wm density <dpi>` (px = dp × dpi ÷ 160). Reset: `adb shell wm size reset`, `adb shell wm density reset`.
- Samakan status bar (Android demo mode, iOS `xcrun simctl status_bar booted override --time 9:41`) atau crop dari kedua gambar.
- Run app, buka screen target.

### 1. Ukur sumber
Artefak (script, screenshot, JSON) di folder temp di luar repo. Hasil wajib: `ref-full.png` (1x, lebar = lebar logis Device), screenshot per section, spec.
- **URL / File**: Playwright, viewport = Device, `isMobile: true`, `hasTouch: true`, `deviceScaleFactor: 1` (1 CSS px = 1 dp/pt). Tunggu `document.fonts.ready` dan network idle, matikan animasi/transition. file:// tidak ter-render benar: serve foldernya (`npx serve`). Dump computed style elemen terlihat ke `ref-styles.json`: selector, teks (≤ 40 karakter), bbox, font-family/size/weight, line-height, letter-spacing, color, background, padding, margin, gap, display + flex/grid, border, border-radius, box-shadow, opacity.
- **Gambar**: skala = lebar gambar ÷ lebar logis Device, bikin versi 1x. Jarak dan warna dari sampling pixel (canvas `getImageData` di Playwright). Font dari bentuk huruf; ragu antara 2 font: render keduanya, bandingkan.
- **MCP**: screenshot frame 1x, nilai layout/style (ukuran, auto layout, gap, font, warna, radius, shadow) dan variable/token ke `ref-styles.json`. Kode hasil generate MCP cuma sumber angka, jangan di-copy.
- Spec: font (sudah di-bundle atau belum), token (warna, skala font, spacing, radius, shadow), kerangka (status bar / safe area, tinggi app bar, tinggi bottom nav / tab bar, padding horizontal), daftar section atas ke bawah.

### 2. Bangun per section
Urutan: font → token → kerangka → section. Tiap section:
1. Implementasi, pastikan app ter-update (hot reload / fast refresh / rebuild), lalu screenshot:
   - Android: `adb exec-out screencap -p > target.png`
   - iOS: `xcrun simctl io booted screenshot target.png`
   - Flutter, lebih cepat: golden test di ukuran dan devicePixelRatio yang sama; load font asli dulu (font bawaan test bukan font app).
   - MCP mobile (mis. mobile-mcp) boleh untuk screenshot dan navigasi kalau tersambung.
   - Screen lebih panjang dari layar: screenshot per posisi scroll vs potongan sumber yang sesuai.
2. Bandingkan:
   - Visual: tumpuk `ref-full.png` dan screenshot target di HTML sederhana, lebar CSS keduanya = lebar logis Device, gambar atas `mix-blend-mode:difference`. Render pakai Playwright, screenshot, LIHAT gambarnya. Hitam = cocok, terang = meleset.
   - Angka (kalau bisa): Android native / React Native `adb shell uiautomator dump`, bounds px ÷ (dpi ÷ 160) = dp. Flutter: DevTools layout explorer atau `debugDumpRenderTree()`. Bandingkan dengan `ref-styles.json`.
3. Ulangi sampai posisi/ukuran selisih ≤ 1dp, font/warna/radius/shadow sama, overlay tinggal noise tipis di tepi teks. Maks 5 putaran; masih meleset: catat penyebabnya, lanjut.

Terakhir bandingkan satu screen penuh.

Paling sering meleset:
- Line-height: React Native Android `includeFontPadding`, Flutter `height` + `TextHeightBehavior`.
- Font weight yang tidak di-bundle jatuh ke weight lain.
- Shadow: Android `elevation` tidak sama dengan shadow desain.
- Safe area, status bar, tinggi app bar / tab bar bawaan library navigasi.
- Touch target minimum yang menambah ukuran (Material 48dp).

### 3. Laporan (ringkas)
- File yang dibuat/diubah.
- Platform dan device yang dicek (platform lain bisa beda sedikit karena rendering font).
- Tabel per section: status, sisa selisih, alasan.
- Nilai di luar token; komponen shared yang di-override lokal.
- Perubahan sementara yang masih ada dan status reset emulator.
- Path screenshot sumber, hasil, dan overlay terakhir.
