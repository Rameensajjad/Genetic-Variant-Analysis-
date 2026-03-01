# Assignment #2 — ClinVar/OMIM/UCSC + ACMG/AMP + VCF + Annotation Pipeline

This repository documents a reproducible workflow to:
1) select 3 rare genetic disorders and 1 key variant per disorder  
2) collect evidence from **ClinVar**, phenotype context from **OMIM**, and in-silico scores from **UCSC Genome Browser** (AlphaMissense + RAVEL)  
3) summarize ACMG/AMP interpretation  
4) create a **patient-style VCF** and annotate it with ClinVar fields using **bcftools annotate**  

> Note: The provided VCFs in this repo are **simulated examples** (format-correct, patient-style) intended for the assignment workflow. Replace coordinates/alleles if your selected ClinVar record uses a different representation or genome build.

---

## Selected disorders & example variants

| Disorder | Gene | Example Variant |
|---|---|---|
| Cystic Fibrosis | CFTR | NM_000492.4:c.1521_1523delCTT (p.Phe508del) |
| Huntington’s Disease | HTT | CAG repeat expansion (example shown as STR-style record in VCF) |
| Duchenne Muscular Dystrophy | DMD | NM_004006.3:c.31dupT (p.Ser11fs) |

---

## Folder structure

```
.
├─ data/
│  ├─ patient_data.vcf
│  ├─ patient_data_annotated.vcf
│  ├─ CFTR_Phe508del.vcf
│  ├─ HTT_CAG_expansion.vcf
│  └─ DMD_c31dupT.vcf
├─ scripts/
│  └─ validate_annotation.py
└─ README.md
```

---

## Requirements

- Conda (recommended)  
- `bcftools`  
- `wget` (or curl)  
- Python 3 (optional validation)

---

## Step-by-step reproducible workflow

### 1) Create environment

```bash
conda create -n variant_annotation -c bioconda bcftools wget -y
conda activate variant_annotation
```

### 2) Download ClinVar VCF (choose build to match your patient VCF)

**GRCh37 (hg19) example:**
```bash
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh37/clinvar.vcf.gz
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh37/clinvar.vcf.gz.tbi
# If .tbi isn't downloaded, build it:
bcftools index clinvar.vcf.gz
```

> Critical: chromosome naming must match (e.g., both use `7` or both use `chr7`).

### 3) Prepare your patient VCF

Use `data/patient_data.vcf` as a template (VCFv4.2, 1 sample).  
If your assignment requires **only SNVs/indels**, remove the HTT STR-style record.

### 4) Annotate patient VCF with ClinVar fields

```bash
bcftools annotate   -a clinvar.vcf.gz   -c ID,CLNSIG,CLNDN,CLNACC,GENEINFO   --pair-logic first   data/patient_data.vcf   -o data/patient_data_annotated.vcf
```

Common additional ClinVar fields you may also add (optional):
- `CLNREVSTAT` (review status)
- `CLNSIGCONF` (conflicting significance)
- `CLNHGVS` (HGVS strings, if present in your ClinVar build)

Example:
```bash
bcftools annotate -a clinvar.vcf.gz   -c ID,CLNSIG,CLNREVSTAT,CLNDN,CLNACC,GENEINFO   --pair-logic first   data/patient_data.vcf   -o data/patient_data_annotated.vcf
```

### 5) Validate annotations (optional)

A minimal parser is provided at `scripts/validate_annotation.py`.

Run:
```bash
python3 scripts/validate_annotation.py data/patient_data_annotated.vcf
```

---

## ClinVar / OMIM / UCSC evidence capture (what to include in your report)

### ClinVar
For each variant record, capture:
- Clinical significance (e.g., Pathogenic)
- Review status (e.g., criteria provided, multiple submitters)
- Condition name(s)
- Evidence notes / submissions summary

### OMIM
For each gene/condition, capture:
- Phenotype MIM number
- Inheritance pattern
- Clinical synopsis (key features)

### UCSC Genome Browser
At the variant coordinate, enable tracks:
- **AlphaMissense**
- **RAVEL**
Take screenshots and paste them into your Excel/report fields.

---

## ACMG/AMP reporting (what to write)

For each variant, write:
- Which ACMG criteria apply (PVS1, PS1, PS3, PM2, PP3, etc.)
- Short justification for each
- Final classification

---

## Notes on structural variants (HTT CAG expansion)

Many standard VCF annotation pipelines are optimized for SNVs/indels.
Repeat expansions (like HTT CAG) may:
- not match ClinVar SNV/indel entries
- require specialized representation and tools (repeat callers / STR-specific formats)

The included `HTT_CAG_expansion.vcf` is a **simplified STR-style example** for the assignment.

---

## Source report

This README is adapted from the submitted assignment report content. 
