---
name: natural-rock-placement
description: Place rocks and boulders naturally in 3D environments using Blender workflows, scale rules, grouping, terrain integration, and composition guidance.
---

# Natural Rock & Boulder Placement Skill

A comprehensive guide for placing rocks and boulders in 3D environments with natural, visually pleasing compositions using Blender MCP. Based on AAA studio techniques, GDC presentations, and professional environment art practices.

---

## 1. SCALE & PROPORTION

### Real-World Boulder Sizes (in Meters)

**Player Reference**: Standard character is 1.8m height. Always verify scale against this.

| Category | Size Range | Polygon Budget | Use Case | Frequency |
|----------|------------|----------------|----------|-----------|
| **Small Detail** | 0.3-0.5m | 200-500 tris | Gap filling, ground scatter | 60-70% |
| **Medium** | 0.5-1.5m | 500-1,200 tris | Secondary composition | 20-25% |
| **Large** | 1.5-3.0m | 1,200-3,000 tris | Structural elements | 10-15% |
| **Hero** | 3.0-5.0m+ | 3,000-8,000 tris | Focal points, landmarks | 2-5% |

### Golden Ratio Scale Relationships (1:1.618)

For a **4-meter hero boulder**:
- Large supporting rocks: **2.0-2.6m** (50-65% of hero)
- Medium transition rocks: **1.2-1.8m** (30-45% of hero)
- Small scatter rocks: **0.6-1.0m** (15-25% of hero)

### The 80/20 Distribution Rule

- **80%** of visible rocks: Small-to-medium (background, filler)
- **20%** of visible rocks: Large-to-hero (focal points, anchors)

### Scale Variation Within Categories

**CRITICAL**: Apply non-uniform scaling to EVERY instance:
- Minimum range: **0.8x to 1.2x** base scale
- Aggressive for small rocks: **0.6x to 1.4x**
- Conservative for heroes: **0.9x to 1.1x**

---

## 2. GROUNDING & BURIAL

### Burial Depth by Rock Type

**Industry standard: 20-30% of rock height below ground**

| Rock Type | Burial % | Visual Rationale |
|-----------|----------|------------------|
| Hero boulders (stable, ancient) | 25-35% | Weathered into landscape |
| Large rocks (structural) | 20-30% | Established presence |
| Medium rocks (scattered) | 15-25% | Recently deposited |
| Small detail rocks | 10-20% | Surface scatter |

### Burial Calculation

```python
# Example: 2.4m tall rock
rock_height = 2.4  # meters
burial_percent = 0.25  # 25%
burial_depth = rock_height * burial_percent  # 0.6m
z_position = ground_z - burial_depth  # Rock extends 0.6m below ground
```

### Slope Considerations

On steep terrain (>30° angle), reduce burial to **15-20%** as rocks naturally appear more exposed on vertical surfaces.

### Avoiding "Floating Rock" Appearance

1. **Shadow Contact**: Paint ambient occlusion into terrain vertex colors at rock contact points
2. **Terrain Resolution**: Locally subdivide terrain to 300+ subdivisions around hero rocks
3. **Edge Blending**: Never place rocks exactly at Z=0 with no interpenetration

### Ground Blending Techniques

**Vertex Painting Setup**:
- Red channel: Rock base material
- Green channel: Grass/vegetation
- Blue channel: Dirt/soil
- Paint soft gradients (20-30% brush strength) over 0.3-0.5m distance

**Grass Exclusion**: Create landscape mask layer where rocks exist, exclude foliage from that layer

**Debris Scatter**: Place small pebbles (0.05-0.15m) around large rock bases using same material

---

## 3. CLUSTERING & DISTRIBUTION

### Poisson Disc Sampling (Gold Standard)

Generates points that are:
- Randomly distributed (no visible grid)
- Never closer than minimum radius (prevents overlap)
- Tightly packed (maximizes coverage)

