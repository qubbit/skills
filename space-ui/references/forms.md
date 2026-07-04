# Mode: Forms & Input Elements

Sci-fi form controls — inputs, selects, toggles, checkboxes, radios, sliders, and
validation — all themed from the accent tokens. Works in HTML (add minimal vanilla JS
only for validation/interactivity) or React (`references/react.md`).

## Field scaffold

Every field pairs a `<label>` with its control (accessibility requirement).

```html
<div class="field">
  <label class="field-label" for="email">Email</label>
  <input class="input" type="email" id="email" placeholder="you@ship.io" autocomplete="email">
  <p class="field-error" role="alert"></p>   <!-- populated on invalid -->
</div>
```

## Core input / textarea

```css
.field { display: grid; gap: .5rem; margin-bottom: 1.1rem; }
.field-label {
  font-family: var(--display); font-size: 0.66rem; letter-spacing: .2em;
  text-transform: uppercase; color: var(--dim);
}
/* Text inputs & textarea. (For dropdowns use the custom select below — NOT a
   native <select>, whose open list can't be themed.) */
.input, .textarea {
  width: 100%; min-height: 44px; padding: .6rem .75rem; border-radius: 0; /* sharp */
  font-family: var(--body); font-size: .95rem; color: var(--ink);
  background: var(--bg); border: 1px solid var(--line);
}
.input::placeholder { color: var(--dim); }
.input:hover, .textarea:hover { border-color: var(--accent-ring); }
.input:focus-visible, .textarea:focus-visible {
  outline: 2px solid var(--accent-ring); outline-offset: 2px; border-color: var(--accent);
}
.input:disabled { opacity: .5; cursor: not-allowed; }
/* strip number spinners for a clean HUD look */
.input[type=number]::-webkit-inner-spin-button,
.input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; margin: 0; }
```

## Custom select (dropdown)

**Do NOT use a native `<select>` for the dropdown.** You can style its collapsed box,
but the moment it opens the browser renders the OS-native option list (rounded, gray,
off-theme) which cannot be styled with CSS and shatters the aesthetic. Build a custom
listbox instead — a `<button role="combobox">` + a `<ul role="listbox">` you own and
style. Works identically in vanilla HTML+JS and React (same markup/ARIA/CSS; only the
event wiring differs). Verified in Chrome.

Key rule that trips people up: the **collapsed button shows the label ONLY — never a
checkmark**. The `✓` appears only on the selected option *inside the open list*. Keep the
option label in its own `.txt` span so the button copies the label without the tick.

