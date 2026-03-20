# Policy Owner POC — Project Context

## What this is
A static HTML/CSS/JS SPA (no framework) that implements a 7-step "Intended Policy Owner" onboarding flow for Ethos insurance agents. Pixel-perfect from Figma designs.

## File structure
```
policy-owner.html          ← SPA shell (sidebar + progress bar + step 1 content + ALL CSS/JS)
po-email.html              ← Step 2: Email input
po-form.html               ← Step 3: Personal details form
po-loading.html            ← Step 4: Loading/processing animation
policy-owner-confirm.html  ← Step 5: Review & confirm details
po-mobile.html             ← Step 6: Select mobile number
po-otp.html                ← Step 7: OTP verification
```

## Architecture — SPA navigation
- `policy-owner.html` is the shell. It has a fixed sidebar, a persistent `#global-progress` bar, and a `.content-stack` div where page content is swapped in.
- Navigation uses `navigateTo(filename)` which `fetch()`es the target file, extracts its `.content-stack` innerHTML (or falls back to `doc.body.innerHTML` for partial files), fades out/in, and calls the appropriate init function.
- Steps 2–7 are **partial HTML files** — just the inner content, no `<html>`/`<head>`/`<body>` wrapper.
- `policy-owner-confirm.html` (step 5) is a **full HTML file** with its own `<html>` wrapper and stylesheet — the SPA extracts its `.content-stack`.

## Progress bar
- `#global-progress` lives outside `.content-stack` so it's never hidden by the fade transition.
- Widths: step 1 = 0% (hidden), step 2 = 14%, step 3 = 28%, step 4 = 43%, step 5 = 57%, step 6 = 71%, step 7 = 100%.
- On step 1, the track is hidden (`visibility: hidden`).
- Animated via `setProgress(width, animate)` using double-rAF.

## Per-step init functions
| File | Init function |
|------|--------------|
| policy-owner.html | `initStep1()` |
| po-email.html | `initStep2()` |
| po-form.html | `initFormPage()` |
| po-loading.html | `initStep4()` |
| policy-owner-confirm.html | `initConfirmPage()` |
| po-mobile.html | `initStep6()` |
| po-otp.html | `initStep7()` |

## Form data persistence
- `initFormPage()` saves form data to `sessionStorage` as `policyOwnerData` (JSON) on every input change.
- When navigating back to the form, `initFormPage()` restores values from `sessionStorage` including custom `<select>` dropdowns (matched by `data-value` or visible text).

## Icons — Figma SVG exports
All icons use the Figma nested inset structure:
```html
<div style="width:24px;height:24px;overflow:hidden;position:relative;flex-shrink:0;">
  <div style="position:absolute;inset:16.67%;">  <!-- inset varies per icon -->
    <img alt="" style="position:absolute;display:block;max-width:none;width:100%;height:100%;"
         src="https://www.figma.com/api/mcp/asset/ASSET_ID"/>
  </div>
</div>
```
> ⚠️ Figma asset URLs expire after 7 days. Re-fetch via `get_design_context` MCP tool.

## Figma node IDs (for re-fetching designs)
| Step | Description | Figma Node ID |
|------|-------------|---------------|
| Step 1 | Pledge / Get Started | (inline in shell — check policy-owner.html) |
| Step 2 | Email input | `139-1615` |
| Step 3 | Personal details form | `103-3019` |
| Step 4 | Loading screen | (check po-loading.html) |
| Step 5 | Confirm details | `113-1281` |
| Step 6 | Select mobile | `139-2371` |
| Step 7 | OTP verification | `139-4037` |

## Key CSS rules to know
```css
/* Card uses flex column so gap works */
.card { display: flex; flex-direction: column; }

/* Progress bar */
.progress-fill { transition: width 0.55s cubic-bezier(0.4, 0, 0.2, 1); }

/* Pledge icons (step 1) */
.pledge-icon { width: 24px; height: 24px; flex-shrink: 0; position: relative; overflow: hidden; }
.pledge-icon-inner { position: absolute; display: flex; }
.pledge-icon-inner img { position: absolute; inset: 0; width: 100%; height: 100%; }

/* Back button — no underline */
.back-btn { text-decoration: none; display: flex; align-items: center; gap: 6px; }

/* OTP boxes — flex:1 with min-width:0 to prevent overflow */
.otp-box { flex: 1 0 0; min-width: 0; }
```