```python
import bpy
import random
import math
from mathutils import Vector

def poisson_disc_sampling(center, radius, min_distance, num_samples, k=30):
    """
    Poisson disc sampling for natural rock distribution

    Args:
        center: Vector - center of placement area
        radius: float - radius of placement area
        min_distance: float - minimum distance between points
        num_samples: int - approximate number of points to generate
        k: int - rejection samples per point (30 for tight, 10 for sparse)
    """
    cell_size = min_distance / math.sqrt(2)
    grid = {}
    points = []
    active = []

    # First point at center
    first_point = center.copy()
    points.append(first_point)
    active.append(first_point)

    grid_key = (int(first_point.x / cell_size), int(first_point.y / cell_size))
    grid[grid_key] = first_point

    while active and len(points) < num_samples:
        idx = random.randint(0, len(active) - 1)
        point = active[idx]
        found = False

        for _ in range(k):
            # Generate candidate in annulus
            angle = random.uniform(0, 2 * math.pi)
            dist = random.uniform(min_distance, 2 * min_distance)
            candidate = Vector((
                point.x + dist * math.cos(angle),
                point.y + dist * math.sin(angle),
                center.z
            ))

            # Check if within bounds
            if (candidate - center).length > radius:
                continue

            # Check minimum distance from all neighbors
            valid = True
            cx, cy = int(candidate.x / cell_size), int(candidate.y / cell_size)
            for dx in range(-2, 3):
                for dy in range(-2, 3):
                    neighbor_key = (cx + dx, cy + dy)
                    if neighbor_key in grid:
                        if (candidate - grid[neighbor_key]).length < min_distance:
                            valid = False
                            break
                if not valid:
                    break

            if valid:
                points.append(candidate)
                active.append(candidate)
                grid[(cx, cy)] = candidate
                found = True
                break

        if not found:
            active.pop(idx)

    return points
```

### Spacing Formula

```
min_distance = (rock_A_radius + rock_B_radius) * separation_multiplier
```

**Separation Multipliers**:

| Rock A + Rock B | Multiplier | Visual Result |
|-----------------|------------|---------------|
| Hero + Hero | 3.0-4.0 | Distinct focal points |
| Hero + Large | 2.0-2.5 | Clear hierarchy |
| Large + Large | 2.0-2.5 | Clear hierarchy |
| Large + Medium | 1.5-2.0 | Clustered but discernible |
| Medium + Medium | 1.5-2.0 | Clustered but discernible |
| Medium + Small | 1.2-1.5 | Dense natural scatter |
| Small + Small | 1.2-1.5 | Dense natural scatter |

**Example**: Hero rock (2m radius) + Large rock (0.75m radius)
- Base sum: 2.75m
- With 2.5x multiplier: **6.9m minimum spacing**

### Golden Angle Spiral (137.5°)

For radial distributions around focal points:

```python
def golden_angle_distribution(center, count, max_radius, start_radius=0):
    """Generate points using golden angle spiral"""
    golden_angle = 137.5 * (math.pi / 180)
    points = []

    for i in range(count):
        angle = i * golden_angle
        # Spiral expands outward with sqrt for even area distribution
        radius = start_radius + (max_radius - start_radius) * math.sqrt(i / count)
        pos = Vector((
            center.x + radius * math.cos(angle),
            center.y + radius * math.sin(angle),
            center.z
        ))
        points.append(pos)

    return points
```

### Cluster Compositions

**Small Cluster (3-5 rocks)** - Triangular, asymmetric:
- 1 hero/anchor (100% scale)
- 2 medium support (40-60% scale)
- 2-3 small detail (20-30% scale)

**Medium Cluster (5-9 rocks)**:
- 1 hero (largest)
- 2-3 large secondary (60-80% scale)
- 3-4 medium tertiary (30-50% scale)
- 3-5 small scatter (15-25% scale)

**Large Cluster (9-15+ rocks)**:
- 1-2 hero anchors
- 3-5 large structural
- 5-8 medium rocks
- Open-ended small scatter

### Cluster Ratio Rules

