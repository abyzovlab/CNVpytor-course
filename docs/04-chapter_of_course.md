
# Data import 




## Learning Objectives

- Import read depth signal
- Import variant information

## Import read depth signal

Make sure that you have indexed alignment (SAM, BAM or CRAM) file.
Initialize your CNVpytor project by running:
```
> cnvpytor -root file.pytor -rd file.bam [-chrom name1 ...] [-T ref.fa.gz]
```
where:
```
* file.pytor -- specifies output CNVpytor file (HDF5 file)
* name1 ... -- specifies chromosome name(s).
* file.bam -- specifies bam/sam/cram file name.
* -T ref.fa.gz -- specifies reference genome file (only for cram file without reference genome).
```

Chromosome names must be specified the same way as they are described in the sam/bam/cram header, 
e.g., chr1 or 1. One can specify multiple chromosomes separated by space. If no chromosome is specified, 
read mapping is extracted for all chromosomes in the sam/bam file. 
Note that the pytor file is not being overwritten.

**Examples:**

```
> cnvpytor -root NA12878.pytor -chrom 1 2 3 -rd NA12878_ali.bam
```
for bam files with a header like this:
```
@HD VN:1.4 GO:none SO:coordinate
@SQ SN:1 LN:249250621
@SQ SN:2 LN:243199373
@SQ SN:3 LN:198022430
```

or
```
> cnvpytor -root NA12878.pytor -chrom chr1 chr2 chr3 -rd NA12878_ali.bam
```
for bam files with a header like this:
```
@HD VN:1.4 GO:none SO:coordinate
@SQ SN:chr1 LN:249250621
@SQ SN:chr2 LN:243199373
@SQ SN:chr3 LN:198022430
```


After -rd step file file.pytor is created and read depth data binned to 100 base pair bins will be stored 
in _pytor_ file.

Chromosome names and lengths are parsed from the input file header and used to 
detect reference genome.

### check reference genome
To check is reference genome detected use:

```
> cnvpytor -root file.pytor -ls
```

CNVpytor will print out details about file.pytor including line that specify which reference genome is
used and are there available GC and mask data:
```
Using reference genome: hg19 [ GC: yes, mask: yes ]
```

Command `-ls` is useful if you want to check content of _pytor_ file but also date and version of CNVpytor 
that created it.

### Predicting CNV regions

First we have to chose bin size. By CNVpytor design it have to be divisible by 100. 
Here we will use 10 kbp and 100 kbp bins.

To calculate read depth histograms, GC correction and statistics type:
```
> cnvpytor -root file.pytor -his 10000 100000
```

Next step is partitioning using mean-shift method:
```
> cnvpytor -root file.pytor -partition 10000 100000
```

Finally we can call CNV regions using commands:
```
> cnvpytor -root file.pytor -call 10000 > calls.10000.tsv
> cnvpytor -root file.pytor -call 100000 > calls.100000.tsv
```

Result is stored in tab separated files with following columns:
```
* CNV type: "deletion" or "duplication",
* CNV region (chr:start-end),
* CNV size,
* CNV level - read depth normalized to 1,
* e-val1 -- e-value (p-value multiplied by genome size divided by bin size) calculated using t-test statistics between RD statistics in the region and global,
* e-val2 -- e-value (p-value multiplied by genome size divided by bin size) from the probability of RD values within the region to be in the tails of a gaussian distribution of binned RD,
* e-val3 -- same as e-val1 but for the middle of CNV,
* e-val4 -- same as e-val2 but for the middle of CNV,
* q0 -- fraction of reads mapped with q0 quality in call region,
* pN -- fraction of reference genome gaps (Ns) in call region,
* dG -- distance from closest large (>100bp) gap in reference genome.
```
Using viewer mode we can filter calls based on five parameters: 
```
CNV size, e-val1, q0, pN and dG:
```

