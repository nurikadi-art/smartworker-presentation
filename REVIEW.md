# Code Review: SmartWorker Presentation

**Reviewer:** Claude (automated)
**Date:** 2026-01-27
**Branch:** `claude/review-all-files-7NFaZ`

---

## Files Reviewed

| File | Size | Status |
|------|------|--------|
| `index.html` | 676 B | Entry point for Vite dev server |
| `package.json` | 364 B | Project metadata & dependencies |
| `vite.config.js` | 209 B | Build configuration |
| `README.md` | 1.2 KB | Developer documentation |
| `presentation_backup.html` | 57 KB | Standalone backup (18 slides) |
| `docs/index.html` | 112 KB | Built production output (57 slides) |

---

## Critical Issues

### 1. Missing Source Files (`src/` directory)
**Severity:** Critical
**Files:** `index.html`, `README.md`

The `index.html` entry point references `<script type="module" src="/src/main.jsx">`, but there is no `src/` directory in the repository. The README references `src/App.jsx`, `src/assets/`, and `src/styles.css` which also do not exist. This means:

- `npm run dev` will fail (no source to serve)
- `npm run build` will fail (no source to compile)
- The build output in `docs/` cannot be reproduced

**Recommendation:** Either restore the source files or remove the Vite project setup (`index.html`, `package.json`, `vite.config.js`) and treat the HTML files as the source of truth.

---

### 2. Content Divergence Between Backup and Production
**Severity:** Critical
**Files:** `presentation_backup.html` (18 slides), `docs/index.html` (57 slides)

The production build has 57 slides while the backup has only 18. The two files have diverged significantly:

| Aspect | `presentation_backup.html` | `docs/index.html` |
|--------|---------------------------|-------------------|
| Slides | 18 | 57 |
| Background color | `#0b0f1f` | `#09090b` |
| Default accent | `#22d3ee` (cyan) | `#818cf8` (indigo) |
| Images | CloudFront URLs (real images) | placehold.co placeholders |
| Design style | Glassmorphism, bold gradients | Minimal, Apple-like, subtle |
| Scroll snap | CSS only | CSS set then JS disables it |
| Navigation | Dots only | Dots + slide counter + keyboard hints |

**Recommendation:** Decide which file is the canonical source and archive/remove the other. If the backup serves as a rollback, document that clearly.

---

### 3. Hardcoded `is-active` Class on Slide 2
**Severity:** Medium
**Files:** `presentation_backup.html:1115`, `docs/index.html:1073`

Slide 2 has `class="slide slide--dark is-active"` hardcoded in the HTML. This means slide 2's animations play immediately on page load regardless of scroll position. The IntersectionObserver will manage it afterwards, but on initial load the user sees slide 2 animated even though it's off-screen.

**Recommendation:** Remove the `is-active` class from the HTML. Let the IntersectionObserver handle all activation consistently.

---

## Moderate Issues

### 4. Scroll-Snap CSS/JS Conflict
**Severity:** Medium
**File:** `docs/index.html:2326`

The CSS sets `scroll-snap-type: y mandatory` on `.deck`, but the JavaScript immediately overrides it with `deck.style.scrollSnapType = 'none'`. This creates a flash where scroll snapping is briefly active before JS runs, then it's disabled.

**Recommendation:** Either remove the CSS `scroll-snap-type` declaration (since JS removes it anyway) or remove the JS override if scroll snapping is desired.

---

### 5. 57 Navigation Dots
**Severity:** Medium
**File:** `docs/index.html:2287-2299`

With 57 slides, the navigation dots column on the right side would be extremely cramped (57 dots at ~22px each = 1254px, exceeding most viewport heights). On desktop, this creates an unusable scrollbar-like strip.

**Recommendation:** Replace dots with the slide counter (which already exists) or group dots by section.

---

### 6. Placeholder Images in Production
**Severity:** Medium
**File:** `docs/index.html`

All images use `placehold.co` placeholder URLs. Since `docs/index.html` is the GitHub Pages deployment, live visitors see placeholder text like "Remote Work", "Phone", "USA" instead of actual images.

Placeholder URLs found:
- `https://placehold.co/280x380/1a1a1a/e2e8f0?text=App+Screen`
- `https://placehold.co/1920x1080/0b0f1f/e2e8f0?text=Remote+Work`
- `https://placehold.co/1920x1080/0b0f1f/e2e8f0?text=Stressed+BG`
- `https://placehold.co/240x320/1f2937/e2e8f0?text=Phone`
- `https://placehold.co/120x80/1f2937/e2e8f0?text=USA` (and similar flag images)
- Various other placeholder images throughout

Meanwhile, `presentation_backup.html` contains actual CloudFront image URLs that could be used.