- **Odd numbers ONLY**: 3, 5, 7, 9, 11, 13 rocks per cluster
- **Size ratio between levels**: Minimum 1.5:1, ideal 1.8:1 to 2.5:1
- **Large-to-small ratio**: Minimum 3:1, up to 10:1 for drama

---

## 4. MATERIAL & TEXTURE

### PBR Settings for Rocks

**CRITICAL VALUES**:

| Parameter | Correct Value | Common Mistake |
|-----------|---------------|----------------|
| **Roughness** | 0.35-0.45 | Too high (0.7-1.0) or too low (<0.3) |
| **Specular** | 0.25-0.33 | Left at default 0.5 |
| **Metallic** | 0.0 (always!) | Any value above 0 |
| **Base Color** | 0.2-0.4 (sRGB) | Too dark (<0.15) |

**By Rock Condition**:
- Dry rock: Roughness 0.4-0.5
- Slightly weathered: Roughness 0.35-0.45
- Wet rock: Roughness 0.2-0.35

### Fixing Shiny/Plastic Rock Materials

```python
import bpy

def fix_rock_materials():
    """Fix overly reflective rock materials"""
    for mat in bpy.data.materials:
        if mat.use_nodes and mat.node_tree:
            for node in mat.node_tree.nodes:
                if node.type == 'BSDF_PRINCIPLED':
                    # Fix roughness (0.35-0.45 range)
                    if 'Roughness' in node.inputs:
                        current = node.inputs['Roughness'].default_value
                        if current < 0.35 or current > 0.6:
                            node.inputs['Roughness'].default_value = 0.4

                    # Fix specular (reduce to 0.25-0.33)
                    for spec_name in ['Specular IOR Level', 'Specular']:
                        if spec_name in node.inputs:
                            node.inputs[spec_name].default_value = 0.3
                            break

                    # Ensure metallic is 0
                    if 'Metallic' in node.inputs:
                        node.inputs['Metallic'].default_value = 0.0

                    print(f"Fixed: {mat.name}")
```

### Triplanar/Box Projection Settings

**When to Use**: Sculpted rocks without clean UVs, procedural terrain, rotating rocks

**Blender Settings for Image Texture Nodes**:
```python
tex_node.projection = 'BOX'
tex_node.projection_blend = 0.2  # 0.2-0.4 for natural rocks
```

**Blend Sharpness Guide**:
- 1.0: Too soft, visible gradients
- **2.0-3.0**: Natural rock blending (RECOMMENDED)
- 4.0-6.0: Architectural, hard edges
- 8.0+: Visible seams, avoid

### Color Matching to Scene

**Albedo Value Ranges**:
- Wet rock: 0.08-0.23 (dark)
- Dry rock: 0.15-0.4 (mid-range)
- Weathered/lichen: 0.25-0.5 (lighter)
- Sandstone/limestone: 0.35-0.55 (bright)

**Saturation**: Rocks should be **15-30% less saturated** than vegetation

---

## 5. COMPOSITION

### Rule of Thirds for Hero Rocks

Place hero boulders at **power points** (grid intersections):
- Top-left: Dynamic, suggests upward movement
- Top-right: Stable, traditional strength
- Bottom-left: Grounded, supports mid-ground
- Bottom-right: Balanced, gentle focal point

**NEVER place hero rocks at dead center** - appears static and artificial.

### Sanzon (Three-Stone) Principle

Arrange three stones in a **scalene triangle** (all sides different lengths):

```
        [PRINCIPAL]          <- Tallest, 100% scale
           /    \               Position at 61.8% from frame edge
          /      \              Slight tilt toward viewer (5-10°)
         /        \
   [SECONDARY]  [TERTIARY]   <- 60-75% and 30-50% of principal
                                Form 60-80° angle from viewer
```

**Extended Groupings**:
- 5 stones: Principal + 2 secondary (60%, 70%) + 2 tertiary (30%, 40%)
- 7 stones: 5-stone base + 2 accent (20%, 25%)

