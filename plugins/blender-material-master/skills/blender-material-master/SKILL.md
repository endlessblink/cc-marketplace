---
name: blender-material-master
description: Create production-ready Blender materials for game engines with Dutch Golden Age painterly terrain, texture blending, UV, baking, and export workflows.
---

# Blender Material Master

## Skill Metadata

**Skill ID:** `blender-material-master`

**Trigger Phrases:**
- "create material"
- "fix UV stretching"
- "terrain material"
- "painterly material"
- "export material"
- "bake textures"
- "blend textures"
- "Rembrandt material"

**Purpose:** Create production-ready Blender materials for game engines with Dutch Golden Age painterly aesthetic. Specializes in terrain materials with height-based texture blending, triplanar projection, and proper baking for export to Unreal/Godot.

**Applies To:**
- Terrain mesh materials
- Organic landscape materials
- Procedural material creation
- Material export workflows
- UV mapping issues

---

## Core Workflow

### Step 1: UV Mapping Strategy

**Problem:** UV stretching on organic terrain destroys texture quality.

**Solution:** Triplanar Box Projection

```
Mesh Type Assessment:
- Organic terrain (sculpted, irregular) → Box Projection
- Hard surface (buildings, props) → UV unwrap
- Procedural geometry → Object/Generated coordinates
```

**Node Setup:**
```
Texture Coordinate (Object or Generated)
  → Mapping (Scale: 5-10, adjust per texture size)
    → Image Texture (Projection: Box, Blend: 0.15)
      → Principled BSDF
```

**Box Projection Settings:**
- Projection: Box
- Blend: 0.10-0.20 (controls edge transition)
- Lower blend = sharper axis transitions
- Higher blend = smoother but blurrier

**When to NOT use Box Projection:**
- Architectural elements (use UV unwrap)
- Props with clear UVs
- Assets from external sources with proper UVs

---

### Step 2: Principled BSDF Painterly Configuration

**Goal:** Achieve Rembrandt oil painting surface quality - matte, subsurface scatter, minimal specularity.

```
Principled BSDF Rembrandt Preset:

Base Color: [connected from texture blend]
Metallic: 0.0
Roughness: 0.85 (matte surface, no gloss)
Specular IOR Level: 0.2 (minimal reflections)
Specular Tint: 0.0

Subsurface Weight: 0.15 (subtle light penetration)
Subsurface Scale: 0.1 (thin layer)
Subsurface Radius: [0.8, 0.5, 0.3] (warm undertones)
Subsurface IOR: 1.4

Coat Weight: 0.0 (no varnish layer)
Sheen Weight: 0.0 (not fabric)

Emission Color: Black (unless adding fill light)
Emission Strength: 0.0

Alpha: 1.0
Normal: [connected if using normal maps]
```

**Why These Values:**
- High roughness (0.85) = canvas texture diffusion
- Low specular (0.2) = no wet paint shine
- Subsurface scatter = pigment depth, warm glow
- No coat = pre-varnish painting appearance

---

### Step 3: Color Palette Enforcement

**Dutch Golden Age Hex Palette:**

```
Dark Umber:    #2D261E  RGB(45, 38, 30)   - Deep shadows
Raw Umber:     #4A3D2F  RGB(74, 61, 47)   - Mid-shadows
Yellow Ochre:  #8C6D46  RGB(140, 109, 70) - Base midtones
Raw Sienna:    #A68758  RGB(166, 135, 88) - Warm highlights
Burnt Sienna:  #B37233  RGB(179, 114, 51) - Golden accents
```

**Desaturation Node Setup:**

```
[Texture Color Output]
  → Hue/Saturation/Value Node
    - Hue: 0.0 (no shift)
    - Saturation: 0.70 (muted earth tones)
    - Value: 1.0 (preserve brightness)
    - Factor: 0.0 (bypassed - enable by setting to 1.0)
  → Color Ramp (optional remapping)
    - Stop 0: #2D261E (Pos 0.0)
    - Stop 1: #8C6D46 (Pos 0.5)
    - Stop 2: #B37233 (Pos 1.0)
  → Principled BSDF Base Color
```

