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