### Visual Hierarchy Layers

| Depth Layer | Distance | Rock Types | % of Frame |
|-------------|----------|------------|------------|
| Foreground | 0-5m | Small-medium detail | 10-20% |
| Mid-ground | 5-20m | Hero + large structural | 30-50% |
| Background | 20m+ | Silhouette shapes, LODs | 20-30% |

### Empty Space (Ma)

Leave **40-50% of hero rock cluster area open** (no rocks). Creates breathing room and guides navigation.

---

## 6. COMMON MISTAKES TO AVOID

### The Big 7 Errors

1. **Uniform Spacing (Grid Syndrome)**: Rocks align to grid, equal spacing everywhere
2. **Identical Scale Repetition**: Same mesh at 1:1 scale creates copy-paste look
3. **Insufficient Burial**: Rocks sitting on surface appear "pasted"
4. **No Size Variation**: All "medium rocks" exactly the same size
5. **Perfect Alignment**: Rocks in straight lines or symmetric patterns
6. **Symmetrical Placement**: Mirror arrangements across pathways
7. **Overly Reflective Materials**: Wrong PBR values create plastic appearance

### Hiding Repetition

**Rotation Protocol**:
- Random rotation all axes: X (0-360°), Y (0-360°), Z (0-15° tilt)
- Avoid 90° increments - use primes: 37°, 73°, 127°
- Flip meshes (scale -1 on X or Y) for mirror variations

**Scale Protocol**:
- ALWAYS non-uniform: X (0.8-1.2), Y (0.85-1.15), Z (0.9-1.1)
- More aggressive for small rocks: 0.6-1.4

**Never place two identical rocks within 5m of each other**

---

## 7. COMPLETE IMPLEMENTATION

### Main Placement Function