**Alternative: RGB Curves**
- Reduce green channel slightly (0.95 multiplier)
- Warm tilt: boost red in midtones
- Crush blacks to #2D261E minimum

---

### Step 4: Multi-Texture Height Blending

**Use Case:** Blend 4 terrain textures (dirt, rock, grass, sand) based on height maps.

**Complete Node Graph:**

```
TEXTURE INPUTS:
Texture_1 (dirt):    Color → Mix1, Height → Normalize1
Texture_2 (rock):    Color → Mix2, Height → Normalize2
Texture_3 (grass):   Color → Mix3, Height → Normalize3
Texture_4 (sand):    Color → Mix4, Height → Normalize4

HEIGHT NORMALIZATION:
Math Node (Add):
  Height1 + Height2 + Height3 + Height4 = TotalHeight

Math Node (Divide) for each:
  Height1 / TotalHeight = Mask1
  Height2 / TotalHeight = Mask2
  Height3 / TotalHeight = Mask3
  Height4 / TotalHeight = Mask4

BLENDING STACK:
Mix Shader (Factor: Mask1)
  Shader1: Texture_1 → Principled BSDF
  Shader2: Mix Shader (Factor: Mask2)
    Shader1: Texture_2 → Principled BSDF
    Shader2: Mix Shader (Factor: Mask3)
      Shader1: Texture_3 → Principled BSDF
      Shader2: Texture_4 → Principled BSDF
```

**Height Map Requirements:**
- Color Space: Non-Color
- Value range: 0.0 (low) to 1.0 (high)
- Format: Grayscale PNG or single-channel texture

**Optional: Vertex Color Control**

```
Attribute Node (Name: "Col")
  → Separate RGB
    - R channel → Texture_1 weight multiplier
    - G channel → Texture_2 weight multiplier
    - B channel → Texture_3 weight multiplier
    - A channel → Texture_4 weight multiplier
```

Enable in Vertex Paint mode for manual touch-up.

---

### Step 5: Slope-Based Auto-Blending

**Goal:** Automatically apply rock texture on steep slopes, grass on flat areas.

**Node Setup:**

```
Geometry Node (Normal output)
  → Separate XYZ (use Z output = upward facing)
  → ColorRamp
    - Black handle: 0.70 (steep slopes)
    - White handle: 0.90 (flat ground)
    - Interpolation: Ease
  → Math (Multiply by contrast factor 1.5)
  → Mix Shader Factor
    Shader1: Rock material
    Shader2: Grass material
```

**Z-Normal Values:**
- Z = 1.0 → perfectly flat (facing up)
- Z = 0.7 → 45° slope
- Z = 0.0 → vertical cliff

**Blend Hardness Control:**
- ColorRamp range 0.7-0.9 = gradual blend
- ColorRamp range 0.75-0.85 = sharp transition
- Add Math Multiply after ColorRamp to boost contrast

**Advanced: Curvature Detection**

```
Geometry (Normal) → Vector Math (Dot Product with World Up Vector)
  → ColorRamp (curvature mask)
    → Mix with height-based blend
```

---

### Step 6: Procedural Variation

**Problem:** Repeated textures look artificial. Break repetition with procedural noise.

**Color Variation:**

```
Noise Texture
  - Scale: 15-25 (larger = softer variation)
  - Detail: 2
  - Roughness: 0.5
  - Distortion: 0.0
  → ColorRamp (compress range 0.45-0.55 for subtle)
  → Mix RGB (Overlay, Factor: 0.10-0.20)
    Color1: Base texture color
    Color2: Noise mask
  → Principled BSDF Base Color
```

**World-Position Variation (Prevents Identical Duplicates):**

```
Object Info (Location output)
  → Vector Math (Add with Noise Texture Vector input)
  → Noise Texture (now unique per object instance)
```

**Roughness Variation:**

```
Noise Texture (Scale: 50, Detail: 4)
  → Math (Multiply 0.1, then Add base roughness 0.85)
  → Clamp (0.7-0.95 range)
  → Principled BSDF Roughness
```

