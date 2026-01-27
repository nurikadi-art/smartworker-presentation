# SmartWorker Presentation

This is a scroll-driven React presentation built with Vite.

## Run locally

```bash
npm install
npm run dev
```

## Add custom images

1. Put images in `src/assets/`
2. Import them in `src/App.jsx`
3. Attach to a slide as `image: yourImage`

Example:

```jsx
import authorPhoto from "./assets/author.jpg";

const slides = [
  {
    id: 4,
    theme: "light",
    accent: "#facc15",
    image: authorPhoto,
    content: (...)
  }
];
```

The slide will render the image as a soft background layer. If you want
inline placement instead, you can also add your own `<img>` inside
`slide.content` and style it with a custom class.

## Add new wow effects

- Change or add slide variants: `variant: "aurora" | "neon" | "prism" | "ember" | "iris" | "oasis"`
- Edit animations in `src/styles.css` (`@keyframes sweep`, `pulse`, `drift`)
- Adjust glow/particles with `--accent`, `--accent-soft`, `--accent-strong`

Each slide uses an IntersectionObserver to animate in as it enters view.
You can control the intensity from `src/styles.css` under:

- `.slide__inner > *` (content reveal)
- `.slide__sweep`, `.slide__particles`, `.slide__noise` (background layers)
- `.card` and `.story-card` (cards and media)