## Custom select dropdowns
- Sex and Birth State use custom `<div>`-based dropdowns (not native `<select>`) because of styling requirements.
- They set `data-value` on the trigger element when a choice is made.
- Restoration from sessionStorage matches by `data-value` or inner text.

## Navigation flow
```
Step 1 (Pledge) → Step 2 (Email) → Step 3 (Form) → Step 4 (Loading, auto-advances) → Step 5 (Confirm) → Step 6 (Mobile) → Step 7 (OTP)
```
Back buttons reverse the flow. Step 4 auto-advances to Step 5 after ~3s animation.

## To run locally
```bash
cd /Users/pranjal.jain/modal-preview
python3 -m http.server 8080
# open http://localhost:8080/policy-owner.html
```

## Pending / known issues (as of last session)
- [ ] Icons on steps 2–7: SVG asset URLs need re-fetching (expire every 7 days) and some icons may not be using the proper Figma inset wrapper structure
- [ ] Step 7 OTP boxes: need `min-width: 0` on `.otp-box` to prevent overflow
- [ ] Step 1 progress bar track: should be hidden (not just 0% width)

---

# Design System Rules

## Color Tokens
Never hardcode one-off hex values. Always use these project tokens:

| Token name | Value | Usage |
|---|---|---|
| `--color-brand` | `#056257` | Primary CTA, active states, focus rings, progress bar |
| `--color-brand-tint` | `#f3f7f7` | Hover bg on dropdown options, selected bg |
| `--color-brand-ring` | `rgba(5,98,87,0.12)` | Focus ring on inputs / dropdowns |
| `--color-text-primary` | `#272727` | Headings, input values, primary labels |
| `--color-text-secondary` | `#525252` | Field labels, body text, agent role |
| `--color-text-muted` | `#7e7e7e` | Placeholders, inactive stepper labels |
| `--color-border` | `#d4d4d4` | Input borders, step connectors, progress track |
| `--color-bg-page` | `#f4f4f4` | Page background, disabled input bg |
| `--color-bg-card` | `#fff` | Cards, inputs, dropdowns |
| `--color-error` | `#d92d20` | Validation error borders and messages |
| `--color-error-ring` | `rgba(217,45,32,0.12)` | Error focus ring |
| `--color-info-text` | `#336cc3` | Info bar text and links |
| `--color-info-bg` | `#f5f8fc` | Info bar background |
| `--color-info-border` | `#cfdef5` | Info bar border |
| `--color-info-icon-bg` | `#ccdaf0` | Info bar icon container bg |
| `--color-sidebar-gradient` | `linear-gradient(-70.43deg, #e6f5ec 0%, #ebf0f9 100%)` | Sidebar background |
| `--color-shadow-base` | `rgba(16,24,40,…)` | All box shadows use #101828 with varying opacity |

## Typography Scale
Font: **Theinhardt** (local OTF files in `Theinhardt/`). Fallback: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif`

| Role | Size | Weight | Line-height | Letter-spacing |
|---|---|---|---|---|
| Display heading | 28px | 500 | 40px | -0.28px |
| Card title | 24px | 500 | 32px | -0.24px |
| Section title | 20px | 500 | 28px | — |
| Back button / large body | 18px | 500 | 26px | — |
| Body / input text / label | 16px | 400/500 | 24px | — |
| Small label / stepper | 14px | 500/700 | 20px | -0.14px |
| Caption / error msg | 13px | 400 | 18–20px | — |

- **IMPORTANT**: Never use font sizes or weights outside this scale.
- Field labels → 16px weight 500 `#525252`
- Input values → 16px weight 400 `#272727`
- Placeholder text → 16px weight 400 `#7e7e7e`

## Spacing Scale
Base unit: **4px**. Only use multiples: `4, 8, 12, 16, 24, 32, 48, 56px`.

## Elevation / Shadows
```css
/* Card */
box-shadow: 0px 1px 2px rgba(16,24,40,0.06), 0px 1px 3px rgba(16,24,40,0.1);
/* Input / trigger */
box-shadow: 0px 1px 2px rgba(16,24,40,0.05);
/* Dropdown panel */
box-shadow: 0px 4px 16px rgba(16,24,40,0.12);
/* Sidebar */
box-shadow: 2px 0px 8px rgba(16,24,40,0.08);
/* Avatar */
box-shadow: 0px 1.33px 2.67px -2px rgba(16,24,40,0.06), 0px 2.67px 5.33px -2px rgba(16,24,40,0.1);
```

