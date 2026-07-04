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

## Core input / select / textarea

```css
.field { display: grid; gap: .5rem; margin-bottom: 1.1rem; }
.field-label {
  font-family: var(--display); font-size: 0.66rem; letter-spacing: .2em;
  text-transform: uppercase; color: var(--dim);
}
.input, .select, .textarea {
  width: 100%; min-height: 44px; padding: .6rem .75rem; border-radius: 0; /* sharp */
  font-family: var(--body); font-size: .95rem; color: var(--ink);
  background: var(--bg); border: 1px solid var(--line);
}
.input::placeholder { color: var(--dim); }
.input:hover, .select:hover, .textarea:hover { border-color: var(--accent-ring); }
.input:focus-visible, .select:focus-visible, .textarea:focus-visible {
  outline: 2px solid var(--accent-ring); outline-offset: 2px; border-color: var(--accent);
}
.input:disabled { opacity: .5; cursor: not-allowed; }
/* strip number spinners for a clean HUD look */
.input[type=number]::-webkit-inner-spin-button,
.input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; margin: 0; }
```

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
