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
