# Cube Buddy

**This is an AI-generated project**

Live page: https://ravinsp.github.io/cube-buddy/

A Rubik's cube solver for 3×3, 4×4 and 5×5 cubes. It is a single web page: you point the phone's camera at each side of a mixed-up cube, and it shows how to solve it one turn at a time with an animated 3D cube and plain-language instructions.

Everything is in one file, [index.html](index.html). There is no build step and no server code.

## Features

- **Camera scanning.** A grid overlay shows the colours it sees live. A small 3D guide cube animates how to turn the real cube between photos. Snapping a side moves straight on to the next one, and ↩️ redoes the previous side.
- **Check before solving.** After all six sides, a draggable 3D copy of the scanned cube is shown. Any single side can be rescanned.
- **Step-by-step solving.**
  - Each turn is animated on a loop, with a purple arrow on the layer to turn.
  - Instructions use plain words, for example "Turn the RIGHT side UP", with standard notation (R, U', 2R…) in the corner for older kids.
  - Big cubes are split into parts ("Make the middles", "Match the edges", "Finish the outside").
- **Kid-friendly touches:** read-aloud instructions (🔊 toggle), cheers along the way, confetti at the end, and an "I got mixed up" help button.
- **Practice mode.** A pretend cube for trying the app without a real cube or camera.
- **Friendly scan errors.** Impossible scans, such as a twisted corner or wrong colour counts, get a kid-level explanation and a prompt to rescan.
- **Phone-first layout.** Every screen fits a phone without scrolling, and buttons stay on screen (tested at 412×915, 375×667 and 360×640).

## Running it

The camera only works on pages served over **https://** or from **localhost**. This is a browser rule, not an app setting. Opening the file directly (`file://`) still works for practice mode, but not for scanning.

On a computer:

```sh
# any static server works, for example:
npx serve .
# or
python -m http.server 8000
```

Then open `http://localhost:8000` (or the port shown).

On a phone, host `index.html` anywhere that serves https, for example GitHub Pages, Netlify or Cloudflare Pages, and open that address. The phone's back camera is used by default; 🔄 switches cameras.

The page needs an internet connection the first time it loads. It pulls [three.js r128](https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js) from cdnjs and the Fredoka font from Google Fonts. Camera images never leave the device; all colour reading and solving happens in the browser.

## How to use it

1. Pick the cube size.
2. Hold the cube with any side facing the camera and **keep the same side on top**. Fit the cube inside the squares and tap **Snap!**.
3. Follow the guide for the other five sides: turn left three times, then tip the top towards you, then tip it twice more.
4. Check the 3D copy on the review screen, then tap **Solve it**.
5. Hold the cube as shown (the same way as the first photo) and follow the arrow. Tap **Done!** after each turn.

Tips: scan in bright, even light without glare. The 🔦 button turns on the phone's flashlight when available.

## How it works

The page has two parts:

- **Solver core** (`<script id="core">`): the cube model and all solvers, in plain JavaScript. The same code also runs in a Web Worker created from that script's text, so solving never freezes the page.
- **App** (the second script): screens, camera, colour reading and the three.js views.

### Cube model

Any N×N cube is a list of 6·N² stickers. Every turn (axis, layer, quarter turns) is precomputed as a sticker permutation.

### 3×3

The 3×3 uses Kociemba's two-phase algorithm, with move and pruning tables built in about half a second. It searches for up to 1 second for shorter solutions, typically giving about 20 turns.

### 4×4 and 5×5 (reduction method)

1. **Undo light scrambles.** A beam search scored on "stickers in the right place + same-colour neighbours" often solves a lightly mixed cube outright. On 5×5 it may turn the middle layer.
2. **Centres by table search.** Three phases, like Thistlethwaite's method:
   1. Top/bottom colours go onto the top/bottom faces.
   2. Left/right colours go onto left/right, and top is separated from bottom.
   3. Every centre sticker goes to its own face.

   Each phase uses exact distance tables for each centre type (24-bit masks, up to about 900k entries). The search is exact (IDA*) on 4×4 and a table-guided beam on 5×5. If phase 3 hits a combination its turns can't finish, a step-by-step centre solver takes over.
3. **Edges.**
   - 4×4: wings go to their home spots.
   - 5×5: wings are paired with the middle edge piece next to them.

   Each step picks the move that fixes the most edge pieces per turn. The candidates are:
   - pure 3-cycles;
   - multi-piece blocks;
   - short "slice, face turns, slice back" pairing moves.

   The move library is generated automatically at start-up by testing commutator patterns. A standard parity algorithm fixes odd wing order first. If the 5×5 pairing stage ever gets stuck, it falls back to solving the outer 3×3 first and then the wings.
4. **Finish as a 3×3** with the Kociemba solver, using outer-layer turns only.

Every solution is replayed on the scanned cube and checked before it is shown.

### Typical results

Measured on a desktop PC in Chrome, fully random scrambles:

| Cube | Average turns | Solve time |
|---|---|---|
| 3×3 | about 20 | about 1 s |
| 4×4 | about 117 | about 3 s |
| 5×5 | about 128 (range roughly 105–160) | about 4–6 s |

Lightly mixed cubes are often solved in about as many turns as were used to mix them. Phones are slower; expect roughly 2–3× these times.

The 4×4/5×5 tables take a few seconds to build (about 8 s for 5×5 on a desktop PC). The app starts building them in the background as soon as a cube size is picked, so they are normally ready before scanning finishes.

### Colour reading

- **Live dots.** Each square is judged on its colour angle and brightness. Red and orange are separated by colour angle plus brightness (orange is more yellowish and brighter). When a frame shows both, they are split at the biggest gap. The app also remembers the split from snapped sides, so it adapts to warm indoor or webcam light.
- **Final colours.** After all six sides, the stickers are sorted into six groups of exactly N² each, using k-means clustering in Lab colour space. On odd cubes the fixed centre stickers seed the groups. Each group is then named from what the live dots showed.

## Limitations

- A camera scan needs reasonably even light. A single wrong sticker makes the cube impossible to solve. The app detects this, but the child has to rescan the side that looks wrong.
- 4×4 and 5×5 solutions are long by nature (well over 100 turns for a full scramble), so children may need breaks. The progress bar keeps their place.
- A front-facing camera (for example a laptop webcam) works, but the turning instructions between photos are written for the phone's back camera.
- Supported sizes are 3×3, 4×4 and 5×5.

## Browser support

Recent Chrome, Edge, Safari (iOS 16+) and Firefox. It needs WebGL, Web Workers and camera access (`getUserMedia`). Read-aloud uses the browser's built-in speech synthesis where available.
