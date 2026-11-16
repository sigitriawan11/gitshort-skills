# Blender 5.3 Universal Procedural SceneCraft & Aesthetic Guide

Gunakan panduan universal ini untuk SEMUA episode dan tema cerita apa pun agar setiap panggung 3D terlihat detail, sinematik, dan jauh dari kesan blok polosan:

## 1. Universal Camera & Color Grading
- **Color Management:** Set `scene.view_settings.view_transform = 'AgX'` (fallback `'Filmic'`) dan `scene.view_settings.look = 'AgX - Medium High Contrast'` agar gradasi terang-gelap dan warna terlihat seperti film animasi modern.
- **Depth of Field (DOF):** Aktifkan `cam.data.dof.use_dof = True`, arahkan `focus_distance` ke aktor yang sedang berbicara, dan gunakan `aperture_fstop = 2.8` agar latar depan/belakang memiliki efek blur (bokeh) alami.
- **Compositor Glow:** Tambahkan node `CompositorNodeGlare` tipe `'FOG_GLOW'` untuk memberi pendar lembut pada sumber cahaya apa pun (lampu ruangan, lampu jalan, matahari, api, layar, atau kilatan efek).

## 2. Procedural PBR Materials (Berlaku untuk Semua Tema)
Hindari warna datar (*flat color*) tanpa tekstur. Selalu kombinasikan `Roughness`, `Metallic`, dan `Bump`:
- **Kayu & Furnitur:** `Noise Texture` + `Wave Texture` -> `ColorRamp` -> `Principled BSDF (Roughness=0.55)` + `Bump (Strength=0.12)`.
- **Logam / Besi / Kendaraan / Peralatan:** `Metallic = 0.88`, `Roughness = 0.22–0.32`.
- **Kain / Pakaian / Gorden / Karpet:** `Principled BSDF` + `Sheen Weight = 0.8`, `Roughness = 0.50`, dengan micro-noise `Bump (Strength=0.06)`.
- **Tanah / Aspal / Batu / Dinding:** `Noise Texture (Scale=25, Detail=8)` -> `Bump (Strength=0.25)` + variasi warna halus lewat `ColorRamp` dengan `Roughness = 0.75–0.90`.
- **Dedaunan / Rumput / Alam:** `Noise Texture` -> `ColorRamp` (2-3 gradasi hijau/cokelat) + `Subsurface Weight = 0.15`, `Roughness = 0.60`.
- **Kaca / Air:** `Transmission Weight = 0.9`, `Roughness = 0.05`, `IOR = 1.45`.

## 3. Universal Geometry & Multi-Part Props Rules (Anti-Boxy)
- **Wajib Bevel:** Setiap objek buatan manusia (bangunan, meja, kursi, tembok, kendaraan, jembatan, tiang) wajib memakai `BevelModifier` (`width = 0.015–0.03`, `segments = 3`) agar sudut-sudutnya menangkap pantulan cahaya (*specular highlight*).
- **Kurva untuk Objek Organik & Panjang:** Gunakan `bpy.data.curves` tipe `'3D'` dengan `bevel_depth` untuk membuat kabel, tali, pipa, dahan pohon, pagar lengkung, atau jalan setapak berkelok.
- **Aturan Multi-Part & Clutter (Kepadatan Detail):** Jangan pernah menaruh ruangan atau jalan kosong melompong. Selalu pecah objek utama menjadi beberapa komponen dan tambahkan *background clutter* sesuai tema cerita:
  - *Setting Interior (Rumah/Sekolah/Kantor/Warung):* Lantai bertekstur + dinding ber-list bawah (*baseboard*) + jendela berbingkai + furnitur utama + 3-5 pernak-pernik kecil di atas meja/lantai (gelas, buku, kabel, lampu meja, kotak).
  - *Setting Eksterior / Alam / Jalanan:* Permukaan tanah/jalan yang tidak rata sempurna (variasi elevasi ringan) + pepohonan/semak berlapis (`icosphere` bervariasi skala & warna) + tiang lampu/rambu + elemen latar jauh (*siluet bangunan/bukit*) agar horizon tidak kosong.

## 4. Universal Three-Point & Mood Lighting
- **Key Light:** `AREA` light utama (`size = 2.0–3.0`) untuk menerangi aktor utama dengan bayangan lembut.
- **Rim / Back Light:** `SPOT` atau `AREA` light berlawanan arah kamera dengan warna kontras (hangat untuk siang/senja, biru dingin atau neon untuk malam/misteri) untuk memisahkan siluet aktor 2D dari latar 3D.
- **Practical Lights:** Setiap objek sumber cahaya di dalam adegan (lampu gantung, lilin, lampu motor, layar TV, obor) wajib memiliki `POINT` light kecil di dalamnya agar cahayanya benar-benar memantul ke lingkungan sekitar.

## 5. Living 3D Environment & Dynamic Props (Anti-Static Background Rule)
Never leave the 3D environment completely frozen like a still painting. In every scene, animate at least **2–3 environmental props** in `build_and_render.py` so the world feels alive:
- **Continuous Mechanical Motion:** Rotate fan blades (`rotation_euler` continuous spin), swing hanging lamps or signs gently (`sin()` wave ±3°–5°), or spin vehicle wheels when moving.
- **Atmospheric & Ambient Motion:** Animate rising steam puffs above hot food/drinks (cyclic Z-rise + scale fade), swaying curtains/banners/foliage in the breeze (subtle rhythmic rotation/scale wave), or drifting fireflies/dust motes in outdoor night scenes.
- **Interactive Story Props:** Whenever a character interacts with a 3D prop (e.g., tripping over a cable, knocking over a glass, opening a door, camera flash firing, or rice mound shrinking per spoonful), animate the 3D object's transform (`location`, `rotation_euler`, `scale`) or material `Emission Strength` in sync with the action beat.
