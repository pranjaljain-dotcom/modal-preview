# /new-screen

Create a new screen for the Policy Owner flow.

## Usage
```
/new-screen <filename> <screen-purpose> [input-type]
```

**Input types:** `email` | `otp` | `form` | `confirm` | `loading` | `landing`

## What this does
1. Creates `{filename}.html` as a **standalone full HTML page** (not a partial — this is a multi-page app)
2. Includes Theinhardt font declarations, Ethos DS CSS inline
3. Adds the correct input pattern for the screen type
4. Links to the next screen in the flow via `window.location.href`

## Template (landing type)
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1.0"/>
  <title>Policy Owner — {PURPOSE}</title>
  <style>
    @font-face { font-family:'Theinhardt'; src:url('Theinhardt/Theinhardt-Regular.otf') format('opentype'); font-weight:400; }
    /* … full Ethos DS CSS … */
  </style>
</head>
<body>
  <header class="top-bar">…</header>
  <main class="main">
    <div class="card">
      <h1 class="q-heading">{HEADING}</h1>
      <p class="q-desc">{DESCRIPTION}</p>
      <!-- input or CTA -->
    </div>
  </main>
</body>
</html>
```

## OTP Screen Rules
- 6 individual `<input maxlength="1">` boxes in a flex row
- Auto-advance focus on input, auto-back on backspace
- Paste handler splits clipboard value across boxes
- Submit enabled only when all 6 filled

## Loading Screen Rules
- No user input
- Animated progress or spinner using `#056257`
- Auto-advance to next screen after animation completes
- Use `@keyframes` — no JS animation libraries
