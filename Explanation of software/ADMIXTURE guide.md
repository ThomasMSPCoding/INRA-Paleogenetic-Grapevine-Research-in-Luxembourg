# ADMIXTURE Guide - Model based on ancestry for ancient grapevine

This covers clustering model, which complements to PCA. Where PCA gives you an unsupervised ordination, ADMIXTURE gives you per-individual ancestry proportions across *K* inferred ancestral components, i.e. "this Romann pip is x% Near-Eastern-like, y% Western-*sylverstris*-like." It runs on the same clean, pruned, merged dataset which is build in PLINK and follows up a continuation.

## 1. Where this fits

```
merged, QC'd, LD-pruned dataset  (from the PLINK guide)
        │  remove clonal duplicates + close relatives first (ADMIXTURE assumes unrelated!)
        ▼
   ┌─────────────────────────────────────────────┐
   │ learn K ancestral components on MODERN panel │  admixture (unsupervised) + --cv
   └─────────────────────────────────────────────┘
        │  then place the ancient pips without letting them distort the model:
        ├── projection  (-P)           ← learn on modern, project ancients
        └── supervised  (--supervised) ← fix modern pops, estimate ancient proportions
        ▼
 per-pip ancestry proportions (.Q)  →  RQ3  ancestry / clustering
```

Two things to internalise before running anything:
1. **ADMIXTURE** does not model LD or relatedness. **You must pre-prune for LD (done in PLINK) and pre-remove clones/relatives 
2. **Low-coverage ancient samples must not drive the clustering.** Same problem as PCA: if you throw sparse pips into an unsupervised run, they pull the model around. The fix is projection or supervised mode (will be explained later), not naive combined run.

## 2. Software

```bash
# easiest: bioconda (into the same env as plink/paleomix)
conda install -c bioconda admixture
 
# or the official Linux binary (requires kernel ≥ 3.17)
wget https://dalexander.github.io/admixture/binaries/admixture_linux-1.4.0.tar.gz
tar -xzf admixture_linux-1.4.0.tar.gz
```

Input is binary PLINK (`.bed` + `.bim` + `.fam`), ordinary PLINK (`.ped` + `.map`), or EIGENSTRAT (`.geno` + `.map`). Format is inferred from the extension. Outputs are two plain-text matrices per run: **`.Q`** (ancestry fractions per individual) and **`.P`** (allele frequencies per inferred component). Filenames carry *K*, e.g. `merged.4.Q`.

Manual: <https://dalexander.github.io/admixture/admixture-manual.pdf>.

## 3. Input prep

### 3.1 Chromosome names -> Integers
Since v1.3, ADMIXTURE refuses non-numeric chromosme names. Grapevine `.bim` files usually carry `chr1...chr19`, which will error out. Recode to plain integers and drop everything that isn't a nuclear chromosome:

```bash
# keep only chr1..chr19, strip the "chr" prefix in the .bim
awk '$1 ~ /^chr([1-9]|1[0-9])$/ {print $2}' merged_pruned.bim > nuclear.snps
plink --bfile merged_pruned --extract nuclear.snps \
      --make-bed --allow-extra-chr --out merged_nuc
sed -i 's/^chr//' merged_nuc.bim        # chr7 → 7
```

>!! Check afterwrads that column 1 of `merged_nuc.bim` is now `1`-`19` with nothing else. A stray `chrPt`/scaffold line is the most common reason ADMIXTURE dies on the first SNP.

### 3.2 Pseudo-haploid coding

Our pips are pseudo-haploid ( one random read per site), written as homozygous. The dominant aDNA practice is to run this in ADMIXTURE's **default diploid mode**, treating each pseudo-haploid call as a homozygous diploid genotype. It slightly misrepresent the data, but is standrd and works in the major aDNA papers.

The more-correct alternative is the `--haploid` flag (`--haploid="*"` = all data haploid), but it requires numeric chromosomes names and errors on any heterozygous call. Since pseudo-haploidisation already removed heterozygotes, it's safe to try, but deafult diploid mode is the pragmatic, widley used choice.

## 4. Remove clones and relatives first

ADMIXTURE's model assumes **unrelated** individuals. Grapevine reference panels are full of **clonal duplicates** and close relatives. Left in,they inflate specific components and disort *K* selection.

