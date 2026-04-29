# Darshan — High Level Design Document

## What This Is
A mobile-first single-page web experience designed as a personal sacred gift. It simulates the act of **digital darshan** — the experience of approaching a temple, opening its doors, and being welcomed into the presence of a deity. Built for one person, from one person, with love.

---

## The Experience Flow

**Stage 1 — Arrival**
The user lands on a dark, warm screen. A temple door greets them — arched at the top like a mandir entrance, deep red and gold, with an ॐ symbol. Ambient golden particles float upward in the background. A gentle pulsing hint invites them to tap.

**Stage 2 — Opening**
The user taps the door. Both panels swing outward from the centre simultaneously, like true temple doors (mandir dwaar). As this happens, a sound sequence plays: a conch (shankh) first, then firecrackers and pooja bells layered together as the conch fades.

**Stage 3 — Darshan**
The doors reveal what is inside. An image — of Ganesh Ji — sits in a warm golden frame with a marigold garland border. Background music (a bhajan or devotional track) fades in and loops continuously. A mute button appears for the user's control.

**Stage 4 — Presence**
A constant shimmer stays alive over the image — golden light rays slowly sweeping, with soft sparkle dots twinkling. This never stops. It feels like candlelight or diya-glow catching the deity's ornaments. The blessing text "गणपति बप्पा मोरया 🙏" fades in beneath the frame.

---

## The Elements

### Visuals
- **Temple Door** — Two-panel arched door in deep red and gold. Simple version now; designed to be swappable with an ornate carved version later
- **ॐ Symbol** — Sits at the crown of the arch, softly pulsing
- **Lotus Motifs** — Decorative elements on each door panel
- **Inner Sanctum** — Dark warm background that the door reveals
- **Image Frame** — Gold-bordered frame with marigold garland effect, holds the Ganesh Ji image
- **Shimmer Overlay** — Lives permanently in front of the image; golden light rays + sparkle dots
- **Ambient Particles** — Floating warm dots in the background, like embers or diya light
- **Blessing Text** — Sanskrit text fades in after the reveal

### Audio
- **Conch (Shankh)** — Plays the moment the door is tapped. Fades in, plays, fades out
- **Firecrackers** — Begins near the tail of the conch. Fades in and out
- **Pooja Bells** — Plays simultaneously with firecrackers. Fades in and out
- **Background Music** — Starts after the full reveal. Loops forever. Fades in on start, fades out only when the user leaves the page
- **Mute Toggle** — A single button, visible only after reveal, to silence/restore all audio

### Interaction
- The only interaction is the door tap. Everything else is automatic and continuous
- Touch and click both work
- The door can only be opened once per visit

---

## Colour & Mood
Saffron, deep marigold gold, vermillion red, maroon, and warm ivory. The palette of a Ganesh puja. Dark background to make the gold glow feel alive.

---

## Files Needed to Complete It

| File | Purpose |
|---|---|
| `darshan.html` | The entire application — one self-contained file |
| Your Ganesh Ji image | The image displayed inside after the doors open |
| `conch.mp3` | Shankh sound |
| `firecrackers.mp3` | Firecracker sound |
| `pooja-bells.mp3` | Pooja bells sound |
| `bg-music.mp3` | Looping background bhajan |

All files sit in the same folder. The HTML references them by name.

---

## What Can Be Built On Top
- **Ornate door** — The simple door panels can be replaced with an SVG carved wooden door with detailed jali patterns, peacock motifs, or deity carvings
- **Personalised message** — A text or letter overlay that appears after the reveal, from the gifter to the recipient
- **Seasonal dressing** — Swap colours and decorations for specific occasions (Ganesh Chaturthi, birthdays, etc.)
- **Transition variations** — The door could dissolve, zoom, or fade rather than swing, depending on mood

---

## Design Philosophy
Every element serves the feeling of reverence and warmth. Nothing is decorative for its own sake — the shimmer feels like diya light, the particles like incense smoke, the sound sequence like a real puja beginning. The experience should feel like a gift, not a website.