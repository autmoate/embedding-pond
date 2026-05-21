# Embedding Pond

A small, browser-only webapp that visualizes multimodal embeddings (text + image) as waterlilies floating on a pond.

This interactive demo was built in the ScaDS.AI Living Lab in Leipzig. It shows how a model can turn text and images into a common numerical representation (an embedding). These embeddings can then be compared, even if the inputs are different types of data. 

For implementation/UX context, start with `OVERVIEW.md`.

## What It Does

- Create a new waterlily from text or an uploaded image.
- Generate a CLIP embedding in the browser.
- Render a lily shape derived from the embedding.
- Place lilies into a shared 2D layout using PCA.
- Click a lily to inspect the original input and the embedding vector.
- Hover a lily to see a lightweight preview popover.
- Open the bottom-left Info panel for plain-language explanations.

## Run Locally

Prereqs: Node.js (18+ recommended) and npm.

```bash
cd embedding-pond
npm install
npm run dev
```

Then open the Vite dev server URL (usually `http://localhost:5173`).

## Tech Notes

- Embeddings are computed client-side via `@huggingface/transformers` using `Xenova/clip-vit-base-patch32`.
- PCA is computed client-side with `ml-pca`.
- Input hardening:
  - text input is capped at 280 characters,
  - image input only accepts `jpg/jpeg/png`,
  - image file size is limited to 3 MB.
- This is a prototype UI: no backend, no persistence (refresh clears lilies), no auth.
- Water background uses a tileable image: `public/water_tileable.jpg`.

## Deployment Notes

- The production Docker image builds static assets (`npm run build`) and serves them via Nginx.
- Security headers (including CSP, frame restrictions, and content type hardening) are set in `nginx/default.conf`.
- Inputs stay browser-local: uploaded images are represented as object URLs and are not sent to a backend by this app.

## Project Structure

- `src/App.vue` main state + orchestration (create/grow/plant/select)
- `src/components/PondScene.vue` pond scene + lily rendering
- `src/components/LilyCreator.vue` modal for text/image input
- `src/components/LilyPopover.vue` inspector panel for a selected lily
- `src/components/InfoPopover.vue` simple info panel
- `src/components/IconButton.vue` shared close button
- `src/utils/lilyPad.ts` shared lily pad path generator
- `src/utils/embedding.ts` CLIP embedding (text + image) + normalization
- `src/utils/pca.ts` PCA -> screen-space layout
- `src/utils/shape.ts` embedding -> SVG path shape

Note: `src/components/CausticsCanvas.vue` exists as an experiment, but the main pond background is CSS + `public/water_tileable.jpg`.

## Known Limitations

- First embedding run can be slow (model download + warmup).
- Image inputs are currently handled as local object URLs.
- PCA layout uses the current viewport size; resizing can change placements.
- Color is derived from the embedding but constrained to a pink-to-near-white palette (see `src/App.vue`).