```
> cnvpytor -root file.pytor [file2.pytor ...] -view 10000

cnvpytor> set Q0_range -1 0.5        # filter calls with more than half not uniquely mapped reads
cnvpytor> set p_range 0 0.0001       # filter non-confident calls 
cnvpytor> set p_N 0 0.5              # filter calls with more than 50% Ns in reference genome 
cnvpytor> set size_range 50000 inf   # filter calls smaller than 50kbp 
cnvpytor> set dG_range 100000 inf    # filter calls close to gaps in reference genome (<100kbp)
cnvpytor> print calls                # printing calls on screen (tsv format)
...
...
cnvpytor> set print_filename file.xlsx   # output filename (xlsx, tsv or vcf)
cnvpytor> set annotate               # turn on annotation (optional - takes a lot of time)
cnvpytor> print calls                # generate output file with filtered calls 
cnvpytor> quit
```

Upper bound for parameters `size_range` and `dG_range` can be _inf_ (infinity).

If there are multiple samples (pytor files) there will be an additional 
column with sample name in tsv format, multiple sheets in Excel format, and
multiple sample columns in vcf format.


## Import SNP data

### From variant file

To import variant data from VCF file use following command:
```
> cnvpytor -root file.pytor -snp file.vcf.gz [-sample sample_name] [-chrom name1 ...] [-ad AD_TAG] [-gt GT_TAG] [-noAD]
```
where:
```
* file.pytor -- specifies cnvpytor file,
* file.vcf -- specifies variant file name.
* sample_name -- specifies VCF sample name,
* name1 ... -- specifies chromosome name(s),
* -ad AD_TAG -- specifies AD tag used in vcf file (default AD)
* -gt GT_TAG -- specifies GT tag used in vcf file (default GT)
* -noAD -- ref and alt read counts will not be readed (see next section)
```
Chromosome names must be specified the same way as they are described in the vcf header, 
e.g., chr1 or 1. One can specify multiple chromosomes separated by space. If no chromosome is specified, 
all chromosomes from the vcf file will be parsed.

If chromosome names in variant and alignment file are different in prefix chr (e.g. in "1" and "chr1")
CNVpytor will detect it and match the names using first imported name for both signals.

### Using SNP positions from variant file and counts from alignment file

In some cases it is useful to read positions of SNPs from vcf file and 
extract read counts from bam file. For example if we have two samples, normal tissue and 
cancer, normal can be used to call germline SNPs, while `samtools mpileup` procedure can be used 
to calculate read counts in cancer sample at the positions of SNPs. CNVpytor have
implemented this procedure. After reading SNP positions (previous step) type:

```
> cnvpytor -root file.pytor -pileup file.bam [-T ref.fa.gz]
```
where
```
* file.pytor -- specifies cnvpytor file,
* file.bam -- specifies bam/sam/cram file,
* -T ref.fa.gz -- specifies reference genome file (only for cram file without reference genome).
```

### Calculating BAF histograms

To apply 1000 genomes strict mask filter:
```
> cnvpytor -root file.pytor -mask_snps
```

To calculate baf histograms for maf, baf and likelihood function for baf use:
```
> cnvpytor -root file.pytor -baf 10000 100000 [-nomask]
```

### Predicting CNV regions using joint caller (prototype)


Finally we can call CNV regions using commands:
```
> cnvpytor -root file.pytor -call combined 10000 > calls.combined.10000.tsv
> cnvpytor -root file.pytor -call combined 100000 > calls.combined.100000.tsv
```

Result is stored in tab separated files with following columns:
```
* CNV type: "deletion", "duplication", or ”cnnloh", 
* CNV region (chr:start-end),
* CNV size,
* CNV level - read depth normalized to 1,
* e-val1 -- e-value (p-value multiplied by genome size divided by bin size) calculated using t-test statistics between RD statistics in the region and global,
* e-val2 -- e-value (p-value multiplied by genome size divided by bin size) from the probability of RD values within the region to be in the tails of a gaussian distribution of binned RD,
* e-val3 -- same as e-val1 but for the middle of CNV,
* e-val4 -- same as e-val2 but for the middle of CNV,
* q0 -- fraction of reads mapped with q0 quality in call segments,
* pN -- fraction of reference genome gaps (Ns) within call region,
* dNS -- fraction of reference genome gaps (Ns) within call segments,
* pP -- fraction of P bases (1kGP strict mask) within call segments,
* bin_size – size of bins
* n – number of bins within call segments,
* delta_BAF – change in BAF from ½,
* e-val1 -- e-value RD based (repeted, reserved for future upgrades),
* baf_eval – e-value BAF based,
* hets – number of HETs,
* homs – number of HOMs,
* cn_1 – most likely model copy number,
* genotype_1 - most likely model genotype,
* likelihood_1 – most likely model likelihood,
* cf_1 -- most likely model cell fraction,
* cn_2 – the second most likely model copy number,
* genotype_2 - the second most likely model genotype,
* likelihood_2 – the second most likely model likelihood,
* cf_2 -- the second most likely model cell fraction.
```