**Subsurface Variation (Simulates Pigment Density):**

```
Musgrave Texture (Scale: 20, Dimension: 0.5)
  → ColorRamp (0.1-0.2 output range)
  → Principled BSDF Subsurface Weight
```

---

### Step 7: Chiaroscuro Lighting Response

**Goal:** Materials should respond dramatically to low-angle golden light with deep shadows.

**Shadow Terminator Fix:**

```
Object Properties → Shading
  Shadow Terminator Offset: 0.1
  Shadow Terminator Geometry Offset: 0.05
```

Prevents black artifacts on curved geometry.

**Optional: Fill Light Emission (Simulates Ambient Bounce):**

```
Mix Shader (Factor: 0.05-0.10)
  Shader1: Principled BSDF (main material)
  Shader2: Emission Shader
    Color: #A68866 (warm ochre)
    Strength: 0.2
```

Subtle emission prevents pure black shadows, simulates canvas reflection.

**Normal Map Strength Control:**

```
Normal Map Node
  Strength: 0.6-0.8 (subtle bump, not harsh)
  Space: Tangent
  UV Map: [your UV map name]
  → Principled BSDF Normal
```

Lower strength = softer lighting, more painterly.

**World-Scale Shading:**

```
Texture Coordinate (Camera output)
  → Map Range (clamp 0.3-0.7)
  → Mix (Multiply) with Base Color
```

Adds subtle camera-distance-based shading (atmospheric perspective).

---

### Step 8: Baking for Export (CRITICAL)

**Why Bake:** Game engines don't support procedural Blender nodes. All procedural elements MUST be baked to textures.

**Pre-Bake Checklist:**
- [ ] UV unwrap mesh (Smart UV Project or manual unwrap)
- [ ] Create blank Image Texture nodes (name them clearly)
- [ ] Select output image in Shader Editor (must be selected!)
- [ ] Set correct Color Space per map type

**UV Unwrap Settings:**

```
Select mesh → U (Unwrap menu)
  Smart UV Project:
    - Angle Limit: 66°
    - Island Margin: 0.02 (2% padding)
    - Area Weight: 1.0

  OR Manual Unwrap:
    - Mark Seams on sharp edges
    - U → Unwrap
    - Layout in UV Editor (avoid overlap)
```

**Create Bake Target Images:**

```
Shader Editor → Add → Texture → Image Texture
  New Image:
    - Name: "Material_Diffuse"
    - Width/Height: 2048x2048 (or 4096 for hero assets)
    - Color: sRGB or Linear (see table below)
    - Alpha: Unchecked (unless transparency)
    - 32-bit Float: Unchecked (8-bit sufficient)
```

**Image Color Space by Map Type:**

| Map Type       | Color Space | Format | Channels |
|----------------|-------------|--------|----------|
| Base Color     | sRGB        | PNG    | RGB      |
| Roughness      | Non-Color   | PNG    | Grayscale|
| Metallic       | Non-Color   | PNG    | Grayscale|
| Normal         | Non-Color   | PNG    | RGB      |
| Ambient Occl.  | Non-Color   | PNG    | Grayscale|
| Emission       | sRGB        | PNG    | RGB      |
| Height         | Non-Color   | EXR    | Grayscale|

**Baking Steps:**

```
1. Select mesh object
2. Render Properties → Bake panel
3. Bake Type: Combined (or specific type)
4. Influence settings:
   - For Base Color: Only Diffuse + Indirect enabled
   - For Roughness: Use "Roughness" bake type
   - For Normal: Use "Normal" bake type (Space: Tangent)
5. Output settings:
   - Margin Type: Extend
   - Margin: 16 pixels (32 for large textures)
6. Selected to Active: Disabled (single object)
7. Click "Bake"
8. Image Editor → Image → Save As → PNG
```

**Multi-Map Bake Workflow:**

```
Bake 1: Combined (Diffuse only)
  → Save as "Material_Diffuse.png" (sRGB)

Bake 2: Roughness
  → Save as "Material_Roughness.png" (Non-Color)

Bake 3: Normal
  → Save as "Material_Normal.png" (Non-Color)
  → Check OpenGL vs DirectX format (see Step 10)

Bake 4: Ambient Occlusion
  → Save as "Material_AO.png" (Non-Color)
```

