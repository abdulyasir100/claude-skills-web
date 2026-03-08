---
name: pmx-to-vrm
description: Convert MMD (PMX) models to VRM format with facial expressions, materials, and physics. Uses Blender 3.6 with CATS plugin, mmd_tools, and VRM addon. Automates via Blender MCP where possible.
---

# PMX to VRM Conversion

Convert MMD (MikuMikuDance) PMX models into fully functional VRM avatars with facial expressions, proper materials, and spring bone physics. Based on established community workflows for Hololive/VTuber model porting.

## Prerequisites

- **Blender 3.6** (Steam version at `F:\SteamLibrary\steamapps\common\Blender`)
- **Blender Addons** (all pre-installed):
  - CATS Blender Plugin (unofficial Blender 3.6 fork): `Cats-Blender-Plugin-Unofficial--blender-36`
  - mmd_tools: `mmd_tools` (for PMX import)
  - VRM format plugin (for VRM export and bone assignment)
- **Blender MCP** connected (`uvx blender-mcp`, addon on port 9876)
- Source PMX model file

## Pipeline Overview

```
PMX → Import (mmd_tools) → Fix Model (CATS) → Translate Bones → Materials (MToon2)
    → Texture Fixes → Shape Keys (bone morphs → vertex) → VRM Expression Mapping
    → Spring Bone Physics → Export VRM
```

## Step 1: Import and Fix Model

### 1.1 Import PMX
```python
import bpy
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
bpy.ops.mmd_tools.import_model(filepath=r"<PMX_PATH>")
```

### 1.2 Select All and Make Single User
Prevents conflicts during export:
```python
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.make_single_user(object=True, obdata=True, material=True)
```

### 1.3 Fix Model with CATS
Merges meshes, standardizes bones, fixes rig for VRM/VRChat compatibility:
```python
bpy.ops.cats_armature.fix()
```

### 1.4 Translate Bone Names
Convert Japanese bone names to English for VRM compatibility:
```python
bpy.ops.cats_translate.all()  # or translate specific elements
```

### 1.5 Apply Transforms
```python
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)
```

## Step 2: Material Conversion

### 2.1 Convert to VRM MToon2
For each material, enable VRM MToon2 shader to preserve anime-style flat shading. This initially turns the model white — textures must be manually reassigned.

**Material properties to set per material:**
- Enable VRM MToon2 material
- Reassign Base Color texture (body, hair, eyes, clothes)
- Set Alpha Mode: `Opaque` for solid parts, `Transparent` for eyelashes/hair fringes
- Enable Double-Sided rendering where needed (fixes black shading on limbs)

### 2.2 Common Texture Issues
| Issue | Fix |
|-------|-----|
| White/blank eyes | Reassign eye texture to Base Color slot |
| Missing hair texture | Reassign hair texture manually |
| Hidden feet/body parts visible | Set Alpha Mode to Transparent, adjust alpha value |
| Black shading on legs | Enable Double-Sided rendering |
| Face shading artifacts | Set face material Alpha Mode to Transparent |

## Step 3: Facial Expressions (Shape Keys)

### 3.1 The Core Problem
Many MMD/TDA models store facial expressions as **bone morphs** (not vertex morphs). VRM requires **vertex-based shape keys** (blend shapes). Bone morphs don't carry over during conversion.

### 3.2 Check What Morph Types Exist
```python
import bpy
for obj in bpy.data.objects:
    if obj.mmd_type == 'ROOT':
        root = obj.mmd_root
        print(f"Vertex Morphs: {len(root.vertex_morphs)}")
        print(f"Bone Morphs: {len(root.bone_morphs)}")
        for m in root.bone_morphs:
            print(f"  {m.name}")
```

### 3.3 Convert Bone Morphs to Vertex Shape Keys
For models with bone morphs (like TDA models), each bone morph must be baked into a vertex shape key:
- Use CATS `pose_to_shape` to convert each bone pose to a shape key
- Or manually: activate bone morph → apply as shape key → repeat

### 3.4 Map Shape Keys to VRM Expressions
Using the VRM Blend Shape Proxy, bind shape keys to VRM expression presets:

