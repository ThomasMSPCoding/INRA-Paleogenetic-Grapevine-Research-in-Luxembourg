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