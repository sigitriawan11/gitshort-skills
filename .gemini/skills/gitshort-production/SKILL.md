---
name: gitshort-production
description: Portable end-to-end automated pipeline for producing 2.5D paper-cutout YouTube Shorts narrative videos (standalone one-shot stories, mystery, horror, folklore, thriller, or multi-part episodes) using VectorCraft MCP, Edge-TTS, FFmpeg/Python SFX, EffectCraft MCP, and FilmCraft MCP without Blender. Use whenever the user asks to create a new short video, story, or content folder.
---

# GitShort 2.5D Video Production Pipeline (Portable Edition)

This skill automates the full 4-phase production workflow for creating vertical (`9:16`, `1080x1920`, `30fps`) paper-cutout narrative Shorts without Blender. All paths are workspace-relative and environment-agnostic so anyone cloning the repository can run it on any machine.

## Workspace & Cross-Platform Path Resolution
1. **Git Blueprint Directory (Tracked in Git):**
   - Store lightweight text files (`produksi.md` and `vo_narasi.srt`) inside `./konten/<content_slug>/` relative to the active repository root.
2. **Media & Render Directory (Git-Ignored):**
   - By default, store heavy media files (`assets/`, `audio/`, `comps/`, `output/`) inside `./workspace-render/<content_slug>/` (or use `RENDER_ROOT_DIR` if defined in `.env`).
3. **Automatic OS / WSL Path Translation for MCP Tools:**
   - Resolve `./workspace-render/<content_slug>` into a native Windows path before calling any MCP tool:
     - **If running inside WSL:** Run `wslpath -w ./workspace-render/<content_slug>` via bash to obtain the exact Windows path.
     - **If running on native Windows:** Resolve the absolute Windows path directly.
   - Always pass the resolved Windows path to MCP tools, and use standard POSIX/relative paths for local Python/FFmpeg commands.

## Visual & Technical Standards
1. **No Blender Policy:** Achieve 3D spatial depth strictly via **2.5D multi-plane parallax layers** and **Glow/Lighting effects** inside EffectCraft.
2. **Articulated 2D Cartoon Character Standard (CRITICAL):**
   - **NEVER build characters out of primitive geometric blocks** (no boxy rectangle torsos, no floating circle hands without arms, no featureless circle heads).
   - Every human character MUST be drawn with **full articulated cartoon anatomy**: distinct head with ears, hair/headwear (e.g., peci/cap), expressive eyebrows, eyes with pupils, nose, and mouth; a distinct neck; structured torso with shirt collar, buttons, or clothing folds; **connected upper and lower arms with sleeves/cuffs and hands**; separated legs with trousers/sarong folds; and detailed shoes/sandals.
   - Maintain a **three-quarter (3/4) front-facing stance** with bold, consistent dark outlines (`#1A1A1A`), flat cel-shaded colors with subtle clothing patterns/folds, a thick white paper-cutout sticker border (`#FFFFFF`), and a soft drop shadow on a transparent PNG background.
3. **Full-Bleed Backgrounds:** Landscape/sky backgrounds (`latar_utama.png`, `2160x1920`) must be full-bleed without white sticker borders.

---

## Phase 0: Workspace Initialization & Story Blueprint
When asked to create a new standalone video or episode with slug `<content_slug>`:
1. Create directories relative to the repository root:
   - `./konten/<content_slug>/`
   - `./workspace-render/<content_slug>/assets/characters/`
   - `./workspace-render/<content_slug>/assets/environment/`
   - `./workspace-render/<content_slug>/audio/sfx/`
   - `./workspace-render/<content_slug>/comps/`
   - `./workspace-render/<content_slug>/output/`
2. Write `./konten/<content_slug>/produksi.md` containing:
   - **Clean Narration Script (Plain Text):** Original Indonesian story (~180–220 words for 75–85 seconds) with a strong 3-second hook, rising tension, climax, and twist/resolution. Must be strictly plain text (no timestamps or bracketed stage directions) so it feeds directly into TTS.
   - **Asset Manifest:** Exact list of character poses/expressions and environment props required.
   - **SFX & Cue Sheet:** Sound effects and lighting/animation triggers mapped to story beats.

---

