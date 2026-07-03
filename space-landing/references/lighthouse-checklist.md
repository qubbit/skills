# Lighthouse Perfect-Score Checklist

This is the definitive checklist for achieving 95–100 in all four Lighthouse categories on a static landing page with no JavaScript. Read this before writing any code.

---

## Performance Audits

### Critical (will fail the score if missed)

- [ ] **No render-blocking resources**: The only external resource is Google Fonts. Use `<link rel="preconnect">` to eliminate connection latency. The font stylesheet uses `display=swap` so text renders immediately with a fallback.
- [ ] **No JavaScript**: Zero `<script>` tags. This eliminates Total Blocking Time, Time to Interactive delays, and main-thread work.
- [ ] **Inline CSS**: All styles live in a `<style>` tag in `<head>`. No external `.css` file to fetch. Keep total CSS under 6KB.
- [ ] **Image dimensions declared**: The hero `<img>` tag must have explicit `width` and `height` attributes matching the image's intrinsic aspect ratio. This reserves layout space and prevents Cumulative Layout Shift (CLS = 0).
- [ ] **Hero image priority**: Add `fetchpriority="high"` and `decoding="async"` to the hero `<img>`. Do NOT add `loading="lazy"` — it's above the fold.
- [ ] **No `@import` in CSS**: CSS `@import` is render-blocking. Never use it. Load Google Fonts via `<link>` in `<head>`.

### Important (can cost 5-15 points if missed)

- [ ] **Efficient CSS**: No unused selectors. No CSS reset libraries. Write only the rules you need. Avoid deeply nested selectors.
- [ ] **Font display swap**: The Google Fonts URL must include `&display=swap` to prevent invisible text during loading (FOIT).
- [ ] **Minimal DOM**: Keep the total element count under 200. A hero landing page should have 30-60 elements.
- [ ] **No large layout shifts**: Reserve space for all content. Use `vh` units for hero height, `clamp()` for text sizing, and explicit image dimensions.
- [ ] **Preconnect hints**: Include BOTH preconnect links for Google Fonts:
  ```html
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  ```

### Nice-to-have (marginal gains)

- [ ] **System font fallback stack**: In your `font-family` declarations, include a fallback stack so text renders before custom fonts load:
  ```css
  font-family: 'Orbitron', ui-sans-serif, system-ui, sans-serif;
  ```
- [ ] **Avoid expensive CSS properties on large areas**: `backdrop-filter`, `filter: blur()`, and large `box-shadow` on full-viewport elements can slow paint. Use them sparingly and on small elements.

---

## Accessibility Audits

### Critical (will fail)