```python
import bpy
import random
import math
from mathutils import Vector

def place_rocks_professionally(rock_sources, terrain_center, terrain_radius, terrain_z):
    """
    Professional rock placement following AAA environment art principles

    Args:
        rock_sources: dict with keys 'hero', 'large', 'medium', 'small' containing object lists
        terrain_center: Vector (x, y) center of placement area
        terrain_radius: float - radius of terrain
        terrain_z: float - ground level Z coordinate
    """
    placed_rocks = []
    placed_positions = []  # Track for spacing validation

    # Minimum spacing by category (meters)
    MIN_SPACING = {
        'hero': 15.0,    # Hero rocks need lots of space (3-4x their size)
        'large': 6.0,    # Large rocks well separated
        'medium': 2.5,   # Medium rocks moderate spacing
        'small': 1.0     # Small rocks can be closer
    }

    # Burial depth by category
    BURIAL_DEPTH = {
        'hero': 0.30,    # 30% buried
        'large': 0.25,   # 25% buried
        'medium': 0.20,  # 20% buried
        'small': 0.15    # 15% buried
    }

    # Scale ranges by category (in meters)
    SCALE_RANGE = {
        'hero': (3.0, 5.0),
        'large': (1.5, 3.0),
        'medium': (0.5, 1.5),
        'small': (0.3, 0.5)
    }

    def is_valid_position(x, y, category):
        """Check spacing from existing rocks"""
        min_dist = MIN_SPACING[category]
        for px, py, pcat in placed_positions:
            dist = math.sqrt((x - px)**2 + (y - py)**2)
            required = max(min_dist, MIN_SPACING.get(pcat, 1.0))
            if dist < required:
                return False
        return True

    def find_valid_position(center_x, center_y, radius_min, radius_max, category, attempts=50):
        """Find position with proper spacing"""
        for _ in range(attempts):
            angle = random.uniform(0, 2 * math.pi)
            dist = random.uniform(radius_min, radius_max)
            x = center_x + dist * math.cos(angle)
            y = center_y + dist * math.sin(angle)
            if is_valid_position(x, y, category):
                return x, y
        return None, None

    def place_rock(source_obj, x, y, scale, category):
        """Place a single rock with proper burial and variation"""
        if source_obj is None:
            return None

        # Create instance
        new_rock = source_obj.copy()
        new_rock.data = source_obj.data  # Share mesh data

        # Calculate burial
        burial = scale * BURIAL_DEPTH[category]

        # Position with burial
        new_rock.location = Vector((x, y, terrain_z - burial))

        # Non-uniform scale (0.85-1.15 variation)
        sx = scale * random.uniform(0.9, 1.1)
        sy = scale * random.uniform(0.85, 1.15)
        sz = scale * random.uniform(0.8, 1.0)  # Slightly flattened
        new_rock.scale = (sx, sy, sz)

        # Natural rotation
        new_rock.rotation_euler = (
            random.uniform(-0.1, 0.1),      # Slight X tilt
            random.uniform(-0.1, 0.1),      # Slight Y tilt
            random.uniform(0, 2 * math.pi)  # Full Z rotation
        )

        # Make visible
        new_rock.hide_set(False)
        new_rock.hide_render = False

        bpy.context.collection.objects.link(new_rock)
        placed_positions.append((x, y, category))

        return new_rock

    center_x, center_y = terrain_center.x, terrain_center.y

    # === HERO ROCKS (3 rocks in Sanzon arrangement) ===
    print("Placing HERO rocks (Sanzon arrangement)...")

    # Principal stone at golden ratio position
    principal_x = center_x + terrain_radius * 0.3
    principal_y = center_y
    principal_scale = random.uniform(*SCALE_RANGE['hero'])

    rock = place_rock(
        random.choice(rock_sources.get('hero', rock_sources.get('large', []))),
        principal_x, principal_y, principal_scale, 'hero'
    )
    if rock:
        rock.name = "hero_principal"
        placed_rocks.append(rock)
        print(f"  Principal: {principal_scale:.1f}m at ({principal_x:.0f}, {principal_y:.0f})")

    # Secondary stone (60-75% of principal, offset)
    secondary_scale = principal_scale * random.uniform(0.6, 0.75)
    secondary_x = principal_x + principal_scale * 1.2
    secondary_y = principal_y - principal_scale * 0.8

    rock = place_rock(
        random.choice(rock_sources.get('hero', rock_sources.get('large', []))),
        secondary_x, secondary_y, secondary_scale, 'hero'
    )
    if rock:
        rock.name = "hero_secondary"
        rock.rotation_euler.z = 0.3  # Angled toward principal
        placed_rocks.append(rock)
        print(f"  Secondary: {secondary_scale:.1f}m")

    # Tertiary stone (30-50% of principal)
    tertiary_scale = principal_scale * random.uniform(0.3, 0.5)
    tertiary_x = principal_x - principal_scale * 0.9
    tertiary_y = principal_y + principal_scale * 0.6

    rock = place_rock(
        random.choice(rock_sources.get('large', rock_sources.get('medium', []))),
        tertiary_x, tertiary_y, tertiary_scale, 'large'
    )
    if rock:
        rock.name = "hero_tertiary"
        rock.rotation_euler.z = -0.4  # Angled toward principal
        placed_rocks.append(rock)
        print(f"  Tertiary: {tertiary_scale:.1f}m")

    # === LARGE ROCKS (5 rocks, supporting composition) ===
    print("\nPlacing LARGE rocks...")
    large_count = 0
    for i in range(5):
        x, y = find_valid_position(center_x, center_y,
                                    terrain_radius * 0.2, terrain_radius * 0.7, 'large')
        if x is not None:
            scale = random.uniform(*SCALE_RANGE['large'])
            rock = place_rock(
                random.choice(rock_sources.get('large', [])),
                x, y, scale, 'large'
            )
            if rock:
                rock.name = f"large_rock_{large_count:02d}"
                placed_rocks.append(rock)
                large_count += 1
    print(f"  Placed {large_count} large rocks")

    # === MEDIUM ROCKS (9 rocks, fill and transition) ===
    print("\nPlacing MEDIUM rocks...")
    medium_count = 0
    for i in range(9):
        x, y = find_valid_position(center_x, center_y,
                                    terrain_radius * 0.1, terrain_radius * 0.8, 'medium')
        if x is not None:
            scale = random.uniform(*SCALE_RANGE['medium'])
            rock = place_rock(
                random.choice(rock_sources.get('medium', [])),
                x, y, scale, 'medium'
            )
            if rock:
                rock.name = f"medium_rock_{medium_count:02d}"
                placed_rocks.append(rock)
                medium_count += 1
    print(f"  Placed {medium_count} medium rocks")

    # === SMALL ROCKS (15 rocks, detail scatter) ===
    print("\nPlacing SMALL rocks...")
    small_count = 0
    for i in range(15):
        x, y = find_valid_position(center_x, center_y,
                                    terrain_radius * 0.05, terrain_radius * 0.9, 'small')
        if x is not None:
            scale = random.uniform(*SCALE_RANGE['small'])
            rock = place_rock(
                random.choice(rock_sources.get('small', rock_sources.get('medium', []))),
                x, y, scale, 'small'
            )
            if rock:
                rock.name = f"small_rock_{small_count:02d}"
                placed_rocks.append(rock)
                small_count += 1
    print(f"  Placed {small_count} small rocks")

    # === SUMMARY ===
    print(f"\n=== TOTAL: {len(placed_rocks)} rocks ===")
    print(f"  Hero: 3 (Sanzon arrangement)")
    print(f"  Large: {large_count}")
    print(f"  Medium: {medium_count}")
    print(f"  Small: {small_count}")

    return placed_rocks
```

