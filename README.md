# น้ำท่วมซ้ำ 20 จังหวัด — Vue 3 + Vite Clone

## Running

```bash
npm install
npm run dev
```

Open the URL printed by Vite (normally `http://localhost:5173`).

Production build:

```bash
npm run build
npm run preview
```

## Architecture / fidelity

- Vue 3 + Vite application (`src/App.vue`) provides the SPA container and shareable province route (`?province=nan`).
- **All 21 original HTML documents are preserved byte-for-byte** under `public/original/` (the overview plus 20 provinces).
- Original rendering, map data, styles, charts and JS interactions execute inside a same-origin iframe, preserving the source behavior as closely as possible.
- Original cross-page links are bridged to the Vue host; province selectors using `location.href` still operate in the iframe.
- External resources referenced by the original pages require network access. The source has not been rewritten into native Vue components; this is a fidelity-first Vue host rather than a full component-by-component migration.

## Pages

- `/` — overview for 20 provinces
- `/?province=nan` — example deep link to Nan province
- Other accepted province slugs are listed in `src/App.vue`.
