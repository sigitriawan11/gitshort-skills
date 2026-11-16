---
name: gitshort-production
description: Portable end-to-end automated pipeline for producing Hybrid 3D Low-Poly + 2D Cutout Billboard YouTube Shorts narrative videos (standalone one-shot stories, mystery, horror, folklore, thriller, comedy, or multi-part episodes) using VectorCraft MCP, Edge-TTS, Blender CLI (bpy), and FilmCraft MCP. Use whenever the user asks to create a new short video, story, or content folder.
---

# GitShort Hybrid 3D + 2D Video Production Pipeline

This skill automates the full 4-phase production workflow for creating vertical (`9:16`, `1080x1920`, `30fps`) narrative Shorts using a **Hybrid 3D Low-Poly Environment + 2D Articulated Cutout Billboard Characters** pipeline (inspired by *Paper Mario* / *Konon* storytelling style).

## Workspace & Cross-Platform Path Resolution
1. **Git Blueprint Directory (Tracked in Git):**
   - Store lightweight text files (`produksi.md` and `vo_narasi.srt`) inside `./konten/<content_slug>/` relative to the active repository root.
2. **Media & Render Directory (Git-Ignored):**
   - By default, store heavy media files (`assets/`, `audio/`, `scenes/`, `output/`) inside `./workspace-render/<content_slug>/` (or use `RENDER_ROOT_DIR` if defined in `.env`).
3. **Automatic OS / WSL Path Translation:**
   - **For Windows MCP tools (`vectorcraft`, `filmcraft`):** Resolve `./workspace-render/<content_slug>` via `wslpath -w ./workspace-render/<content_slug>` (if in WSL) or native Windows path.
   - **For Blender CLI (`blender.exe` in WSL):** Locate Windows Blender executable (e.g., `/mnt/c/Program Files/Blender Foundation/Blender */blender.exe` or `blender` if native) so GPU hardware acceleration (EEVEE) is utilized, passing Windows-formatted paths (`wslpath -w`) into Python `bpy` scripts.

## Visual & Technical Standards
1. **Hybrid 3D + 2D Architecture:**
   - **3D Stage (`Blender bpy`):** Build stylized low-poly 3D environments (ground terrain, roads, buildings/interiors, trees, props, sky dome, directional sun/lamp lighting, and volumetric/distance fog) with dynamic 3D camera cinematography (high-angle establishing shots, smooth dolly-ins, camera pans, and dramatic reveals).
   - **2D Cutout Billboard Characters (`VectorCraft`):** Draw characters as clean 2D articulated cartoon figures and import them into the 3D Blender scene as upright 3D planes (`Principled BSDF` / `Emission` + `Alpha` transparency, facing the camera, casting real 3D shadows onto the ground).
2. **Clean Cutout & True Alpha Transparency (CRITICAL):**
   - **NO white sticker borders & NO drop shadows baked into PNGs:** Do NOT draw thick white outlines, white backing shapes, or outer drop shadows around characters or 2D decals. Keep edges crisp with bold dark line art (`#1A1A1A`) so Blender's real 3D lighting and ground shadows handle depth naturally.
   - **Strict Transparent Canvas (`alpha=0`):** Ensure the VectorCraft document background is completely transparent and export as 32-bit RGBA PNG. Verify with Python (`PIL.Image`) that corner pixels have `alpha == 0`; strip/fix any opaque background before Phase 3.
3. **Articulated 2D Cartoon Character Standard:**
   - **NEVER build characters out of primitive geometric blocks** (no boxy rectangle torsos, no floating circle hands without arms, no featureless circle heads).
   - Every human character MUST have **full articulated cartoon anatomy**: distinct head with ears, hair/headwear, expressive eyebrows, eyes with pupils, nose, and mouth; distinct neck; structured torso with shirt collar, buttons, or clothing folds; **connected upper and lower arms with sleeves/cuffs and hands**; separated legs with trousers/sarong folds; and detailed shoes/sandals.
4. **Rich Facial Expressions & Lip-Sync Mouth Variants:**
   - Every speaking character MUST have both **closed-mouth** (`<char>_diam.png`) and **open-mouth speaking** (`<char>_bicara.png`) variants with identical body/head anchor coordinates, plus story-specific emotional poses (`<char>_jalan.png`, `<char>_kaget.png`, `<char>_senyum.png`, `<char>_kesal.png`, `<char>_bingung.png`).

---

## Phase 0: Workspace Initialization & Story Blueprint
When asked to create a new standalone video or episode with slug `<content_slug>`:
1. Create directories relative to the repository root:
   - `./konten/<content_slug>/`
   - `./workspace-render/<content_slug>/assets/characters/`
   - `./workspace-render/<content_slug>/assets/decals/`
   - `./workspace-render/<content_slug>/audio/sfx/`
   - `./workspace-render/<content_slug>/scenes/`
   - `./workspace-render/<content_slug>/output/`
2. Write `./konten/<content_slug>/produksi.md` containing:
   - **Clean Narration Script (Plain Text):** Original Indonesian story (~180–220 words for 75–85 seconds) with a strong 3-second hook, rising tension/comedy, climax, and twist/resolution. Strictly plain text (no timestamps or stage directions) for TTS.
   - **Character & 3D Set Manifest:** List of 2D character poses (including `_bicara` variants) AND description of the 3D low-poly set elements to construct in Blender.
   - **Dialogue, Camera & Cue Sheet:** Map quoted dialogue lines to speaking characters, camera angles per beat, and SFX cues.

---