## Phase 1: Paper-Cutout Asset Generation (`vectorcraft` MCP)
Resolve `<win_render_dir>` first, then use the `vectorcraft` MCP server to generate and export transparent PNG assets into `<win_render_dir>\assets\`:
1. **Characters (`assets\characters\`):**
   - Build a reusable base character rig/group first (head, neck, articulated arms with sleeves, torso with clothing details, legs, and footwear) so proportions, outfit details, and face remain 100% consistent across `<char>_jalan.png`, `<char>_diam.png`, and `<char>_kaget.png`—only adjusting limb angles, eyebrows, eyes, and mouth per pose.
   - Wrap the final character group in a smooth contour-following white sticker border and drop shadow.
2. **Environment (`assets\environment\`):** Generate props/structures with sticker borders plus one wide parallax background (`latar_utama.png`, `2160x1920`, no white border).
3. **Verification:** Confirm every PNG file in `./workspace-render/<content_slug>/assets/` exists and is non-empty (`> 0 KB`) before proceeding.

---

## Phase 2: Voice Over (`edge-tts`) & SFX Generation
Execute local shell/Python commands to generate audio in `./workspace-render/<content_slug>/audio/`:
1. **Voice Over & Subtitles:**
   - Ensure `edge-tts` is available (`pip install -q edge-tts`).
   - Synthesize the plain-text script using voice `id-ID-ArdiNeural` (`--rate="-5%" --pitch="-2Hz"`).
   - Save audio to `./workspace-render/<content_slug>/audio/vo_narasi.mp3`.
   - Save subtitles to BOTH `./workspace-render/<content_slug>/audio/vo_narasi.srt` AND `./konten/<content_slug>/vo_narasi.srt` (for Git tracking).
2. **SFX & BGM (`audio/sfx/`):**
   - Synthesize (via Python/FFmpeg) or fetch clean royalty-free audio files matching the cue sheet (ambient loop, sharp transient clicks/stingers, footsteps, and dark ambient `bgm_misteri.mp3`).
   - Measure exact duration of `vo_narasi.mp3` via `ffprobe` or Python.

---

## Phase 3: 2.5D Parallax Animation & Lighting (`effectcraft` MCP)
Read timestamps from `./konten/<content_slug>/vo_narasi.srt` and use `effectcraft` MCP to build `<win_render_dir>\comps\scene_utama.ecproj` (`1080x1920`, `30fps`, exact duration of `vo_narasi.mp3`):
1. **Multi-Plane Depth Setup:** Layer background, ground path, midground structures, and characters on separate Z-depth/parallax planes with atmospheric color grading and edge vignette.
2. **Animation & Lighting Choreography:**
   - Animate character movement with vertical step-bobbing (`0.25s` cycle) and horizontal scale flips (`100%` to `-100%`).
   - Synchronize expression swaps and sudden lighting changes (e.g., warm yellow `Glow` layers jumping from `0%` to `100%` opacity via hold keyframes) to the exact `.srt` timestamps.
   - Animate camera panning and a dramatic zoom-in on the climax/twist reveal.
3. **Render Raw Video:** Export to `<win_render_dir>\output\animasi_mentah.mp4`.

---

## Phase 4: Final Audio Mix & Shorts Subtitles (`filmcraft` MCP)
Use `filmcraft` MCP to assemble the final video at `<win_render_dir>\output\final_shorts.mp4`:
1. **Video & Safe-Zone Subtitles:** Place `animasi_mentah.mp4` on the main video track and burn in bold, high-contrast subtitles (white/yellow fill with thick black outline) from `vo_narasi.srt` inside the YouTube Shorts center-lower safe zone.
2. **Audio Mixing:**
   - **Track 1 (VO):** `vo_narasi.mp3` at `0 dB` (`100%`).
   - **Track 2 (Ambience):** Loop ambient SFX at `-14 dB`, automating sudden silence (`0%`) during suspense/shock beats.
   - **Track 3 (Point SFX):** Align transient sound effects to exact animation timestamps.
   - **Track 4 (BGM):** `bgm_misteri.mp3` at `-20 dB` throughout.
3. **Export:** Render H.264 MP4 (`1080x1920`, `30fps`, AAC audio) to `<win_render_dir>\output\final_shorts.mp4` and report completion to the user.
