---
name: space-landing
description: Build blazing-fast, futuristic/space-age static landing pages with a single dominant hero image that fills the browser viewport. Use this skill whenever the user asks to create a landing page, hero page, splash page, or single-page site with a futuristic, space, sci-fi, cyberpunk, or high-tech aesthetic — especially when performance, Lighthouse scores, or page speed are mentioned. Also trigger when the user says "space age website", "futuristic landing page", "hero image page", "full-screen image site", "fast landing page", or mentions wanting near-perfect Lighthouse scores on a visually striking page. This skill produces a single self-contained HTML file with zero JavaScript, inlined critical CSS, and semantic markup engineered for 95-100 across all four Lighthouse categories.
---

# Space Landing — Futuristic Hero-Image Landing Pages

Build a single-file static landing page where one striking image dominates the viewport, wrapped in a futuristic/space-age design language. The page must score 95–100 in **all four** Lighthouse categories: Performance, Accessibility, Best Practices, and SEO.

## Design Philosophy

The aesthetic is **cinematic futurism** — think deep space, orbital stations, neon-lit horizons, or planetary surfaces. The page should feel like the opening frame of a sci-fi film: one arresting image, minimal but purposeful typography, and an atmosphere of vastness and precision.

### Visual Identity

- **Color palette**: Deep blacks (#0a0a0f, #0d0d1a), cool blue-grays (#1a1a2e), electric accents (cyan #00f0ff, magenta #ff00aa, amber #ffaa00). Pick ONE accent color per page — restraint is power.
- **Typography**: Use exactly TWO fonts from Google Fonts. One geometric/display face for headings (e.g., Orbitron, Rajdhani, Exo 2, Chakra Petch, Audiowide, Bruno Ace) and one clean sans-serif for body (e.g., Outfit, Plus Jakarta Sans, Sora, Manrope). Never use Inter, Roboto, or Arial.
- **Atmosphere**: Subtle scanline overlays, fine grid patterns via CSS, soft radial glows behind the hero image, thin luminous borders (1px with low-opacity accent color).
- **Layout**: The hero image is the centerpiece — it occupies 85-95vh. All other content is secondary and minimal: a wordmark/logo, a single headline, an optional one-line subhead, and one CTA button.

### What Makes It Unforgettable

The hero image is not decoration — it IS the page. Everything else (typography, glow effects, scanlines) exists to frame and elevate that image. The page should feel like looking through a viewport into another world.

## Architecture — Single HTML File

Everything lives in one `.html` file. No external JS. No build step. No framework. This is the fastest possible page.

```
index.html
├── <head>
│   ├── Preconnect to Google Fonts (2 links)
│   ├── Google Fonts stylesheet link (display=swap)
│   ├── Inlined <style> block (all CSS)
│   └── SEO meta tags
└── <body>
    ├── <header> — logo/wordmark + optional nav links
    ├── <main>
    │   ├── <section class="hero"> — the hero image + headline overlay
    │   └── <section class="cta"> — optional call-to-action
    └── <footer> — minimal copyright line
```

## Lighthouse-Perfect Engineering

Every decision serves the Lighthouse score. Read `references/lighthouse-checklist.md` for the full checklist before writing any code.

### Performance (target: 100)

- **Zero JavaScript**. None. Not a single `<script>` tag. All interactions are CSS-only (`:hover`, `:focus-visible`).
- **Inline all CSS** in a `<style>` tag in `<head>`. No external stylesheet except Google Fonts.
- **Google Fonts**: Use `<link rel="preconnect">` for both `fonts.googleapis.com` and `fonts.gstatic.com` (with crossorigin). Then one `<link>` with `display=swap`. This is the ONLY external resource.
- **Hero image**: Use `<img>` with explicit `width` and `height` attributes (matching the image's intrinsic dimensions or a proportional scale). Add `fetchpriority="high"` and `decoding="async"`. Do NOT lazy-load the hero — it's above the fold.
- **No layout shift**: The `width`/`height` on the `<img>` tag reserves space. The CSS uses `object-fit: cover` to fill the container without distortion.
- **Minimal CSS**: Target under 6KB of inlined CSS. No unused rules. No CSS resets — write only what you need.
- **No `@import`** in CSS. Ever.

### Accessibility (target: 100)

- **Semantic HTML**: `<header>`, `<main>`, `<footer>`, `<nav>`, `<section>`, `<h1>` (exactly one).
- **Image alt text**: The hero `<img>` must have a descriptive, meaningful `alt` attribute (not "hero image" — describe what's actually in the image).
- **Color contrast**: All text must meet WCAG AA (4.5:1 for body, 3:1 for large text). Light text on dark backgrounds makes this straightforward — but verify accent colors against their backgrounds.
- **Focus states**: Every interactive element (`<a>`, `<button>`) needs a visible `:focus-visible` outline. Use the accent color with a 2px offset.
- **Language attribute**: `<html lang="en">` (or the correct language).
- **Skip link**: Include a visually-hidden skip-to-content link as the first element in `<body>`.
- **Touch targets**: Buttons and links must be at least 44×44px.
- **Reduced motion**: Wrap any CSS animations in `@media (prefers-reduced-motion: no-preference) { }`.

### Best Practices (target: 100)

- **HTTPS-ready**: All external URLs use `https://`.
- **No deprecated APIs** (not applicable — there's no JS).
- **Charset**: `<meta charset="UTF-8">` as the first child of `<head>`.
- **Doctype**: `<!DOCTYPE html>` on line 1.
- **No console errors**: Since there's no JS, this is automatic.
- **Aspect ratio**: The hero image `width`/`height` attributes prevent layout shift and satisfy the "image has correct aspect ratio" audit.

### SEO (target: 100)

- `<title>` tag with the page title (under 60 characters).
- `<meta name="description" content="...">` (under 160 characters).
- `<meta name="viewport" content="width=device-width, initial-scale=1">`.
- One `<h1>` element.
- All links have descriptive text (no "click here").
- Valid, crawlable HTML.
- `<html lang="...">` set.

## CSS Techniques for the Aesthetic

Use these CSS-only techniques to build the futuristic atmosphere without JavaScript:

```css
/* Scanline overlay via pseudo-element */
.hero::after {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    to bottom,
    transparent,
    transparent 2px,
    rgba(0, 0, 0, 0.03) 2px,
    rgba(0, 0, 0, 0.03) 4px
  );
  pointer-events: none;
}

/* Soft glow behind hero image */
.hero::before {
  content: '';
  position: absolute;
  top: 50%; left: 50%;
  width: 60%; height: 60%;
  transform: translate(-50%, -50%);
  background: radial-gradient(ellipse, rgba(0, 240, 255, 0.08), transparent 70%);
  pointer-events: none;
  z-index: 1;
}

/* Luminous border */
.cta-button {
  border: 1px solid rgba(0, 240, 255, 0.3);
  box-shadow: 0 0 20px rgba(0, 240, 255, 0.1);
}

/* Hover glow — CSS only */
.cta-button:hover {
  box-shadow: 0 0 30px rgba(0, 240, 255, 0.25);
  border-color: rgba(0, 240, 255, 0.6);
}
```

Wrap all transitions and animations in a reduced-motion media query:

```css
@media (prefers-reduced-motion: no-preference) {
  .cta-button {
    transition: box-shadow 0.3s ease, border-color 0.3s ease;
  }
}
```

## Image Handling

The user will either provide an image or describe what they want. The skill handles both cases:

**User provides an image file**: Reference it with a relative path. Set `width` and `height` to the image's actual pixel dimensions (or a proportional downscale). Use the computer tools to check actual dimensions: `identify -format "%wx%h" image.jpg` (ImageMagick) or `file image.jpg`.

**User describes what they want**: Use a placeholder `<img>` pointing to a high-quality stock source like `https://images.unsplash.com/photo-XXXXX?w=1920&q=80` or instruct the user to replace the `src` with their own image. Set reasonable dimensions like `width="1920" height="1080"`.

Always apply these to the hero `<img>`:
```html
<img
  src="hero.jpg"
  alt="Descriptive alt text of the actual image content"
  width="1920"
  height="1080"
  fetchpriority="high"
  decoding="async"
  class="hero-image"
/>
```

And this CSS:
```css
.hero-image {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

## Responsive Design

Mobile-first is not the goal — cinematic-first is. But the page must work on all screens:

- The hero section uses `height: 90vh` (not a fixed pixel value).
- Text scales with `clamp()`: headings use `clamp(1.8rem, 5vw, 4.5rem)`, body uses `clamp(0.9rem, 1.2vw, 1.1rem)`.
- The CTA button has generous padding and a min-width for touch targets.
- On narrow screens (<768px), reduce hero height to `75vh` and adjust overlay text positioning.

## Output

Save the final HTML file to `/mnt/user-data/outputs/index.html` (or a user-specified filename) and present it with `present_files`. The file should be completely self-contained and immediately openable in any browser.

## Checklist Before Delivering

Before saving the file, mentally verify:

1. `<!DOCTYPE html>` is on line 1
2. `<html lang="en">` is set
3. `<meta charset="UTF-8">` is first in `<head>`
4. `<meta name="viewport">` is present
5. `<title>` is present and under 60 chars
6. `<meta name="description">` is present and under 160 chars
7. Google Fonts: preconnect links + one stylesheet link with `display=swap`
8. All CSS is inlined in `<style>` — no external CSS files
9. Zero `<script>` tags
10. Hero `<img>` has `width`, `height`, `alt`, `fetchpriority="high"`, `decoding="async"`
11. Exactly one `<h1>`
12. Skip-to-content link is first in `<body>`
13. All interactive elements have `:focus-visible` styles
14. All animations wrapped in `prefers-reduced-motion` media query
15. Color contrast meets WCAG AA on all text
16. Semantic landmarks: `<header>`, `<main>`, `<footer>`
17. File is saved to `/mnt/user-data/outputs/`
