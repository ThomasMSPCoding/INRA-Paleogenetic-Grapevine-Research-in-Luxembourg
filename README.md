# INRA-Paleogenetic-Grapevine-Research-in-Luxembourg

*A Paleogenetic Investigation of INRA, Luxembourg*

---

## Description

[Provide a brief overview (3–5 sentences) of the project: what archaeological context is being studied, what biological material is analyzed (e.g. ancient DNA from human/faunal/plant remains), the geographic and chronological scope in Luxembourg, and why this study matters — e.g. its contribution to regional archaeogenetics, population history, biodiversity, or agricultural history.]

**Site:** INRA
**Period:** [e.g. Neolithic / Bronze Age / Roman / Medieval — insert chronological range]
**Sample type:** Grapevine seeds - Type will be  seen soon

---

## Aim and Objectives

**Aim:**

**Objectives:**
1. A bibliographic review of recent research on ancient grape seed DNA analysis and grapevine genetics in Europe.
2. A summary of the latest findings in this field, written for a non-specialist archaeological audience (including reflections on what data from Luxembourg sites could contribute).
3. A practical guide for the conservation of waterlogged grape seeds and the preservation of ancient DNA (recommendations for best practices).

**Research Questions:**
1.	In the sense of preservation of ancient DNA (aDNA), does waterlogged and other types of contamination in pips, be able to recover endogenous content and fragmented reads?
a.	Assessing if there is endogenous content, fragmented reads distribution, and post-mortem damage (C-T/G-A deamination /water damage in DNA which removes nitrogen and changes the structure of DNA) to look if the material is usable and reproducible data can be managed, which can considered and used for downstream analysis.

2.	Do methods like whole-genome shotgun and/or targeted SNP (Single nucleotides Polymorphism) capture better usable data for poorly preserved pips/fragmented genetic material?
a.	After building the preservation data/baseline of RQ1, we could compare the two approaches. Shotgun sequencing from Noraz et al., 2026 versus SNP GrapeReSeq capture used by Ramos-Madrigal et al., 2019. This could define the best practice and most cost effective route for reliable genotypes given by the coverage of real expectancy.

3.	What kind of modern ancestry cluster does Luxembourgish pips carry, and does the (paleo)genetic profile fit to the southern-French and Mediterranean pips?
a.	After building a reliable and stable genotype base from fractured and mutation DNA, we will be assessing the location of the grapevine and can be compared to reference panels (which are already know, e.g. France and Italy). This can be tested whether Luxemburgish viticulture drew on the same Mediterranean lineages (documented in Noraz et al., 2026 and Ramos-Madrigal et al., 2019) or represent another distinct region/location.

4.	Are there any pips that share clonal or first/second degree relatedness from other known sides or periods (Luxembourg wider image of exchange/trade network)?
a.	With established ancestry, relatedness and clone detection, a possible comparison with French and Mediterranean records could be assessed, but only if the Luxemburgish pips in RQ3 are detected to not only be specific Luxemburgish location.


---

## Dataset

| Item | Description |
|------|-------------|
| **Sample origin** | [Excavation site, coordinates, excavation year, responsible institution] |
| **Number of samples** | [n = ] |
| **Sample type** | [bone, tooth, seed, sediment, etc.] |
| **Storage/Accession** | [Museum/lab accession numbers, if applicable] |
| **Sequencing platform** | [e.g. Illumina NovaSeq, etc.] |
| **Sequencing type** | [shotgun / targeted capture / amplicon] |
| **Reference genome(s)** | [Reference genome build(s) used] |
| **Comparative/reference datasets** | [Public ancient/modern reference panels used, with citations] |

