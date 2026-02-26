# 3D View Assets

This folder contains GLB (GL Transmission Format Binary) files for 3D PCB visualization on the AttentioLight website.

## Source

These models were exported from **KiCad** PCB design software. KiCad exports highly detailed meshes optimized for accuracy, which can result in large file sizes.

## Current Files

| File | Description |
|------|-------------|
| `attentiolight-1-mainboard1.glb` | Main PCB board with all components |
| `attentiolight-1-1ledmodule1.glb` | LED module PCB |

## Mobile Optimization

The mainboard GLB file is too large for reliable rendering on mobile devices. Mobile browsers have limited GPU memory and may fail to render large 3D models, or the WebGL context may be lost.

**Recommended target sizes for mobile:**
- Mainboard: < 5 MB (ideally < 3 MB)
- LED Module: < 500 KB

## How to Compress GLB Files

### Option 1: Using gltf-transform CLI (Recommended)

[gltf-transform](https://gltf-transform.dev/) is a powerful CLI tool for optimizing glTF/GLB files.

#### Installation

```bash
# Instal npm
sudo apt insatll npm -y

# Install globally via npm
npm install -g @gltf-transform/cli

# Verify installation
gltf-transform --version
```

#### Basic Compression (Draco)

Draco compression typically reduces file size by 5-10x with minimal visual impact:

```bash
cd assets/3dview
# Compress mainboard (24MB -> ~3-5MB expected)
gltf-transform optimize attentiolight-1-mainboard1.glb \
  attentiolight-1-mainboard1-compressed.glb \
  --compress draco

# Compress LED module (970KB -> ~150-300KB expected)
gltf-transform optimize attentiolight-1-1ledmodule1.glb \
  attentiolight-1-1ledmodule1-compressed.glb \
  --compress draco
```

#### Aggressive Compression (Draco + Mesh Simplification)

For even smaller files, add mesh simplification. This reduces polygon count which may slightly affect visual quality:

```bash
# Reduce to 50% of original polygons + Draco compression
gltf-transform optimize attentiolight-1-mainboard1.glb \
  attentiolight-1-mainboard1-mobile.glb \
  --compress draco \
  --simplify \
  --simplify-ratio 0.5

# More aggressive: 30% of original polygons
gltf-transform optimize attentiolight-1-mainboard1.glb \
  attentiolight-1-mainboard1-mobile-aggressive.glb \
  --compress draco \
  --simplify \
  --simplify-ratio 0.3
```

#### Inspect File Details

To analyze the contents of a GLB file before/after optimization:

```bash
gltf-transform inspect attentiolight-1-mainboard1.glb
```

### Option 2: Using Blender (GUI Alternative)

If you prefer a graphical interface:

1. **Import**: File > Import > glTF 2.0 (.glb/.gltf)
2. **Simplify meshes** (optional):
   - Select objects in the scene
   - Add "Decimate" modifier
   - Set ratio to 0.5 (50% of polygons)
   - Apply modifier
3. **Export**: File > Export > glTF 2.0 (.glb/.gltf)
   - Enable "Compression" in export settings
   - Set compression level (6 is a good default)

### Option 3: Re-export from KiCad with Lower Detail

When exporting from KiCad's 3D Viewer:

1. Open the PCB in KiCad's 3D Viewer
2. Before exporting, consider:
   - Hiding non-essential components
   - Using simplified 3D models for components (if available)
3. Export to STEP, then convert to GLB using Blender or other tools

## Updating the Website

After creating optimized files, update `index.html` to use them:

```html
<!-- Before -->
<model-viewer src="assets/3dview/attentiolight-1-mainboard1.glb" ...>

<!-- After (using compressed version) -->
<model-viewer src="assets/3dview/attentiolight-1-mainboard1-compressed.glb" ...>
```

### Device-Specific Loading (Advanced)

To serve different files to mobile vs desktop:

```javascript
const isMobile = window.matchMedia('(max-width: 768px)').matches;
const modelViewer = document.querySelector('model-viewer');
modelViewer.src = isMobile 
  ? 'assets/3dview/attentiolight-1-mainboard1-mobile.glb'
  : 'assets/3dview/attentiolight-1-mainboard1.glb';
```

## Verification

After optimization, verify the results:

1. **Check file size**: Should be significantly smaller
   ```bash
   ls -lh *.glb
   ```

2. **Test in browser**: Open in Chrome/Firefox and check DevTools console for errors

3. **Test on mobile**: Use actual mobile device or Chrome DevTools device emulation

4. **Compare visual quality**: Ensure the model still looks acceptable

## Troubleshooting

### Model doesn't load on mobile
- File is still too large (target < 5MB)
- Try more aggressive simplification
- Check browser console for WebGL errors

### Model looks blocky after simplification
- Increase `--simplify-ratio` (e.g., 0.7 instead of 0.5)
- Try Draco compression only without simplification

### Draco compression not working
- Ensure gltf-transform is up to date: `npm update -g @gltf-transform/cli`
- The model-viewer library supports Draco-compressed files natively

## Resources

- [gltf-transform documentation](https://gltf-transform.dev/cli)
- [Google Model Viewer](https://modelviewer.dev/)
- [Draco 3D compression](https://google.github.io/draco/)
- [KiCad 3D Viewer documentation](https://docs.kicad.org/master/en/pcbnew/pcbnew.html#_3d_viewer)
