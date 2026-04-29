# Darshan — AI Agent Instructions

## Purpose of This File
This file is the primary reference for any AI agent working on this project. Read this before making any change. After every change, update this file and the LLD if the change affects structure, behaviour, audio logic, styling architecture, or file references.

---

## Project Identity
A single-file mobile web experience. A sacred gift. The user approaches a temple door, taps it, the two panels swing open, a sound sequence plays, and a deity image is revealed with a permanent shimmer effect. Every decision — visual, audio, interaction — serves the feeling of reverence and warmth. Do not introduce elements that feel modern, techy, or decorative without purpose.

---

## The One File
**`darshan.html`** contains everything — HTML structure, all CSS, all JavaScript. There are no external JS or CSS files. Do not split this into multiple files unless explicitly instructed. Do not introduce a build system, bundler, or package manager.

The only external resource is a Google Fonts `<link>` in `<head>`. This can be removed for offline use by embedding the fonts as base64 or removing them entirely.

---

## Folder Layout
```
darshan/
├── darshan.html           ← the entire application
├── ganesh.jpg             ← deity image (name must match src in #ganesh-image)
├── conch.mp3
├── firecrackers.mp3
├── pooja-bells.mp3
└── bg-music.mp3
```
All asset filenames are configurable inside the `AUDIO_FILES` object in the JS and the `src` attribute of `#ganesh-image`. If filenames change, update both the file on disk and the reference inside the HTML.

---

## Semantic Structure — Every Element Explained

### `#loading`
Full-screen dark overlay showing a pulsing ॐ symbol. Exists to mask font/resource load jitter. Fades out 600ms after `window.load`. Has no interactivity. Remove or restyle only if the loading experience needs to change.

### `#scene`
The root visual container. `position: fixed; inset: 0`. Everything visible lives inside this. Do not change its positioning model — it anchors all child absolute/fixed elements.

### `#bg-glow`
A pure CSS radial gradient layer — dark brown at edges, slightly lighter brown at centre. Creates the warm, dim room feeling before the door opens. Lives behind everything (z-index: 0). Can be changed in colour but should remain dark and warm.

### `#ambient`
An empty container at init. JavaScript injects 18 `div.diya-particle` elements into it on page load. These float upward continuously using `@keyframes floatUp`. They represent diya embers or incense smoke. Do not add interactive logic here.

### `#door-frame`
The most important structural element. It is the sized, positioned wrapper for the entire door experience. It receives two state classes:
- `.open` — triggers the door swing CSS transition
- `.revealed` — triggers all inner reveal transitions (image, shimmer, blessing text)

Its dimensions (`min(88vw, 340px)` wide, `clamp(300px, 75vh, 560px)` tall) control the entire door size responsively. Change these only if resizing the door for different screens.

### `#inner-sanctum`
Sits inside `#door-frame` but behind the door panels (z-index: 5, panels are z-index via stacking context). Always rendered — the door swing animation reveals it visually by rotating the panels away. Contains the image frame and is styled as a dark warm chamber.

### `#image-frame`
The gold-bordered container for the deity image. Has a marigold garland border effect via `::before` pseudo-element using `repeating-conic-gradient`. Starts `opacity: 0; transform: scale(0.92)` — transitions to full visibility when `#door-frame.revealed` class is applied. Aspect ratio is 3:4 (portrait). Change this if the image is landscape.

### `#ganesh-image`
The `<img>` tag. Its `src` is the only thing to change when swapping the image. Has an `onerror` handler that hides it if the file is missing, falling back to `#image-placeholder`. Do not remove the onerror handler.

### `#image-placeholder`
Visible when `#ganesh-image` fails to load or has no src. Shows a lotus emoji and instructional text. Hidden automatically when the image loads. Remove or restyle freely — it is a dev convenience only.

### `#shimmer-overlay`
A full-inset absolute div that lives inside `#image-frame`, in front of the image (z-index: 30). Contains the 4 `.shimmer-ray` divs and the 14 JS-injected `.sparkle` divs. Starts `opacity: 0`, transitions to `opacity: 1` when `.revealed` is applied (1s delay). Once visible, all its children animate forever. Do not add `pointer-events` to this element — it must stay transparent to touch.

### `.shimmer-ray` (×4)
Tall narrow divs with a vertical gold gradient. Each has a unique `left` position, rotation angle, animation duration, and delay — creating an asynchronous sweeping light effect. Driven entirely by `@keyframes raySweep`. Do not synchronise their timings — the stagger is intentional.

### `.sparkle` (×14)
Injected by `createSparkles()` in JS. Small circles with random positions, sizes, durations, and delays. Driven by `@keyframes sparkleTwinkle` (scale + opacity pulse). Represent light glinting off the idol's ornaments.

### `#door-arch`
The visible arched shell of the door. `border-radius: 50% 50% 0 0 / 25% 25% 0 0` creates the arch shape. Contains `#om-symbol` and `#doors-container`. Has `overflow: hidden` — this clips the door panels to the arch shape during the swing. Do not remove `overflow: hidden`.

