# Paleomix guide -> From RAW Fastq to authenticated aDNA BAMs
This covers the read-processing and aligment stage thats sits *after* sequencing and *before* variant calling / population genetics: turning raw Illumina FASTQ reads into trimmed, aligned, duplicated-filtered, damaged assessed BAM files againt a *Vitis vinifera* refernce. Paleomixbundles the whole aDNA read-handling chain (AdapaterRemoval -> BWA/Bowtie2 -> samtools -> mapDamage)behind a single YAML "make file", so the pipleline is reproducible and resumable.

Everything in this pipline is contructed for Paleomix v 1.4.0, if there is an new update, some changes can occure.

## 1. Mapping the structure

```
raw Illumina FASTQ (per library, per lane; SE and/or PE)
        │  PALEOMIX BAM pipeline
        │    ├─ AdapterRemoval  → trim adapters/low-qual/Ns, collapse overlapping PE
        │    ├─ BWA / Bowtie2   → map trimmed reads to Vitis reference
        │    ├─ samtools        → fixmate, sort, index, calmd
        │    ├─ markdup / rmdup → filter PCR duplicates (per library)
        │    └─ mapDamage2      → postmortem damage model (+ optional rescaling)
        ▼
final BAM (+ .bai) per sample, + coverage / depths / damage / summary tables
        │  downstream
        ▼
genotyping → phylogenetics / popgen  (PALEOMIX Phylogenetic pipeline, or your own)
```

## 2. Software
 
PALEOMIX itself is a Python 3 orchestrator; it does not re-implement the tools, it calls them. So you install PALEOMIX **and** the underlying programs.

### 2.1 How to install paleomix?
The cleanest option that pulls in all the aligner/samtools/mapDamage dependencies automatically is **conda + bioconda**. That's the route I'd recommend for the project, since it avoids hand-installing five separate tools with pinned versions.

```bash
# 1. Download the environment template for this release and build the env
curl -fL https://github.com/MikkelSchubert/paleomix/releases/download/v1.4.0/paleomix_environment.yaml > paleomix_environment.yaml
conda env create -n paleomix -f paleomix_environment.yaml
conda activate paleomix
 
# 2. Picard is only needed if you enable BAM validation, but the pipeline
#    expects the JAR in a fixed location — symlink it into place:
mkdir -p ~/install/jar_root/
ln -s ~/*conda*/envs/paleomix/share/picard-*/picard.jar ~/install/jar_root/
```
 
If you'd rather keep it off conda, install into an isolated virtual environment instead:
 
```bash
python3 -m venv venv
./venv/bin/pip install paleomix==v1.4.0
# link the executable onto your PATH without exposing the venv's python:
mkdir -p ~/.local/bin/
ln -s ${PWD}/venv/bin/paleomix ~/.local/bin/
# make sure ~/.local/bin is on PATH (add to ~/.bashrc if not):
echo 'export PATH=~/.local/bin:$PATH' >> ~/.bashrc
```
 
On Debian/Ubuntu the pip route may also need a few dev headers first: `sudo apt-get install libz-dev libbz2-dev liblzma-dev python3-dev python3-venv`.
 
Verify:
 
```bash
paleomix
# → PALEOMIX - pipelines and tools for NGS data analyses
#   Version: v1.4.0
```

### 2.2 Underlying tools (only needed if NOT using the conda env)
 
Minimum supported versions for the BAM pipeline:
 
| Tool | Min version | Role |
|---|---|---|
| AdapterRemoval | v2.2.0 | adapter/quality trimming, PE collapsing |
| SAMtools | v1.6.0 | fixmate, sort, index, calmd, markdup |
| BWA | v0.7.15 | short-read aligner (default) |
| Bowtie2 | v2.3.0 | alternative aligner |
| mapDamage | 2.2.1 | postmortem damage model / rescaling |
| Picard Tools | v1.137 | optional BAM validation only |
 
On Debian-based systems most of these come straight from apt:
 
```bash
sudo apt-get install adapterremoval samtools bowtie2 bwa mapdamage
```
 
If you plan to use mapDamage **rescaling** (not just plotting/modelling), it additionally needs the GNU Scientific Library and several R packages (`inline`, `gam`, `Rcpp`, `RcppGSL`, `ggplot2`). Check them with:
 
```bash
gsl-config                      # should print usage, not "command not found"
mapDamage --check-R-packages    # → "All R packages are present"
```

## 3. Prepare the reference genome 

Mapping is done against one or more FASTA reference, which PALEOMIX calls **prefixes** (because the aligner index files are generated alongside the FASTA, sharing its path prefix).

- For *Vitis vinifera*, the standard reference is the **PN40024** (use tools like 12X.v2). Download the FASTA and if relevant, the chloroplast genome as seperate prefix -> mapping cpDNA separately is often useful for aDNA because organellar reads are far more abundant and give independent damage readout.
- Name the file sensibly, e.g. `Vitis_vinifera_PN40024_v4.fasta`, and let the prefix name match the filename stem.
- **You do not index it yourself** — the pipeline runs `bwa index` / `samtools faidx` automatically on first run.

> !! The pipeline needs **write access** to the folder containing the FASTA, since it writes index files there. If the reference lives in a read-only shared location, make a local folder of symbolic links to the FASTA and point the makefile at that instead —> the generated indices then land in your writable folder.

## 4. Create and edit the makefile

### 4.1 Generate a template

```bash
mkdir grapevine_bam && cd grapevine_bam
paleomix bam makefile > makefile.yaml
```

The template has three sections: **`Options`** (global defaults), **`Prefixes`** (references), and then one block per **target/sample** listing the input FASTQ. Open it in any text editor (it's plain YAML — indentation matters, use spaces not tabs).