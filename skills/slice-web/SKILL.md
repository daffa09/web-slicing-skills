---
name: slice-web
description: >
  Slicing UI website / web app 1:1 (termasuk tampilan mobile web / responsive)
  dari sumber desain: URL mockup, file HTML, gambar, atau MCP desain (mis.
  Figma). Sumber diukur, hasil dicek pakai Playwright dan overlay screenshot
  sampai cocok. Use when user says "slice web", "slicing website", "1:1",
  "pixel perfect", "tiru mockup", or gives a design that must be implemented
  exactly in a web project. For Flutter / React Native / native apps use
  slice-mobile.
user-invocable: true
argument-hint: "[sumber: url | path .html | path gambar | mcp] [target: route | path file]"
---

# Slicing Web 1:1

Argumen: $ARGUMENTS

## Input

| Field    | Wajib | Isi / default |
|----------|-------|---------------|
| Sumber   | ya    | URL, path .html / folder mockup, path gambar, atau MCP + link frame |
| Target   | ya    | route atau path file (boleh belum ada) |
| Scope    | tidak | satu halaman penuh |
| Viewport | tidak | 1440x900; mobile web 390x844; gambar: lebar ÷ skala (2880px @2x → 1440) |
| Catatan  | tidak | komponen wajib, cara login, dll |

- Sumber atau Target kosong: tanya keduanya dalam satu pesan, sebut jenis sumber, MCP desain yang tersambung (`mcp__*`), dan default field opsional. Tunggu.
- MCP tidak tersambung: tawarkan sambungkan atau ganti sumber.
- Gambar di-paste tanpa path: minta path-nya (sampling pixel butuh file).
- Lengkap: tulis ulang Input dalam satu blok, lanjut tanpa konfirmasi.

## Aturan

1. Tiru persis. Tanpa redesign, ganti teks, atau tambah/kurang elemen.
2. Semua nilai hasil ukur (computed style / data MCP / sampling pixel), bukan tebakan.
3. Sumber kode/MCP: nilai presisi, jangan dibulatkan ke token; di luar token pakai nilai persis dan catat. Sumber gambar: bulatkan ke token terdekat, catat kalau selisih > 2px.
4. Pakai komponen dan token project. Komponen shared beda ukuran: jangan diubah, override lokal, catat.
5. Jangan tambah dependency. `$CLAUDE_PLUGIN_DATA/node_modules/playwright` ada: pakai itu. Belum ada: pasang di folder temp di luar repo. Font sumber boleh ditambah dengan cara standar framework, sebut di laporan.
6. Data dummy sama persis dengan sumber (teks, jumlah item).
7. Hanya Viewport Input: tanpa breakpoint atau state yang tidak ada di sumber.
8. Kode tanpa komentar `ponytail:`, banner/pemisah section, atau komentar yang mengulang kode. Komentar hanya untuk alasan yang tidak kelihatan dari kode.
9. Berhenti tanya hanya kalau sumber tidak bisa dibuka atau target butuh login tanpa cara mock.

## Hemat token: graphify

- Ada `graphify-out/graph.json`: cari komponen, token/theme, route, dan layout pakai `graphify query "<apa>"`, `graphify explain "<Simbol>"`, `graphify path "<A>" "<B>"`. Baca file mentah hanya yang akan diubah. Hasil `[!] TRUNCATED`: sempitkan ke nama simbol.
- Belum ada graph: build hanya kalau `command -v graphify` dan `git check-ignore -q graphify-out/graph.json` lolos → `graphify extract . --code-only` (0 token). Kalau tidak, jangan build (bisa ke-commit); pakai grep.
- Sumber folder mockup, atau URL mockup lokal (mis. `localhost`) yang foldernya punya graph: cari kode screen-nya pakai `graphify query "<screen>" --graph <folder>/graphify-out/graph.json`. Angka tetap dari ukur Playwright.
- Setelah semua section selesai (bukan tiap putaran): `graphify update .`.

## Langkah

### 0. Kenali project
Framework, styling (Tailwind / CSS modules / dll), file token/theme, komponen yang ada, konvensi folder, cara run. Jalankan dev server, pastikan route target kebuka.

### 1. Ukur sumber
Artefak (script, screenshot, JSON) di folder temp di luar repo. Hasil wajib: `ref-full.png` (1x, lebar = Viewport), screenshot per section, spec.
- **URL / File**: Playwright di Viewport (mobile web: `isMobile: true`, `hasTouch: true`, `deviceScaleFactor: 1`). Tunggu `document.fonts.ready` dan network idle, matikan animasi/transition. file:// tidak ter-render benar: serve foldernya (`npx serve`). Dump computed style elemen terlihat ke `ref-styles.json`: selector, teks (≤ 40 karakter), bbox, font-family/size/weight, line-height, letter-spacing, color, background, padding, margin, gap, display + flex/grid, border, border-radius, box-shadow, opacity. Cek state hover/focus elemen interaktif pakai `hover()` / `focus()`.
- **Gambar**: skala = lebar gambar ÷ Viewport, bikin versi 1x. Jarak dan warna dari sampling pixel (canvas `getImageData` di Playwright). Font dari bentuk huruf; ragu antara 2 font: render keduanya, bandingkan.
- **MCP**: screenshot frame 1x, nilai layout/style (ukuran, auto layout, gap, font, warna, radius, shadow) dan variable/token ke `ref-styles.json`. Kode hasil generate MCP cuma sumber angka, jangan di-copy.
- Spec: font, token (warna, skala font, spacing, radius, shadow), kerangka (container, gutter, grid, tinggi header/sidebar), daftar section atas ke bawah.

### 2. Bangun per section
Urutan: font → token → kerangka → section. Tiap section:
1. Implementasi, lalu screenshot target di Viewport yang sama.
2. Bandingkan:
   - Visual: tempel `ref-full.png` di atas target (data URL, `position:absolute; top:0; left:0; pointer-events:none; mix-blend-mode:difference`), screenshot, LIHAT gambarnya. Hitam = cocok, terang = meleset.
   - Angka (sumber kode/MCP): computed style elemen yang sama vs `ref-styles.json`.
3. Ulangi sampai posisi/ukuran selisih ≤ 1px, font/warna/radius/shadow sama, overlay tinggal noise tipis di tepi teks. Maks 5 putaran; masih meleset: catat penyebabnya, lanjut.

Terakhir bandingkan satu halaman penuh.

Paling sering meleset: line-height, letter-spacing, font-weight 600 vs 700, border yang menambah ukuran box, ukuran/stroke icon, shadow, max-width dan gutter container.

### 3. Laporan (ringkas)
- File yang dibuat/diubah.
- Tabel per section: status, sisa selisih, alasan.
- Nilai di luar token; komponen shared yang di-override lokal.
- Path screenshot sumber, hasil, dan overlay terakhir.