**Common Bake Errors:**

| Error | Cause | Fix |
|-------|-------|-----|
| Black output | Wrong image selected | Select target image in Shader Editor |
| Seams visible | Margin too small | Increase to 32px, use Extend mode |
| Wrong colors | sRGB vs Non-Color | Set Color Space correctly |
| Low resolution | Small texture size | Use 2K minimum (2048x2048) |
| No procedural detail | Viewport shading | Switch to Rendered mode before bake |

---

### Step 9: Export Settings

**FBX Export (Unreal Engine, Unity):**

```
File → Export → FBX
  Include:
    ☑ Selected Objects (or all if needed)
    ☑ Mesh
    ☑ Apply Modifiers
    ☐ Animation (disable for static)

  Transform:
    Scale: 1.00
    Apply Scalings: FBX All
    Forward: X Forward
    Up: Z Up
    ☑ Apply Unit
    ☑ Apply Transform

  Geometry:
    ☑ Mesh
    ☐ Smoothing: Face (use Normals instead)
    ☑ Export Subdivision Surface
    ☐ Apply Modifiers (already checked above)

  Armature: (disable all if no skeleton)

  Bake Animation: (disable if static)

  Path Mode: Copy
    ☑ Embed Textures
    ☑ Copy Textures
```

**glTF Export (Godot, Web):**

```
File → Export → glTF 2.0
  Include:
    ☑ Selected Objects
    ☐ Cameras / Punctual Lights (unless needed)

  Transform:
    ☑ Y Up

  Data:
    ☑ UVs
    ☑ Normals
    ☑ Vertex Colors (if using)
    ☑ Materials: Export
    ☐ Compression

  Geometry:
    ☑ Apply Modifiers
    ☑ Tangents

  Material:
    Images: Automatic (embedded or separate)

  Format: glTF Separate (.gltf + .bin + textures)
```

**Post-Export Checklist:**

- [ ] Textures exported to same folder or subfolder
- [ ] Normal map format correct (OpenGL/DirectX)
- [ ] Material paths relative or embedded
- [ ] Scale correct (1 Blender unit = 1 meter default)
- [ ] Up axis correct (Z-up for Unreal/Godot)

---

### Step 10: Normal Map Format Conversion

**Problem:** Blender uses OpenGL normal maps, Unreal uses DirectX.

**Format Differences:**

| Engine | Format | G Channel |
|--------|--------|-----------|
| Blender | OpenGL | G+ (up) |
| Unreal | DirectX | G- (inverted) |
| Godot | OpenGL | G+ (no conversion) |
| Unity | OpenGL | G+ (no conversion) |

**Conversion in Blender (Pre-Export):**

```
Normal Map Image in Compositor:
  Input → Image (Normal map texture)
    → Separate RGBA
      R → Combine RGBA (R)
      G → Math (Multiply -1) → Combine RGBA (G)
      B → Combine RGBA (B)
    → Output to new image
```

**Conversion in Unreal (Import-Time):**

```
Content Browser → Import Normal Map
  In Texture Editor:
    Flip Green Channel: ☑ Enabled
    Compression: TC_Normalmap
    sRGB: ☐ Disabled
```

**Conversion via Script (Batch):**

```python
from PIL import Image
import numpy as np

img = Image.open("Normal_OpenGL.png")
arr = np.array(img)
arr[:,:,1] = 255 - arr[:,:,1]  # Invert G channel
Image.fromarray(arr).save("Normal_DirectX.png")
```

**Verification:**

- OpenGL: Green = top surfaces, Magenta = bottom surfaces
- DirectX: Magenta = top surfaces, Green = bottom surfaces

---

## Complete Node Setups (Copy-Paste Ready)

### 1. Triplanar Box Projection (No UV Required)

