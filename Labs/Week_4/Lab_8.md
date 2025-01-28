# Multiple sequence alignment and phylogenetic tree construction

## Orientation
We would like to better examine the evolutionary origin of the virus causing the respiratory disease outbreak. In the last lab we used tools in R to narrow down our BLAST results to a manageable number of sequences and to create a new FASTA file that had just the sequences that we wanted.

Today we will do a multiple sequence alignment on those sequences and then build a phylogenetic tree.

Basic background on multiple sequence alignment and phylogenetic trees can be found in [Chapter 9 of Bioinformatics for Beginners](https://www.sciencedirect.com/science/article/pii/B9780124104716000098). You should be able to download this file on campus, but if you have any problems let me know.

## Download the sequences
Just to make sure we are all starting with the same sequences, download my output from Assignment 3, part 1 and use these.

Download the sequences into your output folder with the code below. Make sure that your working directory is scripts, or, run this from within your assignment_3_template_2 script:
```
download.file(url="https://bis180ldata.s3.amazonaws.com/downloads/Assignment3/selected_viral_seqs.fa",
              destfile = "../output/selected_viral_seqs_195v2.fa") # use this to put the file in a different directory
```
## Multiple sequence alignment
There are many sequence alignment algorithms and programs. Here we will use MAFFT because it is reasonably quick and does a reasonably good job. Obtaining a good alignment is as much of an art as a science. I tried a few settings and found that we had to reduce the gap opening penalty to get a good alignment.

It takes about an hour to align (remember this is a big file, 196 sequences up to 24,000 bases in length). I give the command below, but YOU DO NOT NEED TO RUN THE COMMAND BELOW. IT WILL TAKE AN HOUR. DOWNLOAD MY RESULTS INSTEAD
```
#time mafft --maxiterate 100 --thread 3 --reorder --op 0.5 selected_viral_seqs.fa  > mafft_maxiter100_195_op.5.fa
```
```
download.file(url="https://cluster.hpcc.ucr.edu/~dkoenig/COURSE_DATA/mafft_maxiter100_195_op.5.fa",
              destfile = "../output/mafft_maxiter100_195v2_op.5.fa") # use this to put the file in a different directory
```

## View alignment
It is always important to view your alignment to see if the aligner seems to have done a good job. There are many alignment visualization packages and websites. We will use an online tool called [Wasabi](http://was.bi).

Go to Wasabi now to view the MAFFT output file. You can do this from your instance or from your own computer. You do not need to create a Wasabi account.

At Wasabi, click on the left most icon (document icon) to upload (import) a file. Select and upload the `mafft_maxiter100_195v2_op.5.fa file`

**STOP and THINK**: What is each row? What is each column? What are the colors?

Click on the “-“ icon to zoom out; I recommend zooming so far out that you just see colors, not letters. Scroll through the alignment. You will see that there are some regions that are aligned far better than others.

An example of a poorly aligned region:
![](/GNBT120/assets/images/Was.bi.bad.png)

An example of a well-aligned region:
![](/GNBT120/assets/images/Was.bi.good.png)

If you look at the first 1,000bp, you will see that there are some sequences that have gaps in otherwise very well conserved regions. A similar problem exists at the end of the alignment. Probably these are incomplete sequences, so we will plan to trim them. We will trim both of these regions, later in R.

There are also many gap-rich regions, where the alignment did not work well. We will also plan to trim these in R. You can get an idea for what removing gappy sites looks like by clicking on the gear icon in Wasabi and choosing “Hide gaps” and choosing a 75% threshold. This makes the missing data at the beginning and end of the alignment easier to see.

In theory we could do the trimming and export from Wasabi itself, but in practice that does not work well with this many sequences of this length, so we will turn to R.

**Exercise 15:**

If you turned on a gap threshold in the viewer, undo it by clicking on the undo button:

For each of the the following regions classify them as well- or poorly aligned, or uncertain/intermediate and explain your reasoning:

* 39.1K to 39.4K
* 31.6K to 31.9K
* 24.7K to 24.9K
* 14.5K to 14.7K

### Some additional notes on alignments:
We are really doing a pretty quick and dirty job of editing of the alignment. If we were going to do this in more detail, we might use a program like [gblocks](http://phylogeny.lirmm.fr/phylo_cgi/one_task.cgi?task_type=gblocks) to more systematically prune poorly aligned regions (although for the most part I think the 75% gap cutoff results in a reasonable alignment). An alternative approach would be to focus on a few genes that are conserved among these viruses, and then use an amino-acid or codon guided alignment using software like [PRANK](https://ariloytynoja.github.io/prank-msa/).

## Subset sequences
Now that we have found a region we want to use for tree construction, let’s use R to subset those sequences.

**RECOMMEND CUT and PASTE for this lab.**
```
library(tidyverse)
library(Biostrings)
```

Load the alignment using Biostrings, and set up our file names.

```
inpath <- "../output/mafft_maxiter100_195v2_op.5.fa"
outpath <- "../output/mafft_maxiter100_195v2_op.5_trimmed_75pct.fa"
alignment <- readDNAMultipleAlignment(inpath)
alignment
```

Subset to trim off the ends with incomplete sequence

```
alignment <- DNAMultipleAlignment(alignment,start=1000,end=48449)
```

Mask sites that have more than 25% gaps

```
alignment <- maskGaps(alignment, min.fraction=0.25, min.block.width=1)
maskedratio(alignment) #what proportion got masked? (rows and columns)
## [1] 0.0000000 0.5212013
```
Change the alignment into a stringset so that we can more easily manipulate the names

```
alignment <- alignment %>% as("DNAStringSet")
```

For tree viewing we need to remove the “ “ (spaces) so that the names do not get truncated. We will also select a subset of fields. Many of these commands will be covered in a subsequent lab. OK to CUT and PASTE

```
newnames <- names(alignment) %>%
  tibble(name=.) %>%
  mutate(name=str_replace_all(name," ","_")) %>% #replace " " with "_" because some programs truncate name at " "
  separate(name,
           into=c("acc", "isolate", "complete", "name", "country", "host"),
           sep="_?\\|_?") %>%
  mutate(name=str_replace(name, "Middle_East_respiratory_syndrome-related","MERS"),   # abbreviate
         name=str_replace(name, "Severe_acute_respiratory_syndrome-related", "SARS"), # abbreviate
         newname=paste(acc,name,country,host,sep="|")) %>% # select columns for newname
  pull(newname) #return newname
```

```
## Warning: Expected 6 pieces. Missing pieces filled with `NA` in 1 rows [1].
```

```
head(newnames)
```

```
## [1] "Seq_H|NA|NA|NA"
## [2] "MG772933|SARS_coronavirus|China|Rhinolophus_sinicus"
## [3] "MG772934|SARS_coronavirus|China|Rhinolophus_sinicus"
## [4] "KU182964|Bat_coronavirus|China|Rhinolophus_ferrumequinum"
## [5] "KY938558|Bat_coronavirus|South_Korea|Rhinolophus_ferrumequinum"
## [6] "KY770860|Bat_coronavirus|China|Rhinolophus_ferrumequinum"
```

Write out the trimmed file

```
names(alignment) <- newnames
alignment %>% writeXStringSet(outpath)
```

## Build a tree
Like everything else in this lab, there are a myriad of phylogenetic tree reconstruction algorithms and programs. There is a consensus that you want to use a Maximum Likelihood or Bayesian approach, not Maximum Parsimony and not Neighbor Joining.

We will use [FastTree](http://www.microbesonline.org/fasttree/) because it offers a good balance of accuracy and speed. Again this can be more of an art.

1. Use the shell to log on to the server and start an interactive job
2. At the COMMAND LINE and in your output directory, use:

```
module load fasttree
FastTree -nt -gtr -gamma -out mafft_maxiter100_195v2_op.5_trimmed.fasttree.tre mafft_maxiter100_195v2_op.5_trimmed_75pct.fa
```
(Takes less than 2 minutes)

Where

* `-nt` specifies that this is a nucleotide alignment
* `-gtr` specifies that we want to use the General Time Reversible model for nucleotide substitution
* `-gamma` improves the maximum likelihood estimates
* `-out` specifies what we want the output file to be named
* the final argument is the input file.

## Examine the tree
Again, there are many tree viewing programs. We will use the web-based viewer [iTOL](https://itol.embl.de/).

Upload the `mafft_maxiter100_195v2_op.5_trimmed.fasttree.tre` file to iTol.

For improved viewing, do the following:

* Basic tab
    * Mode: “Rectangular”
    * Label options, Position: “At tips”
* Advanced tab
    * Bootstraps / metadata: “Display”
    * Legend: On
(Now the circles at each internal node indicate support for the node: 1 is best, 0 is worst).

You can now use the + and - buttons to make the tree larger and smaller. Clicking on the tree and scrolling allows you to move around.

## Exercise 16:

**A:** What is the sister taxon to Seq_H? What is the host for the virus in this group (provide the Latin and common names)
hint: if you are having trouble finding Seq H in the tree, search for it using the (Aa) magnifying glass

**B:** Consider Seq_H plus its sister taxa as defining one taxonomic group of three species. Look at the sister taxa to this group. What is a general description for the viruses in this group? List at least 3 different hosts found in this group. Give Latin and common names (if there is a common name).

**C:** Now consider Seq_H plus all of the other taxa used in question B as one taxonomic group. The sister to this group has three members. What are the hosts of this group.

**D:** Given your above findings, what do you think the host of the most recent common ancestor to the viruses in parts A and B was? (You can answer giving the common name for the general type of organism, e.g. rat, mouse, bat, ape, etc. you do not need to give a precise species).

**E:** Do you think that Seq_H evolved from a virus with a human host? Why or why not? If not, what did it evolve from?

Congratulations! You have identified the likely evolutionary origin of the COVID-19 virus.

## Evolution of the Spike (S) protein
The SARS Covid-19 S or Spike protein is a glycoprotein found on the surface of the virus and responsible for binding to host cells and subsequent membrane fusion and entry ([wikipedia spike protein](https://en.wikipedia.org/wiki/Coronavirus_spike_protein)). It is also highly immunogenic and the target of most Covid-19 vaccines.

Given its importance to viral function it might be interesting to examine the evolutionary changes that occurred between the bat and human hosts.

We will do a simple analysis that compares the rate of nucleotide substitutions that cause amino acid changes (non-synonymous changes) to those that do not (synonymous changes). The ratio of these rates (dN/dS or Ka/Ks) is a measure of the evolutionary pressure acting on the gene ([wikipedia KaKs](https://en.wikipedia.org/wiki/Ka/Ks_ratio)).

Ka/Ks ratios of 1 indicate neutral selection (amino acid changes are neither selected for or against). Ratios of less than one indicate purifying selection (most amino acid changes are selected against). Ratios greater than one indicate positive selection (amino acid changes are favored). In the simple test we are going to do here it is important to remember that we will get an estimate of the average Ka/Ks across the whole protein.

**Exercise 17:**

Make a hypothesis about what you expect the Ka/Ks ratio to be when comparing the S gene from SARS2 with a human versus a bat host. Explain you reasoning. There is no wrong answer here so long as you explain your reasoning.

*End of Exercise 17*

### Gather the sequences
To perform this analysis we need to do a pairwise sequence alignment of the S genes from Bat and Human infecting viruses.

First, let’s load the unaligned sequences:
```
selected.seqs <- readDNAStringSet("../output/selected_viral_seqs.fa")
```

**Exercise 18:**

1. Retrieve Seq_H from one of the `original data sets (before alignment)`.
2. Use the `subseq()` function (look it up) to keep the bases from position 21531 to 25352 of Seq_H. This is the S gene. Name the new object `Seq_H_Spike`
3. Retrieve the sequence of the sister sequence to Seq_H (use the one ending in “34”) and place it in an object `Bat_Seq`. Hint: Because we reformatted the names of the sequences used to build the tree, it might be tricky to use the full name to retrieve the Bat sequence. Instead you can use grep, as below (Change the XXXXX to match the identifier of your sequence).
```
Bat_Seq <- selected.seqs[grep("XXXXXXX", names(selected.seqs))]
```

### Align the sequences
To find the S gene from the Bat sequences we will perform a pairwise sequence alignment. We can use a tool from the Biostrings package for this.
```
spike_aligned <- pairwiseAlignment(Seq_H_Spike,
                                   Bat_Seq,
                                   type="overlap",
                                   substitutionMatrix=
                                     nucleotideSubstitutionMatrix(
                                       baseOnly = TRUE, match=2, mismatch = -1),
                                   gapOpening=10,
                                   gapExtension=4,
                                   )
spike_aligned
```
```
## Overlap PairwiseAlignmentsSingleSubject (1 of 1)
## pattern:     [1] ATGTTTGTTTTTCTTGTTTTATTGCCACTAGTCTCTAGTCAG...TCTGAGCCAGTGCTCAAAGGAGTCAAATTACATTACACATAA
## subject: [21486] TTGTTTTTCTTGTTTCTTCAGTTCGCCTTAGTAAACTCCCAG...TCTGAGCCTGTGCTCAAAGGAGTCAAATTACATTACACGTAA
## score: 4420
```

Convert it to a more useful stringset object and write it out

```
spike_aligned_stringset <- c(alignedPattern(spike_aligned), alignedSubject(spike_aligned))
writeXStringSet(spike_aligned_stringset,"../output/spike_aligned.fa")
```

**Exercise 19**

View the alignment in Wasabi. Any concerns? It may be helpful to display the alignment as codons and/or protein (click on the gears and select `translate`). Note: if you view as protein, you will have to reload the file to see nucleotides again.

*End of Exercise 19*

### Calculate KaKs
Regardless of any concerns about the alignment, let’s go ahead and calculate KaKs

The tool that we will use for KaKs calculation requires a different object type than a DNAStringSet. The easiest thing to do is just to read the alignment back in.
```
spike_aligned_seqinr <- seqinr::read.alignment("../output/spike_aligned.fa", format = "fasta")
spike_kaks <- seqinr::kaks(spike_aligned_seqinr)
spike_kaks
```

```
## $ka
##              Seq_H
## MG772934 0.1190342
##
## $ks
##             Seq_H
## MG772934 1.154126
##
## $vka
##                 Seq_H
## MG772934 0.0001063036
##
## $vks
##                Seq_H
## MG772934 0.009775355
```

** Exercise 20 **

Calculate Ka/Ks, using the `spike_kaks` object. Do this without hand-typing the numbers. Then, interpret the result. Assuming our alignment was okay, what does this tell you about how this protein is evolving? Does this match your hypothesis? If the results do not match your hypothesis, could your hypothesis still be correct, in a modified form?
