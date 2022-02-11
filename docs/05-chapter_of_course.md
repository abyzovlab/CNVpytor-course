
# Visualization



## Learning Objectives


This chapter will cover:  

- Visualization in command line mode
- Interactive visualization



### Plot from command line

Chromosome wide plots:
```
> cnvpytor -root file.pytor -plot [rd BIN_SIZE] [likelihood BIN_SIZE] [baf BIN_SIZE] [snp] [-o IMAGE_FILENAME]
```
where
* rd BIN_SIZE -- plots RD signal for all chromosomes
* likelihood BIN_SIZE -- plots baf likelihood for all chromosomes
* baf BIN_SIZE -- plots baf/maf/likelihood peak position for all chromosomes
* snp -- plots baf for each snp for all chromosomes
* -o IMAGE_FILENAME -- if specified, saves plot in file instead to show on the screen

Manhattan plot:
```
> cnvpytor -root file.pytor -plot manhattan BIN_SIZE [-chrom name1 ...] [-o IMAGE_FILENAME]
```

Circular plot:
```
> cnvpytor -root file.pytor -plot circular BIN_SIZE [-chrom name1 ...] [-o IMAGE_FILENAME]
```

Plot genomic regions:
```
> cnvpytor -root file.pytor -plot regions [reg1[,| ]...] BIN_SIZE [-panels [rd] [likelihood] [baf] [snp] ...] [-o IMAGE_FILENAME]
```
where
* reg1 -- comma or space separated regions in form CHR[:START-STOP], e.g. 1:1M-20M 2 3:200k-80000010
* if regions are comma separated they will be plotted in the same subplot
* space will split regions in different subplots
* -panels -- specify which panels to plot: rd likelihood baf snp
* -o IMAGE_FILENAME -- if specified, saves plot in file instead to show on the screen


### Plot from interactive mode

The best way to visualize CNVpytor results is interactive mode. Enter interactive mode by typing:
```
cnvpytor -root file.pytor -view BIN_SIZE
```

There is tab completion and help similar to man pages. Type double tab or help to start.

