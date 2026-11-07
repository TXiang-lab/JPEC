# JPEC <img src="https://img.shields.io/badge/Issues-%2B-brightgreen.svg" /><img src="https://img.shields.io/badge/license-GPL3.0-blue.svg" /><img src="https://img.shields.io/badge/version-2.0.0-orange.svg" />

JPEC is a tool for pedigree error correction in animal and plant breeding.

It compares phased haplotypes between each offspring and its candidate parents, scores every offspring–candidate pair, and reports the most likely sire or dam. It checks the recorded pedigree, finds likely parentage errors and writes a corrected pedigree.

JPEC 2.0 is a single compiled program (`bin/JPEC`). It does not need Julia, Python or a network connection.

## Contents

- [What's New in 2.0](#whats-new-in-20)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Input Files](#input-files)
- [How JPEC Decides](#how-jpec-decides)
- [Command-Line Options](#command-line-options)
- [Output Files](#output-files)
- [Larger Example](#larger-example)
- [Troubleshooting](#troubleshooting)

<a id="whats-new-in-20"></a>

## What's New in 2.0

Compared with JPEC V1.1.0:

- **One program.** `bin/JPEC` prepares the inputs, scores sires and/or dams and writes the corrected pedigree in one command, without any extra scripts.
- **Opposing-homozygote (OH) check.** For each pair JPEC also counts the genome-wide rate of opposing homozygous genotypes (AA vs BB). A true parent has an OH rate close to 0.
- **Adaptive write-back limit (default).** The best-scoring candidate replaces the recorded parent only when its OH rate is below a limit estimated from the data itself (see [How JPEC Decides](#how-jpec-decides)).
- **Sex inference.** Missing sexes are inferred from the pedigree roles, so a dam is not proposed as a sire.
- **Safer targets.** By default only offspring whose recorded parent is genotyped are searched; records that cannot be checked are left unchanged.
- **Detailed outputs.** Per-pair evidence, per-candidate summaries, run metadata and a log for every run.

Options of V1.1.0 that are no longer available: `--sex-swap`, `--noMC`, `--nGC`.

<a id="installation"></a>

## Installation

Download the compiled release (Linux x86_64, glibc ≥ 2.28):

```bash
wget https://github.com/TXiang-lab/JPEC/releases/download/V2.0.0/JPECCompiled.tar.gz
tar -xzvf JPECCompiled.tar.gz
```

If `wget` is not available, download `JPECCompiled.tar.gz` from the assets on the release page:

https://github.com/TXiang-lab/JPEC/releases/tag/V2.0.0

The package contains:

```text
JPECCompiled/
├── bin/JPEC        the program
├── lib/  share/    runtime libraries used by bin/JPEC
└── example/        simulated example data, reference output and simulation truth
```

Keep the folder together: `bin/JPEC` finds `lib/` and `share/` by relative path.

<a id="quick-start"></a>

## Quick Start

```bash
cd JPECCompiled/example
../bin/JPEC \
  --blocks example_data/plink.blocks.det \
  --vcf example_data/haplotype.vcf \
  --ped example_data/ped.txt \
  --findex 3 \
  --o result \
  --julia-args --threads=2
```

The example data are simulated (no real animals): 560 genotyped individuals, 33,363 SNPs on 18 autosomes and 4,552 LD blocks. 27 of the 381 target offspring have a wrong recorded sire.

The run takes about 10 seconds. Expected result: the 27 wrong sires are replaced by the true sires, no correct record is changed, and no dam is changed.

Check the result:

```bash
for f in expected_output/*; do cmp $f result/$(basename $f) && echo "SAME $(basename $f)"; done
python3 sim_truth/check_accuracy.py result/JPEC_plus_ped.txt    # optional, needs python3
```

`sim_truth/` holds the true parents of the simulation. Use it only for checking, never as input.

The example prints `WARNING ... false-acceptance rate of 1.42 % instead of 0.1 %`. This is expected and not an error: the example is small, so JPEC relaxes the rate automatically (see [How JPEC Decides](#how-jpec-decides)).

<a id="input-files"></a>

## Input Files

Every run needs three files: a pedigree, a phased VCF and an LD block file. Only individuals present in the VCF are analysed.

### 1. Pedigree File

No header. 4 columns, or 5 columns if sex is known.

| Column       | Meaning                  | Example                         |
| ------------ | ------------------------ | ------------------------------- |
| 1            | Individual ID            | `76080`                         |
| 2            | Sire ID                  | `63032`                         |
| 3            | Dam ID                   | `67598`                         |
| 4            | Generation or birth date | `3`, `20231001` or `2023-10-01` |
| 5 (optional) | Sex: `M` or `F`          | `M`                             |

Use `0`, `NA` or `-999` for an unknown sire or dam.

```text
54903 0 0 1 M
55402 0 0 1 F
76080 63032 67598 3 M
76092 64806 69141 3 F
```

When column 4 holds birth dates, add `--age-mode birthdate`.

If column 5 is missing, sexes are inferred from the pedigree: an individual recorded only as a sire is male, only as a dam is female.

### 2. Phased VCF File

Genotypes must be phased (`0|0`, `0|1`, `1|0`, `1|1`); an unphased heterozygote (`0/1`) stops the run. Sites that are not strict A/C/G/T biallelic are dropped automatically. Missing genotypes (`.`, `./.`) are skipped but carry no information, so phase and impute first.

```text
##fileformat=VCFv4.2
#CHROM  POS     ID   REF  ALT  QUAL  FILTER  INFO  FORMAT  55402  55685
1       6260    M2   T    A    .     PASS    .     GT      0|0    0|1
1       152890  M17  A    T    .     PASS    .     GT      1|0    1|1
```

Phasing and imputation with Beagle 5.2 ([download](https://faculty.washington.edu/browning/beagle/b5_2.html#download)):

```bash
java -Xmx50g -jar beagle.5.2.jar gt=genotype_file.vcf out=haplotype impute=true nthreads=10
```

### 3. LD Block File

PLINK `.blocks.det` format. JPEC uses the first three columns (chromosome, start, end).

```text
CHR      BP1      BP2       KB  NSNPS SNPS
1     282580   431960  149.381      4 M30|M33|M37|M46
1    3161360  3358350  196.991     12 M344|M347|M349
```

With PLINK 1.9 ([download](https://www.cog-genomics.org/plink/)):

```bash
plink --vcf haplotype.vcf --blocks no-pheno-req --blocks-max-kb 200
```

### 4. Optional Lists

`--offspring FILE` (target offspring) and `--parents FILE` (candidate parents): one ID per line, no header, no `NA` rows.

<a id="how-jpec-decides"></a>

## How JPEC Decides

1. **Targets.** Genotyped offspring with a recorded parent of the searched role (sire or dam) whose recorded parent is also genotyped.
2. **Candidates.** All genotyped individuals of the right sex that are old enough (at least `--gen` generations, or `--day` days, older than the offspring). Candidates with a genome-wide OH rate above 5 % are not block-scored.
3. **Score.** Each pair gets a block-matching score; the best-scoring candidate is the *top-1*.
4. **Write-back (default `--oh-gate np`).** The top-1 is written to the corrected pedigree only when its OH rate is below a limit. JPEC sets this limit from the known non-parents in your own data, so that only about 0.1 % of non-parents would pass it (`--oh-gate-far 0.001`). When the top-1 is above the limit, the recorded parent is kept and the offspring is listed in `JPEC_plus_oh_unconfirmed_<Role>.txt`.

The adaptive limit needs enough data:

- At least 200 targets per role whose top-1 is the recorded parent; otherwise the run stops before writing the pedigree. On smaller data, use a fixed limit (`--oh-gate 0.004`), `--accept-error-rate 0.05` or `--top1`.
- About 5,000 known non-parents per role to estimate the 0.1 % quantile. With fewer, JPEC relaxes the rate and prints a warning (as in the Quick Start example).

<a id="command-line-options"></a>

## Command-Line Options

```bash
bin/JPEC [options] --julia-args --threads=N
bin/JPEC --help        # full list
bin/JPEC --version
```

Options before `--julia-args` are JPEC options; `--julia-args` and the thread setting must come last.

### Required

| Option          | Meaning                              |
| --------------- | ------------------------------------ |
| `--blocks FILE` | LD block file (PLINK `.blocks.det`)  |
| `--vcf FILE`    | Phased VCF                           |
| `--ped FILE`    | Pedigree (4 or 5 columns, no header) |
| `--o DIR`       | Output directory; must not exist yet |

### Parent Search

| Option                                         | Meaning                                                      | Default                                    |
| ---------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| `--findex 1\|2\|3`                             | Sires / dams / both                                          | `1`                                        |
| `--offspring FILE`                             | Target offspring list                                        | genotyped offspring with a recorded parent |
| `--parents FILE`                               | Candidate parent list                                        | all genotyped individuals                  |
| `--all-candidates` / `--pedigree-parents-only` | Score every eligible candidate, or only individuals recorded as a parent of that role (faster) | `--all-candidates`                         |
| `--gen N`                                      | Minimum generation gap parent → offspring                    | `1`                                        |
| `--age-mode generation\|birthdate`             | Meaning of pedigree column 4                                 | `generation`                               |
| `--day N`                                      | Birth-date mode: minimum parent–offspring gap in days        | estimated from the recorded pedigree       |
| `--allow-ungenotyped-parent`                   | Also search offspring whose recorded parent is not genotyped | off                                        |

### Sex

| Option                                                  | Meaning                                           | Default    |
| ------------------------------------------------------- | ------------------------------------------------- | ---------- |
| (default)                                               | Infer missing sexes from pedigree roles           | on         |
| `--infer-sex-min-count N`, `--infer-sex-min-fraction X` | Thresholds for individuals recorded in both roles | `1`, `0.9` |
| `--no-infer-sex`                                        | No inference                                      | off        |
| `--allow-unknown-sex`                                   | Let candidates of unknown sex compete             | off        |


### Scoring

| Option                                     | Meaning                                                      | Default |
| ------------------------------------------ | ------------------------------------------------------------ | ------- |
| `--score`                                  | Also write the full offspring × candidate score matrix (`score_matrix.tsv`) | off     |
| `--oh-exclusion-rate R` / `--no-oh-filter` | Candidates with genome-wide OH rate above `R` are not block-scored / no pre-filter | `0.05`  |
| `--include-unknown`                        | Also score offspring without a recorded parent               | off     |

### Runtime

| Option                     | Meaning                            | Default                        |
| -------------------------- | ---------------------------------- | ------------------------------ |
| `--julia-args --threads=N` | Number of threads (last arguments) |                                |
| `--threads N`              | Threads of the scoring runs        | `--julia-args` value, else `2` |

### Sub-Commands

| Command                       | Meaning                                                      |
| ----------------------------- | ------------------------------------------------------------ |
| `bin/JPEC score ...`          | Run only the scoring core (`bin/JPEC score --help`)          |
| `bin/JPEC write-pedigree ...` | Write a corrected pedigree from existing scores with another rule, without re-scoring (`bin/JPEC write-pedigree --help`) |

<a id="output-files"></a>

## Output Files

All files are written to the `--o` directory.

| File                                          | Content                                                      |
| --------------------------------------------- | ------------------------------------------------------------ |
| `JPEC_plus_ped.txt`                           | Corrected pedigree, same columns as the input                |
| `JPEC_plus_vs_Original_Sire.txt` / `_Dam.txt` | Changed records                                              |
| `JPEC_plus_oh_gate_<Role>.txt`                | How the OH limit was set                                     |
| `JPEC_plus_oh_unconfirmed_<Role>.txt`         | Offspring whose top-1 was above the OH limit (recorded parent kept) |
| `JPEC_plus_role_conflicts.txt`                | Changes refused because the candidate is already the other parent of that offspring |
| `jpec_plus_<role>/`                           | Detailed results: `assignments.tsv` (one row per target), `pair_evidence.tsv` (per pair), `candidate_summary.tsv`, `metadata.toml` |
| `jpec_plus_<role>.log`, `input/`              | Run log and prepared inputs                                  |

`JPEC_plus_vs_Original_Sire.txt`:

```text
offspring Original JPEC_plus score gap status oh_rate
69797 62670 67113 0.98418278 0.24340949 SCORE_ONLY 0
70424 65248 64156 0.9677065 0.1599297 SCORE_ONLY 0
```

`score` is the top-1 score, `gap` the difference to the second-best candidate, `oh_rate` the OH rate between offspring and new parent.

`JPEC_plus_oh_unconfirmed_Sire.txt`:

```text
offspring recorded top1 score oh_rate case group limit
```

<a id="larger-example"></a>

## Larger Example

`JPEC_sim_example_medium.tar.gz` (85 MB; 1.1 GB unpacked; release asset) is a larger simulated data set: 8,645 individuals and 7,415 target offspring, 569 of them with a wrong recorded sire. It has more than 5,000 known non-parents per role, so the default limit is estimated at 0.1 % without a warning.

```bash
tar -xzf JPEC_sim_example_medium.tar.gz
cd JPEC_sim_example_medium
/path/to/JPECCompiled/bin/JPEC \
  --blocks example_data_sim_medium/plink.blocks.det \
  --vcf example_data_sim_medium/haplotype.vcf \
  --ped example_data_sim_medium/ped.txt \
  --findex 3 \
  --o result_sim_medium \
  --julia-args --threads=8
```

About 4 minutes with 8 threads and 1.4 GB memory. Expected result: all 569 errors corrected and no wrong changes; 30 further wrong records of parent individuals are corrected as well. Reference output and simulation truth are included in the package.

<a id="troubleshooting"></a>

## Troubleshooting

| Message or problem                                           | Cause and fix                                                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Output directory exists                                      | `--o` must be a new directory. Remove it or choose another name. |
| `WARNING ... false-acceptance rate of X % instead of 0.1 %`  | Fewer than about 5,000 known non-parents. Results are still written; the limit is less strict. |
| `--oh-gate np: only N pseudo-true pairs ... need >= 200` (no corrected pedigree) | Fewer than 200 targets per role whose top-1 is the recorded parent. Rerun with `--oh-gate 0.004`, `--accept-error-rate 0.05` or `--top1`, or apply another rule to the existing scores with `bin/JPEC write-pedigree`. |
| Many offspring are `PARENT_NOT_GENOTYPED`                    | Their recorded parent is not in the VCF. Add `--allow-ungenotyped-parent` to search them anyway. |
| A dam is never proposed as a sire, or a candidate is missing | Candidates need sex `M`/`F` (given or inferred) and must be older than the offspring. See `candidate_summary.tsv`. |
| Very few SNPs or OH rates of exactly 0                       | Use all chromosomes. With few SNPs, many non-parents (e.g. full sibs of the true parent) have OH = 0 and the limit becomes 0. |
| Long run time or high memory                                 | Use more threads, or `--pedigree-parents-only` to score only recorded parents. |
