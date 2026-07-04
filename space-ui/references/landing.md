# Mode: Landing Page

A single-file static page where **one striking hero image dominates the viewport**,
wrapped in the space-ui design language. In HTML this is the Lighthouse-95–100 mode —
read `lighthouse-checklist.md` too. (Also buildable as a React page; then drop the
zero-JS rule but keep the layout and tokens.)

## Layout

The hero image IS the page (85–95vh). Everything else is minimal: wordmark, one `<h1>`,
optional one-line subhead, one CTA. Structure:

```
<body>
  <a class="skip" href="#main">Skip to content</a>
  <header>          wordmark + optional nav
  <main id="main">
    <section class="hero">   hero <img> + overlaid headline + CTA
    <section class="cta">    optional secondary CTA
  <footer>          minimal copyright
```

## Hero image

```html
<img class="hero-image" src="hero.jpg"
     alt="Describe the actual image content, not 'hero image'"
     width="1920" height="1080" fetchpriority="high" decoding="async">
```
```css
.hero { position: relative; height: 90vh; min-height: 600px; overflow: hidden; }
.hero-image { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; }
/* Legibility gradient over the image */
.hero::before {
  content: ''; position: absolute; inset: 0; z-index: 1; pointer-events: none;
  background: linear-gradient(180deg,
    color-mix(in srgb, var(--bg) 55%, transparent) 0%,
    color-mix(in srgb, var(--bg) 15%, transparent) 35%,
    var(--bg) 100%);
}
.hero-content { position: relative; z-index: 2; }
```

If the user provides an image, set `width`/`height` to its real pixels (`identify -format
"%wx%h" img.jpg`). If they only describe it, use an Unsplash placeholder
(`https://images.unsplash.com/photo-XXXX?w=1920&q=80`) or tell them to swap `src`.

## Typography

```css
h1 {
  font-family: var(--display); font-weight: 700; text-transform: uppercase;
  font-size: clamp(2.2rem, 6vw, 5rem); line-height: 1.02; color: #fff;
  text-shadow: 0 2px 30px rgba(0,0,0,.5);
}
h1 .accent { color: var(--accent); text-shadow: 0 0 30px var(--accent-glow); }
.subhead { color: var(--muted); font-weight: 300; max-width: 56ch;
  font-size: clamp(.95rem, 1.4vw, 1.15rem); }
```

## CTA button

```css
.cta {
  display: inline-flex; align-items: center; justify-content: center; gap: .6rem;
  min-height: 48px; padding: 1rem 2rem; border-radius: 0;
  font-family: var(--display); letter-spacing: .16em; text-transform: uppercase;
  color: var(--on-accent); background: var(--accent);
  border: 1px solid var(--accent);
  box-shadow: 0 0 24px var(--accent-glow);
  text-decoration: none;
}
.cta:hover { background: var(--accent-bright); box-shadow: 0 0 34px var(--accent-glow); }
.cta:focus-visible { outline: 2px solid #fff; outline-offset: 3px; }
```

## Optional atmosphere

Add `.grid-bg` behind the hero content, an eyebrow chip above the `<h1>`, and small
"coordinate" readouts in a corner (monospace-ish display font, `--dim`) for the sci-fi
HUD feel. Keep `backdrop-filter`/`blur` off full-viewport elements (paint cost).

## Responsive

Hero `height: 90vh` → `75vh` under 768px. Text via `clamp()`. CTA full-width on narrow
screens with a max-width.

## Checklist (landing)

- One `<h1>`; hero `<img>` has width/height/alt/fetchpriority/decoding.
- Legibility gradient ensures text passes contrast over the image.
- HTML target: zero `<script>`, inlined CSS, Google-Fonts-only external — run the full
  `lighthouse-checklist.md`.