## Phase 1: Clean 2D Character Asset Generation (`vectorcraft` MCP)
Resolve `<win_render_dir>` first, then use `vectorcraft` MCP to generate and export transparent PNG assets into `<win_render_dir>\assets\`:
1. **Characters (`assets\characters\`):**
   - Build a reusable base character rig/group first so proportions, outfit details, and face remain 100% consistent across all poses.
   - Export varied emotional expressions AND a dedicated open-mouth `<char>_bicara.png` variant for every character with spoken dialogue.
   - Export with true transparent background (`alpha=0`), no white sticker border, and no baked shadow.
2. **Optional 2D Signs/Decals (`assets\decals\`):** Export any flat signage, posters, or face icons needed for 3D props with true transparent alpha.
3. **Verification:** Confirm every PNG file exists, is non-empty (`> 0 KB`), and has `alpha == 0` corner transparency before proceeding.

---

## Phase 2: Voice Over (`edge-tts`) & SFX Generation
Execute local shell/Python commands to generate audio in `./workspace-render/<content_slug>/audio/`:
1. **Voice Over & Subtitles:**
   - Ensure `edge-tts` is available (`pip install -q edge-tts`).
   - Synthesize the plain-text script using voice `id-ID-ArdiNeural` (`--rate="-5%" --pitch="-2Hz"` for mystery/horror, or `--rate="+3%" --pitch="+2Hz"` for comedy/upbeat).
   - Save audio to `./workspace-render/<content_slug>/audio/vo_narasi.mp3`.
   - Save subtitles to BOTH `./workspace-render/<content_slug>/audio/vo_narasi.srt` AND `./konten/<content_slug>/vo_narasi.srt` (for Git tracking).
2. **SFX & BGM (`audio/sfx/`):**
   - Synthesize (via Python/FFmpeg) or fetch clean royalty-free audio files matching the cue sheet.
   - Measure exact duration of `vo_narasi.mp3` via `ffprobe` or Python.

---

## Phase 3: Hybrid 3D Stage & Animation (`Blender CLI bpy`)
Parse timestamps from `./konten/<content_slug>/vo_narasi.srt` and write a self-contained Python script `./workspace-render/<content_slug>/scenes/build_and_render.py` executed headless via Blender CLI (`blender.exe --background --python <win_script_path>`):
1. **Render Engine & Output Config:**
   - Use `BLENDER_EEVEE_NEXT` (or `BLENDER_EEVEE`), resolution `1080x1920`, `30 fps`, total frames = `ceil(audio_duration * 30)`, output to `<win_render_dir>\output\animasi_mentah.mp4` (`FFMPEG` / `MPEG4` / `H264`).
   - Enable shadows and clean stylized color management (`Standard` or `AgX` with vibrant contrast).
2. **Procedural Low-Poly 3D Environment:**
   - Build the full 3D scene matching the story setting using clean low-poly meshes and matte/stylized materials (ground/grass/road/floor, stylized faceted trees/foliage, buildings/walls/furniture, props, sky gradient/world lighting, key sun/area light casting soft shadows, and subtle depth fog).
3. **2D Billboard Character Rigging & Lip-Sync in 3D Space:**
   - Import each character's PNG poses as textured upright 3D planes (`Alpha` connected to `Principled BSDF` Alpha + `Backface Culling` off, bottom-center origin snapped cleanly to `Z = 0` ground level).
   - Stack all pose planes of a character at the same local anchor and switch active pose per story beat using stepped (`CONSTANT`) keyframes on `hide_render` / `scale`.
   - **Lip-Sync Animation:** During the exact `.srt` frame range of any spoken dialogue line, alternate visibility every `4–5` frames (`~0.15s`) between the character's closed-mouth pose and `_bicara` open-mouth pose, paired with subtle Z-axis bounce (`1.02` scale pulse).
   - **Movement & Action:** Animate character walking/running with rhythmic vertical hop cycles (`Z` sine/bounce) and horizontal Y/Z-flip rotations when turning around, or tilt rotations when falling/reacting.
4. **Dynamic 3D Cinematography:**
   - Animate the 3D camera across multiple distinct shots synced to story beats (e.g., wide high-angle establishing shot $\rightarrow$ medium tracking shot $\rightarrow$ close-up over-the-shoulder/reaction shot $\rightarrow$ dramatic punchline zoom) using smooth `BEZIER` interpolation between shots and instant cuts (`CONSTANT`) when switching camera angles.
5. **Headless Render Execution:** Run Blender CLI to render `<win_render_dir>\output\animasi_mentah.mp4` and verify the output file is non-empty.

---

## Phase 4: Final Audio Mix & Shorts Subtitles (`filmcraft` MCP)
Use `filmcraft` MCP to assemble the final video at `<win_render_dir>\output\final_shorts.mp4`:
1. **Video & Safe-Zone Subtitles:** Place `animasi_mentah.mp4` on the main video track and burn in bold, high-contrast subtitles (white/yellow fill with thick black outline) from `vo_narasi.srt` inside the YouTube Shorts center-lower safe zone.
2. **Audio Mixing:**
   - **Track 1 (VO):** `vo_narasi.mp3` at `0 dB` (`100%`).
   - **Track 2 (Ambience):** Loop ambient SFX at `-14 dB`, automating sudden silence (`0%`) during punchlines or suspense beats.
   - **Track 3 (Point SFX):** Align transient sound effects to exact animation frames.
   - **Track 4 (BGM):** Background music at `-20 dB` throughout.
3. **Export:** Render H.264 MP4 (`1080x1920`, `30fps`, AAC audio) to `<win_render_dir>\output\final_shorts.mp4` and report completion to the user.
