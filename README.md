# Considerations for YaHS Hi-C Scaffolder Parameters & Input Data Preparation
Hi-C scaffolding is a versatile method to order and bridge the contigs into chromosome-scale genomic scaffolds. One of the standard computational tools for Hi-C scaffolding is ["YaHS" (Yet another Hic Scaffolding tool)](https://github.com/c-zhou/yahs), which is employed in the genome assembly pipeline provided by the Vertebrate Genome Project. \
Here, we outline several insights for running this tool based on our experience with shark genome assembly ([Niwa et al., <I>PNAS</I>, 2025](https://www.pnas.org/doi/10.1073/pnas.2513676122)).

## Input preparation with HiC-Pro
YaHS requires a BAM/BED/BIN/PA5 file as input, which records how Hi-C reads were mapped onto contigs. Mapping of Hi-C reads requires a specific strategy distinct from usual genomic read mapping due to their chimeric nature and ligation junctions. The original YaHS GitHub repository recommends the Arima Genomics' mapping pipeline, the Omni-C's mappping pipeline and HiC-Pro. We have utilized HiC-Pro for the read mapping because of its all-in-one feature, which considers MAPQ, multi-mapped reads, PCR duplicates, read-name sorting and even more it outputs a Hi-C read-pair statistics. \
However, YaHS GitHub repository does not specify which HiC-Pro output file should be used as its input. When comparing major HiC-Pro (bowtie mapping step) output bam files: \
 ```outdir/bowtie_results/bwt2/xx/xxx.bwt2pairs.bam```\
 ```outdir/bowtie_results/bwt2/xx/xxx r1 or r2.bwt2merged.bam```\
, we found that xxx.bwt2pairs.bam only contains the reads above the MAPQ threshold set in the config file. Therefore, we always use xxx.bwt2pairs.bam directly as the YaHS input.

## MAPQ threshold
The MAPQ threshold is one of the most important factors, defining how many reads will contribute to the contact frequency calculation. In a highly repetitive genome, many reads were classified as uniquely-mapped reads with low MAPQ or multi-mapped reads (MAPQ=0). To incorporate unique read pair information as much as possible, we tested several MAPQ thresholds defined in the HiC-Pro config file and YaHS -q option, and then set it to MAPQ=1.

## Assembly error check
By default, YaHS performs assembly error checks at both contig and scaffold levels based on contradictions in contact frequency. These error check steps can be suppressed by two different options: ```--no-contig-ec --no-scaffold-ec```, corresponding to the ```-r``` option in 3d-dna.\
In some cases, these steps may fragment contigs and scaffolds excessively. In one of our experiences, either options (both check steps OFF) increased the continuity of the assembly (e.g. proportion of sequences > 10 Mb), but this was not in the other case, which resulted in increased misassembly. We recommend to try the default setting first (both check steps ON), and then if too many fragmented contigs are left, try other settings.

## Contact map resolution
Hi-C scaffolding involves a series of contig/scaffold assignment steps where the resolution of contact matrices gradually decreases. When smaller contigs are included, the starting resolution should be higher, corresponding to the smallest contig size. The default resolution starts from 10 kb. When we tried a smaller value, we specified all the resolution values as follows:\
```-r 2000,5000,10000,20000,50000,100000,200000,500000,1000000,2000000,5000000,10000000,20000000,50000000,100000000,200000000,500000000```\
Be aware that this will consume more memory.

## Memory check
```--no-mem-check``` option could be critical in some cases. By default, YaHS defines an upper limit for memory occupancy, which is much lower than the actually available RAM size. Scaffolding processes at higher-resolutions (i.e. smaller window sizes) will be skipped when the memory usage exceeds the limit. We missed this error at the first attempt, which prevented small contigs from being assigned into scaffolds. \
This error is reported in the standard output as: \
```[I::run_scaffolding] No enough memory. Try lower resolutions... End of scaffolding round.```\
at each round of scaffolding without any critical problem. Pay attention to this  error message, and add the option ```--no-mem-check``` if this message appears.


## References
[Chenxi Zhou, Shane A. McCarthy, Richard Durbin. YaHS: yet another Hi-C scaffolding tool. Bioinformatics, 39(1), btac808.](https://github.com/c-zhou/yahs)

[Servant N., Varoquaux N., Lajoie BR., Viara E., Chen CJ., Vert JP., Dekker J., Heard E., Barillot E. HiC-Pro: An optimized and flexible pipeline for Hi-C processing. Genome Biology 2015, 16:259](https://github.com/nservant/HiC-Pro)

[VGP assemblies and genome assembly pipeline: Rhie et al., Towards complete and error-free genome assemblies of all vertebrate species, Nature 2021.](https://galaxyproject.org/projects/vgp/workflows)