| VRM Expression | Japanese Morph Name | Shape Key Purpose |
|---------------|--------------------|--------------------|
| A (aa) | あ | Mouth open "ah" |
| I (ih) | い | Mouth "ee" |
| U (ou) | う | Mouth "oo" |
| E (ee) | え | Mouth "eh" |
| O (oh) | お | Mouth "oh" |
| Blink | まばたき | Both eyes closed |
| Blink_L | ウィンク | Left eye wink |
| Blink_R | ウィンク右 | Right eye wink |
| Joy | 笑い / にっこり | Happy/smile eyes |
| Angry | 怒り | Angry expression |
| Sorrow | 困る | Sad/troubled |
| Fun | にっこり2 | Playful expression |
| Surprised | びっくり | Surprised eyes |

### 3.5 Bind in VRM Blend Shape Proxy
1. Open VRM panel → Blend Shape Proxy
2. For each expression preset, click Binds → Add
3. Select Body mesh → pick the corresponding shape key
4. Use slider to preview (0.0 = off, 1.0 = full)
5. Multiple shape keys can be layered for complex expressions

## Step 4: Spring Bone Physics

### 4.1 Create Collider Groups
Define collision boundaries to prevent clipping:
```
Collider Groups needed:
- Hips (center body)
- Spine (upper body)
- Chest (chest area)
- Left Leg / Right Leg
```

**Important:** Resize colliders to ~0.1m — default size is too large and causes clothes to puff out.

### 4.2 Create Spring Bone Groups
Add physics to hair, clothing, ribbons, accessories:

| Parameter | Default | Effect |
|-----------|---------|--------|
| Stiffness | 1.0 | How quickly bones return to rest (higher = stiffer) |
| Drag Force | 0.4 | Movement softness (lower = softer, more fluid) |
| Gravity Direction | (0, -1, 0) | Downward gravity |
| Center Bone | Hips | Physics relative to body, not world |

### 4.3 Bone Selection Rules
- **DO** add: Hair strands, skirt bones, ribbon bones, clothing tails, accessories
- **DON'T** add: Main body bones (chest, neck, head), eyeballs, eyebrows
- Only the **first bone in a chain** needs to be added; children follow automatically
- Assign relevant collider groups to each spring bone group

### 4.4 Unity Alternative
Unity provides better visualization and real-time preview for physics:
- Import VRM into Unity project with UniVRM
- Add `Spring Bone Collider Group` components to body bones
- Add `VRM Spring Bone` components with bone references
- Play scene to preview physics in real-time
- Export VRM from Unity when satisfied

## Step 5: Export VRM

### 5.1 Pre-Export Checklist
- [ ] All bone names in English
- [ ] Materials converted to MToon2 with correct textures
- [ ] Shape keys exist for at least: A, I, U, E, O, Blink
- [ ] Shape keys bound in VRM Blend Shape Proxy
- [ ] Spring bones configured (optional but recommended)
- [ ] Transforms applied (Ctrl+A)
- [ ] Save .blend file before export (Blender may crash)

### 5.2 Export
```
File → Export → Export VRM
```
Export may take several minutes as Blender processes textures. Blender may appear frozen — this is normal.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Model turns white after MToon2 | Reassign textures to each material |
| Eyes appear blank/white | Re-link eye texture in Base Color slot |
| Clothes disappear after combining materials | Undo combine, do it selectively |
| Shape keys missing in VRM | Bone morphs need baking to vertex shape keys first |
| Physics causes body to wobble | Remove spring bones from main body bones |
| Clipping through clothes | Add more colliders, adjust collider sizes |
| Blender crashes during export | Save frequently, reduce texture sizes if needed |
| Eyeballs move backward into head | Known limitation; use overlay eye meshes instead |

## File Locations (Project-Specific)

- PMX sources: `F:\3D Avatars\`
- Converted VRMs: `F:\OS\suisei-companion\`
- VRM for Unity: `F:\OS\VRMCompanion\Assets\StreamingAssets\`
- PMX2VRM Converter (alternative): `F:\3D Avatars\PMX2VRMConverter\PMX2VRMConverter.exe`
- Video transcripts: `F:\OS\3d-converter\`

## Notes

- Blender 3.6 is unstable during this workflow — **save frequently**
- The CATS "Fix Model" step is destructive — always work on a copy
- TDA-style models typically use bone morphs, not vertex morphs
- Some models have 15+ materials — combining same materials speeds up conversion
- Physics setup is optional for basic VRM but required for natural-looking avatars
- Unity provides better physics tuning than Blender (real-time preview)