> Raw sequencing data availability is described in the [Data Availability](#data-availability) section.

---

## Tools and Software

| Tool | Version | Purpose |
|------|---------|---------|
| [Zotero] | [9.0.5] | Bibliographic review |
| [ArcGIS] | [3.7] | Paleogenetic Archaeological Mapping |
| [Positron] | [2026.01.0] | Coding software |
| [e.g. samtools] | [ ] | BAM processing/filtering |
| [e.g. MapDamage2] | [ ] | aDNA damage pattern assessment |
| [e.g. ANGSD / PMDtools] | [ ] | Genotype likelihoods, damage-aware calling |
| [e.g. ADMIXTURE / PCAngsd] | [ ] | Population structure analysis |
| [e.g. R / Bioconductor packages] | [ ] | Statistical analysis and visualization |
| [Add additional tools as needed] | | |

**Environment:** [e.g. Conda environment file / Docker image / Singularity container — link if available]

```bash
# Example environment setup (edit as needed)
conda env create -f environment.yml
conda activate [env-name]
```

---

## Repository Structure

```
├── data/
│   ├── raw/                # Raw sequencing data (or pointers/links, not committed if large)
│   ├── processed/          # Cleaned, filtered, and processed data
│   └── reference/          # Reference genomes and comparative datasets
│
├── scripts/
│   ├── 01_preprocessing/   # QC, trimming, filtering scripts
│   ├── 02_alignment/       # Mapping and BAM processing scripts
│   ├── 03_analysis/        # Population genetics / statistical analysis scripts
│   └── 04_visualization/   # Plotting and figure-generation scripts
│
├── results/
│   ├── figures/             # Output figures
│   ├── tables/               # Output tables
│   └── reports/              # Summary reports
│
├── docs/                    # Supplementary documentation, protocols, metadata
├── environment.yml          # Conda environment specification
├── LICENSE
└── README.md
```

[Adjust folder names/structure to match your actual repository layout.]

---

## Analysis Pipeline and Methods

### 1. Sample Preparation and Sequencing
[Describe DNA extraction protocol, library preparation method, and sequencing strategy.]

### 2. Data Preprocessing
[Describe QC, adapter trimming, and read filtering steps.]

### 3. Alignment and Post-Processing
[Describe reference mapping, duplicate removal, and quality/damage filtering specific to ancient DNA (e.g. deamination pattern checks, contamination estimates).]

### 4. Authentication of Ancient DNA
[Describe criteria used to authenticate aDNA: fragment length distribution, C-to-T deamination pattern, contamination estimates, endogenous DNA content.]

### 5. Population/Comparative Analysis
[Describe downstream analyses: PCA, admixture, kinship, phylogenetics, or other methods relevant to your research questions.]

### 6. Statistical Analysis and Visualization
[Describe statistical tests and visualization approaches used to interpret results.]

---

## Timeline

| Phase | Task | Period |
|-------|------|--------|
| Phase 1 | Bibliographic review | [2 weeks] |
| Phase 2 | DNA extraction and library preparation | [ ] |
| Phase 3 | Sequencing | [ ] |
| Phase 4 | Bioinformatic preprocessing and alignment | [ ] |
| Phase 5 | Authentication and quality control | [ ] |
| Phase 6 | Population/comparative genetic analysis | [ ] |
| Phase 7 | Interpretation and report writing | [ ] |
| Phase 8 | Publication / dissemination | [ ] |

---

## Data Availability

[State where raw and processed data are or will be deposited, e.g. ENA/SRA accession numbers, institutional repository, or embargo status. Include any relevant restrictions on data use due to ethical, cultural heritage, or institutional agreements.]

- **Raw sequencing data:** [Accession number / repository link / "Available upon reasonable request"]
- **Processed data and scripts:** [This repository / DOI if archived via Zenodo, etc.]
- **Restrictions:** [Any embargo, ethical, or heritage-related access restrictions]

---

## Acknowledgements

[Institutions, collaborators, funding bodies, and archaeological teams involved.]

---

## License

[Specify license, e.g. MIT, CC-BY-4.0]

---

**Thomas Addabbo**