### Usage Example

```python
# Get terrain info
terrain = bpy.data.objects.get('Terrain')
terrain_z = terrain.location.z if terrain else 0
center = Vector((terrain.location.x, terrain.location.y, terrain_z))
radius = 100  # meters

# Organize rock sources by category
rock_sources = {
    'hero': [bpy.data.objects.get('cliff_rock'), bpy.data.objects.get('large_boulder')],
    'large': [bpy.data.objects.get('boulder_01'), bpy.data.objects.get('boulder_02')],
    'medium': [bpy.data.objects.get('rock_01'), bpy.data.objects.get('rock_02')],
    'small': [bpy.data.objects.get('pebble_01'), bpy.data.objects.get('pebble_02')]
}

# Remove None values
for category in rock_sources:
    rock_sources[category] = [r for r in rock_sources[category] if r is not None]

# Place rocks
placed = place_rocks_professionally(rock_sources, center, radius, terrain_z)
```

---

## Quick Reference Checklist

Before finalizing rock placement, verify:

- [ ] **Scale ranges correct**: Hero 3-5m, Large 1.5-3m, Medium 0.5-1.5m, Small 0.3-0.5m
- [ ] **Burial depth**: 20-30% of height below ground
- [ ] **Odd numbers** in each grouping (3, 5, 7, 9)
- [ ] **Sanzon arrangement** for hero rocks (scalene triangle)
- [ ] **Spacing validated**: No rocks closer than minimum distance
- [ ] **No grid patterns**: Positions have noise, no alignments
- [ ] **Rotation varied**: Full Z rotation, slight X/Y tilt, no parallels
- [ ] **Non-uniform scale**: 0.85-1.15 variation on every rock
- [ ] **PBR values**: Roughness 0.35-0.45, Specular 0.25-0.33, Metallic 0
- [ ] **Box projection**: Enabled on all rock textures, blend 0.2-0.3
- [ ] **Rule of thirds**: Hero rocks at power points, not centered
- [ ] **40-50% empty space** around hero clusters

---

## References

- Professional Environment Art Techniques (AAA Studio Research 2024-2025)
- GDC Environment Art Talks
- Japanese Garden Stone Arrangement (Sanzon Ishigumi)
- Poisson Disc Sampling (Bridson's Algorithm)
- PBR Material Standards for Game Environments
