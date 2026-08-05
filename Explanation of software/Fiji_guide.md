# Fiji (ImageJ) Guide - Photographing seed to creating outline by using open-software Fiji

This process covers the step that sits *before* the R/Momocs pipeline: turning raw photographs of gravine pips into binary black-on-white silhouette that `import_jpeg()` expects. If a step below doesn't match your screen exactly, check the last note on the end.

## 1. Where this fits

```
raw photograph (RGB, scale bar, one or more seeds)
        │  Fiji: threshold → binary mask → clean up → isolate specimens
        ▼
binary silhouette, one seed per .jpg (black seed, white background)
        │  R: import_jpg()
        ▼
Out object → efourier() → PCA → ...
```
## 2. Software

- **Fiji**(the "batteries-included" ImageJ distribution): [fiji.sc](https://fiji.sc) -> down, unzip, run `ImageJ-win64.exe`. No installer, no admin rights needed.
- Update it once after install **Help -> Update...** -> keeps the bundled plugins current.

## 3. Photography prerequisistes

Before opening Fiji, the source photo should have:

- A **scale bar or ruler** visisble in every frame for calibration.
- Even, diffuse lightning with no strong specular highlights on the seed surface (highlights creates holes in the treshold later).
- The seed(s) against abackground with clearly different tone from the seed itself -> for charred pips (dark) a light/white background gives the best sparatoin; for pale waterlogged pips a darker background may separate better. Keep the convention consistent within a batch/session.
-If following the dorsal+lateral view protocol from Bouby et al.,2024, photographed and process each view as separate file, same naming, and suffix.

## 4. Step-by-step workflow (single image)

### 4.1 Open the image
**File → Open…** (or drag the file onto the Fiji toolbar).

### 4.2 Set the scale
**Analyze -> Set Scale...**
- Draw a straight line along the scale bar first(line tool, then draw), *then* open this dialog -> it pre-fills "Distance in pixels."
- Enter **Known distance**(e.g. `10`) and **Unit of length** (`mm`)
- Click **Global** of every photo in the session used the same magnification/camera distance.
- This applies the calibration to all images opened afterward without repeating the step !!!!!

### 4.3 Convert to 8-bit grayscale
**Image -> Type -> 8-bit**
Thresholding works on a single grayscale channel, this discards colour information you don't need for outline extraction.

### 4.4 Treshold
**Image → Adjust → Threshold…** (`Ctrl+Shift+T`)
- **Method: Otsu** -> a good default for bimodal seed vs background histogram, try **Default** as a fallback of Otsu clips part of the seed.

- **Dark background: unchecked** -> This assumes the seed is the *darker* object on a *lighter* background (typical setup above). If you backgrund is darker than the seed, check this box instead.

- Watch the red preview overlay, it should hug the seed outline without eating into it or leaving a fringe of background. Nudge the treshold sliders manuallyif the auto value is slightly off.

- Click **Apply** to convert to a binary mask

### 4.5 Chack plarity (black object, white background)

Momocs `import_jpeg()` expects a **black seed silhouette on a white background**. Fiji's binary output can come out inverted depending on the **Black background** setting under **Process -> Binary -> Options...**
- If your result shows a **white** seed on **black**, either unclick **Blackbackground** in that Options dialog and re-run, or simply invert the current image: **Edit -> Invert**.
- Confirm visually before moving on - this is the single most common reason `import_jpeg()` silently fails or returns a nonsense outline later.

### 4.6 Cleam up the mask
Process in this order:
1. **Process -> Binary -> Fill Holes** -> closes small internal gaps
2. **Process -> Binary -> Despeckle** -> removes unwanted noises in the image (median filter)
3. Optional, if edges still look funky: **Process -> Binary -> Open** then **Close** (erosion + dilation pair) to smooth the boundary slightly - use sparingly, this can round off genuine morphological detail like the seed beak, which matters for EFA.

## 4.7 Isolate individual specimens ( multiple seeds per photo)
**Analyze -> Analyze Particles...**
- **Size**: start around `5-30mm^2` for grapevine pips and adjust after inspecting results
- **Circularity**: `0.30-1.00` as a starting filter to exclude thin debris/scratches while still keeping the pip's natural elongated, beaked shape
- Click **Exclude on edges** (drops partially cropped specimens touching the frame border)
- **Show: Masks**, thick **Add to Manager**
- If two seeds are touching and get merged into one particle, undo, go back to section 4.6 and add **Process -> Binary -> Watershed** before re-runing Analyze Particles ; it splits touching convex objects at their narrowest connection.
- Use the **ROI Manager** (now populated) to crop and save each specimen individually: select an ROI -> **Image -> Duplicate...** -> save

### 4.8 Export
**File -> Save As -> jpeg**, one file per specimen. Match the naming convention used in your metadata table from the EFT guide (sampleIDs, site, perservation mode) so `import_jpeg()` out put rows line up with `read.csv("data/seed_metadata.csv")`

## 5. Batch processing 
For many photos, record a macro once replay it across a folder rather than repeating section 4-4.6 by hand.

**Plugins -> Macros -> Record**, walk through section 4.3-4.6 manually once, than adapt the recorded code into this:

```javascript
// Fiji macro: raw seed photo -> cleaned binary silhouette
input  = getDirectory("Choose input folder");
output = getDirectory("Choose output folder");
list   = getFileList(input);
 
for (i = 0; i < list.length; i++) {
    open(input + list[i]);
    run("8-bit");
    setAutoThreshold("Otsu");
    run("Convert to Mask");
    run("Fill Holes");
    run("Despeckle");
    saveAs("Jpeg", output + list[i]);
    close();
}
```

Run it via **Plugins -> Macro -> Run**, or paste into a new **Plugins -> New -> Macro** window and hit Run. This handles the single-seed-per-photo case end to end. For multi-seed photos, keep the Analyze Particles + crop step (section 4.7) as manual pass; automated particles splitting on irregular archaeological material benefits from a visual check per image rather than a blind batch run.

## 6. Quality control
- Spot-check a subsample of exported silhouettes: no seeds clipped at the frame edge, no two seeds merged into one blob, outline follows the visible seed boundary without a jagged stair case
- Log the thresjold method and any manual slider adjustments per session - lightning drift between photograpgy sessions can shift the ideal treshold, and this note lets you explain any batch-to-batch variation later.
- Keep the raw (unprocessed) photographs archived alongside the silhouettes- reprocessing is only possible if the originals are kept.

## 7. Naming and metadata linkage

Use one consitent stem across photo, silhouette, and metadata rowe.g. `SITE_sampleID_view.jpg` → `LUX014_dorsal.jpg`. The row order (or a matching ID column) in `seed_metadata.csv` from the EFA guide must correspond to the file order `import_jpg()` reads, so decide the convention before processing a full batch, not after.
---
 
Menu names, dialog layouts, and some default checkbox states (e.g. "Black background") can shift slightly between Fiji/ImageJ releases — if a specific item isn't exactly where described above, it's almost certainly still there under a neighboring menu (Image / Process / Analyze), just check the current version's tooltip or the Fiji changelog rather than assuming the function was removed.