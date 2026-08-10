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

### 4.2 Options

Most defaults are finefor modern Illumina data. For the grapevine aDNA project, pay attention to these:

**`Platform: Illumina`** -> matches with the shotgun libraries

**`QualityOffset: 33`** —> correct for any modern Illumina run (Illumina 1.8+). Only change to `64` for genuinely old data.
> !! AdapterRemoval expects max quality-score 41 (ASCII `J`) at offset 33. Some NovaSeq/binned-quality data can exceed this, if the pipeline complains, pass `--qualitymax` accordingly. Also note offset 64 is **not** supported with BWA `mem`/`bwasw`; convert with seqtk first if you ever hit that.
 
**`AdapterRemoval :: --adapter1 / --adapter2`** —> the single most important thing to get right. If you know the exact adapters used for library construction, uncomment these and paste them in (in the orientation they appear in the reads, i.e. greppable). With AdapterRemoval v3, auto-detection kicks in when left blank, but for aDNA I'd set them explicitly if the sequencing facility gave you the adapter sequences. The BAM-pipeline defaults also override a couple of AR defaults you'll see there: `--minlength: 25`, `--mm: 3`, and collapsing/trimming of Ns and qualities all enabled.
 
**`Aligners :: Program: BWA`** —> BWA `backtrack` (`aln`) is the standard, best-validated choice for short, damaged aDNA reads. Keep `Algorithm: backtrack` rather than `mem` for typical <100 bp ancient fragments.
 
**`BWA :: MinQuality: 30`** —> filter out low-confidence mappings (Phred 30 = error prob 0.001). Reasonable default; set to `0` (plus `FilterUnmappedReads: no`) only if you deliberately want *every* read retained.
 
**`BWA :: UseSeed: no`** —> **change this from `yes` to `no` for ancient DNA.** Postmortem damage concentrates near read termini (the seed region), which BWA's seeding assumes is low-error. Disabling the seed recovers additional true alignments from damaged fragments, at some runtime cost. This is the key aDNA-specific tweak.

### 4.3 Features 
```yaml
Features:
  PCRDuplicates: filter   # remove PCR duplicates (per library). Correct grouping
                          # of libraries matters — see the read-data section.
  mapDamage: model        # 'plot' = damage plots only;
                          # 'model' = plots + postmortem damage model (recommended
                          #           for authenticating aDNA — gives you the C→T / G→A
                          #           misincorporation curves that show the reads are
                          #           genuinely ancient);
                          # 'rescale' = also writes BAMs with damage-downweighted
                          #             quality scores (use if downstream genotyping
                          #             should not trust damaged positions).
  Coverage: yes
  Depths: yes
  Summary: yes
```

For grapevine aDNA I'd start with `mapDamage:model` -> you get the diagnostic damageplots and the model without altering the base qualities in your BAM. Switch to `rescale` only once you're confident and want damage-aware genotyping downstream.

### 4.4 Prefixes section - point at the reference 
```yaml
Prefixes:
  Vitis_vinifera_PN40024:
    Path: prefixes/Vitis_vinifera_PN40024_v4.fasta
    # Optional: extra regions for targeted coverage/depth stats
#    RegionsOfInterest:
#      NAME: path/to/regions.bed
```
 
Add a second prefix block for the chloroplast if mapping organellar reads separately. Each prefix name must be unique — it becomes part of every output filename.
 
### 4.5 Read data — target → sample → library → lane
 
This is the hierarchy PALEOMIX uses, and getting the **library** level right is critical: PCR-duplicate filtering happens *per library*, so wrongly merging two independent amplifications loses real reads (falsely called duplicates), while wrongly splitting one library lets true duplicates through.
 
```yaml
# Target name = prefixed to all output for this sample
LUX_Vitis_Roman01:
  # Sample name = recorded in the BAM read groups & summary tables
  LUX_Vitis_Roman01:
    # One block per library (I like to name libraries by their index barcode)
    Lib_ACGATA:
      # One entry per lane; name is free-form. Single-end = just the path:
      Lane_1: data/ACGATA_L1_*.fastq.gz
    Lib_GCTCTG:
      # Paired-end = use the {Pair} keyword where the 1/2 mate number sits:
      Lane_1: data/GCTCTG_L1_R{Pair}_*.fastq.gz
      Lane_2: data/GCTCTG_L2_R{Pair}_*.fastq.gz
```
 
Notes on paths:
- For **paired-end**, put `{Pair}` where the mate number (1/2) appears, e.g. `..._R{Pair}_...` matches both `_R1_` and `_R2_`. Wildcards (`*`) are fine for multiple files per lane.
- For **single-end**, just give the path — no `{Pair}`.
- Absolute or relative paths both work. Uncompressed, `.gz`, and `.bz2` FASTQ are all auto-detected — the extension doesn't matter.
## 5. Run the pipeline
 
Dry-run first to validate the makefile and see the planned task graph without executing anything:
 
```bash
paleomix bam run --dry-run makefile.yaml
```
 
Then run for real:
 
```bash
paleomix bam run makefile.yaml
# useful flags:
#   --max-threads N     cap total cores (default: all available)
#   --destination DIR   write outputs somewhere other than beside the makefile
#   --help              full option list
```
 
The pipeline runs steps in parallel where it can, and is **resumable**: if it's interrupted, re-running picks up from the last completed intermediate.
 
> !! Resumption is based on the *existence* of intermediate files, not on makefile contents. If you edit the makefile after a (partial) run, affected steps may **not** automatically re-run. When you change settings mid-project, manually delete the affected intermediate files (in the `<Target>/` working folder) before re-running. See the docs' File Structure section for the layout.
 
## 6. Outputs
 
For a target `LUX_Vitis_Roman01` mapped to prefix `Vitis_vinifera_PN40024`, you'll get:
 
```
LUX_Vitis_Roman01.Vitis_vinifera_PN40024.bam        # final BAM
LUX_Vitis_Roman01.Vitis_vinifera_PN40024.bam.bai    # index
LUX_Vitis_Roman01.Vitis_vinifera_PN40024.coverage   # mean coverage table
LUX_Vitis_Roman01.Vitis_vinifera_PN40024.depths     # per-site depth histogram
LUX_Vitis_Roman01.Vitis_vinifera_PN40024.mapDamage/ # one damage plot set per library
LUX_Vitis_Roman01.summary                           # whole-analysis summary table
LUX_Vitis_Roman01/                                  # intermediate files (see below)
```
 
The **summary** table is where you check the run at a glance: raw read counts, reads surviving trimming/collapsing, fraction mapped to each prefix, and fraction filtered as PCR duplicates. The **mapDamage** folder is your authenticity check — genuinely ancient grapevine pips should show the characteristic elevated C→T at 5′ ends and G→A at 3′ ends, decaying inward.
 
> The `<Target>/` folder holds intermediates and typically uses **3–4×** the final BAM's disk space. It's only kept to allow resumption — once the run is complete and validated, it's safe to delete to reclaim space.
 
## 7. Where the BAMs go next
 
The BAM pipeline stops at aligned, filtered, damage-assessed BAMs. From here the grapevine project can go to:
- **PALEOMIX Phylogenetic pipeline** — genotyping (per-sample consensus / SNP calling) and phylogenetic inference directly from these BAMs, if you want to place the ancient pips relative to modern reference accessions (e.g. the Remich vineyard samples).
- **Your own downstream** — `bcftools`/`GATK` for variant calling, then whatever popgen you're planning with Copenhagen; the BAMs are standard and tool-agnostic.
If you enabled `mapDamage: rescale`, prefer the rescaled BAMs for genotyping so damaged positions don't masquerade as real variants.
 