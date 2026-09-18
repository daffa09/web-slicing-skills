# web-slicing-skills

Dua skill Claude Code buat slicing UI 1:1 dari mockup: `/slice-web` buat website/web app
(termasuk mobile web), `/slice-mobile` buat Flutter, React Native/Expo, Android native,
dan iOS native. Sumbernya boleh URL, file HTML, gambar, atau MCP desain kayak Figma.
Hasilnya dicek pakai Playwright — screenshot ditumpuk di atas sumber, diff-nya kelihatan
sampai selisihnya tinggal noise tipis di tepi teks.

## Pasang

**Cara 1 — plugin (satu baris, Playwright otomatis kepasang):**

```
/plugin marketplace add daffa09/web-slicing-skills
/plugin install web-slicing-skills@web-slicing-skills
```

Skill-nya jadi `/web-slicing-skills:slice-web` dan `/web-slicing-skills:slice-mobile`.
Playwright + Chromium kepasang otomatis di sesi Claude Code pertama setelah install —
bukan seketika pas `/plugin install` selesai, tapi begitu sesi berikutnya dibuka. Butuh
Node.js di mesin kamu dan sekali unduh internet (~150 MB buat Chromium).

**Cara 2 — copy manual (nama tetap `/slice-web` dan `/slice-mobile`):**

```
cp -r skills/slice-web skills/slice-mobile ~/.claude/skills/
```

Playwright belum otomatis di jalur ini — skill-nya sendiri yang pasang ke folder temp
pas pertama kali dipanggil.

## Butuh apa

- Claude Code.
- Node.js (buat Playwright).
- Koneksi internet sekali di awal buat unduh Chromium.
- Opsional: MCP desain (mis. Figma) kalau sumbernya MCP.
- Opsional: `graphify` di `PATH` buat hemat token nyari komponen/token di repo Next.js,
  Flutter, atau React Native yang gede. Nggak ada pun jalan — skill-nya pakai grep biasa.

## Cara pakai

```
/slice-web https://contoh.com/pricing src/app/pricing/page.tsx
/slice-mobile ./mockup/onboarding-3.png OnboardingScreen
```

Sumber atau target kosong, skill-nya nanya balik satu kali lalu jalan tanpa konfirmasi
lagi. Hasilnya laporan ringkas: file yang diubah, sisa selisih per section, dan path
screenshot overlay terakhir.

## Aturan yang dipegang

- Tiru persis — tanpa redesign, ganti teks, atau tambah/kurang elemen.
- Semua nilai dari hasil ukur (computed style / data MCP / sampling pixel), bukan
  tebakan.
- Pakai komponen dan token project yang sudah ada, bukan bikin baru.
- Nggak nambah dependency ke project kamu — Playwright cuma jalan di folder terpisah,
  di luar repo yang di-slice.

Detail lengkap tiap aturan ada di `skills/slice-web/SKILL.md` dan
`skills/slice-mobile/SKILL.md`.

## Troubleshooting

Chromium gagal keunduh otomatis (jaringan kepotong pas sesi pertama)? Minta Claude
jalanin ulang, atau pasang manual:

```
cd ~/.claude/plugins/data/web-slicing-skills-web-slicing-skills
npx playwright install chromium
```

## License

MIT