Use the identity results from the PLINK stage (IBS distance / `--genome` PI_HAT) to build a clone-free, relative-thinned reference set, then run ADMIXTURE on that. This is where RQ4's clonal-detection output feeds directly into RQ3:
 
```bash
# from PLINK: pairs with PI_HAT above ~0.9 are clone/near-clone candidates
# keep one representative per clone group, drop the rest:
plink --bfile merged_nuc --remove clone_duplicates.txt \
      --make-bed --allow-extra-chr --out ref_unrelated
```

## 5. Choosing K -cross validation

There is no single "true" *K*; you pick the value with the best predicitive accuracy via ADMIXTURE's built-in cross validation. Run a range and compare CV error:

```bash
for K in 1 2 3 4 5 6 7 8; do
    admixture --cv ref_unrelated.bed $K -j4 | tee logs${K}.out
done
grep -h CV log*.out
# CV error (K=2): 0.481
# CV error (K=3): 0.478 <- pick the K at/near the minimum
```

`--cv` defaults to 5-fold (--cv=10 for 10-fold); `-j4` uses 4 threads. Pick the *K* at the lbow/minimum, bzt don't treat it as gospel, inspect a few values around it, because the biologically interesting structure may sit at *K* that isn't the strict CV minimum.

**What K means for grapecine.** Expect low components to separate the big axes of *Vitis* structure: wild *sylvestris* vs domesticated *vinifera*. Differentiation between cultivar groups is subtle (low F_ST), so you need **many markers** to resolve it (the manual's rule of thumb is ~100k for within-continent-scale F_ST < 0.01). That collides with the limited SNP overlap of low-coverage pips.

## 6. Placing the ancient pips

### 6.1 Projection (`-P`) ; Admixture analog of PCA projection

Learn the components on the modern unrelated reference, then hold those allele frequencies fixed and estimate only the pips' proportion. Thos stops sparse ancients from reshaping the model. The same logic as `lsproject` in smartpca.

```bash
# reference and study files MUST share an identical SNP set:
diff -s ref_unrelated.bim pips.bim
 
# 1. learn structure on the modern reference at chosen K (say 4)
admixture ref_unrelated.bed 4 -j4
 
# 2. hand the learned allele frequencies to the projection run
cp ref_unrelated.4.P pips.4.P.in
 
# 3. project the ancient pips onto the fixed components
admixture -P pips.bed 4 -j4
# → pips.4.Q holds each pip's ancestry proportions on the modern components
```

### 6.2 Supervised (`--supervised`) - when your reference groups are well-defined

If you have confidently-labelled modern population, you can fix those as training labels and estimate against them. Provide a `.pop` file, one line per individual matching `.fam` order: the population name for refernce, `-` for rhe ancients to be estimated.

```bash
# merged.fam order must match merged.pop line-f or-line - verify with:
paste merged.fam merged.pop | less
admixture --supervised merged.bed 3 -j4
```

> !! Supervised mode **assumes the reference individuals are 100% ancestry from their labelled population** and that you've specified the right ancestral populations. Those are strong assumptions for grapevine, where cultivars are themselves admixed. Supervised mode answers a sharper question ("what fraction Near-Eastern vs Western?") but is only as good as your labels. I'd run projection as the primary analysis and supervised as a targeted follow-up, and report both.

## 7. Robustness - run multiple seeds
Block relaxation can settle into different local optima, so a single run per *K* isn't enough. Run each *K* several times with different seeds and check the `.Q` matrices agree:

```bash
for s in 1 2 3 4 5; do
      admixture -s ${s} --cv ref_unrealted.bed 4 -j4 | tee log4_seed${s}.out
done
```

Align and summarise replicate runs with **pong** or **CLUMPAK** (they match components across runs and flag unstable *K* values). Genuine structure is reproducible across seeds; a components that only appears in some runs is an artifact.

## 8. Plotting the Q matrix
 
The `.Q` file is a plain matrix, a stacked barplot is a few lines of R:
 
```r
Q <- read.table("pips.4.Q")
barplot(t(as.matrix(Q)), col = rainbow(4),
        xlab = "Individual", ylab = "Ancestry", border = NA, space = 0)
```
 
Order individuals sensibly (by site, by period, or grouped modern-then-ancient) and keep the colour/label mapping identical across *K* values and figures, or the story gets unreadable fast. For publication-grade multi-*K* panels, pong/CLUMPAK output is cleaner than base R.