## Border Radius Scale
| Element | Radius |
|---|---|
| Card | 16px |
| Button, input, dropdown, info bar, step icon | 8px |
| Step icon | 10px |
| Progress bar | 4px |
| Avatar (circle) | 200px |

## Component Patterns

### Text Input
```html
<input class="text-input" type="text" placeholder="…" />
```
- Height: 56px, padding: 16px 14px, border: 1px solid #d4d4d4, border-radius: 8px
- Focus: border-color #056257 + box-shadow ring rgba(5,98,87,0.12)
- Error (use inline style, not class, to avoid specificity issues):
  ```js
  el.style.borderColor = '#d92d20';
  el.style.boxShadow = '0 0 0 4px rgba(217,45,32,0.12)';
  ```

### Custom Select Dropdown
```html
<div class="custom-select" id="my-select" tabindex="0" role="combobox">
  <div class="custom-select-trigger">
    <span class="select-value">Option name</span>
    <div class="chevron-wrap"><div class="chevron-inner"><img src="icons/8cf6f3a7-f783-429a-92ce-595723750830.svg" alt=""/></div></div>
  </div>
  <div class="custom-select-dropdown" id="my-options">
    <!-- populated by JS using US_STATES or similar -->
  </div>
</div>
```
- Options display as: `Name (ABBR)` e.g. `Arizona (AZ)`
- Selected option: background `#f3f7f7`, color `#056257`, weight 500
- Chevron icon: always `icons/8cf6f3a7-f783-429a-92ce-595723750830.svg`
- Open state adds `.open` class; `overflow: visible` on open to avoid clipping

### Primary Button
```html
<button class="btn-continue" id="btn-step-continue">Continue</button>
```
- Full width, height 60px, bg `#272727`, color `#fff`, border-radius 8px
- Font: 20px weight 500
- Hover: bg `#3a3a3a`
- Disabled: add class `btn-disabled` (opacity 0.4, cursor not-allowed)

### Card
```html
<div class="card"> … </div>
```
- bg `#fff`, border-radius 16px, padding 24px, shadow (see Elevation above)

### Validation Error Pattern
- Use **inline styles** (not CSS classes) for error borders — avoids specificity conflicts
- Always append a `<p class="field-error-msg error-msg">` sibling after the input
- Clear error on next `input` event using `{ once: true }` listener

### Icon (Figma nested inset structure)
```html
<div style="width:24px;height:24px;overflow:hidden;position:relative;flex-shrink:0;">
  <div style="position:absolute;inset:16.67%;">
    <img alt="" style="position:absolute;display:block;max-width:none;width:100%;height:100%;"
         src="icons/ASSET_ID.svg"/>
  </div>
</div>
```
- ⚠️ Figma MCP asset URLs expire after 7 days. Re-fetch via `get_design_context` and save locally to `icons/`.
- IMPORTANT: Never use `<img src="https://www.figma.com/api/…">` directly — always download and reference from `icons/`.
- Inset percentage varies per icon (16.67% for 24px icons, 8.33% for 20px icons, 12.5% horizontal for calendar).

## Figma MCP Integration Workflow
**Follow this order every time you implement from Figma:**

1. `get_design_context` — fetch structured representation + screenshot for the node
2. If response too large → `get_metadata` first, then re-fetch specific child nodes
3. `get_screenshot` — visual reference for pixel comparison
4. Download any new icon/image assets to `icons/` directory
5. Translate the Figma output to this project's plain HTML/CSS/JS conventions (not React/Tailwind)
6. All CSS goes into `policy-owner.html` `<style>` block (the shell stylesheet)
7. Validate final UI against Figma screenshot before marking complete

### Translation rules (Figma → this project)
- Figma React + Tailwind output → convert to semantic HTML + the CSS classes above
- Figma color references → use the token table above
- `font-family: Inter` → replace with `font-family: 'Theinhardt', …`
- Figma `gap`, `padding`, `border-radius` → verify against spacing/radius scale before writing
- New step page files → partial HTML (no `<html>/<head>/<body>`) except step 5 which is full HTML
- Always add `<script>if(typeof navigateTo==='undefined'){window.location.replace('policy-owner.html');}</script>` at top of every partial file

## Asset Handling
- IMPORTANT: If Figma MCP returns a localhost source for an asset, use it directly during development
- Save all SVG icons to `icons/` with the Figma asset UUID as filename
- DO NOT install icon libraries — all icons come from the Figma payload
- Images/blobs: save to project root or `icons/` as appropriate