**Recommendation:** Replace placeholder URLs with actual images, either by restoring the CloudFront URLs from the backup or by adding images to the repository.

---

### 7. External Font Dependency Without Fallback Strategy
**Severity:** Low-Medium
**Files:** `index.html:11-16`, `presentation_backup.html:8-10`, `docs/index.html:8-10`

Google Fonts (`Manrope`) is loaded as a render-blocking resource. If the CDN is slow or blocked:
- The font stack falls back to `system-ui`, which works
- But `font-display: swap` is not explicitly set (Google Fonts defaults to `swap` in modern browsers, but it's implicit)

**Recommendation:** Add `&display=swap` to the Google Fonts URL for explicit control:
```
https://fonts.googleapis.com/css2?family=Manrope:wght@300;400;500;600;700;800&display=swap
```
(This is already present - confirmed. Low priority.)

---

## Minor Issues

### 8. Missing `<meta name="description">`
**Severity:** Low
**Files:** All HTML files

None of the HTML files include a meta description tag, which affects SEO and social sharing previews.

**Recommendation:** Add:
```html
<meta name="description" content="SmartWorker — система удалённой работы на компании США из стран СНГ">
```

---

### 9. Accessibility Concerns
**Severity:** Low-Medium
**Files:** `presentation_backup.html`, `docs/index.html`

**Positive:**
- Decorative elements properly use `aria-hidden="true"`
- `prefers-reduced-motion` is respected
- Semantic HTML (`<section>`, `<h1>`-`<h3>`, `<ul>`) is used

**Issues:**
- No skip-to-content link for keyboard users
- Navigation dots are `aria-hidden="true"`, so keyboard users can only scroll
- No `<main>` landmark wrapping the content
- Photo card div has `aria-hidden="true"` but the author photo is content, not decoration
- The `lang="ru"` attribute is correct for the Russian content

**Recommendation:** Add `role="main"` to `.deck`, add a skip-to-content link, and reconsider `aria-hidden` on content images.

---

### 10. Performance: Animation Load
**Severity:** Low
**File:** `docs/index.html`

With 57 slides, each having 7 decorative overlay elements (frame, sweep, particles, noise, 2 orbs, grid) plus CSS animations, there are approximately:
- 57 x 7 = 399 animated decorative elements
- 30+ CSS `@keyframes` rules running continuously

The IntersectionObserver only toggles `is-active` for content reveal but does not pause background animations for off-screen slides.

**Recommendation:** Consider using `animation-play-state: paused` for slides not in the viewport, or use `content-visibility: auto` for off-screen slides.

---

### 11. Dead CSS in `presentation_backup.html`
**Severity:** Low
**File:** `presentation_backup.html`

The `.title--lg, .title--xl` text shimmer animation (lines 1006-1012) applies a background-clip text effect, but this conflicts with the gradient text in the hero orbit which uses `-webkit-text-fill-color: transparent`. The shimmer animation may not be visible on the hero slide titles.

---

### 12. Inline Styles
**Severity:** Low
**Files:** Both HTML files

CSS custom properties (`--accent`, `--accent-soft`, `--accent-strong`) are set via inline `style` attributes on each `<section>`. While this works, it creates verbose HTML. A data attribute approach or CSS classes per variant would be cleaner.

Current:
```html
<section style="--accent: #7dd3fc; --accent-soft: rgba(125, 211, 252, 0.10); --accent-strong: rgba(125, 211, 252, 0.28);">
```

Alternative:
```html
<section data-accent="sky">
```

**Recommendation:** Low priority. The current approach works and is maintainable enough.

---

## Security Notes

### 13. Exposed Contact Information
**Severity:** Informational
**File:** `presentation_backup.html:1532`

The CTA slide contains a Telegram handle (`@thirsty_invest31`) and phone number (`+7 700 131 0304`). This is intentional for the presentation's purpose but be aware this is publicly accessible on GitHub Pages.

---

## Summary

| Priority | Count | Items |
|----------|-------|-------|
| Critical | 2 | Missing source files, content divergence |
| Medium | 4 | Hardcoded is-active, scroll-snap conflict, 57 nav dots, placeholder images |
| Low | 6 | Meta description, accessibility, performance, dead CSS, inline styles, contact info |

### Recommended Actions (in order):
1. **Restore `src/` directory** or restructure the project to use the HTML files directly
2. **Replace placeholder images** in `docs/index.html` with actual images
3. **Remove hardcoded `is-active`** from slide 2
4. **Resolve scroll-snap CSS/JS conflict** in `docs/index.html`
5. **Reduce navigation dots** or replace with grouped navigation for 57 slides
6. **Add meta description** to all HTML files
7. **Consider animation performance** for 57 slides with heavy CSS animations
