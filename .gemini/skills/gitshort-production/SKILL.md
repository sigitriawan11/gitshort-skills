---
name: gitshort-production
description: Portable end-to-end automated pipeline for producing 2.5D layered 2D cartoon YouTube Shorts narrative videos (standalone one-shot stories, mystery, horror, folklore, thriller, comedy, or multi-part episodes) using VectorCraft MCP, Edge-TTS, FFmpeg/Python SFX, EffectCraft MCP, and FilmCraft MCP without Blender. Use whenever the user asks to create a new short video, story, or content folder.
---

# GitShort 2.5D Video Production Pipeline (Portable Edition)

This skill automates the full 4-phase production workflow for creating vertical (`9:16`, `1080x1920`, `30fps`) 2.5D layered cartoon narrative Shorts without Blender. All paths are workspace-relative and environment-agnostic so anyone cloning the repository can run it on any machine.

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
2. **Clean Cutout & True Alpha Transparency (CRITICAL):**
   - **NO white sticker borders & NO drop shadows baked into PNGs:** Do NOT draw thick white outlines, white backing shapes, or outer drop shadows around characters or props. Keep edges crisp with dark line art (`#1A1A1A`) so assets blend naturally into the scene.
   - **Strict Transparent Canvas (`alpha=0`):** When generating and exporting characters (`assets/characters/*.png`) and props (`assets/environment/*.png`) in VectorCraft, ensure the document/artboard background is completely transparent (no white background rectangle/fill) and export as 32-bit RGBA PNG. Verify with Python (`PIL.Image`) that corner pixels have `alpha == 0`; if any PNG has an opaque white background, strip/fix it before Phase 3.
3. **Articulated 2D Cartoon Character Standard:**
   - **NEVER build characters out of primitive geometric blocks** (no boxy rectangle torsos, no floating circle hands without arms, no featureless circle heads).
   - Every human character MUST be drawn with **full articulated cartoon anatomy**: distinct head with ears, hair/headwear, expressive eyebrows, eyes with pupils, nose, and mouth; a distinct neck; structured torso with shirt collar, buttons, or clothing folds; **connected upper and lower arms with sleeves/cuffs and hands**; separated legs with trousers/sarong folds; and detailed shoes/sandals.
   - Maintain a **three-quarter (3/4) front-facing stance** with bold, consistent dark outlines (`#1A1A1A`) and flat cel-shaded colors.
4. **Rich Facial Expressions & Lip-Sync Mouth Variants:**
   - Every speaking character MUST have both **closed-mouth** and **open-mouth speaking** variants (e.g., `<char>_diam.png` and `<char>_bicara.png`) plus distinct emotional expressions tailored to the story (`<char>_jalan.png`, `<char>_kaget.png`, `<char>_senyum.png`, `<char>_kesal.png`, `<char>_bingung.png`).
   - Keep body, head position, and clothing anchor coordinates identical between `<char>_diam.png` and `<char>_bicara.png` so alternating them produces clean, jitter-free lip movement.
5. **Full-Bleed Backgrounds:** Landscape/room backgrounds (`latar_utama.png`, `2160x1920`) must be full-bleed edge-to-edge without borders.

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
   - **Clean Narration Script (Plain Text):** Original Indonesian story (~180–220 words for 75–85 seconds) with a strong 3-second hook, rising tension/comedy, climax, and twist/resolution. Must be strictly plain text (no timestamps or bracketed stage directions) so it feeds directly into TTS.
   - **Asset Manifest:** Exact list of character poses/expressions (explicitly including `_bicara` open-mouth variants for any character with spoken dialogue) and environment props required.
   - **Dialogue & Cue Sheet:** Map which character speaks during each quoted dialogue line, along with sound effects and lighting/animation triggers.

---

