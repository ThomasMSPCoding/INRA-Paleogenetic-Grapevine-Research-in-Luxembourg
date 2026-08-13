# PLINK Guide - Population Genetics for Ancient Grapevine

This covers the population-genetics stage that sits *after* mapping/genotyping and drives the ancestry and clonal identity question: taking pseudo-haploid genotype calls from your ancient pips, merging them with a modern *Vitis* reference panel, cleaning the merge dataset, and running PCA identity analyses. PLINK is the workhorse for all the data-mangement and QC steps; the final figure (PCA, F-statistics) are usually produced by EIGENSOFT/AdmixTools downstream, but PLINK is what get you a clean, merged, correctly-formatted dataset to implement them.

Everything below targets **PLINK v.1.9**, although the version 2.0 exist and will be implemented in this guide for refernce.

## 1. Where this fits

```
per-sample BAM  (from the mapping guide)
      │  pseudo-haploid genotype calling at a fixed SNP panel
      │    (ANGSD -doHaploCall  /  pileupCaller  — NOT PLINK)
      ▼
ancient genotypes  (EIGENSTRAT or PLINK format)
      │  PLINK: merge with modern Vitis reference panel
      ▼
merged dataset
      │  PLINK: QC filters → LD prune
      ├──────────────► PCA            → RQ3  ancestry / clustering
      ├──────────────► IBS / IBD      → RQ4  clonal identity
      └──────────────► (export)       → smartpca / AdmixTools (F-stats, projection)
```
The keything to understand: **PLINK does not call genotypes from BAM files.** It operates on genotype tables. So the first job os turning your alingned reads into genotype calls, and for low-coverage aDNA that means *pseudo-halpoidisation*, not standard diploid calling.

# 2. Software

| Tool | Version | Role |
|---|---|---|
| PLINK | **1.9** (stable) | all data management, QC, PCA, IBD in this guide |
| PLINK | 2.0 (alpha) | optional, for very large panels; some flags differ |
| ANGSD *or* pileupCaller | current | pseudo-haploid calling from BAMs (upstream) |
| EIGENSOFT (convertf, smartpca) | current | format conversion + projected PCA (downstream) |

Install via conda/biconda alongside the paleomix enviroment:

```bash
conda install -c biconda plink

conda install -c biconda angsd eigensoft
```

Docs: PLINK 1.9 at <https://www.cog-genomics.org/plink/1.9/>, 2.0 at <https://www.cog-genomics.org/plink/2.0/>.

## 3. Getting your pips into PLINK (the upstream step)

**Why pseudo-haploid.** At ~0.01-x coverage you can't confidently call a dipoid gentype (you rarely see both alleles). The aDNA-standard fix: ate each SNP position. draw **one random high-quality read**and record its allele as a haploid call. This avoids refernce bias and stops post-mortem damgae from inlfating apparent heterozygosity.

**How?** Two common routes, both operating on your final BAMs at a **fixed list of SNP positions**.

```bash
# Option A — ANGSD random-haploid call at your SNP panel
angsd -i pip01.bam -r <region> \
      -doHaploCall 1 -doCounts 1 \
      -minMapQ 30 -minQ 30 \
      -sites vitis_panel.sites -out pip01
 
# Option B — pileupCaller (sequenceTools), reads samtools mpileup at the panel
samtools mpileup -R -B -q30 -Q30 \
      -l vitis_panel.bed -f Vitis_PN40024.fasta pip01.bam | \
  pileupCaller --randomHaploid \
      --snpFile vitis_panel.snp \
      --sampleNames pip01 \
      --eigenstratOut pip01
```

**Damage handling:** Deamination modifies C->T (5') and G->A (3') transition. Either trim terinal bases before calling, or **restrict analyses to transversion SNPs** (drop C/T and G/A sites) - the latter is the simplest robust options for authenticity-sensitivity comparisons. mapDamage output from the previous stage tells you how many terminaly bases to worry about.

The output lands in **EIGENSTRAT** (`.geno/.snp/.ind`) or PLINK format; `convert`(EIGENSOFT) bridges the two in either directions.

## 4. The SNP panel - the grapevine-specific problem

Human aDNA has the universal **1240k capture panel**. Grapevine has no single standard equivalent, so you *ascertain* your own SNP set from modern diversity data and call your pips at exactly those positions.

