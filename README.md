# JPEC
Julia package (Binary version) for pedigree correction in animal and plant breeding


#### Install JEPC via github (way1)
```Julia
git lfs install
git clone git clone https://github.com/TXiang-lab/JPEC.git
```

#### Usage of JPEC in Linux (command line)
```Bash
..../bin/JPEC \
	   --blocks  plink.blocks.det \  # generated by Plink
	   --vcf  17488.vcf \            # vcf file
	   --ped  ped.txt \              # pedgiree file
	   --findex 1 \                  # index of finding parents, 1 means only find father, 2 means only find Mother, 3 means both find father and mother
	   --score \                     # write the score matrix for each comparsion between offspring and parents   
	   --o /data2/fck/test2 \        # output path of JEPC
           --julia-args --threads=20      # Julia arguments, specify the number of threads used in JPEC
```