### `#om-symbol`
A text node containing `ॐ`. Uses `@keyframes omPulse` to gently glow. Positioned at the top centre of the arch. Uses `pointer-events: none` so it does not intercept taps meant for the door.

### `#doors-container`
A flex container holding both panels. `perspective: 900px` is set here — this is what enables 3D rotation of child panels. Do not move `perspective` to a parent element — it must be on the direct parent of the rotating elements.

### `.door-panel.left` and `.door-panel.right`
The two door halves. Each has:
- `transform-origin: left center` (left panel) or `right center` (right panel)
- `transition: transform 1.4s cubic-bezier(0.76, 0, 0.24, 1)`
- A wood-grain texture via `repeating-linear-gradient` in `::before`
- Gold panel insets (`.panel-inset.top`, `.panel-inset.bottom`)
- A door knob (`.door-knob`)
- A lotus motif (`.door-lotus`)

On `.open`, left rotates to `rotateY(105deg)`, right to `rotateY(-105deg)`. The 105° (not 90°) ensures the panels appear to fully clear the opening. Click and touchstart listeners are attached to both panels and to `#door-arch`.

**To swap to an ornate door:** Replace the CSS and inner child elements of these two divs. The `transform-origin`, `transition`, and open-state `rotateY` values must be preserved.

### `#tap-hint`
Italic text below the door frame. Pulses via `@keyframes hintPulse`. Hidden with `display: none` when `.open` is applied. Lives outside `#door-arch` so it is not clipped by the arch's overflow.

### `#blessing-text`
Sanskrit text `गणपति बप्पा मोरया 🙏`. Starts `opacity: 0`. Fades in with `transition: opacity 1.5s ease 1.8s` when `.revealed` is applied. The 1.8s delay staggers it after the image reveal. Lives outside `#door-arch`.

### `#mute-btn`
Fixed position, bottom-right. Starts `opacity: 0`. Gets class `.visible` (opacity: 1) when `.revealed` is applied. Toggles `isMuted` boolean and ramps `bgGainNode.gain` smoothly. Only controls background music gain — one-shot sounds (conch, bells, firecrackers) are not affected after they finish.

---

## JavaScript — Every Function and Variable

### Configuration Block
```js
const AUDIO_FILES = {
  conch:        'conch.mp3',
  firecrackers: 'firecrackers.mp3',
  bells:        'pooja-bells.mp3',
  bgMusic:      'bg-music.mp3'
}
```
This is the only place audio filenames are defined. Change filenames here, not elsewhere.

### Global State Variables
```js
audioCtx        — the Web Audio API context. Null until first tap.
bgMusicSource   — the BufferSourceNode for looping bg music. Used for stop if needed.
bgGainNode      — the GainNode for bg music. Used for mute/unmute and page-unload fade.
isMuted         — boolean. Tracks mute state.
audioBuffers    — cache object. Decoded AudioBuffers keyed by name (conch, firecrackers, etc.)
doorOpened      — boolean. Guards against double-firing the open sequence.
```

### `createAmbientParticles()`
Runs once at init. Injects 18 `.diya-particle` divs into `#ambient`. All animation is handled by CSS after injection. No return value. No dependencies.

### `createSparkles()`
Runs once at init. Injects 14 `.sparkle` divs into `#shimmer-overlay`. Randomises position, size, duration, delay per sparkle. CSS handles animation after injection. No return value. No dependencies.

### `initAudio()`
Creates `AudioContext` if it does not already exist. Called at the start of `openDoor()`. Must be called inside a user gesture (the tap handler) — never on page load.

### `loadBuffer(key)`
Async. Fetches and decodes one audio file. Caches result in `audioBuffers`. Returns `null` on failure without throwing — all callers must handle null. The key must match a key in `AUDIO_FILES`.

### `playBuffer(buffer, options)`
The single playback function for all audio. Never call the Web Audio API directly for playback elsewhere — route everything through this function.

Parameters:
```
buffer      — decoded AudioBuffer or null (null = silent no-op)
loop        — boolean (default false)
fadeIn      — seconds for gain ramp up (default 0.5)
fadeOut     — seconds for gain ramp down before end (default 0.5, ignored if loop=true)
duration    — explicit playback length in seconds (default = buffer.duration)
startDelay  — seconds from now to begin (default 0)
```
Returns `{ source, gainNode }` or `null` if buffer is null. The caller must store the return value if they need to control the sound later (mute, stop). Internally creates a fresh `BufferSourceNode` and `GainNode` per call — Web Audio nodes are single-use by design.

### `openDoor()`
The master sequence function. Called by tap/click on door panels.

