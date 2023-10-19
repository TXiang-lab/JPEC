# JPEC <img src="https://img.shields.io/badge/Issues-%2B-brightgreen.svg" /><img src="https://img.shields.io/badge/license-GPL3.0-blue.svg" />    
### Julia package (Binary version) for pedigree correction in animal and plant breeding
## Contents

-   [Overview](#overview)
-   [Installation](#installation)
-   [Data preparation](#data-preparation)
-   [Usage](#usage)
-   [Output](#output)

------------------------------------------------------------------------
### <u>Overview</u>

`JPEC` is a useful and user-friendly tool for pedigree correction in animal and plant breeding, especially when users doubt the accuracy of the pedigree records.  This is the update Julia version of our R package **rPEC**. 

**Haplotype file** in **VCF** format which has been phased by phasing software like **Beagle**, and  **LD blocks**  which are constructed by **Plink**, are necessary.

**Note that** only individuals who and whose parent have genotypes would be checked. Apart from **Haplotype file** and **LD blocks**, `JPEC`  can be used under three situations below: 1. providing **pedigree**; 2.  providing **pedigree** and **target individuals' id**; 3. providing **target individuals' id** and **candidate individuals' id**.

### <u>Installation</u>

#### Install JEPC via github

```R
git lfs install  #key step
git clone https://github.com/TXiang-lab/JPEC.git
```

**👉 Note:In the situation where the user can not download the JPEG as shown above, users can download it as follows:** 
```R
wget https://github.com/TXiang-lab/JPEC/releases/download/V1.0.0/JPECCompiled.zip
unzip JPECCompiled.zip
```

### <u>Data preparation</u>

#### ①Pedigree

It can be four or five columns(NO tittle). 4 columns are id, sire, dam, and generation(e.g.1)/birthday(e.g.20231001). The extra fifth column is sex, represented by **F** and **M**.

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

#### ② VCF (ONLY autosomes)

Haplotype data in VCF format is recommended to be phased by software **Beagle5.2**.

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

PLUS: 

Download linkage of **Beagle5.2** 

https://faculty.washington.edu/browning/beagle/beagle.html#download

command of phasing and imputing by software **Beagle5.2** 

``` {.r}
java -Xmx50g -jar beagle.5.2.jar gt=genotype_file.vcf out=out_put_file_name impute=true nthreads=10
```

#### ③ LD blocks

LD blocks (**plink.blocks.det**) are recommended to be constructed by software **PLink1.9**

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

PLUS: 

Download linkage of **PLink1.9**

https://www.cog-genomics.org/plink/

command of constructing LD blocks by software **PLink1.9**

``` {.r}
plink --vcf haplotype.vcf --blocks no-pheno-req --blocks-max-kb 200
```

## <u>Usage</u>

#### 🌈Usage of JPEC in Linux

``` R
../bin/JPEC \
	   --blocks  plink.blocks.det \  # generated by Plink
	   --vcf  haplotype.vcf \            # phased vcf file
	   --ped  ped.txt \              # pedgiree file with 4 columns
	   --findex 1 \                  # index of finding parents, 1 means only find father, 2 means only find Mother, 3 means both find father and mother
	   --score \                     # write the score matrix for each comparsion between offspring and parents   
	   --o /result \        # output path of JEPC
       --julia-args --threads=20     # Julia arguments, specify the number of threads used in JPEC
```

##### **Parameter:**

◼ **--gen [number]** specifies the generation between a kid and a parent when the forth column of the pedigree is generation, default is **1**.

◼ **--day [number]** specifies the least interval days between a kid and a parent when the forth column of the pedigree is birthday, which can be calculated **automatically**.

◼ **--offspring [target_individuals.txt]** specifies target individuals file with one column.

◼ **--parents [candidate_individuals.txt]** specifies candidate individuals file with one column.

## <u>Output</u>

**JPEC_vs_Original_Sire.txt** shows the individuals whose parents are corrected, the first column is the individuals' id, the second column is their original parents, and the third column is their parents corrected by JPEC.

``` R
offspring Original JPEC
76092 64806 63032
76271 64712 62673
76475 65120 69518
76533 65365 66269
76720 66740 67916
```

**JPEC_ped.txt** is the corrected pedigree, 4 columns are id, sire, dam, generation/birthday.

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

**JPEC_Sire_socred_matrix.txt** is the score of each candidate individuals per target individual.

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

**JPEC_Sire_off_id.txt** is target individuals in pedigree.

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

**JPEC_Sire_par_id.txt** is candidate individuals in pedigree.

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

























