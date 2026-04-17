# JPEC <img src="https://img.shields.io/badge/Issues-%2B-brightgreen.svg" /><img src="https://img.shields.io/badge/license-GPL3.0-blue.svg" />    
### A Julia package (Binary version) for pedigree correction in animal and plant breeding
## Contents

-   [Overview](#overview)
-   [Installation](#installation)
-   [Input](#Input)
-   [Usage](#Usage)
-   [Output](#output)

------------------------------------------------------------------------
### <u>Overview</u>

`JPEC` is a user-friendly tool for pedigree error correction in animal and plant breeding. It is especially useful for checking the accuracy of recorded pedigree information and identifying likely parentage errors.

`JPEC` is implemented in Julia. To run JPEC, **genotypes** and **LD blocks** are required. Only individuals with genotypes, and whose candidate parents also have genotypes, can be included in pedigree correction.

JPEC provides a flexible framework for defining target offspring and candidate parents.
Because **genotypes** and **LD blocks** are mandatory for all analyses, different analysis objectives are specified through the additional input files:

1. **Pedigree only**  
   JPEC automatically derives target offspring and candidate parents from the pedigree and attempts to correct all eligible parentage records. In this setting, the search is constrained by pedigree-based rules such as **--delta_gen** or **--delta_day**.

2. **Pedigree + target offspring**  
   JPEC attempts to correct the parents of the specified target offspring, while candidate parents are identified from the pedigree under the same pedigree-based constraints.

3. **Target offspring + candidate parents**  
   JPEC attempts to correct the parents of the specified target offspring using only the provided candidate parents.

In addition, JPEC provides an optional **sire–dam swap check**. This module can use:

- pedigree evidence only,
- chrX-based sex inference only, or
- both pedigree and chrX evidence together.

This function is useful for identifying records in which the sire and dam may have been entered in reversed columns.

---

## <u>Installation</u>

⚠️ **Attention:** You should download `JPEC` from the Releases

```bash
#### Install JEPC package from Releases

```R
wget https://github.com/TXiang-lab/JPEC/releases/download/V1.1.0/JPECCompiled.tar.gz
tar -xzvf JPECCompiled.tar.gz
```

👉 **Note**: If users can not download JPEG using the method above, please click on **V1.1.0** under **Releases** on the webpage (https://github.com/TXiang-lab/JPEC/releases/tag/V1.1.0), and click on **JPECCompiled.tar.gz** in **Assets** to download **JPEC**.


### <u>Input</u>

#### 1.Pedigree

The pedigree file should contain four or five columns and no column names.

The first four required columns are:
	1.	**id**
	2.	**sire**
	3.	**dam**
	4.	**generation** (e.g. 1 or 1.5, as a numeric value) or birth date (e.g. 20231001, as an integer)

An optional fifth column, sex, may also be included and should be coded as: **F** for female and **M** for male

For pedigrees without generation records, we recommend using the `trace_pedigree` function in our R package [blupADC](https://github.com/TXiang-lab/blupADC) to trace and reorder the pedigree from older to younger individuals. The reordered pedigree can then be used directly as the input pedigree for JPEC.

``` R
54903 0 0 1
55402 0 0 1
55685 0 0 1
56228 0 0 1
56286 0 0 1
56345 0 0 1
56463 0 0 1
56480 0 0 1
56487 0 0 1
56732 0 0 1
57052 0 0 1
```

#### 2.Genotypes

Genotypes should be phased into haplotypes in VCF format. One option is using **Beagle 5.2** for phasing.

``` R
##fileformat=VCFv4.2
##filedate=20210627
##source="beagle.29May21.d6d.jar"
##INFO=<ID=AF,Number=A,Type=Float,Description="Estimated ALT Allele Frequencies">
##INFO=<ID=DR2,Number=A,Type=Float,Description="Dosage R-Squared: estimated squared correlation between estimated REF dose [P(RA) + 2*P(RR)] an
##INFO=<ID=IMP,Number=0,Type=Flag,Description="Imputed marker">
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
##FORMAT=<ID=DS,Number=A,Type=Float,Description="estimated ALT dose [P(RA) + 2*P(AA)]">
#CHROM  POS     ID      REF     ALT     QUAL    FILTER  INFO    FORMAT  55402   55685   56228   56345   56480   56487   57052   57347   57495
1       6260    M2      T       A       .       PASS    .       GT      0|0     0|0     0|0     0|0     0|0     0|1     0|1     0|0     0|0
1       152890  M17     A       T       .       PASS    .       GT      0|0     1|1     0|0     0|1     1|1     0|0     0|0     0|0     0|0
1       217560  M24     A       T       .       PASS    .       GT      0|0     1|1     0|0     0|1     1|1     0|1     0|1     0|0     0|0
1       282580  M30     A       T       .       PASS    .       GT      1|1     0|0     0|0     0|0     0|0     0|0     0|0     1|0     0|1
1       298030  M33     T       A       .       PASS    .       GT      1|1     0|0     0|0     0|0     0|0     0|0     0|0     1|0     0|1
1       312770  M36     T       A       .       PASS    .       GT      0|1     0|0     0|0     0|0     0|0     0|0     0|0     0|0     0|0
1       322090  M37     T       A       .       PASS    .       GT      1|1     0|0     0|0     0|0     0|0     0|0     0|0     1|0     0|1
1       431960  M46     T       A       .       PASS    .       GT      1|1     0|0     0|0     0|0     0|0     0|0     0|0     1|0     0|1
1       566310  M53     T       A       .       PASS    .       GT      0|0     0|0     0|1     0|0     1|0     0|1     0|1     0|1     0|0
1       627770  M63     T       A       .       PASS    .       GT      0|0     1|0     0|0     0|1     0|0     0|0     0|0     0|0     0|0
1       769890  M79     T       A       .       PASS    .       GT      0|0     1|0     0|0     0|1     0|0     0|0     0|0     0|0     0|0
```

Download **Beagle 5.2** here:  
https://faculty.washington.edu/browning/beagle/b5_2.html#download

Example command for phasing and imputing with **Beagle 5.2**:

```bash
java -Xmx50g -jar beagle.5.2.jar gt=genotype_file.vcf out=output_file_name impute=true nthreads=10
```

#### 3.LD blocks

We recommend using **PLink1.9** for constructing LD blocks (**plink.blocks.det**) from **genotypes**.

``` R
 CHR          BP1          BP2           KB  NSNPS SNPS
   1       282580       431960      149.381      4 M30|M33|M37|M46
   1      3161360      3358350      196.991     12 M344|M347|M349|M350|M354|M356|M357|M360|M361|M362|M363|M364
   1      4505080      4545150       40.071      4 M499|M500|M501|M502
   1      4592550      4785890      193.341     11 M507|M510|M511|M512|M515|M516|M518|M520|M521|M527|M533
   1      5126710      5311850      185.141      5 M569|M572|M576|M589|M591
   1      5509420      5621020      111.601     10 M611|M614|M618|M619|M620|M621|M622|M623|M624|M626
   1      6538020      6556580       18.561      2 M735|M738
   1      6577500      6666710       89.211      9 M740|M741|M742|M743|M746|M747|M748|M751|M753
```

Download **PLink1.9** here:  
https://www.cog-genomics.org/plink/

Example command for phasing and imputing with **PLink1.9**:

```bash
plink --vcf haplotype.vcf --blocks no-pheno-req --blocks-max-kb 200
```

## <u>Usage</u>

#### 🌈Usage of JPEC in Linux

```bash
../bin/JPEC \
  --blocks plink.blocks.det \      # LD blocks
  --vcf haplotype.vcf \            # genotypes
  --ped ped.txt \                  # pedigree file with 4 columns
  --findex 1 \                     # parent identification: 1 = Sire (father) only, 2 = Dam (mother) only, 3 = both sire and dam
  --score \                        # write the score matrix for each comparison between offspring and parents
  --o /result \                    # output path of JPEC
  --julia-args --threads=20        # Julia arguments: specify the number of threads used
```


##### **Main Parameter:**

◼ **--blocks [file]**: LD block file, e.g. plink.blocks.det.

◼ **--vcf [file]**: Phased genotype file in VCF format.

◼ **--ped [file]**: Pedigree file with 4 or 5 columns.

◼ **--findex [number]**
Parent identification mode:
	•	1 = sire only
	•	2 = dam only
	•	3 = both sire and dam

◼ **--ped [file]**: Pedigree file with 4 or 5 columns.

◼ **--score**: Write the score matrix for each comparison between offspring and candidate parents.

◼ **--o [path]**: Output directory.

◼ **--julia-args --threads=[number]**: Number of Julia threads.

##### **Additional Parameter:**

◼ **--gen [number]**: 
  Minimum generation gap between offspring and parent when the 4th column of the pedigree is **generation**. This parameter can be numeric, for example 1, 1.5, or 2.0.
Default: 1

◼ **--day [number]**:
Minimum offspring–parent interval (in days), used when the 4th column of the pedigree file is **birth date**. If this option is not specified, JPEC computes it automatically as the minimum birth-date difference across all recorded offspring–parent pairs in the pedigree.

◼ **--offspring [file]**: file of target offsprings, one column.

◼ **--parents [file]**: file of candidate parents, one column.

##### **Optional inputs for sire–dam swap checking:**

JPEC provides an optional sire–dam swap checking module. Depending on the `--sex-swap` setting, this module can use:
###### **Swap-check mode**

The swap-check mode is controlled by: `--sex-swap` (default: `nothing`)

optional values are:

- **`pedigree`** : use pedigree evidence only
- **`chrX`** : use chrX evidence only
- **`pedigree_chrX`** : use both pedigree and chrX evidence; if they conflict, pedigree has priority
- **`chrX_pedigree`** : use both pedigree and chrX evidence; if they conflict, chrX has priority

###### **Pedigree-based swap checking**
When `--sex-swap pedigree` is used, JPEC evaluates whether an individual appears predominantly in the sire column or the dam column across the pedigree. This can help identify records in which sire and dam IDs may have been entered in reversed columns.

**`--swap-threshold`**  defines how strongly an individual must be associated with one pedigree column before JPEC treats it as role-specific.

For example, if an ID appears 10 times in the pedigree and 9 of those appearances are in the sire column, then its sire-column proportion is `0.90`. With `--swap-threshold 0.90`, this ID would be treated as a sire-like ID. Similarly, if an ID appears mainly in the dam column, it can be treated as dam-like.

**`--swap-min-count`**  defines the minimum number of pedigree occurrences required before an ID is evaluated in pedigree-based swap checking.

For example, if an ID appears only once or twice in the pedigree, there is usually not enough information to decide whether it is mainly sire-like or dam-like. With `--swap-min-count 3`, JPEC only evaluates IDs that appear at least 3 times across the sire and dam columns combined.


###### **Optional chrX VCF for swap checking**

If the sire-dam swap checking module uses chrX evidence, an additional **chrX VCF file** can be provided.

If the sex chromosome in the VCF is not labeled as `X`, users can specify the corresponding chromosome label(s) with `--chrX-names` (e.g.: `19`) so that JPEC can recognize chromsome 19 as sex chromosome and read the sex chromosome correctly.

chrX-based swap checking is controlled by:

- `--chrX-vcf-file` (default: not used)
- `--chrX-names` (default: `X,chrX`)
- `--male-het-range` (default: `0.0,0.05`)
- `--female-het-range` (default: `0.20,1.0`)

The sex inference is controlled by two heterozygosity intervals:

- `male_het_range`
- `female_het_range`

For example:

- **Pig chrX**
  - `male_het_range = (0.0, 0.05)`
  - `female_het_range = (0.20, 1.0)`

- **Chicken chrZ** (opposite heterozygosity pattern)
  - `male_het_range = (0.20, 1.0)`
  - `female_het_range = (0.0, 0.05)`

When both pedigree and chrX evidence are used together, JPEC integrates the two sources according to the selected `--sex-swap` mode.


## <u>Output</u>

**JPEC_vs_Original_Sire.txt** lists the individuals whose parents were corrected. The first column contains the offspring ID, the second column contains the original parent, and the third column contains the parent corrected by JPEC.

``` R
offspring Original JPEC
76092 64806 63032
76271 64712 62673
76475 65120 69518
76533 65365 66269
76720 66740 67916
```

**JPEC_ped.txt** is the corrected pedigree. Its 4 columns are id, sire, dam, generation/birthday.

``` R
76080 63032 67598 3
76092 63032 69141 3
76269 62673 65612 3
76271 62673 65612 3
76450 69518 64937 3
76475 69518 68025 3
76514 66269 69048 3
76533 66269 62981 3
76701 67916 65726 3
76720 67916 66079 3
```

**JPEC_Sire_scored_matrix.txt** contains the matching scores between target individuals and candidate sires if you search only for sires using `--findex 1`. Column names are candidate sires, and row names are target individuals. If you search only for dams using `--findex 2`, the output file will be **JPEC_Dam_scored_matrix.txt**. If you search for both sires and dams using `--findex 3`, both output files will be generated.

``` R
Offspring_Parents	61202	58769	60903	59261	57881	59926	60884	60886	57726	60896	58864	59150	56463	59815	57558	54903	59990	57052	63032	64806	62673	64712	69518	65120	66269	65365	67916	66740
62673	0.9668446192750773	0.4072841893698867	0.38018951023786796	0.2787803194460987	0.296780298513859	0.4070335746315158	0.31327593669380294	0.3351974314339646	0.3618083343829631	0.3186467155036397	0.3384568667233928	0.2861703636060899	0.31349162972260963	0.38699273821940494	0.3143185983700242	0.3127742810118328	0.23211451843089936	0.26668253984374324	
62981	0.23413276521622983	0.9772407979769598	0.29958317444707583	0.25950072016382775	0.30180431955649045	0.35562040862624095	0.29007716082737156	0.31286961466138846	0.2853719561961806	0.2954168080311522	0.3817931721250195	0.3052109285203438	0.2564330941230786	0.29768885863650574	0.2921828374643071	0.27736665608150685	0.26069407903139286	0.32334482787980967	
63032	0.3581071653162529	0.9901657769036246	0.38716446755458656	0.29836450041189777	0.3320430737015859	0.44888638108639894	0.2778886183875142	0.29150496869201437	0.37727319684806415	0.29376001105657434	0.4199847728158508	0.32639487617397545	0.2539111837349316	0.30802626473460976	0.444008877779452	0.33854061470788377	0.32776291631865195	0.3014284303972418	
64712	0.36646098501525465	0.2658175723585793	0.9794886203989884	0.23494225421708925	0.293605876356698	0.30417808389148654	0.3108567594966165	0.3756013785115057	0.2339393023941997	0.3554807391830273	0.3096142998688066	0.21574142422409648	0.23190554368873095	0.2438444641519153	0.3481182439417475	0.25870715937261435	0.24129365540919612	0.24986658881683987
64806	0.33930911710485634	0.35337870390501364	0.2596764828441511	0.989884799100871	0.1903523486539358	0.42650830730357703	0.24852814064129614	0.20011948748881603	0.2957953122710862	0.34788798351596895	0.333673638248836	0.2699965834154501	0.43322084911156833	0.3507122749635218	0.3464196211159032	0.2579501695276766	0.29094255343558917	0.22992627000188504	
64937	0.25182348670871335	0.29802705663771945	0.32045839521534936	0.24779707893272507	0.9859511098623209	0.4215790952514751	0.26946624981487755	0.3010417602229365	0.413045232460509	0.313892774663315	0.35831494199514463	0.27790904903621727	0.2968993991674146	0.20567542939337158	0.36883632998104976	0.20125949853401381	0.37576388121685134	0.21474958068275884	
65120	0.37140650429678856	0.35069790345148594	0.3344782945646583	0.32288736415629615	0.35412455776208734	0.9747119977521775	0.18572272534059553	0.25088340759175265	0.37461476464411947	0.36768526702940263	0.40959681657847935	0.289404161777102	0.37963529750197356	0.29968445586323866	0.36332416607774853	0.24544752253354807	0.2855522739495435	0.30416580624267225
65365	0.2947496911894904	0.39185021501945416	0.4318025576267659	0.3017924547788029	0.359381620463014	0.32427710818971117	0.9676875526833381	0.4247592780477176	0.3734560704240447	0.38013552468713574	0.35385510609178	0.3188913751851087	0.2740384333303562	0.2949593351988206	0.3356550972824024	0.26369804758397053	0.2963447222564031	0.2838889170378491
```

**JPEC_Sire_off_id.txt** contains the target individuals for sire identification in the pedigree when only a pedigree file is provided and you search only for sires using `--findex 1`. **JPEC_Dam_off_id.txt** contains the target individuals for dam identification in the pedigree when only a pedigree file is provided and you search only for dams using `--findex 2`. Both output files will be generated if you search for both sires and dams using `--findex 3`. Both files use the format below.

``` R
off_id
62673
62981
63032
64712
64806
64937
65120
65365
```

**JPEC_Sire_par_id.txt** contains the candidate sires for parent identification in the pedigree when only a pedigree file is provided and you serach only for sires using `--findex 1`. **JPEC_Dam_par_id.txt** contains the candidate individuals for dam identification in the pedigree when only a pedigree file is provided and you search only for dams using `--findex 2`. Both output files will be generated if you search for both sires and dams using `--findex 3`. Both files use the format below.


``` R
par_id
61202
58769
60903
59261
57881
59926
60884
60886
```

























