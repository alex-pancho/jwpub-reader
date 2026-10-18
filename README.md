# JWPUB Reader

A single-file, offline viewer for `.jwpub` publications. Open the HTML file in a browser, drop a `.jwpub` onto it, and read. Everything runs locally in the page: the file is never uploaded anywhere.

The decryption logic follows [jwpub-toolkit](https://github.com/thenights20/jwpub-toolkit) and its `JWPUB_FORMAT.md`.
<img width="1634" height="815" alt="image" src="https://github.com/user-attachments/assets/4c5e5c47-d2b8-4bb2-ab55-b27c17a1af4c" />

## Features

- Opens `.jwpub` files by button or drag and drop
- Decrypts and decompresses document content in the browser (Web Crypto + `DecompressionStream`)
- Contents tree built from `PublicationViewItem`, with a flat document list as a fallback
- Renders each document in a sandboxed frame, including the built-in `du-*` color, background and font-size classes
- Full-text search across all documents with highlighted matches
- Internal `jwpub://p/...` links jump to the target document
- Text size controls, previous/next navigation, light and dark theme (follows the system)
- "About this publication" page with metadata, manifest fields and the derived key string
- Raw HTML view of the decrypted document (the `</> HTML` button), handy for checking what a publication contains
- Switchable interface language through a plain JSON file

## Quick start

1. Download `jwpub-reader.html`.
2. Double-click it to open it in Chrome, Edge, Firefox or Safari.
3. Click **Open .jwpub** or drop a file onto the page.

No installation, build step or network connection is needed. The page ships with a small demo publication (`Legenda colori`, built with jwpub-toolkit) so it shows something on first load. Opening your own file replaces it.

Fonts are loaded from Google Fonts when online and fall back to system fonts when offline.

## Interface language

The interface is in English by default. Translations are JSON files:

```
jwpub-reader.html
ua.json        ← Ukrainian
en.json        ← template with every key
```

To use a translation:

- **Opened from disk (double-click):** browsers do not let a page read files next to it, so click **Language** and pick the JSON file, or drop the JSON file onto the page. The choice is remembered in that browser. The ✕ button next to **Language** switches back to English.
- **Served over HTTP** (for example `python -m http.server` in the folder): if a `ua.json` file sits next to the HTML file, it is loaded automatically.

### Adding a language

Copy `en.json`, translate the values, and set the language name in `_meta`:

```json
{
  "_meta": { "language": "Deutsch", "code": "de" },
  "open": "…",
  "toc.title": "…"
}
```

Keep the keys and any `{placeholders}` (such as `{name}` or `{n}`) unchanged. Keys missing from a translation fall back to English.

## How it works

A `.jwpub` file is a ZIP archive:

```
publication.jwpub
├── manifest.json
└── contents            (a nested ZIP)
    ├── <symbol>.db     (SQLite)
    └── media files
```

Document text in the `Document.Content` column is encrypted. The reader derives the key like this:

1. Build the string `{MepsLanguageIndex}_{Symbol}_{Year}`, with `_{IssueTagNumber}` appended when it is not zero. All values come from the `Publication` table.
2. Take the SHA-256 hash of the string.
3. XOR the hash with the constant `11cbb5587e32846d4c26790c633da289f66fe5842a3a585ce1bc3a294af5ada7`.
4. The first 16 bytes are the AES-128-CBC key and the last 16 bytes are the IV.
5. Decrypt the blob, then inflate it with zlib. The result is the document's HTML.

SQLite is read with [sql.js](https://github.com/sql-js/sql.js) (WebAssembly), and archives are opened with [JSZip](https://stuk.github.io/jszip/). Both are embedded in the HTML file.

## Limitations

- Images are matched by file name against the files in the archive. Publications that link images only through the `Multimedia` table may show no pictures.
- Bible links (`jwpub://b/...`) show the reference but do not open a Bible, because it is not part of the file.
- Only the `Document` table is shown. Other content tables are not read.
- The whole file is loaded into memory, so very large publications may be slow on weak devices.
- A modern browser is required (`DecompressionStream` is available in current Chrome, Edge, Safari and Firefox).

## Development

The viewer is one self-contained HTML file. Styles come first, then the embedded libraries (JSZip, sql.js, the SQLite WebAssembly binary and the demo publication as base64), and the application code is the last `<script>` block. Interface strings live in the `EN` object in that block and mirror `en.json`.

## Disclaimer

This is an unofficial tool. It is not produced, approved or distributed by Jehovah's Witnesses or the Watch Tower Bible and Tract Society. Use it only with publications you are entitled to read.

## License

Add your license here. The bundled libraries are sql.js (MIT) and JSZip (MIT or GPLv3, dual-licensed). jwpub-toolkit, which this project follows, is MIT-licensed.