Reasonable sources for a *Vitis* panel:
- **Dong et al. 2023 (Science)** grapevine domestication dataset — thousands of accessions, the richest current SNP resource and a natural reference for European/Near-Eastern structure.
- Published diversity panels from the group you're working with (the Ramos-Madrigal / Copenhagen lineage of grapevine aDNA work)
- The Vitis SNP arrays (e.g. the 18K array) 

Three non-negotiables for the panel:
1. **One frozen assembly:** Ancient calls and the modern refernce must ise the *same* PN40024 version. Mismatched coordinates silently corrupt a merge.
2. **Boallelic, autosmal:** Keep grapevine nuclear chromosome (chr1-19); eclude unplaced scaffolds and the chloroplast contig from popgen (handle cpDNA separetely).
3. **The identical SNP list:** used for calling the pips and present in the reference; the merge only keeps intersection anyway, so define it once.

## 5. Merge ancient + modern reference panel

Convert both sets to PLINK binary, the merge:

```bash
# make binary sets (repeat for the reference)
plink --file pip01 --make-bed --allow-extra-chr --out pip01
 
# merge ancient into modern reference
plink --bfile vitis_modern_ref \
      --bmerge pips_all.bed pips_all.bim pips_all.fam \
      --allow-extra-chr --make-bed --out merged
```

Merges fail on allele mismateches; the standard fix cycle:

```bash
# 1. merge attemp reports strand/triallelic conflicts in merged-merge.missnp
# 2. flip the conflicting SNPs in one set, or just exclude them:
plink --bfile pips_all --flip merged-merge.missnp --make-bed -oot pips_flip
# (re-run bmerge; if triallelic conflicts remain, exclude these)
plink --bfile pips_flip --exclude merged-merge.missnp --make-bed --out pips_clean
# 3. re-merge pips_clean into the reference
```

Keep only the shared SNPs (`--extract`) and biallelic sites; drop anything triallelic rather than guessing strand.

## 6. QC filtering / aDNA-aware and grapevine-aware

Strandard PLINK QC, but several defaults are **wrong for this data** and one is actively harmful for grapevine:

```bash
plink --bfile merged \
      --geno 0.99 \          # per-SNP missingness: keep SNPs typed in ≥1% of samples (LENIENT — ancients are sparse)
      --maf 0.01 \           # minor-allele freq: see note, base this on the MODERN panel
      --allow-extra-chr \
      --make-bed --out merged_qc
```

- **`mind` (per-sample missingness): usually skip it .** It drops samples with too much missing data. Applying it aggressively ydeletes the samples. If you use it at all, applyit only to the modern reference beforehand.
- **`--maf` should be drivem by the reference, not the ancients.** A rare allele seen only  in one low-coverage pip is noise; compute MAF on the modern panel and apply that SNP set to the merged data.
- **`--hwe`: do NOT apply for grapevine.** Hardy_weinberg filtering assumes random mating. Grapevine is clonally propagated, domesticated/wild split and cultivar structure are strong, and pooling them violates HWE at real, informativeloci. HWE filtering here rmoves true signal. This is genuine departurefrom human-popgen habit.

Then LD-prune before PCA:

```bash
plink --bfile merged_qc --indep-pairwise 200 25 0.4 --allow-extra-chr --out prune
plink --bfile merged_qc --extract prune.prune.in --make-bed --allow-extra-chr --out merged_pruned
```

## 7. PCA / ancestry and clustering

For a quick exploratory look, PLINK's own PCA is fine:
```bash
plink --bfile merged_pruned --pca 10 --allow-extra-chr --out pca_explore
```

> !! **But dont use plain `--pca` for final ancestry result.** With sparse, damaged ancient samples, missung data and damagedrive the top PCs, so pips cluster by *coverage* rather than *ancestry* (A classic aDNA artifact).The standard fix **preojection**: build the PCA on the *modern* accessions only, then project the ancient pips onto that fixed space.PLINK doent project cleanly: use **smartpca (EIGENSOFT) with `lsproject: YES`** and the ancients listed in `poplistname` as the projected population. Export the merged data to EIGENSTRAT and run smartpca.

So the division of labour: PLINK for the merge/QC/pruning and a first exploratory PCA, smartpca for the publication PCA that places your Roman pips relative to modern European vs Near-Eastern cultivars.

## 8. Idenity / relatedness (clonal detection)

Because grapevine cultivars are clones, the meaningful question here is **"is this pip a clonal match to another pip, or to a modern cultivars?"** The same reframing from the READ/KIN stage. PLINK givesyou two complementary readouts.