```html
<div class="xselect" data-open="false">
  <button type="button" class="xselect-btn" id="sec-btn" role="combobox"
    aria-haspopup="listbox" aria-expanded="false" aria-controls="sec-list"
    aria-labelledby="sec-lbl sec-btn">
    <span class="val">Helios Ring — Inner</span>
    <span class="arrow" aria-hidden="true"></span>
  </button>
  <ul class="xselect-list" id="sec-list" role="listbox" aria-labelledby="sec-lbl" tabindex="-1">
    <li class="xselect-opt" role="option" id="sec-0" aria-selected="true">
      <span class="tick" aria-hidden="true">✓</span><span class="txt">Helios Ring — Inner</span></li>
    <li class="xselect-opt" role="option" id="sec-1" aria-selected="false">
      <span class="tick" aria-hidden="true">✓</span><span class="txt">Kuiper Gate — Outer</span></li>
    <li class="xselect-opt" role="option" id="sec-2" aria-selected="false">
      <span class="tick" aria-hidden="true">✓</span><span class="txt">Lagrange Station L5</span></li>
  </ul>
  <input type="hidden" name="sector" value="Helios Ring — Inner"><!-- for form submit -->
</div>
```
```css
.xselect { position: relative; }
.xselect-btn {
  width: 100%; min-height: 44px; padding: .6rem .75rem; border-radius: 0;
  font: inherit; font-size: .95rem; color: var(--ink); text-align: left; cursor: pointer;
  background: var(--bg); border: 1px solid var(--line);
  display: flex; align-items: center; justify-content: space-between; gap: .5rem;
}
.xselect-btn:hover { border-color: var(--accent-ring); }
.xselect-btn:focus-visible { outline: 2px solid var(--accent-ring); outline-offset: 2px; border-color: var(--accent); }
/* CSS chevron (rotates when open) */
.xselect-btn .arrow { width: 8px; height: 8px; flex: none;
  border-right: 1.5px solid var(--accent); border-bottom: 1.5px solid var(--accent);
  transform: rotate(45deg) translate(-1px,-1px); transition: transform .18s ease; }
.xselect[data-open=true] .xselect-btn { border-color: var(--accent); }
.xselect[data-open=true] .arrow { transform: rotate(-135deg) translate(-2px,-2px); }
/* Popup list — themed like a panel: sharp, luminous border, glow */
.xselect-list {
  position: absolute; top: calc(100% + 4px); left: 0; right: 0; z-index: 20;
  margin: 0; padding: 2px; list-style: none; max-height: 240px; overflow: auto;
  background: var(--panel); border: 1px solid var(--accent-border);
  box-shadow: 0 0 16px -2px var(--accent-glow),
              inset 0 0 0 1px color-mix(in srgb, var(--accent) 5%, transparent);
  display: none;
}
.xselect[data-open=true] .xselect-list { display: block; }
.xselect-opt { display: flex; align-items: center; gap: .55rem;
  padding: .55rem .7rem; font-size: .92rem; color: var(--ink); cursor: pointer; }
.xselect-opt:hover, .xselect-opt[data-active=true] { background: var(--accent-soft); color: var(--accent); }
.xselect-opt[aria-selected=true] { color: var(--accent); }
.xselect-opt .tick { width: 14px; flex: none; color: var(--accent); font-family: var(--display); }
.xselect-opt[aria-selected=false] .tick { visibility: hidden; } /* reserve space, hide glyph */
```
```js
// Vanilla controller (one per .xselect). React: same markup/CSS, drive open/selected/
// active with useState and translate these handlers to props — see references/react.md.
function initSelect(root) {
  const btn = root.querySelector('.xselect-btn'),
        list = root.querySelector('.xselect-list'),
        val = btn.querySelector('.val'),
        hidden = root.querySelector('input[type=hidden]'),
        opts = [...list.querySelectorAll('[role=option]')],
        label = o => o.querySelector('.txt').textContent;
  let active = Math.max(0, opts.findIndex(o => o.getAttribute('aria-selected') === 'true'));
  const open = o => { root.dataset.open = o; btn.setAttribute('aria-expanded', o); if (o) { paint(); list.focus(); } };
  const paint = () => { opts.forEach((o,i) => o.dataset.active = (i===active));
    const a = opts[active]; list.setAttribute('aria-activedescendant', a.id); a.scrollIntoView({block:'nearest'}); };
  const choose = i => { opts.forEach(o => o.setAttribute('aria-selected','false'));
    opts[i].setAttribute('aria-selected','true');
    val.textContent = label(opts[i]); if (hidden) hidden.value = label(opts[i]); // button = label only, no tick
    active = i; open(false); btn.focus(); };
  btn.addEventListener('click', () => open(root.dataset.open !== 'true'));
  opts.forEach((o,i) => o.addEventListener('click', () => choose(i)));
  btn.addEventListener('keydown', e => { if (['ArrowDown','ArrowUp','Enter',' '].includes(e.key)) { e.preventDefault(); open(true); } });
  list.addEventListener('keydown', e => {
    if (e.key === 'Escape') { open(false); btn.focus(); }
    else if (e.key === 'ArrowDown') { e.preventDefault(); active = Math.min(active+1, opts.length-1); paint(); }
    else if (e.key === 'ArrowUp')   { e.preventDefault(); active = Math.max(active-1, 0); paint(); }
    else if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); choose(active); }
    else if (e.key === 'Home') { e.preventDefault(); active = 0; paint(); }
    else if (e.key === 'End')  { e.preventDefault(); active = opts.length-1; paint(); }
  });
  document.addEventListener('click', e => { if (!root.contains(e.target)) open(false); });
}
document.querySelectorAll('.xselect').forEach(initSelect);
```

Behaviors this covers: click/keyboard open, arrow-key roving highlight
(`aria-activedescendant`), Enter/Space to choose, Escape to close, click-outside to
close, and a hidden input for form submission. Screen readers announce it as a combobox
+ listbox with the selected option.