- [ ] **`<html lang="en">`**: Always set the language attribute.
- [ ] **Image alt text**: The hero image MUST have a meaningful, descriptive `alt` attribute. Describe the image content, not its role. Bad: `alt="hero"`. Good: `alt="Astronaut floating above Earth's atmosphere with the sun rising on the horizon"`.
- [ ] **Heading hierarchy**: Exactly one `<h1>` on the page. If subheadings exist, they must follow the hierarchy (`<h2>` after `<h1>`, never skip levels).
- [ ] **Color contrast**: All text must pass WCAG AA. Minimum ratios:
  - Normal text (< 18pt): 4.5:1
  - Large text (≥ 18pt bold or ≥ 24pt): 3:1
  - Common safe combos on dark (#0a0a0f) background: white (#ffffff = 19.4:1), light gray (#c0c0c0 = 10.3:1), accent cyan (#00f0ff) CHECK — may need to be lightened to #66f7ff or similar for body text.
- [ ] **Semantic landmarks**: Use `<header>`, `<main>`, `<footer>`. Screen readers rely on these for navigation.
- [ ] **Interactive element focus**: All `<a>` and `<button>` elements need visible `:focus-visible` styling. Default browser focus rings are acceptable but a styled one is better:
  ```css
  :focus-visible {
    outline: 2px solid var(--accent);
    outline-offset: 3px;
  }
  ```
- [ ] **Skip navigation link**: First element inside `<body>` should be:
  ```html
  <a href="#main" class="skip-link">Skip to content</a>
  ```
  Visually hidden but accessible:
  ```css
  .skip-link {
    position: absolute;
    top: -100%;
    left: 0;
    padding: 0.75rem 1.5rem;
    background: var(--accent);
    color: var(--bg);
    z-index: 1000;
    font-weight: 700;
  }
  .skip-link:focus {
    top: 0;
  }
  ```

### Important

- [ ] **Touch targets**: Buttons and links must be at least 44×44px. Apply `min-height: 44px; min-width: 44px;` plus padding.
- [ ] **Link purpose**: Link text must describe the destination. No "click here" or "read more" without context.
- [ ] **Reduced motion**: All CSS transitions and animations MUST be inside:
  ```css
  @media (prefers-reduced-motion: no-preference) {
    /* animations here */
  }
  ```
  This respects users who have enabled reduced motion in their OS settings.

---

## Best Practices Audits

### Critical

- [ ] **`<!DOCTYPE html>`**: Must be the very first line of the file.
- [ ] **`<meta charset="UTF-8">`**: Must be the first element inside `<head>`.
- [ ] **HTTPS URLs**: Every external URL (Google Fonts, images, links) must use `https://`.
- [ ] **No deprecated HTML**: Don't use `<center>`, `<font>`, `<marquee>`, etc.
- [ ] **Image aspect ratio**: The `width` and `height` attributes on `<img>` must match the image's actual aspect ratio. Lighthouse checks this.
- [ ] **No `document.write`**: Not applicable (no JS), but noted for completeness.

### Important

- [ ] **No browser errors**: Since there's no JS, console errors are essentially impossible. But ensure all resource URLs are valid (fonts, images).
- [ ] **Valid HTML**: Run the output through a mental validation — no unclosed tags, no duplicate IDs, no invalid nesting.

---

## SEO Audits

### Critical (will fail)

- [ ] **`<title>` tag**: Present, unique, under 60 characters. Should describe the page content.
- [ ] **`<meta name="description">`**: Present, under 160 characters. Should summarize the page.
- [ ] **`<meta name="viewport" content="width=device-width, initial-scale=1">`**: Required for mobile SEO.
- [ ] **`<h1>` element**: Exactly one. Should contain the primary keyword/topic.
- [ ] **`<html lang="...">`**: Set to the correct language code.
- [ ] **Crawlable links**: All `<a>` tags must have an `href`. No `javascript:void(0)`.
- [ ] **Valid `robots` directives**: If you include a `robots` meta tag, ensure it allows indexing. Best to omit it entirely (defaults to index/follow).

### Important

- [ ] **Descriptive link text**: Links should describe their destination, not "click here".
- [ ] **Status codes**: The page itself should serve a 200. Since it's a static file, this is handled by the server, not the HTML.
- [ ] **Image alt text**: Also counted by SEO audits, not just accessibility.

---

## Template Structure

Here is the exact order of elements in `<head>` for optimal loading:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Page Title Here</title>
  <meta name="description" content="A concise description of this page under 160 characters.">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=DisplayFont:wght@400;700&family=BodyFont:wght@300;400;500&display=swap" rel="stylesheet">
  <style>
    /* ALL CSS INLINED HERE */
  </style>
</head>
```

And the body structure:

```html
<body>
  <a href="#main" class="skip-link">Skip to content</a>

  <header>
    <span class="wordmark">BRAND</span>
    <nav aria-label="Main navigation">
      <a href="#about">About</a>
      <a href="#contact">Contact</a>
    </nav>
  </header>

  <main id="main">
    <section class="hero" aria-label="Hero">
      <img
        src="hero-image.jpg"
        alt="Meaningful description of image content"
        width="1920"
        height="1080"
        fetchpriority="high"
        decoding="async"
        class="hero-image"
      >
      <div class="hero-overlay">
        <h1>Headline Goes Here</h1>
        <p class="tagline">Optional one-line subheading.</p>
      </div>
    </section>

    <section class="cta-section" aria-label="Call to action">
      <a href="#" class="cta-button" role="button">Explore Now</a>
    </section>
  </main>

  <footer>
    <p>&copy; 2026 Brand Name. All rights reserved.</p>
  </footer>
</body>
</html>
```

---

## Common Mistakes That Cost Points

1. **Forgetting `width`/`height` on `<img>`** → CLS penalty, drops Performance by 10-20 points.
2. **Using `loading="lazy"` on the hero image** → Delays LCP (Largest Contentful Paint), drops Performance.
3. **Accent color fails contrast** → Cyan (#00f0ff) on black (#0a0a0f) is only 12.6:1 which passes, but lighter cyans or magentas on dark gray backgrounds may not. Always verify.
4. **Missing skip link** → Automatic accessibility failure.
5. **No `prefers-reduced-motion` guard** → Accessibility penalty for animations.
6. **External CSS file instead of inline** → Adds a render-blocking request, hurts FCP.
7. **Forgetting `<meta name="description">`** → SEO audit failure.
8. **Multiple `<h1>` tags** → SEO audit failure.
9. **`<a>` without `href`** → SEO crawlability failure.
10. **Using `<div>` instead of semantic elements** → Accessibility landmark failures.