```
[Texture Coordinate Node] (Object)
  → [Mapping Node]
      Location: (0, 0, 0)
      Rotation: (0, 0, 0)
      Scale: (8, 8, 8)
    → [Image Texture Node]
        Image: [your texture]
        Projection: Box
        Blend: 0.15
        Extension: Repeat
      → [Principled BSDF] Base Color
```

**When to Use:** Sculpted terrain, organic shapes, UV-less procedural geo.

---

### 2. 4-Texture Height-Based Blend with Vertex Color

```
SETUP:
4x [Image Texture] (Color, Box projection)
4x [Image Texture] (Height, Non-Color, Box projection)

HEIGHT NORMALIZATION:
[Height1] → [Math Add] ← [Height2]
               ↓
          [Math Add] ← [Height3]
               ↓
          [Math Add] ← [Height4]
               ↓
          [TotalHeight]

[Height1] → [Math Divide] (by TotalHeight) → [Mask1]
[Height2] → [Math Divide] (by TotalHeight) → [Mask2]
[Height3] → [Math Divide] (by TotalHeight) → [Mask3]
[Height4] → [Math Divide] (by TotalHeight) → [Mask4]

OPTIONAL VERTEX COLOR CONTROL:
[Attribute "Col"] → [Separate RGB]
  R → [Math Multiply] ← [Mask1] → [AdjustedMask1]
  G → [Math Multiply] ← [Mask2] → [AdjustedMask2]
  B → [Math Multiply] ← [Mask3] → [AdjustedMask3]

SHADER BLENDING:
[Texture1 Color] → [Principled BSDF 1]
  → [Mix Shader] (Fac: AdjustedMask1)
    → [Texture2 Color] → [Principled BSDF 2]
      → [Mix Shader] (Fac: AdjustedMask2)
        → [Texture3 Color] → [Principled BSDF 3]
          → [Mix Shader] (Fac: AdjustedMask3)
            → [Texture4 Color] → [Principled BSDF 4]
              → [Material Output]
```

**Result:** Natural texture transitions based on height variation, manual control via vertex paint.

---

### 3. Painterly Roughness Variation

```
[Noise Texture]
  Scale: 50
  Detail: 4
  Roughness: 0.5
  Distortion: 0.0
  → [Math Multiply] (0.1)
    → [Math Add] (Base: 0.85)
      → [Math Clamp] (Min: 0.7, Max: 0.95)
        → [Principled BSDF] Roughness
```

**Result:** Subtle micro-roughness variation simulating canvas texture.

---

### 4. World-Position Color Variation (Anti-Repetition)

```
[Object Info] Location
  → [Vector Math Add] (B input from:)
    [Noise Texture]
      Scale: 15
      Detail: 2
      → [ColorRamp]
          Black (0.45) → Dark Umber #2D261E
          White (0.55) → Raw Sienna #A68758
        → [Mix RGB] Overlay (Fac: 0.15)
            Color1: [Base Texture Color]
            Color2: [ColorRamp Output]
          → [Principled BSDF] Base Color
```

**Result:** Each mesh instance gets unique color variation, breaks tiling.

---

### 5. Low-Specular Principled BSDF (Rembrandt Preset)

```
[Your Texture Blending Setup]
  → [HSV Node]
      Hue: 0.0
      Saturation: 0.7
      Value: 1.0
      Factor: 0.0 (bypassed - enable by setting to 1.0)
    → [Principled BSDF]
        Base Color: [from HSV]
        Metallic: 0.0
        Roughness: 0.85
        Specular IOR Level: 0.2
        Specular Tint: 0.0
        Subsurface Weight: 0.15
        Subsurface Radius: (0.8, 0.5, 0.3)
        Subsurface Scale: 0.1
        Coat Weight: 0.0
        Normal: [from Normal Map if exists]
      → [Material Output] Surface
```

**Result:** Matte, warm, subsurface-scattered surface matching Dutch Golden Age paintings.

---

### 6. Slope-Based Auto-Blend (Rock on Cliffs, Grass on Flats)

```
[Geometry] Normal
  → [Separate XYZ] Z output
    → [ColorRamp]
        Black (0.70) → Steep
        White (0.90) → Flat
      → [Math Multiply] (1.5 for contrast)
        → [Math Clamp] (0-1)
          → [Mix Shader] Factor
              Shader1: [Rock Material]
              Shader2: [Grass Material]
            → [Material Output]
```

