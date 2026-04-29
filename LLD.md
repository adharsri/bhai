# Darshan — Low Level Design Document

## File Structure
```
darshan/
├── darshan.html        # Entire application (single file)
├── ganesh.jpg          # Your image (name configurable in HTML)
├── conch.mp3
├── firecrackers.mp3
├── pooja-bells.mp3
└── bg-music.mp3
```
All assets are referenced by relative path. No build step, no server required — opens directly in a mobile browser.

---

## HTML Structure
```
<body>
  #loading                        — Full-screen ॐ splash, fades out on load
  #scene                          — Root container, fills viewport
    #bg-glow                      — Radial gradient background layer
    #ambient                      — Floating particle container (JS-populated)
    #door-frame                   — Positioned wrapper; receives .open and .revealed classes
      #inner-sanctum              — Behind the doors; always rendered, revealed by door swing
        #image-frame              — Gold-bordered frame
          <img#ganesh-image>      — The deity image (src = your file)
          #image-placeholder      — Shown when image src is empty
          #shimmer-overlay        — Permanently active shimmer; child of image-frame
            .shimmer-ray ×4       — CSS-animated gold light rays
            .sparkle ×14          — JS-injected twinkling dots
      #door-arch                  — The arched outer shell of the door
        #om-symbol                — ॐ text node at arch crown
        #doors-container          — Flex container; holds both panels; perspective set here
          .door-panel.left        — Left panel; transform-origin: left center
          .door-panel.right       — Right panel; transform-origin: right center
            .panel-inset.top      — Decorative gold rectangle (top)
            .panel-inset.bottom   — Decorative gold rectangle (bottom)
            .door-knob            — Radial-gradient circle
            .door-lotus           — Lotus emoji, low opacity
      #tap-hint                   — "Tap to open" — hidden once .open is added
      #blessing-text              — Sanskrit text — fades in on .revealed
  #mute-btn                       — Fixed bottom-right; visible only after reveal
```

---

## State Machine
The entire UI is driven by two CSS classes added to `#door-frame`:

```
[initial]
    │
    │  user taps door
    ▼
[.open]
    │  door CSS transition plays (1.4s)
    │  audio sequence begins
    │  setTimeout 1500ms
    ▼
[.open.revealed]
    │  image fades in
    │  shimmer activates
    │  blessing text fades in
    │  bg music starts
    │  mute button appears
    ▼
[stable — shimmer + music loop forever]
```

There is no "close" state. The door opens once per page load.

---

## CSS Architecture

### Layout
- `#scene` — `position: fixed; inset: 0` — fills full viewport always
- `#door-frame` — `width: min(88vw, 340px)` — scales on small phones, caps on larger screens
- `#door-frame` height — `clamp(300px, 75vh, 560px)` — responsive without overflow
- `#doors-container` — `display: flex; perspective: 900px` — enables 3D child transforms

### Door Swing
```css
.door-panel.left  { transform-origin: left center; }
.door-panel.right { transform-origin: right center; }

transition: transform 1.4s cubic-bezier(0.76, 0, 0.24, 1);

/* Open state */
.door-panel.left  → rotateY(105deg)
.door-panel.right → rotateY(-105deg)
```
The cubic-bezier gives a slow start, fast swing, slow settle — mimicking real door physics.

### Shimmer Rays
Four absolutely-positioned `div.shimmer-ray` elements, each:
- A tall narrow strip with a vertical linear-gradient (transparent → gold → transparent)
- Different `left` positions, rotation angles, animation durations and delays
- `@keyframes raySweep` — translates horizontally across the frame, opacity fades in/out
- Runs `infinite` — never stops

### Sparkles
14 `div.sparkle` elements, JS-injected into `#shimmer-overlay`:
- Random `top`, `left` positions (5%–95%)
- Random size (2–5px), duration (1.5–4.5s), delay (0–4s)
- `@keyframes sparkleTwinkle` — scale + opacity pulse
- Runs `infinite` — never stops

### Class-Driven Transitions
```css
/* Image reveal */
#image-frame { opacity: 0; transform: scale(0.92); transition: opacity 1.2s ease 0.3s, transform 1.2s ease 0.3s; }
#door-frame.revealed #image-frame { opacity: 1; transform: scale(1); }

/* Shimmer activation */
#shimmer-overlay { opacity: 0; transition: opacity 1s ease 1s; }
#door-frame.revealed #shimmer-overlay { opacity: 1; }

/* Blessing text */
#blessing-text { opacity: 0; transition: opacity 1.5s ease 1.8s; }
#door-frame.revealed #blessing-text { opacity: 1; }
```
All reveal animations are CSS transitions triggered by class addition — no JS animation loops.

---

## JavaScript Architecture

