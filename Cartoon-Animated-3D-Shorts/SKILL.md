---
name: cartoon-animated-3d-shorts
description: Make a ~55-70 s vertical cartoon story short in the "2D characters in a colourful 3D world" style (flat bean-head cartoon cards with ink outlines, painted 3D town/world with black ink lines, saturated sky, depth-of-field backgrounds, gliding cameras, local AI voice narration with a reusable saved voice, synthesized SFX + march-style music, animated single-word subtitles), rendered with three.js inside Remotion. Proven on "The Bridge That Fell for a Song" (2026-10-31). Use when the user asks for another video "like the bridge video", "like the soldiers/bridge short", "in that cartoon 3D style", a history/science story short with cartoon characters in 3D, or says "use the cartoon short skill".
---

# Cartoon 3D story short (three.js + Remotion)

Proven example: **The Bridge That Fell for a Song**. It is an original 67 s short about the 1831 Broughton bridge and 74 soldiers marching in step.
- Code: `src/videos/bridge/` of your Remotion project (composition `BridgeShort`, 1080x1920, 30 fps).
- Script, voice and sound: `~/bridge-short/`.
- Style research: `references/`.

The style reference is a channel studied privately (`references/`: STYLE_BIBLE.md, reports/*.md, frames/).
- Those videos are private study material. Never upload, publish or send them.
- Never copy their scripts, characters or shots. Story, characters and voice are always OUR OWN.
- Never say the channel's name in a voice-over or description.

User rules that apply:
- The user writes to Claude in Polish. Reply to the user in Polish. The narration and the on-screen subtitles are English unless the user explicitly asks for another language.
- Subtitles: ALWAYS burned in, single-word mode (see "Animated subtitles"). No other captions or labels in the video.
- Voice: one steady tone. The voice comes from the user's voice description and is saved and reused (see "Voice").
- Jokes are researched real facts.
- Standard mix: music +3 dB, SFX -6 dB, voice unchanged.
- Final export 2K vertical: 1440x2560, `--scale 1.3333`.
- Opus max verifies every check.
- Save after every step.
- "Just do it" once the script is approved.

## Pipeline
1. **Topic and facts.** Pick one surprising true story with a physical cause and effect (it should be visual), plus a simple everyday analogy that explains it.
   - Write the facts with their source at the top of `SCRIPT.md`.
2. **Script** (~3.7 words/s, 55-70 s). Beat formula from the bridge short:
   - **Hook:** the absurd fact in one line ("Seventy-four soldiers knocked down an iron bridge just by walking across it.").
   - **Setting:** year, place, pride.
   - **Action:** escalation over three or four beats, each a little bigger.
   - **Snap:** the moment it breaks.
   - **Deadpan double line:** "Luckily ... / Unluckily ...".
   - **Explanation:** one physics idea through an everyday analogy (the kid on a swing).
   - **Consequence:** the rule it created.
   - **Callback twist:** the ending.
   - Never describe on screen what the viewer already sees.
   - Keep normal punctuation in the script: it drives natural TTS pronunciation. It is stripped only from the subtitle display text.
3. **Voice** (local Qwen3-TTS, saved voice library, see "Voice"). Result: `voice.wav`.
4. **Word timing.** Forced alignment on the FINAL `voice.wav` -> `words.json` (see "Voice", step 7). Everything is timed to words.json.
5. **Shot plan** in `story.ts`: copy `templates/bridge/` (the final bridge source, same as `src/videos/bridge/` of your Remotion project) to `src/videos/<name>/`, register a new composition next to `BridgeShort` in `src/Root.tsx`, and rename it.
   - `SHOTS`: id, a/b frames, light day/golden, crowd mode, `tau` story time.
   - `XFADE`: crossfades at place/time changes.
   - `FLASH_AT`: a white flash after the hook.
   - `CAMS`: per-shot keys `{f, dur?, cut?, s: f => {pos, look, fov, roll}}`.
   - `KICKS`: impact flip-shakes.
   - `DOF`/`dofAt`: lens blur per beat.
   - `FOREGROUND`: near framing props.
   - Every visual reaction lands 0-3 frames before its word.
6. **World.**
   - `lights.ts`: LIGHTS plus the LOOK sky/river/cloud palette per light.
   - `shade.ts`: two-tone shading and `untone()`.
   - `build.ts`: town, houses and Kenney CC0 kit props, recoloured, with sight-line culling of anything blocking a camera.
   - `paint.ts`: HD Canvas2D painted textures with seams painted in as dark tints.
   - `world.ts`: sky shader, fog = horizon colour, river, per-frame posing, sun frozen per shot and camera-relative.
7. **Characters.**
   - `toon.ts`: the Canvas2D `Pen` drawing API, 600x1000 design canvas, soles at y = 975, bean heads, stick limbs, plum variable-width outline, face library.
   - `cast.ts`: camera-facing billboards, crowds, effect cards, blinks, walk cycles. All billboards go on the `NOINK` layer.
   - `cardshade.ts`: cards lit by the scene light, no emissive.
8. **Post** (`Bridge.tsx` + `ink.ts` + `lensblur.ts` + `grade.ts`).
   - Composer order: ink → lens pre → lens main → bloom → NEUTRAL tone map → grade (MSAA 4).
   - Every pixel parameter scales with output height / 1920.
   - Subtitles (`Subtitles.tsx`) are a DOM overlay ON TOP of the composer output, after the grade (see "Animated subtitles").
9. **Sound** (`~/bridge-short/audio/`: timeline.py, sfx_gen.py, sfx.py, music.py, instruments.py, dsp.py, mix.py).
   - Event table → synthesized SFX plus an original score ducked under the voice.
   - Output: mix.wav (48 kHz, exact length), stems and cues.md.
   - Lock the picture's time constants before the sound is made. Any later timing change means re-running the sound.
10. **Look checks.** Run `node scripts/bridge-stills.mjs out/<name>/x_ 100,400 0.5`.
    - Set `BRIDGE_ENTRY=src/sandboxes/<n>/index.tsx` to render a sandbox copy.
    - Make side-by-sides against reference frames with `python scripts/sbs_sheet.py out.jpg --ours ... --ref ...`.
    - Send them to an Opus max verifier: 1-3 images, a ≤200-word verdict.
    - Also check subtitle stills (see the subtitle QA list).
11. **Final render.**
    - `npx remotion render src/index.ts <Comp> out/<name>/video.mp4 --codec h264 --crf 16 --scale 1.3333`
    - Mux with `-map 0:v:0 -map 1:a:0` (Remotion writes a silent audio track).
    - QA: `python scripts/short_qa.py final.mp4 out/<name>/qa shots.json` checks every frame for black, white, frozen and pop frames, plus loudness and sheets.
    - Fix anything it finds, then run a deep Opus audit.
12. **Deliver.** SendUserFile the final mp4, update memory, write notes.

## Voice (local Qwen3-TTS + saved voice library)

### Models
Both models live in `C:/Users/juzek/models/` (always use forward slashes in code and strings):
- `C:/Users/juzek/models/Qwen3-TTS-12Hz-1.7B-VoiceDesign` -> DESIGNS a new voice from a text description. Used ONLY the first time a voice is created.
- `C:/Users/juzek/models/Qwen3-TTS-12Hz-1.7B-Base` -> clones a voice from a reference clip. Used for ALL narration.

Rules:
- Run both models locally from that folder. Never call an online TTS API.
- Before the first use, read the README / config files inside the model folders and confirm the real inference API. Do not guess function names. Install what is needed (`pip install qwen-tts torch soundfile`, CUDA build of torch if a GPU exists).
- Why the library exists: a VoiceDesign model is random. The same text description run twice gives two DIFFERENT voices. Only a saved reference clip gives an identical voice. So the description is used once to design the voice, then the voice is frozen as a reference clip, and every later short clones that clip.

### Voice library
Folder: `C:/Users/juzek/voices/<voice-id>/`
- `ref.wav`: 10-15 s reference clip of the designed voice (mono, 24 or 48 kHz, clean, no music).
- `ref.txt`: the exact text spoken in ref.wav (needed for cloning).
- `voice.json`:
  `{ "id", "name", "description", "fingerprint", "seed", "language", "gen_params", "models", "created", "used_in": [] }`
- `fingerprint` = sha1 of the normalised description: lowercase, punctuation removed, whitespace collapsed, words kept in the order written.
- `voice-id` = the name the user gives ("narrator1", "deep-storyteller") or, if none, a slug of the first 5 words of the description plus the first 6 characters of the fingerprint.

### Procedure (run every time a short needs a voice)
1. **Get the voice request.** The user writes it in the prompt, e.g. `Głos: calm deadpan British male narrator, warm, steady pace` and optionally `Nazwa głosu: narrator1`. A line like `Głos: narrator1` (just a saved name) is also valid. If the user gives no voice at all, ask once: which voice (new description or a saved name). If the user is not present, list `C:/Users/juzek/voices/` and reuse the most recently used voice.
2. **Look up the library.** Match in this order: exact name -> exact fingerprint -> near match (same words, small edits). Exact match = REUSE. Near match = ask the user "same voice as <id> or a new one?". Never silently create a second voice for an identical description.
3. **Reuse path (voice found).** Do NOT run VoiceDesign. Say in one line that the saved voice `<id>` is reused. Go to step 5. Append the short's name to `used_in`.
4. **Create path (voice not found).**
   - Run VoiceDesign with the description, a fixed integer seed (choose one, store it) and the fixed reference text: one neutral ~35-word English paragraph kept in `C:/Users/juzek/voices/REFERENCE_TEXT.txt` (create it once, never change it, so every voice is judged on the same text).
   - Generate 2-3 candidates with different seeds, listen-check them with a speaker-quality check (no clipping, no artifacts, steady pace, no long silences), pick the best, and SendUserFile the winner so the user can hear it. If the user is present, wait for approval; if the user says it is wrong, change only the description and repeat.
   - Save `ref.wav`, `ref.txt`, `voice.json` in the new voice folder. From this moment the clip is the voice, not the description.
5. **Generate the narration with the Base model, cloning `ref.wav` + `ref.txt`.** Never narrate with VoiceDesign.
   - Use the stored seed and the stored `gen_params` (temperature, top-p, etc.) from `voice.json` for every sentence. Never change them between shorts.
   - Generate sentence by sentence, then concatenate with ~0.15 s gaps, so one bad sentence can be redone without redoing the rest.
   - Keep the script punctuation as written. One steady tone for the whole script.
6. **Consistency check.** Compare each generated sentence with `ref.wav` using a speaker embedding (e.g. Resemblyzer or the model's own encoder), cosine similarity. If a sentence scores clearly lower than the others (below ~0.80 or a drop of more than 0.08 from the median), regenerate it with a new sampling seed (keep the voice clip the same) up to 3 times. Report the median similarity in the notes.
7. **Output and alignment.**
   - Save `voice.wav` (mono, 48 kHz) in `~/<short>/voice/`.
   - Run forced alignment on the FINAL `voice.wav` (WhisperX, stable-ts or equivalent) to get real word timings -> `~/<short>/voice/words.json` as `[{ "word", "start", "end" }]`, in seconds, one entry per spoken word, in script order.
   - Hand-fix wrong timings. Never estimate timing from text length.
   - Lock `voice.wav` before sound design and before subtitles. Any change to the voice means re-running alignment, subtitles and sound.

### Library rules
- Never overwrite or delete a saved voice, its `ref.wav` or its `voice.json` without asking the user.
- Changing the description of an existing name = a NEW voice id (ask the user for a name), the old one stays untouched.
- The user can ask to "regenerate the voice from scratch": only then create a new reference clip (new id, or overwrite with explicit permission).
- Back up the whole `C:/Users/juzek/voices/` folder into the short's `backup/` folder at the end of each short, if it is under 50 MB.
- Mention in the final message which voice id was used and whether it was reused or newly created.

## Animated subtitles (STRICT single-word mode)

Every short gets burned-in subtitles built from the real word timings in `words.json`. This section is extremely important. Follow every rule precisely.

### Content
- EXACTLY ONE WORD on screen at a time. When the narrator says a word, show only that word. When the next word starts, replace it.
- NEVER show full sentences, multiple words at once, previous words or upcoming words. Two subtitle words never overlap visually. At most one word is visible on any frame.
- ALL UPPERCASE. Convert every word with `.toUpperCase()` in code before rendering. Never rely on the transcription model for case. Never lowercase, never title case.
- NO punctuation anywhere: no commas, periods, exclamation marks, question marks, apostrophes, quotation marks, colons, semicolons, hyphens or any other punctuation character. Show clean uppercase words only.
- Strip punctuation from the DISPLAY text AFTER the timestamps are generated. Keep the original script punctuation for the TTS. If stripping leaves an empty word, drop it but keep its neighbours' timing.
- Examples: "The king shouted, but nobody listened." -> THE -> KING -> SHOUTED -> BUT -> NOBODY -> LISTENED. Contractions lose the apostrophe ("don't" -> DONT).

### Look (at 1080x1920, scale every pixel value with output height / 1920; the final 2K render is 1440x2560)
- Pure white `#FFFFFF`. No yellow words, no highlight, no coloured emphasis.
- Modern, bold, clean sans-serif: Montserrat ExtraBold or Inter Black (or a similar heavy font). Load the font file locally in Remotion (`@remotion/fonts` or a local `@font-face`) and wait for it before rendering.
- VERY LARGE: about 110-130 px. Auto-shrink exceptionally long words so they fit the safe width (measure the text, shrink to fit). Centered horizontally.
- Thick black outline about 7-10 px (use `-webkit-text-stroke` with `paint-order: stroke fill`, or SVG text) plus a subtle black drop shadow.
- No coloured backgrounds, no rectangular boxes.
- Never extend outside the frame. Readable on a mobile screen.
- Position: vertical center at about 75 % of the frame height, inside the YouTube Shorts and TikTok safe area. Adjust a little for platform overlays if needed.

### Animation (each word separately)
- Smooth pop-in: opacity fade-in plus scale from about 0.92 to 1.0 with soft easing (easeOutCubic or similar), over about 100-160 ms (3-5 frames at 30 fps) when the word's duration allows. Shorter for very short words.
- Subtle exit fade.
- No excessive bounce, no shaking, no flashing, no exaggerated zoom.
- If two adjacent words are very close, shorten the exit and the entrance. Guarantee only one word on any frame.
- Animation stays synchronized with the actual spoken word. Smoothness never costs word-level accuracy.

### Timing
- Use forced-alignment word timestamps from the final `voice.wav`. One interval per spoken word. Never fake or estimate timestamps.
- At 30 fps, convert each time to frames from the ABSOLUTE time (`Math.round(t * 30)`), never by adding durations up, so there is no accumulated drift.
- Make the end frame of word N never later than the start frame of word N+1.

### Processing order
1. Generate the real voiceover (see "Voice").
2. Forced alignment on the real audio -> start and end of every spoken word.
3. Strip punctuation from the display text.
4. Convert every word to UPPERCASE.
5. Build one animated subtitle element per word.
6. Synchronize the animation with the real audio.
7. Check that only one word is visible per frame.
8. Burn the subtitles into the final MP4 (no separate subtitle track).

### Implementation
- `Subtitles.tsx` in `src/videos/<name>/`, a Remotion component fed by `words.json` (clean -> uppercase -> frames).
- Render it as a DOM overlay ABOVE the composer output, after the grade, so ink, blur, bloom and grade never touch the text.
- Render the overlay on its own layer of the composition, not inside the three.js canvas.

### Subtitle QA (add to the look checks and to `short_qa.py`)
- On stills and on a frame sweep: at most one word per frame, all caps, no punctuation, white fill, black outline, centered, fully inside the frame, center near 75 % of the height.
- Number of distinct subtitle words equals the number of spoken words in `words.json` (minus the dropped empty ones).
- No frame shows two words (cross-fade overlap is forbidden).
- Words start within +-1 frame of their `words.json` start.

### Critical subtitle rule (the final video must ALWAYS follow this)
ONE WORD AT A TIME. ALL UPPERCASE. PURE WHITE. LARGE BOLD FONT. THICK BLACK OUTLINE. NO PUNCTUATION. NO YELLOW HIGHLIGHTING. SMOOTH POP-IN ANIMATIONS. PERFECT WORD-LEVEL SYNCHRONIZATION.

## Final look values (bridge after 4 verifier rounds; all in `templates/bridge/`)
Start every new short from `templates/bridge/` (copy to `src/videos/<name>/`): the look lives in these values, the
story lives in story.ts / build.ts layout / cast.ts roles. Opus max scores at the end: colour day 7, golden 5-6,
lighting 5-6, ink 6, textures 5-6, characters 7, depth 6-7, composition 6-7 (from 5/3/4/5/4/7/4/6 at the start).
- **Light presets** (`lights.ts`, sun camera-relative and frozen per shot):
  - day: az 75 / el 48, lit [1.58,1.5,1.33], shade [.30,.29,.36], lift [.05,.06,.11] (cool tint ONLY in shade), sky
    top #3e7fd2 → horizon #b8d0ec, fog = horizon, 25→300 m, uSat 1.08, ~15 % blue haze past 60 m.
  - golden: az 6 / el 7 (backlit low sun), key toward #FFB36B ×1.5, violet-blue shade (facade ≈ #444380), sky
    #ffc47c → #8a6f8a, peach fog only past 50 m, sun disc #ffcf9e + bloom, contact blobs + real card shadows.
- **Albedos** (`paint.ts`/`build.ts`): muted brick / cream / ochre / terracotta houses with plinth, cornice, sills,
  hoods, door step, chimney + pot, heights 7 / 8.4 / 9.8 m; warm grey/beige stone (saturation −35 % from "storybook");
  ochre wood deck C* ~51-58 with near-black seams; iron dark and desaturated. Never lemon, never lavender lit faces.
- **Ink** (`ink.ts`): #030303, 5.1 px near (6 m) → 3.3 px far (90 m) at 1080x1920, creases ×0.8 from 35°, depth
  jumps from 1 %, fog tint 0.3 only far; character cards on layer `NOINK` (they carry their own plum ink).
- **Lens** (`lensblur.ts` + `dofAt` in story.ts): cInf px at 1080: hook/river CU 15-16, close-ups 12-14, MS 10-12,
  park 7, golden sway 5, wides/aerials 0. Focus on the eyes of the front row.
- **Post** (`Bridge.tsx`): ink → lens pre → lens main → bloom (thr 1.0, int .5, 6 levels, radius .6) → NEUTRAL tone
  map → grade; MSAA 4. Film grain exists but is OFF (reference measures no grain; it quadruples the file size).
- **Characters** (`toon.ts`/`cast.ts`/`cardshade.ts`): bean heads h/w .94, plum ink #241726, eyes .26 W apart, no blush,
  14 expressions; adults 3.5 head-heights; walk bob 5 %, lean 1-7°, planted 0.8 m stride; blinks 0.5-2.3 s; tight
  contact shadow ~.5 under the feet + sun-direction tail by day; golden rim inner only (never an offset outer copy).
- **Camera** (`story.ts`): cuts on big jumps, smootherstep glides 18-40 f, every hold creeps (no static hold ≥4 f),
  flip kicks decaying over 7 f on impacts, 4-4.6° rolls, foreground props (`FOREGROUND`) in ~1/3 of shots.

## Look targets (measured from the reference; see references/reports/)
- **Ink:**
  - World lines: about 3 px at 1080 wide, near-black #0b0a0c, nearly constant with distance. Silhouettes, depth jumps and normal creases only. No line on wall/floor joins or the horizon. Character cards are never outlined by the world ink.
  - Character ink: dark plum #201628-#2f1d28, thicker on the bottom/shadow side.
- **Colour:**
  - Day median luma ~.57. Non-ink p5 ~.23 (no crushed blacks). Lit chroma C* p90 ~55. Day sky top ~#5e93d9, horizon 1.28x brighter.
  - Golden: peach horizon #ffd08a, violet-blue shadows (never brown), backlit sun.
- **Depth:**
  - DOF on about 1/3 of screen time (CU, MS, inserts), cInf 10-16 px at 1080, focus on the eyes. Establishing and wide shots stay sharp.
  - Fog = horizon colour, 25→420 m day, 20→360 m golden.
  - Bloom only on HDR emissives: threshold 1.0.
- **Camera:**
  - Hard cuts on big jumps. Everything else is a smootherstep glide of 18-40 f, peak ≤ 25 px/f at 720p.
  - Every hold pushes in 0.15-0.35 %/f, with 0.6 %/f on punch words.
  - Rolls of 4-5°.
  - No hand-held float. Impact shakes flip sign every frame and decay over 7 f.
- **Characters:**
  - About 3.5 head-heights tall, legs 1.4 HH.
  - Stick limbs about 0.055 of head width, ink-dot hands.
  - Bean head (h/w ~0.93, narrower chin, pointier crown).
  - No blush or eye highlights. Faces at brightness ~.93.
  - Walk bob 3-7 %, lean 1-7°, planted feet.
  - Blinks every 0.5-2.3 s, lasting 4 f.

## Look-match loop (how the bridge look was reached; reuse for any "make it look like X" pass)
1. **Measure the reference first.** One analysis agent per topic (motion, depth, textures, characters, lighting, outlines, grade), each writing numbers, not adjectives, to `references/reports/<topic>.md`. Shared brief: `compare/CONTEXT.md`. Reports are the source of truth for every agent after.
2. **Sandboxes, one owner per file.** Copy the project folder to `src/videos/<name>_{cam,world,mat,char,post}` with entries `src/sandboxes/<n>/index.tsx` (same `index.tsx` as main, only the import path changed). Ownership used for the bridge:
   - cam: story.ts (camera keys, cuts, kicks, DOF table, foreground props)
   - world: world.ts, lights.ts, shade.ts, grade.ts (sun, sky, fog, river, colour, grade)
   - mat: paint.ts, build.ts (painted textures, buildings, props)
   - char: toon.ts, cast.ts, cardshade.ts (character drawing, posing, card lighting, fx cards)
   - post: ink.ts, Bridge.tsx, lensblur.ts, Subtitles.tsx (outline pass, composer, lens blur, bloom, subtitle overlay)
   Agents only edit their own files in their own sandbox and render stills with `BRIDGE_ENTRY=src/sandboxes/<n>/index.tsx node scripts/bridge-stills.mjs out/<name>/<n>/x_ <frames> 0.5`.
3. **Merge** with `python scripts/bridge_merge.py [cam,world,mat,char,post]` (backs main up to `backup/bridge_<time>/`, copies owned files only). Then `npx tsc --noEmit` (filter to the project folder), then **re-sync every sandbox from main** (`cp -f src/videos/<name>/* src/videos/<name>_<n>/`) before the next round.
4. **Verify** with a fresh Opus max agent: side-by-side sheets (`python scripts/sbs_sheet.py out.jpg --ours a b --ref r1 r2`), score each area out of 10 (colour day / golden, lighting, ink, textures, characters, depth, composition) plus a numbered fix list with frame numbers. Map each fix to the file owner and resume that owner's agent with SendMessage (keeps its context). Bridge scores: round 1 5/3/4/5/4/7/4/6 → round 2 5/4/5/6/4/7/6/6 → round 3 see notes.
5. Contracts between owners live in exported constants (e.g. `NOINK` layer from ink.ts, `dofAt`/`FOREGROUND` from story.ts, `LIGHTS`/`LOOK` from lights.ts). Changing a contract means telling every owner that reads it.

## DOF rule (no whole-frame blur)
- For every shot, FIRST decide if there is a character or recognisable prop in the frame to focus on. Set `cInf` per shot in `DOF`/`dofAt` accordingly.
- No character or hero prop in frame (wide, establishing, aerial, pure scenery): `cInf = 0` (fully sharp).
- Character in frame: focus distance = camera-to-eyes distance of the front-row character. Never leave the default focus distance.
- Never blur a whole frame. If nothing in the frame is in focus, that is a bug: set `cInf = 0` for that shot.
- After the shot plan, render a still of EVERY shot (middle frame, scale 0.5) and check that something in each frame is sharp. Fix before the full render.
- Props closer than 2 m to the lens must be removed or moved out of frame.

## Gotchas
- Kenney nature-kit materials are teal: recolour them by material name in build.ts.
- Cameras end up inside houses: the sight-line culler in build.ts/world.ts removes anything blocking any camera ray.
- Ink in the same merged effect pass as DOF gets overwritten. Ink must be its own pass before the blur.
- Pixel-sized post parameters must scale with resolution: stills are at 0.5, the final at 1.333. R3F clamps dpr to ≥ 1, so `--scale 0.5` stills still render a 1080x1920 buffer. The same applies to subtitle font size, outline and shadow.
- **Disk:** every Remotion bundle copies `public/` (~1.3 GB) into `the temp folder\remotion-webpack-bundle-*` and never deletes it. 60+ GB filled the system drive during the bridge passes and a render died with ENOSPC. Always render with `export TEMP='<big drive>/remotion-tmp' TMP='<big drive>/remotion-tmp'` (bridge-stills.mjs sets it itself) and ask the user before deleting old bundle folders in the shared the temp folder (auto mode blocks the sweep). <big drive>/remotion-tmp fills too (~500 folders / 110 GB after the 4 bridge rounds): when a project is finished and nothing is rendering, delete its `remotion-webpack-bundle-*` and `puppeteer_dev_chrome_profile-*` folders there.
- In JS strings write `"<big drive>/remotion-tmp"`, never `"<big drive>\remotion-tmp"` (`\r` becomes a carriage return). The same goes for `C:/Users/juzek/models` and `C:/Users/juzek/voices`: forward slashes only.
- Remotion writes a silent audio track: mux with `-map 0:v:0 -map 1:a:0`.
- Sub-agents can't write report files in this harness: ask them for a ≤150-word final message and save it yourself to `out/<name>/<n>/REPORT.md`.
- Scratch `.ts` files inside `out/` break `tsc`: name them `.txt`.
- 2K renders run ~4 fps (≈10 min for 67 s). Stills at 0.5 take 3-8 s each after a ~40 s bundle.
- Film grain at 2.5 % made the 2K file 910 MB instead of ~250 MB and will turn to mush on YouTube; the reference has none. Keep it off.
- The TTS models and the Remotion render both want the GPU/VRAM: finish voice generation and free the model (`del model; torch.cuda.empty_cache()`) before rendering.
- VoiceDesign output is random: never use it twice for "the same" voice. Always go through the voice library.
- Subtitle font not loaded at render time silently falls back to a thin default font: wait for the font (`delayRender`) and check a still.

## Readability traps the verifier caught (check every new prop/shot for these)
- **A swing drawn as one rope that disappears behind the kid reads as a HANGING.** Always two ropes to a wide plank
  with the kid between them (`swingKid` in toon.ts), and a frame whose legs clear the arc (A-frame at each end).
- Props or flotsam closer than ~2 m to a lens become huge cut-off blobs at the frame edge. When a camera moves, re-check
  every prop near its new position (the river cameras moved into the floating hats).
- A foreground bush cut by the frame edge reads as a "green ball"; a foreground prop must be recognisable or gone.
- Flags: red + yellow star/diamond reads as Vietnam's. Use a period-correct flag (St George's cross for England 1831).
- Painted signs lit by a side sun go grey-blue and unreadable: give the sign face an unlit material.
- Golden rim light done as an offset outer copy turns every hat into a picture frame: inner rim only, head/torso only.
- Balcony/railing rails at chest height through characters: put the lens ~2.5 m up so rails cross legs, not torsos.
- Flat holds read as freezes in QA (frozen span ≥12 f): every hold needs a creeping push-in.
- Subtitles at 75 % height can collide with a character's face or an important prop: check stills and, if needed, nudge the camera framing (never the subtitle position).
