# Viewing Illumina reads in IGV; finding SNPs

# Goals

In the last lab period we learned about Illumina reads and fastq files. We performed quality trimming and then mapped those reads to the _B. rapa_ reference genome.

Today we will pick up where we left off. Our goals are to:

1. Learn about sequence alignment/mapping [(SAM/BAM) files](http://www.htslib.org/doc/sam.html). [More info on SAM](http://genome.sph.umich.edu/wiki/SAM)
2. Examine mapped reads in [IGV–the integrative genomics viewer](https://www.broadinstitute.org/igv/). This will allow us to see how reads are placed on the genome during the mapping process.
3. Find polymorphic positions in our sequencing data.

# Preliminaries

_OK to cut and paste today’s code_

## Data files

For better viewing and SNP calling I compiled all of the IMB211 internode files and all of the R500 internode files and ran STAR on those. Then to keep the download to a somewhat reasonable size I subset the bam file to chromosome A03. `cd` into your `Assignment_6/output` directory and download the files as listed below:

```
wget https://cluster.hpcc.ucr.edu/~dkoenig/COURSE_DATA/STAR_out-IMB211_INTERNODE_A03.tar.gz
tar -xvzf STAR_out-IMB211_INTERNODE_A03.tar.gz

wget https://cluster.hpcc.ucr.edu/~dkoenig/COURSE_DATA/STAR_out-R500_INTERNODE_A03.tar.gz
tar -xvzf STAR_out-R500_INTERNODE_A03.tar.gz
```

## Examine STAR output

`cd` into the `STAR_out-IMB211_INTERNODE_A03/` output directory that you downloaded above

You will see several files there. Some of these are listed below

- `IMB211_INTERNODE_Aligned_A03.bam` – A [bam](https://samtools.github.io/hts-specs/SAMv1.pdf) file for reads that were successfully mapped **(the “A03” is there to designate that I filtered to only retained chromosome A03)**
- `IMB211_INTERNODE_SJ.out.tab` A tab-delimited file giving exon splice junctions
- `IMB211_INTERNODE_Log.final.out` Summarizes the mapping

**Exercise 8**: Take a look at the `IMB211_INTERNODE_Log.final.out` file.
_You only need to answer for the IMB211 alignment_
**a**. What percentage of reads map uniquely to the reference?
**b**. What reasons are given for the reads that don’t map uniquely (list three)
**c**. One category is “unmapped other”. Think about the process of read mapping and that the reference genome is from a different cultivar (B3) than the cultivar we sequenced (IMB211). Give 2 reasons why reads might not map to the reference.

## Look at a bam file

Bam files contain the information about where each read maps. They are in a binary, compressed format so we can not use `less` on its own. Thankfully there is a collection of programs called [`samtools`](http://www.htslib.org/) that allow us to view and manipulate these files.

Let’s take a look at `IMB211_INTERNODE_Aligned_A03.bam`. For this we use the `samtools view` command

```
module load samtools
samtools view -h IMB211_INTERNODE_Aligned_A03.bam | less
```

The file is structured as follows

|Field|Value|
|---|---|
|01|Read Name (just like in the fastq)|
|02|Code providing info about the alignment (bitwise FLAG)|
|03|Template Name (Chromosome in this case)|
|04|Position on the template where the read starts|
|05|Phred based mapping quality for the read|
|06|CIGAR string providing information about the mapping|
|07 - 09|These give information about the “mate pair” if paired end sequencing was done|
|10|Sequence|
|11|Phred+33 quality of sequence|
|Additional fields|Varies; see [SAM page](http://www.htslib.org/doc/sam.html) for more info|

`samtools` has many additional functions. These include

`samtools merge` – merge BAM files
`samtools sort` – sort BAM files; required by many downstream programs
`samtools index` – create an index for quick access; required by many downstream programs
`samtools idxstats` – summarize reads mapping to each reference sequence
`samtools mpileup` – count the number of matches and mismatches at each position
`samtools addreplacerg` – add/replace read group identifiers in a BAM file

And more

## Look at a bam file with IGV

While `samtools view` is nice, it would be nicer to actually see our reads in context. We can do this with [IGV, the Integrative Genome Viewer](https://www.broadinstitute.org/igv/).

**You will have to do this part of the lab on your own computer. Download igv onto your own computer and make sure you can get it to run**

To use IGV, first create an index of the bam file

```
samtools index IMB211_INTERNODE_Aligned_A03.bam
```

**Do the above step for both the IMB211 and the R500 files**

### Prepare the genome reference files for IGV to use

**Use rsync to copy the bam files and there indexes onto your computer.**

By default IGV starts with the human genome. It has a number of built-in genomes, but does not include _B. rapa_. We must upload it ourselves.

#### load the genome fasta
**Use rsync to copy Brapa_gene_v1.5.gff and BrapaV1.5_chrom_only.fa onto your computer. These files are from Assignment_5**

Select `BrapaV1.5_chrom_only.fa` located in input/Brapa_reference of your Assignment 6 repository

In IGV, click on `Genomes > Load Genome from File` (it may take a few seconds for the file select window to open)

#### load the gff

In IGV, click on `File > Load from File`

Select `Brapa_gene_v1.5.gff` located in input/Brapa_reference of your Assignment 6 repository

### Load some tracks

Now to load our mapped reads:

Click on `File > Load From File` ; then select the `IMB211_INTERNODE_Aligned_A03.bam` file. Repeat this for the `R500_INTERNODE_Aligned_A03.bam` file.

### Take a look

Click on the “ALL” pull-down menu and select chromosome A03. Then zoom in until you can see the reads.

- Grey vertical bars are a histogram of coverage
- Grey horizontal bars represent reads.
    - Thin lines connect read segments that were split across introns.
    - Colored vertical lines show places where a read differs from the reference sequence
- In the lower panel, the blue blocks and lines show the **reference annotation**. Blocks are exons and lines are introns. The arrowheads show the direction of transcription.
- The orange lines show junctions inferred by STAR from our reads

**Exercise 9**:
Can you distinguish likely SNPs from sequencing/alignment errors? How?

**Exercise 10**: View each gene listed below in IGV. (You can type the gene ID into the IGV search bar). For each gene, does the computationaly predicted annotation (blue bars) appear to be correct or incorrect given our RNA-seq data? If incorrect, describe what is wrong.
**a** Bra001706
**b** Bra001700
**c** Bra001702
**d** Bra012917

**Exercise 11**: Comment on the trust-worthiness of computational annotation. (To be fair, this annotation was done in 2009 or 2010; tools may have gotten better since then).

## Calling SNPs
For the remainder of the lab you will again work on the cluster.

The goal of this section is to find polymorphisms between IMB211 and R500. There are many tools available. We will use [FreeBayes](https://github.com/ekg/freebayes)
(For more info click on the link above. Once you are on the FreeBayes page, scroll down to the README for more info on FreeBayes)

Make a new directory for this analysis inside the `Assignment_6/output` directory

```
mkdir SNP_analysis
cd SNP_analysis
```

In the library preparation step there is a PCR amplification. This can cause duplication of DNA fragments. We want to remove these duplicate reads because they can skew our SNP analysis (essentially they represent pseudo-replication, giving us artificially high sample numbers). We will use `samtools rmdup` to remove the duplicate reads

```
samtools rmdup -s ../STAR_out-IMB211_INTERNODE_A03/IMB211_INTERNODE_Aligned_A03.bam IMB211_rmdup.bam

samtools rmdup -s ../STAR_out-R500_INTERNODE_A03/R500_INTERNODE_Aligned_A03.bam R500_rmdup.bam
```

Next create an index for each of the new files

```
samtools index IMB211_rmdup.bam
samtools index R500_rmdup.bam
```

Now we use `freebayes` to look for SNPs. `freebayes` calculates the number of reference and alternate alleles at each position in the genome and genotype likelihoods.

We first call `samtools addreplacerg` to add Read Groups to our bam files, enabling us to call SNPs separately for the two genotypes. The output from `samtools addreplacerg` is a bam file with the Read Groups, we will then merge the two bam files, index the merged file and use it as input for `freebayes`.

```
module load freebayes

samtools addreplacerg -o rg_IMB211_rmdup.bam -r "ID:IMB211" -r "SM:IMB211" IMB211_rmdup.bam

samtools addreplacerg -o rg_R500_rmdup.bam -r "ID:R500" -r "SM:R500" R500_rmdup.bam

samtools merge combined_IMB211_R500.bam rg_IMB211_rmdup.bam rg_R500_rmdup.bam

samtools index combined_IMB211_R500.bam

freebayes --fasta-reference ../../input/Brapa_reference/BrapaV1.5_chrom_only.fa combined_IMB211_R500.bam  > IMB211_R500.vcf
```

freebayes will take ~ 3 minutes to run.

## Two other popular SNP callers:

- [bcftools mpileup](https://samtools.github.io/bcftools/howtos/variant-calling.html).
- GATK offers an alternative and popular [genotyping pipeline and caller](https://www.broadinstitute.org/gatk/)