**Result:** Automatic cliff texture on slopes, grass on flat ground, smooth transitions.

---

## Debugging Checklist

### Material Looks Different in Viewport vs Render

**Cause:** Viewport uses Eevee approximation, render uses Cycles.

**Fix:**
- Viewport Shading: Switch to **Rendered** mode (Z → Rendered)
- Match render engine: Render Properties → Render Engine: Cycles
- Enable Ambient Occlusion in Eevee (if using viewport shading)

---

### Normal Map Appears Inverted (Bumps = Dents)

**Cause:** OpenGL vs DirectX format mismatch.

**Fix:**
- Normal Map node → Strength: Set to -1.0 (flips all axes)
- OR: Convert texture G channel before import (see Step 10)
- OR: In Image Texture node, set Color Space: Non-Color

---

### UV Seams Visible After Baking

**Cause:** Bake margin too small, texture bleeding.

**Fix:**
- Bake settings → Margin: Increase to 32 pixels
- Margin Type: Extend (not Adjacent Faces)
- Image Texture → Extension: Clip (not Repeat)
- Re-unwrap with larger Island Margin (0.03-0.05)

---

### Colors Oversaturated or Wrong Tone

**Cause:** sRGB doubling or missing color management.

**Fix:**
- Render Properties → Color Management:
  - View Transform: Filmic
  - Look: Medium Contrast
- Image Texture → Color Space: sRGB (for color maps)
- Add HSV node → Saturation: 0.7, Factor: 0.0 (bypassed by default)
- Verify texture files aren't pre-gamma-corrected

---

### Procedural Noise Doesn't Export

**Cause:** Game engines can't interpret Blender nodes.

**Fix:**
- **MUST BAKE** all procedural elements to textures (see Step 8)
- No exceptions - noise, Voronoi, Musgrave, all must be baked
- After baking, replace procedural nodes with baked Image Texture nodes

---

### Texture Repetition Too Obvious

**Cause:** Uniform tiling without variation.

**Fix:**
- Add Noise Texture color variation (Step 6)
- Use Object Info Location to break repetition
- Blend 2-3 textures at different scales
- Add Detail texture layer (high-frequency overlay)

---

### Material Too Shiny / Reflective

**Cause:** Default Principled BSDF specular is too high.

**Fix:**
- Principled BSDF → Specular IOR Level: 0.2 (from 0.5 default)
- Roughness: 0.85 minimum (matte)
- Coat Weight: 0.0 (no varnish)
- Add roughness variation (Step 3)

---

### Shadows Too Dark / No Detail

**Cause:** No subsurface scatter or fill light.

**Fix:**
- Principled BSDF → Subsurface Weight: 0.15
- Add subtle emission fill light (Step 7)
- Shadow Terminator Offset: 0.1 (Object Properties)
- World Settings → Surface → Strength: 0.05 (ambient)

---

### Export to Unreal Shows Wrong Scale

**Cause:** Blender unit scale vs Unreal unit scale.

