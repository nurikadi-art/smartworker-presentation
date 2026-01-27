# SmartWorker Presentation

A scroll-driven presentation showcasing the SmartWorker system for remote work opportunities in USD from CIS countries.

## Project Structure

| File | Purpose |
|------|---------|
| `docs/index.html` | Production build (57 slides) — deployed to GitHub Pages |
| `presentation_backup.html` | Standalone backup (18 slides) with original design |
| `index.html` | Vite dev server entry point (requires `src/` — see note below) |
| `vite.config.js` | Vite build config (outputs to `docs/`) |

> **Note:** The `src/` directory (React source files) is not currently in the repository.
> The production build in `docs/index.html` and the backup HTML are the current sources of truth.
> Running `npm run dev` or `npm run build` will fail until source files are restored.

## View locally

Open `docs/index.html` or `presentation_backup.html` directly in a browser — no build step needed.

To serve via a local server:

```bash
npx serve docs
```

## Slide Variants

Each slide supports visual theme variants via the `data-variant` attribute:

- `aurora` — cool teal gradients
- `neon` — green/cyan accent
- `prism` — indigo tones
- `ember` — warm orange
- `iris` — pink/magenta
- `oasis` — sky blue

## Animations

CSS animations activate via IntersectionObserver when slides enter the viewport.
Key animation layers per slide:

- `.slide__inner > *` — content fade-in reveal
- `.slide__sweep` — horizontal light sweep
- `.slide__particles` — dot grid drift
- `.slide__orb` — morphing blob shapes
- `.slide__grid` — background grid pattern

Respects `prefers-reduced-motion` for accessibility.

## Keyboard Navigation (docs/index.html)

- `Arrow Down` / `Arrow Right` / `Space` — next slide
- `Arrow Up` / `Arrow Left` — previous slide
- `Home` — first slide
- `End` — last slide