Order of operations:
1. Guard check — exits if `doorOpened` is true, then sets it to true
2. `initAudio()` and `audioCtx.resume()` for iOS unlock
3. Add `.open` to `#door-frame` — triggers CSS door swing
4. `Promise.all` — loads all 4 audio buffers in parallel
5. Play conch immediately (t=0)
6. Calculate `overlapAt = conchDuration - 1.2s` (minimum 1.5s)
7. Play firecrackers and bells at `startDelay: overlapAt`
8. `setTimeout(1500ms)` — adds `.revealed` to `#door-frame`, shows mute button
9. Play bg music at `startDelay: 2.5s` with `loop: true, fadeIn: 2.0s`
10. Store bg music `{ source, gainNode }` in globals

Do not reorder steps 3–9. The timing is intentional and interdependent.

### Mute Toggle (inline on `#mute-btn` click)
Flips `isMuted`. Ramps `bgGainNode.gain` over 0.5s. Updates button emoji. Does not affect in-progress one-shot sounds.

### Page Unload Handler
`beforeunload` event. Ramps `bgGainNode` to 0 over 1.5s. Prevents abrupt audio cutoff. Only fires if `bgGainNode` exists.

### Init Sequence (bottom of script, runs on parse)
```
createAmbientParticles()
createSparkles()
window.load → setTimeout 600ms → hide #loading
```

---

## Audio Timing Reference
```
t = 0s        Conch starts (fadeIn 0.6s)
t = X - 1.2s  Firecrackers start (X = conch file duration)
t = X - 1.2s  Pooja bells start (same moment)
t = 1.5s      .revealed class applied (CSS transitions begin)
t = 2.5s      Background music starts (fadeIn 2.0s)
t = ∞         Background music loops
```
X is dynamic — derived from the decoded conch buffer's `.duration` property. If the conch file is replaced with a shorter or longer file, the overlap timing adjusts automatically.

---

## CSS Class Transitions Reference
```
Class added          Element affected        What changes
─────────────────────────────────────────────────────────────────
.open                #door-frame             activates door swing
.open                .door-panel.left        rotateY(105deg)
.open                .door-panel.right       rotateY(-105deg)
.open                #tap-hint               display: none
.revealed            #image-frame            opacity 0→1, scale 0.92→1
.revealed            #shimmer-overlay        opacity 0→1 (delay 1s)
.revealed            #blessing-text          opacity 0→1 (delay 1.8s)
```
All transitions are defined in CSS. JavaScript only adds the classes and sets the setTimeout. Never use JS to animate properties that have CSS transitions defined on them — they will conflict.

---

## What to Preserve Under All Circumstances
These must never be changed without fully understanding the cascade of effects:

- `overflow: hidden` on `#door-arch` — removes this and panels will visually escape the arch during the swing
- `perspective: 900px` on `#doors-container` — move this and 3D rotation breaks
- `transform-origin` on each `.door-panel` — change these and the doors won't hinge correctly
- `pointer-events: none` on `#shimmer-overlay` — remove this and taps on the image area break
- `doorOpened` guard in `openDoor()` — remove this and audio plays twice on fast double-tap
- `onerror` on `#ganesh-image` — remove this and a broken image icon shows when file is missing
- `initAudio()` inside the tap handler — move this outside a user gesture and browsers will block audio

---

## How to Make Common Changes

**Swap the image**
Change `src="ganesh.jpg"` on `#ganesh-image` to your filename. Place file in the same folder.

**Swap an audio file**
Change the filename in the `AUDIO_FILES` object. Place the new file in the same folder.

**Change the blessing text**
Edit the text content of `#blessing-text` in the HTML.

**Change door colours**
Edit the `background` gradient on `.door-panel` and the `border` colour on `#door-arch`.

**Replace with ornate door**
Replace the CSS and child elements inside `.door-panel.left` and `.door-panel.right`. Keep `transform-origin`, `transition`, and the open-state `rotateY` values intact.

**Add a personalised message after reveal**
Add a new element inside `#inner-sanctum`, below `#image-frame`. Start it `opacity: 0` and add a transition triggered by `#door-frame.revealed`. Follow the same pattern as `#blessing-text`.

**Change shimmer intensity**
Adjust opacity values inside `@keyframes raySweep` and the `background` gradient of `.shimmer-ray`. Adjust sparkle count in `createSparkles()`.

**Make the door swing faster or slower**
Change the `transition` duration on `.door-panel` (currently `1.4s`). If making it significantly faster, also reduce the `setTimeout` in `openDoor()` accordingly so `.revealed` doesn't fire before the door finishes.

---

## Testing
You can use the browser tool to open the page and do any testing to make sure things are working as expected.

## Self-Maintenance Rule
After every change to this project — no matter how small — the agent must:

1. Re-read this instructions file and the LLD
2. Identify if the change affects any element, function, variable, timing, class, or file reference documented in either
3. If yes — update the relevant section(s) in this file and/or the LLD before ending the session
4. If a new element, function, or behaviour is introduced — add a full entry for it in this file under the appropriate section
5. If something documented here is removed — remove its entry from this file and the LLD
6. Never leave the documentation in a state that describes something that no longer exists, or omits something that does

The HTML file is the source of truth for implementation. This file is the source of truth for intent and understanding. They must stay in sync.