**Fix:**
- FBX Export → Apply Scalings: FBX All
- Unreal Import → Transform → Import Uniform Scale: 100 (if 1 BU = 1cm)
- Verify: 1 Blender unit = 1 meter = 100 Unreal units
- Scene Properties → Unit Scale: 1.0 (don't change)

---

## Quality Targets

### Resolution Guidelines

| Asset Type | Texture Resolution | Notes |
|------------|-------------------|-------|
| Terrain (third-person 5-15m) | 2048x2048 | Standard |
| Terrain (first-person <2m) | 4096x4096 | High detail |
| Props (background) | 1024x1024 | Optimized |
| Props (hero/interactable) | 2048x2048 | Standard |
| Characters (main) | 4096x4096 | High detail |

### Performance Targets

- **Texture Samples:** Max 8 per material (each Image Texture = 1 sample)
- **Shader Complexity:** Keep under 200 instructions (check Stats in Shader Editor)
- **Draw Calls:** Batch materials (same shader = 1 draw call)
- **Target FPS:** 60 FPS with 100+ terrain chunks visible (RTX 4070 Ti)

### Artistic Targets (Rembrandt Style)

- **Roughness Range:** 0.70-0.95 (matte, no gloss)
- **Saturation:** 0.65-0.75 (muted earth tones)
- **Contrast Ratio:** 8:1 minimum (deep shadows)
- **Color Palette:** 80% earth tones, 20% accent colors
- **Specular:** Minimal (<0.25), warm tinted if present

### Validation Checklist

Before considering material complete:

- [ ] Material works in Rendered viewport mode (Cycles)
- [ ] All procedural nodes baked to textures
- [ ] Textures saved as PNG (or EXR for height)
- [ ] Color Space correct (sRGB for color, Non-Color for data)
- [ ] UV unwrap complete (no stretching visible)
- [ ] Normal map format correct for target engine
- [ ] Material exports correctly (FBX/glTF test)
- [ ] Roughness in 0.7-0.95 range (painterly)
- [ ] Colors match Dutch Golden Age palette
- [ ] No visible UV seams after baking
- [ ] Texture resolution meets quality target
- [ ] Max 8 texture samples used

---

## Advanced Techniques

### Detail Texture Overlay (Break Macro Tiling)

```
[Base Texture] (Scale: 5)
  → [Mix RGB] Overlay (Fac: 0.3)
    [Detail Texture] (Scale: 50, high-frequency)
  → [Principled BSDF]
```

### Vertex AO Baking (Free Shadows)

```
1. Duplicate mesh
2. Add Modifier → Data Transfer
   Source: Original mesh
   Data Type: Vertex Colors
   Layers: VCol → VCol (AO)
3. Generate: Bake AO to vertex colors
4. In material:
   [Attribute "AO"] → [Mix RGB] Multiply (Fac: 0.7)
     Color1: Base Color
     Color2: Black
```

### Animated Roughness (Wet Ground Effect)

```
[Value Node] (driven by animation)
  → [Math Clamp] (0.3-0.9 range)
    → [Principled BSDF] Roughness

Keyframe: Start 0.85 (dry) → End 0.3 (wet)
```

### PBR Texture Pack Import (CC0 Assets)

```
For standard PBR pack (Color, Roughness, Normal, AO):

1. Load Color → sRGB → Base Color
2. Load Roughness → Non-Color → Roughness
3. Load Normal → Non-Color → Normal Map → Normal
4. Load AO → Non-Color → Mix (Multiply) with Color
5. Apply Rembrandt corrections:
   - HSV Saturation 0.7, Factor 0.0 (node present but bypassed)
   - Roughness Math Add 0.15 (increase matteness)
```

---

## MCP Integration Notes

When using Blender MCP, request materials using this structure:

```
"Create a terrain material with:
- 4 textures: dirt, rock, grass, sand
- Height-based blending
- Triplanar projection (no UVs)
- Rembrandt painterly style (matte, warm, low-specular)
- Bake to 2K textures for Unreal export
- Export as FBX with embedded textures"
```

MCP tools to use:
- `create_material` - Initialize material
- `add_texture_node` - Add image textures
- `connect_nodes` - Build node graph
- `bake_material` - Bake procedural to textures
- `export_fbx` - Export with correct settings

---

## References

**Blender Docs:**
- Shader Nodes: https://docs.blender.org/manual/en/latest/render/shader_nodes/
- Baking: https://docs.blender.org/manual/en/latest/render/cycles/baking.html

**Color Palette Research:**
- Rembrandt Palette Analysis: https://www.rembrandthuis.nl/en/learn/pigments/
- Dutch Golden Age Colors: Burnt Umber, Raw Umber, Yellow Ochre, Raw Sienna

**Texture Resources (CC0):**
- Poly Haven: https://polyhaven.com/textures
- ambientCG: https://ambientcg.com/
- 3D Textures: https://3dtextures.me/

---

**Skill Version:** 1.0
**Last Updated:** 2026-02-02
**Maintained By:** Claude Code (oh-my-claudecode)