## Phase 1: Clean 2D Vector Asset Generation (`vectorcraft` MCP)
Resolve `<win_render_dir>` first, then use the `vectorcraft` MCP server to generate and export transparent PNG assets into `<win_render_dir>\assets\`:
1. **Characters (`assets\characters\`):**
   - Build a reusable base character rig/group first (head, neck, articulated arms with sleeves, torso with clothing details, legs, and footwear) so proportions, outfit details, and face remain 100% consistent across all poses.
   - Export varied emotional expressions AND a dedicated open-mouth `<char>_bicara.png` variant for every character who speaks dialogue in the script.
   - Export directly with transparent background (no white border, no shadow layer).
2. **Environment (`assets\environment\`):** Generate clean transparent props/furniture (no white border, no shadow layer) plus one wide full-bleed parallax background (`latar_utama.png`, `2160x1920`).
3. **Verification:** Confirm every PNG file in `./workspace-render/<content_slug>/assets/` exists, is non-empty (`> 0 KB`), and has true transparent alpha background (except `latar_utama.png`) before proceeding.

---

## Phase 2: Voice Over (`edge-tts`) & SFX Generation
Execute local shell/Python commands to generate audio in `./workspace-render/<content_slug>/audio/`:
1. **Voice Over & Subtitles:**
   - Ensure `edge-tts` is available (`pip install -q edge-tts`).
   - Synthesize the plain-text script using voice `id-ID-ArdiNeural` (`--rate="-5%" --pitch="-2Hz"` for mystery/horror, or `--rate="+3%" --pitch="+2Hz"` for comedy/upbeat).
   - Save audio to `./workspace-render/<content_slug>/audio/vo_narasi.mp3`.
   - Save subtitles to BOTH `./workspace-render/<content_slug>/audio/vo_narasi.srt` AND `./konten/<content_slug>/vo_narasi.srt` (for Git tracking).
2. **SFX & BGM (`audio/sfx/`):**
   - Synthesize (via Python/FFmpeg) or fetch clean royalty-free audio files matching the cue sheet (ambient loop, sharp transient clicks/stingers, footsteps, and fitting BGM).
   - Measure exact duration of `vo_narasi.mp3` via `ffprobe` or Python.

---

## Phase 3: 2.5D Parallax Animation, Lip-Sync & Lighting (`effectcraft` MCP)
Read timestamps from `./konten/<content_slug>/vo_narasi.srt` and use `effectcraft` MCP to build `<win_render_dir>\comps\scene_utama.ecproj` (`1080x1920`, `30fps`, exact duration of `vo_narasi.mp3`):
1. **Multi-Plane Depth Setup:** Layer background, floor/ground, midground structures/furniture, and characters on separate Z-depth/parallax planes with color grading matching the story mood.
2. **Dialogue Lip-Sync & Varied Expression Choreography:**
   - Identify every `.srt` subtitle segment containing spoken character dialogue (quoted speech).
   - During the exact start-to-end timestamps of a character's dialogue line, animate **lip-sync mouth movement** by rapidly toggling visibility/opacity (`0%` / `100%` hold keyframes every `0.15s`) between the character's closed-mouth pose (`_diam` or emotional pose) and open-mouth speaking pose (`_bicara`), accompanied by subtle head/upper-body emphasis (`1%–2%` scale/rotation pulse). Immediately return to the closed-mouth expression when the dialogue line ends.
   - Switch between varied emotional poses (`_jalan`, `_diam`, `_kaget`, `_senyum`, `_kesal`, `_bingung`) across narrative beats so characters never look static.
3. **Movement & Camera:**
   - Animate walking with vertical step-bobbing (`0.25s` cycle) and horizontal scale flips (`100%` to `-100%`).
   - Animate camera panning and zoom-in on key story reveals.
4. **Render Raw Video:** Export to `<win_render_dir>\output\animasi_mentah.mp4`.

---

## Phase 4: Final Audio Mix & Shorts Subtitles (`filmcraft` MCP)
Use `filmcraft` MCP to assemble the final video at `<win_render_dir>\output\final_shorts.mp4`:
1. **Video & Safe-Zone Subtitles:** Place `animasi_mentah.mp4` on the main video track and burn in bold, high-contrast subtitles (white/yellow fill with thick black outline) from `vo_narasi.srt` inside the YouTube Shorts center-lower safe zone.
2. **Audio Mixing:**
   - **Track 1 (VO):** `vo_narasi.mp3` at `0 dB` (`100%`).
   - **Track 2 (Ambience):** Loop ambient SFX at `-14 dB`, automating sudden silence (`0%`) during punchlines or suspense beats.
   - **Track 3 (Point SFX):** Align transient sound effects to exact animation timestamps.
   - **Track 4 (BGM):** Background music at `-20 dB` throughout.
3. **Export:** Render H.264 MP4 (`1080x1920`, `30fps`, AAC audio) to `<win_render_dir>\output\final_shorts.mp4` and report completion to the user.