**Static-HTML landing mode caveat:** that mode is zero-JS, so a dropdown there must use a
native `<select>` (accept the OS popup) — or avoid `<select>` entirely. The custom listbox
is for the form/dashboard/control-center modes (which already allow JS).

## Toggle switch (checkbox-driven, CSS-only)

```html
<label class="switch">
  <input type="checkbox" class="switch-input">
  <span class="switch-track"><span class="switch-thumb"></span></span>
  <span class="switch-label">Shields</span>
</label>
```
```css
.switch { display: inline-flex; align-items: center; gap: .7rem; cursor: pointer; }
.switch-input { position: absolute; opacity: 0; width: 0; height: 0; }
.switch-track {
  width: 44px; height: 24px; border-radius: 999px; position: relative;
  background: var(--bg); border: 1px solid var(--line); transition: border-color .2s, background .2s;
}
.switch-thumb {
  position: absolute; top: 2px; left: 2px; width: 18px; height: 18px; border-radius: 50%;
  background: var(--dim); transition: transform .2s, background .2s;
}
.switch-input:checked + .switch-track { background: var(--accent-soft); border-color: var(--accent); }
.switch-input:checked + .switch-track .switch-thumb {
  transform: translateX(20px); background: var(--accent); box-shadow: 0 0 10px var(--accent-glow);
}
.switch-input:focus-visible + .switch-track { outline: 2px solid var(--accent-ring); outline-offset: 2px; }
```

## Checkbox / radio (custom, accent-checked)

```css
.check-input { appearance: none; width: 20px; height: 20px; border: 1px solid var(--line);
  background: var(--bg); border-radius: 0; display: grid; place-content: center; cursor: pointer; }
.check-input[type=radio] { border-radius: 50%; }
.check-input:checked { border-color: var(--accent); background: var(--accent-soft); }
.check-input:checked::after { content: ''; width: 10px; height: 10px; border-radius: inherit;
  background: var(--accent); box-shadow: 0 0 8px var(--accent-glow); }
.check-input:focus-visible { outline: 2px solid var(--accent-ring); outline-offset: 2px; }
```

## Range slider

```css
input[type=range] { width: 100%; accent-color: var(--accent); cursor: pointer; }
/* For full control, style ::-webkit-slider-thumb / ::-moz-range-thumb with var(--accent)
   and a var(--accent-glow) box-shadow; track in var(--line). */
```

## Buttons

```css
.btn { display: inline-flex; align-items: center; justify-content: center; gap: .5rem;
  min-height: 44px; padding: .75rem 1.1rem; border-radius: 0; cursor: pointer;
  font-family: var(--display); font-size: .78rem; letter-spacing: .14em; text-transform: uppercase;
  border: 1px solid transparent; }
.btn-primary { color: var(--on-accent); background: var(--accent); border-color: var(--accent);
  box-shadow: 0 0 24px var(--accent-glow); }
.btn-primary:hover { background: var(--accent-bright); }
.btn-ghost { color: var(--ink); background: var(--accent-fill); border-color: var(--line); }
.btn-ghost:hover { color: var(--accent); border-color: var(--accent-ring); background: var(--accent-soft); }
.btn:focus-visible { outline: 2px solid var(--accent-ring); outline-offset: 3px; }
.btn:disabled { opacity: .4; cursor: not-allowed; }
```

## Validation & errors

- Error text: `.field-error { color: var(--crit); font-size: .82rem; }` and hide when empty
  (`.field-error:empty { display: none; }`). Place it BELOW the control with positive
  margin — never a negative margin that can overlap the control/buttons above it.
- Announce errors to AT: `role="alert"` (or `aria-live="polite"`) on the error node;
  set `aria-invalid="true"` on the control when invalid.
- HTML: validate with a tiny vanilla-JS handler on submit/blur. React: controlled inputs
  + state; see `references/react.md`.

## Multi-step forms

Show a stepper (numbered chips in the display font, active in `--accent`, done filled).
One `<fieldset>` per step with a `<legend>`. Keep a single `<h1>`; steps use `<h2>`.

## Checklist (forms)

- Every control has an associated `<label>` (`for`/`id` or wrapping).
- Focus rings on all controls; 44px min target; disabled states styled.
- Errors announced (`role="alert"`), placed below with non-overlapping spacing.
- Placeholder is not a substitute for a label. Contrast AA on all text.
