# Eukaryotic Genome assembly & quality control (Nanopore reads)

This contains a README of exemplar core commands for Eukaryotic genome assembly using Oxford Nanopore reads.

## Table of contents 
**0). [Software](#software)**\
**1). [Read quality control](#RAQC)**\
&nbsp;&nbsp;&nbsp;&nbsp;**a). [Dorado basecalling](#dorado)**\
&nbsp;&nbsp;&nbsp;&nbsp;**b). [Porechop adapter removal](#porechop)**\
&nbsp;&nbsp;&nbsp;&nbsp;**c). [Filtlong read subsetting](#filtlong)**\
&nbsp;&nbsp;&nbsp;&nbsp;**d). [Nanoplot QC](#nanoplot)**\
**2). [Genome assembly](#GAQC)**\
&nbsp;&nbsp;&nbsp;&nbsp;**a). [Flye assembly](#flye)**\
&nbsp;&nbsp;&nbsp;&nbsp;**b). [Miniasm assembly](#miniasm)**\
&nbsp;&nbsp;&nbsp;&nbsp;**c). [WTDBG2 assembly](#wtdbg2)**\
&nbsp;&nbsp;&nbsp;&nbsp;**d). [Raven assembly](#raven)**\
&nbsp;&nbsp;&nbsp;&nbsp;**e). [Shasta assembly](#shasta)**\
**3). [Assembly QC](#AQC)**\
&nbsp;&nbsp;&nbsp;&nbsp;**a). [BUSCO](#busco)**\
&nbsp;&nbsp;&nbsp;&nbsp;**b). [Assembly statistics](#asm_stats)**

## Software <a name="software"></a>
PENDING

## Read quality control <a name="RAQC"></a>
### Dorado basecalling <a name="dorado"></a>
```
# Basecall autoselecting a super-accurate model
dorado basecaller sup \
  PATH/TO/pod5_pass/ \
  --emit-fastq > ${sample_id}.sup.fastq
```
### Porechop adapter removal <a name="porechop"></a>
```
# Trim adapter and discarding reads with middle adapters (not doing this results in identical read name errors downstream)
porechop \
  --discard_middle -i ${sample_id}.sup.fastq \
  -o ${sample_id}.trimmed.sup.fastq
```
### Filtlong read subsetting<a name="filtlong"></a>
```
# OPTIONAL - subset down to 60% of reads keeping only the higher quality reads
filtlong \
  --min_length 1000 --keep_percent 60 ${sample_id}.trimmed.sup.fastq | gzip > ${sample_id}.subset06.trimmed.sup.fastq.gz
```
### Nanoplot QC <a name="nanoplot"></a>
```
# Run Nanoplot QC
  NanoPlot -t 2 \
    --fastq ${sample_id}.subset06.trimmed.sup.fastq.gz  \
    --plots hex dot kde \
    -o ${sample_id}.subset06.trimmed.sup.nanoplot.out
```
## Genome assembly & quality control <a name="GAQC"></a>
### Flye assembly <a name="flye"></a>
```
# Assemble using Flye with scaffolding and 2 rounds of polishing
flye \
  --nano-hq ${sample_id}.subset06.trimmed.sup.fastq.gz \
  -g 380m \
  -t 32 \
  --iterations 2 \
  --scaffold \
  -o ./${sample_id}_sup_nanohq/
```
### Miniasm assembly <a name="miniasm"></a>
```
# Assemble using Miniasm
minimap2 \
  -x ava-ont \
  -t14 ${sample_id}.subset06.trimmed.sup.fastq.gz | gzip -1 > ${sample_id}.reads.paf.gz

miniasm -f ${sample_id}.subset06.trimmed.sup.fastq.gz ${sample_id}.reads.paf.gz > ${sample_id}.reads.gfa
```
### WTDBG2 assembly <a name="wtdbg2"></a>
```
# Assemble using WTDBG2 and make consensus
wtdbg2 \
  -x ont \
  -g 380m \
  -i ${sample_id}.subset06.trimmed.sup.fastq.gz \
  -t 12 \
  -fo ${sample_id}.dbg

wtpoa-cns \
  -t 12 \
  -i ${sample_id}.dbg.ctg.lay.gz \
  -fo ${sample_id}.dbg.raw.fa
```
### Raven assembly <a name="raven"></a>
```
# Create and enter output directory
mkdir raven_${sample_id} ; cd raven_${sample_id}

# Run Raven
raven \
  -t 16 \
  ../${sample_id}.subset06.trimmed.sup.fastq.gz > ${sample_id}.asm.fa
```
### Shasta assembly <a name="shasta"></a>
```
shasta \
  --config Nanopore-R10-Fast-Nov2022 \
  --input ${sample_id}.subset06.trimmed.sup.fastq.gz \
  --assemblyDirectory ${sample_id}.trimmed.sup.shasta \
  --threads 24
```

## Assembly QC <a name="AQC"></a>
### BUSCO quality control <a name="busco"></a>
```
# Run BUSCO
busco -i ${sample_id}.assembly.fa -m genome -l diptera_odb10 -c 12
```
### Assembly statistics <a name="asm_stats"></a>
```
assembly-stats ${sample_id}.assembly.fa
```
### RepeatMasking <a name="repm"></a>
```
# Build a database for your assembly
BuildDatabase -name assembly_1a ${sample_id}.assembly.fa

# Run RepeatModeller
RepeatModeler -database assembly_1a -threads 24 -LTRStruct

# Run RepeatMasker
RepeatMasker -pa 12 -lib RM_*/consensi.fa.classified -gff -dir RM_output/ ${sample_id}.assembly.fa
```








