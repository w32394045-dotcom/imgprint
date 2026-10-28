# imgprint — print images on paper, from the browser

A single-file, offline image printer. Drop in a handful of images, pick a paper size, and print.
Everything runs locally: **no upload, no account, no history, no network requests**.

- **Live**: [imgprint.xszx.cc.cd](https://imgprint.xszx.cc.cd/)
- **Mirror**: [imgprint.w32394045.workers.dev](https://imgprint.w32394045.workers.dev/)
- **GitHub Pages**: [w32394045-dotcom.github.io/imgprint](https://w32394045-dotcom.github.io/imgprint/)
- **Source**: this repository (one file: [`index.html`](index.html), ~89 KB)

---

## Why another image printer?

Printing photos from a browser is usually an exercise in wasted paper: images get cropped
wrong, a JPEG lands at 146 dpi on A4, and the whites come out with a mysterious 10 mm border.
imgprint does one thing — lay out N images on paper, exactly as previewed — and refuses to do
the rest (accounts, albums, cloud, "AI enhance").

The whole app is one HTML file. Open it from disk, a USB stick, or your own static host.

## Features

| | |
|---|---|
| **Import** | Click, drag & drop anywhere, or paste a screenshot with `Ctrl+V`. Multi-select. Up to 60 images. |
| **Arrange** | Move each image up/down, remove it. Order is the print order. |
| **Layout** | A4 / A5 / B5 / Letter / 6-inch photo · portrait or landscape · margin none / 5 mm / 12 mm · 1, 2 or 4 images per page · *contain* (whole image, white space) or *cover* (fill, cropped) |
| **Preview** | The preview is the print DOM — the same elements that go to the printer, scaled down. No second rendering path. |
| **DPI** | Every image shows the estimated print resolution for the current paper/layout (green ≥ 300 dpi, gray ≥ 150, amber below). |
| **Color safety** | Detects sRGB / unprofiled / AdobeRGB / Display P3 / ProPhoto / CMYK / HDR per image and warns before you print. See [Color](#color-what-this-tool-does-and-does-not-do). |
| **Watermark** | Optional promo line in the footer of every printed page. On by default, switchable in settings. |
| **7 languages** | 简体中文 · 繁體中文 · 日本語 · 한국어 · Deutsch · Français · English. Auto-detected, remembered. |
| **Offline** | No fonts, no CDN, no telemetry. A strict CSP (`default-src 'none'`) is embedded. |
| **No saving** | Nothing is uploaded or written to disk. Only the settings (language + layout) persist in `localStorage`; images live in memory and vanish when the tab closes. |

## Watermark

Settings → **Footer watermark** (on by default). Each printed page carries:

```
imgprint.xszx.cc.cd · github.com/w32394045-dotcom/imgprint
```

The URLs are printed **as text on purpose** — a link on paper cannot be clicked, so the address
itself is what gets promoted. On screen the same text is a real link. Turn the switch off if you
need clean pages. Forking? Change `SITE_TEXT` / `REPO_TEXT` at the top of the app script.

## Color: what this tool does and does not do

Browsers cannot color-manage printing. `window.print()` gives the page no way to select the
printer's ICC profile, rendering intent, or soft-proofing. imgprint therefore takes the honest
route (strategy A):

- it prints the pixels the browser gives it, **in sRGB, without converting anything**;
- it **reads the image metadata** (JPEG APP2/APP1, PNG iCCP/cICP, WebP ICCP, AVIF/HEIC `colr`) and
  labels each image so you know what you are about to print:

| Badge | Meaning | Risk |
|---|---|---|
| `sRGB` | tagged sRGB / EXIF ColorSpace=1 | none |
| `No ICC — assumed sRGB` | no profile found | probably fine, but the file gives no guarantee |
| `AdobeRGB`, `Display P3`, `ProPhoto` | wide-gamut profile | printed colors will shift; convert to sRGB first |
| `CMYK` | CMYK JPEG | browsers convert it roughly; expect drift |
| `HDR` | PQ/HLG transfer (cICP / `colr`) | prints as SDR — darker, desaturated |
| `ICC profile` | profile present but unparsed | unknown; treat as untrusted |

What the tool **does not** do: convert color spaces, read the printer profile, or promise that
paper matches the screen. For faithful color, convert images to sRGB in an image editor before
printing. This is a deliberate scope limit, stated in the UI and not hidden behind "it just works".

## Printing checklist

1. Settings → paper, orientation, margins, images per page, fit.
2. Print (`Ctrl+P` / the Print button).
3. In the print dialog:
   - **Margins: None** (otherwise the browser adds its own border inside the page);
   - **Scale: 100%** (any shrink breaks the mm-exact layout);
   - **Background graphics: on**, or full-bleed images may print with white gaps.
4. Check the preview page count and the DPI badges first — 150 dpi is the practical floor,
   300 dpi is comfortable.

## Resolution reference

| Paper | Full-bleed pixels at 300 dpi |
|---|---|
| A4 | 2480 × 3508 |
| A5 | 1748 × 2480 |
| B5 | 2079 × 2953 |
| Letter | 2550 × 3300 |
| 6-inch photo (102 × 152 mm) | 1205 × 1795 |

A phone photo (4032 × 3024) prints comfortably at A4. A 1200 × 1600 image stretched across A4
lands at ~146 dpi — the badge will say so.

## Works with the print pipeline, not against it

- Layout is computed in millimetres; `@page { size: <w>mm <h>mm; margin: 0 }` is generated from
  your settings, so the printed page *is* the paper size.
- The preview and the printout share one DOM tree and one stylesheet. Print CSS only drops the
  app chrome and un-scales the sheets (`transform: none`), so what you see is the geometry that
  prints.
- Images are placed with `object-fit` — no canvas resampling, no re-encoding, no profile loss.
- `print-color-adjust: exact` is set so the watermark and image backgrounds are not stripped.

## Internationalization

All UI copy is externalized in an `I18N` pack (7 locales × 81 strings, 567 strings total).
Translations are **sense-for-sense, written as native product copy** — not word-for-word
renderings of the Chinese source. Keys are semantic (`set.watermark`, `badge.dpiLow`, …) and
placeholders are positional (`{0}`); the build checks that every locale has exactly the same key
set and placeholder set.

To add a language: duplicate a pack inside `index.html` (`var PACKS = {...}`), add its code to
`LANGS`, add the native name to `NATIVE`, and set `autodetect` if you want it chosen automatically.

## Privacy & security

- No network calls of any kind (checked by the release gate).
- No `innerHTML` / `eval`; user file names are inserted with `textContent`.
- Embedded CSP: `default-src 'none'; img-src 'self' blob: data:; style-src 'self' 'unsafe-inline';
  script-src 'self' 'unsafe-inline'`.
- Object URLs are revoked on removal and on unload. Nothing about your images is persisted.

## Verification

Checked during development with a Node gate (not shipped — this repo is just the app):

```
gate.js         PASS 122 · FAIL 0
dom-smoke.js    PASS  28 · FAIL 0
```

- 19 real image fixtures generated with Pillow — sRGB/AdobeRGB/ProPhoto/P3 ICC (both `desc` and
  v4 `mluc`), EXIF ColorSpace 1/2, CMYK JPEG, PNG iCCP, PNG cICP (HDR + P3), WebP ICCP, GIF,
  AVIF/HEIC `colr`, junk bytes — all classified correctly.
- The DOM smoke test boots the real `app.js` against a minimal DOM shim: imports real fixtures,
  renders 3 pages, checks the printed watermark, switches to Japanese, toggles the watermark, and
  re-injects the `@page` rule through the actual settings `change` handlers.
- Layout/DPI/pagination/`@page` math asserted against hand-computed millimetre values.
- i18n: 7 × 81 keys, identical key and placeholder sets, no untranslated Chinese in en/de/fr,
  kana present in ja, Hangul present in ko.
- Hygiene: single file, no external script/style/URL, no network APIs, no HTML-injection APIs,
  2 `localStorage` writes (settings only), one inline script, CSP present.

**Not machine-verifiable**: the visual result in a real browser, the print dialog behavior (margin
and background-graphics settings are the user's), and physical printer color. The layout math and
DOM wiring are checked; the appearance needs a human eye — please open it once before trusting it.

## Browser support

Current Chrome / Edge / Safari / Firefox. `AVIF` decodes in all current browsers; `HEIC` only in
Safari (Chrome will show a "can't open" toast — convert to JPEG/PNG first). `deflate` decompression
for PNG iCCP uses `DecompressionStream` when available and degrades to "ICC profile" otherwise.

## License

MIT — see [LICENSE](LICENSE).
