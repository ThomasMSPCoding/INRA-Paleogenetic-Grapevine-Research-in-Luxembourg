# INRA-Paleogenetic-Grapevine-Research-in-Luxembourg

*A Paleogenetic Investigation of INRA, Luxembourg*

---

## Description

[Provide a brief overview (3–5 sentences) of the project: what archaeological context is being studied, what biological material is analyzed (e.g. ancient DNA from human/faunal/plant remains), the geographic and chronological scope in Luxembourg, and why this study matters — e.g. its contribution to regional archaeogenetics, population history, biodiversity, or agricultural history.]

**Site:** [Site name, region/municipality, Luxembourg]
**Period:** [e.g. Neolithic / Bronze Age / Roman / Medieval — insert chronological range]
**Sample type:** [e.g. human skeletal remains, seeds, animal bone, sediment]

---

## Aim and Objectives

**Aim:**
[One overarching sentence describing the main research aim.]

**Objectives:**
1. [Objective 1 — e.g. reconstruct genetic ancestry/diversity of the studied samples]
2. [Objective 2 — e.g. assess preservation/authenticity of ancient DNA]
3. [Objective 3 — e.g. compare findings to regional/temporal reference datasets]
4. [Objective 4 — e.g. produce outputs relevant to archaeological/conservation stakeholders]
5. [Add/remove as needed]

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
| [e.g. FastQC] | [ ] | Raw read quality control |
| [e.g. AdapterRemoval / fastp] | [ ] | Adapter trimming, quality filtering |
| [e.g. BWA / bwa-mem2] | [ ] | Read alignment to reference |
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
| Phase 1 | Sample collection / access to excavation material | [ ] |
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