### Audio Engine
Uses **Web Audio API** throughout. No `<audio>` tags.

```
AudioContext
    └── per sound:
        BufferSourceNode  →  GainNode  →  audioCtx.destination
```

**`initAudio()`**
Creates the `AudioContext`. Called on first tap (required by browsers — AudioContext must be created inside a user gesture).

**`loadBuffer(key)`**
Fetches the audio file, decodes it to an `AudioBuffer`, caches it in `audioBuffers{}`. Fails gracefully — logs a warning if file is missing, returns `null`. Playback functions handle `null` safely.

**`playBuffer(buffer, options)`**
Single reusable function for all playback. Options:
```js
{
  loop: false,          // true for bg music
  fadeIn: 0.5,          // seconds
  fadeOut: 0.5,         // seconds (ignored if loop: true)
  duration: null,       // explicit stop time (uses buffer.duration if null)
  startDelay: 0         // seconds from now to start
}
```
Internally:
```
gainNode.gain.setValueAtTime(0, startTime)
gainNode.gain.linearRampToValueAtTime(1, startTime + fadeIn)
gainNode.gain.setValueAtTime(1, fadeStart)
gainNode.gain.linearRampToValueAtTime(0, endTime)
```
Returns `{ source, gainNode }` — caller holds reference for mute control.

### Audio Sequence in `openDoor()`
```
t=0s      Conch plays            (fadeIn 0.6s, fadeOut 0.8s)
t=Xs      Firecrackers start     where X = conchDuration - 1.2s  (overlap)
t=Xs      Pooja bells start      same moment as firecrackers
t=1.5s    .revealed class added  (setTimeout, independent of audio)
t=2.5s    BG music starts        (fadeIn 2.0s, loops forever)
```
`conchDuration` is read from the decoded buffer — adapts automatically to whatever conch file is provided.

### Mute Toggle
Holds a reference to `bgGainNode` (background music gain node). On toggle:
```js
bgGainNode.gain.linearRampToValueAtTime(isMuted ? 0 : 1, audioCtx.currentTime + 0.5)
```
A smooth 0.5s ramp — no click artifacts. The `isMuted` flag also gates the initial gain value in `playBuffer` so one-shot sounds respect mute state if user mutes before they complete.

### Page Unload Fade
```js
window.addEventListener('beforeunload', () => {
  bgGainNode.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 1.5)
})
```
Prevents abrupt music cutoff when navigating away.

### Ambient Particles
18 `div.diya-particle` elements created in JS at init:
- Random `left`, `bottom`, size, animation duration (6–16s), delay (0–10s)
- Pure CSS `@keyframes floatUp` after injection — no JS animation loop

---

## Key Constants & Thresholds

| Value | What it controls |
|---|---|
| `rotateY(±105deg)` | How wide the doors swing open |
| `perspective: 900px` | Depth of 3D effect on door swing |
| `1.4s cubic-bezier(0.76,0,0.24,1)` | Door swing duration and easing |
| `setTimeout 1500ms` | Delay before `.revealed` is applied |
| `conchDuration - 1.2s` | When firecrackers/bells overlap the conch |
| `startDelay: 2.5s` | When bg music begins after tap |
| `fadeIn: 2.0s` | Bg music fade-in duration |
| `min(88vw, 340px)` | Door width — responsive cap |
| `clamp(300px, 75vh, 560px)` | Door height — responsive range |
| 18 particles, 14 sparkles | Ambient density |

---

## Browser & Device Behaviour

| Concern | How it's handled |
|---|---|
| AudioContext autoplay policy | Created inside tap handler (user gesture) |
| iOS audio unlock | `audioCtx.resume()` called before playback |
| Missing audio files | `loadBuffer` catches fetch errors, returns null; `playBuffer` checks for null |
| Missing image | `onerror` hides `<img>`, placeholder div shows instead |
| Touch vs click | Both `touchstart` and `click` listeners on door panels |
| Double-tap zoom | `maximum-scale=1.0, user-scalable=no` in viewport meta |
| Tap highlight flash | `-webkit-tap-highlight-color: transparent` on `*` |
| Font load delay | Loading screen stays visible for 600ms after `window.load` |

---

## What "Swapping the Door" Means (Future)
The door visuals live entirely inside `.door-panel.left` and `.door-panel.right`. To replace with an ornate version:
- Replace the CSS background, panel-inset borders, and lotus elements inside those two divs
- Or inline an SVG inside each panel div
- The swing animation (`rotateY`) and all audio/reveal logic are completely untouched

---

## What Is NOT in This File
- No frameworks, no bundler, no npm
- No network calls except Google Fonts (one `<link>` in `<head>` — removable for offline use)
- No cookies, no localStorage, no tracking
- No server required — runs from the local filesystem