Using viewer mode we can filter calls based on five parameters: 
```
CNV size, e-val1, q0, pN and dG:
```
```
> cnvpytor -root file.pytor [file2.pytor ...] -view 10000
cnvpytor> set caller combined_mosaic # IMPORTANT, default caller is mean shift
cnvpytor> set Q0_range -1 0.5        # filter calls with more than half not uniquely mapped reads
cnvpytor> set p_range 0 0.0001       # filter non-confident calls 
cnvpytor> set p_N 0 0.5              # filter calls with more than 50% Ns in reference genome 
cnvpytor> set size_range 50000 inf   # filter calls smaller than 50kbp 
cnvpytor> set dG_range 100000 inf    # filter calls close to gaps in reference genome (<100kbp)
cnvpytor> print calls                # printing calls on screen (tsv format)
...
...
cnvpytor> set print_filename file.xlsx   # output filename (xlsx, tsv or vcf)
cnvpytor> set annotate               # turn on annotation (optional - takes a lot of time)
cnvpytor> print calls                # generate output file with filtered calls 
cnvpytor> quit
```

Upper bound for parameters `size_range` and `dG_range` can be _inf_ (infinity).

If there are multiple samples (pytor files) there will be an additional 
column with sample name in tsv format, multiple sheets in Excel format, and
multiple sample columns in vcf format.

Comparison between `CNVnator` and `CNVpytor` callers output format:

<img src="https://raw.githubusercontent.com/abyzovlab/CNVpytor/master/imgs/joint_caller_output.png">


## Genotyping genomic regions

Using -genotype option followed by bin_sizes you can enter region and genotype calculation
for each bin size will be performed:

```
> cnvpytor -root file.pytor -genotype 10000 100000
12:11396601-11436500
12:11396601-11436500    1.933261    1.937531
22:20999401-21300400
22:20999401-21300400    1.949186    1.957068
```

Genotyping with additional information:
```
> cnvpytor -root file.pytor -genotype 10000 -a [-rd_use_mask] [-nomask]
12:11396601-11436500
12:11396601-11436500    2.0152  1.629621e+04    9.670589e+08    0.0000  0.0000  4156900 1.0000  50      4       0.0000  1.000000e+00
```

Output columns are:
```
1. region,
1. cnv level -- mean RD normalized to mean autosomal RD level,
1. e_val_1 -- p value calculated using t-test statistics between RD statistics in the region and global,
1. e_val_2 -- p value from the probability of RD values within the region to be in the tails of a gaussian distribution of binned RD,
1. q0 – fraction of reads mapped with q0 quality within call region,
1. pN – fraction of reference genome gaps (Ns) within call region,
1. dG -- distance from closest large (>100bp) gap in reference genome,
1. proportion of bins used in RD calculation (with option _-rd_use_mask_ some bins can be filtered out),
1. Number of homozygous variants within region,
1. Number of heterozygous variants,
1. BAF level (difference from 0.5) for HETs estimated using maximum likelihood method,
1. p-value based on BAF signal.


Option _-rd_use_mask_ turns on P filtering (1000 Genome Project strict mask) for RD signal.

Option _-nomak_ turns off P filtering of SNPs (1000 Genome Project strict mask) for BAF signal.
```

**Example:**

Genotype all called CNVs:

```
> awk '{ print $2 }' calls.10000.tsv | cnvpytor -root file.pytor -genotype 10000 100000
```

