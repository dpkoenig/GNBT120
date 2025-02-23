# Illumina continued: analyze a VCF file in R

In the first part of today’s lab you used `FreeBayes` to calculate SNPs between IMB211 and the reference sequence and between R500 and the reference sequence. `FreeBayes` returned a [.vcf (variant call format)](https://samtools.github.io/hts-specs/VCFv4.2.pdf) file.

The .vcf file contains information on polymorphisms. It is a tab-delimited file that is easily imported into R. We want to filter the SNPs to remove low quality SNPs. We also could use this for downstream analyses, i.e. use these as genetic markers for mapping or QTL mapping, look for genes with coding changes, etc., although we will not have time to explore those types of analyses in this class.

If you need to download the file you can do so [here](https://cluster.hpcc.ucr.edu/~dkoenig/COURSE_DATA/IMB211_R500.vcf.gz). Place it in your `output/SNP_analysis/` directory.


## Import VCF file and reformat

```
library(tidyverse)
vcf.data <- read_tsv("../output/SNP_analysis/IMB211_R500.vcf",na = c("","NA","."),comment="#",col_names = FALSE)
head(vcf.data)
```

There is no column heading in what we imported. There is a column heading in the .vcf file but it is not loaded into R because its first character is “#”. Here we get it by using a `system()` call. `system()` causes R to issue a command to the Linux shell. The column headings in the .vcf start with “#CHROM” so we use Linux’s version of `grep` to search for that line. The argument `intern = TRUE` tells R that we want to capture the output of the command.

```
vcf.header <- system("grep '#C' ../output/SNP_analysis/IMB211_R500.vcf",intern = TRUE) #might not work on Windows
vcf.header
vcf.header <- vcf.header %>% str_replace("#","") #get rid of the pound sign
```

The header is tab delimited, so we split on tabs and use those as column names

```
vcf.header <- vcf.header %>% str_split(pattern = "\t")
colnames(vcf.data) <- vcf.header[[1]] #we need the [[1]] because str_split returns a list and we want the first element
head(vcf.data)
```

Now with the headers attached the file makes a bit more sense. Detailed information [is here](https://samtools.github.io/hts-specs/VCFv4.2.pdf), but briefly:

- CHROM and POS should be obvious
- ID is not used here
- REF is the reference sequence (From the fasta file)
- ALT is the allele found in our sample(s)
- QUAL is the PHRED scaled probability that the locus is actually homozygous for the _reference_ allele, or 1 - P(locus does have the alternate allele). So QUAL of 20 means that there is a 0.01 probability that the locus is actually not polymorphic (i.e. there is a 0.01 probability that a SNP has been called incorrectly). _Note that the definition of this field varies depending on which SNP calling program you use._
- FILTER is not used here. There are programs that will place flags in this field to indicate that a SNP should be filtered out.
- INFO lots of INFO. We can learn about these from the beginning of the vcf file (See below)
- FORMAT tells us the identity of the numbers in the next two fields (IMB211 & R500)
    - GT: The most probable genotype. 1/1 is homozygous alternate; 0/0 is homozygous reference; 0/1 is heterozygous.
    - DP: Read Depth
    - AD: Allele Depth. The counts of each of the observed alleles.
    - RO: Reference allele observation count
    - QR: Sum of quality of the reference observations
    - AO: Alternate allele observation count
    - QA: Sum of quality of the alternate observations
    - GL: Genotype Likelihood, log10-scaled likelihoods of the data given the called genotype for each possible genotype generated from the reference and alternate alleles given the sample ploidy

This information is at the beginning of the .vcf file which we could look at with `less` at the Linux command line, or we can look at it in R using system calls. (Might not work on Windows)

```
system("grep '##INFO' ../output/SNP_analysis/IMB211_R500.vcf")

system("grep '##FORMAT' ../output/SNP_analysis/IMB211_R500.vcf")
```

It turns out that FreeBayes called insertion/deletions as well as SNPs. To keep things simple, we will filter to retain only snps:

```
vcf.data <- vcf.data %>%
  filter(str_detect(INFO, "TYPE=snp"))
```

To be able to use the SNP data in R we need to break up the data in the final two columns. We use the command `separate` for this. `separate` splits a character column based on every occurrence of a character or characters. You can read more about separate in the [R for data science book](http://r4ds.had.co.nz/tidy-data.html#separating-and-uniting)

```
vcf.data <- vcf.data %>% separate(IMB211,
                                  into = paste("IMB211",c("gt","tot.depth","allele.depth", "ref.depth","ref.qual","alt.depth","alt.qual","gt.lik"),sep="_"), # new column names
                                  sep=":", #separate on ":"
                                  convert=TRUE #converts numeric columns to numeric
)

vcf.data <- vcf.data %>% separate(R500,
                                  into = paste("R500",c("gt","tot.depth","allele.depth","ref.depth","ref.qual","alt.depth","alt.qual","gt.lik"),sep="_"), # new column names
                                  sep=":", #separate on ":"
                                  convert=TRUE #converts numeric columns to numeric

)
```

## Look at Quality and Filter

**Exercise 1**
To explore the quality of our data, make a histogram of genotype quality. It is hard to get a feel for the distribution of QUAL scores at the low end of the scale (less than 500) using the default settings, so try making a second histogram that illustrates this region better. (Hint: one option is to filter the data)

**Exercise 2**
We only want to keep positions that have a reasonable probability of being correct.

**a** At a quality of 40 what is the probability that the SNP call is wrong?

**b** Filter the data to only keep positions where the quality score is 40 or greater. Put the retained SNPs in an object called `vcf.data.good`

**c** What percentage of SNPs were retained?

**Use the `vcf.data.good`** for the next exercises

## Look at allele classes.

Let’s examine the way SNPs are coded a bit more. We’ll just take a look at a few of the columns, filtering to keep SNPs with reasonable depth in both IMB211 and R500, and where IMB211 and R500 differ:

```
vcf.data.good <- vcf.data.good %>%
  filter(IMB211_gt != R500_gt,
         IMB211_tot.depth > 20,
         R500_tot.depth > 20) %>%
  select(CHROM, POS, REF, ALT, IMB211_gt, IMB211_tot.depth, R500_gt, R500_tot.depth)
```

**Exercise 3:** Examine the `vcf.data.good` object. What do the “0/0”, “0/1”, and “1/1” values indicate? Use IGV to look at a few of the positions above and then explain what “0/0”, “0/1”, and “1/1” values indicate.

We can count the number of SNPS of different classes using table:

```
table(vcf.data.good$IMB211_gt)
table(vcf.data.good$R500_gt)
```

We can even count the numbers common and unique to each cultivar

```
vcf.data.good %>% select(IMB211_gt, R500_gt) %>% ftable
```

**Exercise 4**
**a** Which SNPS would be most useful for a downstream mapping analysis of F2 progeny generated from a cross of IMB211 and R500? (Ignore the allele categories that have “2”, “3”, or “4”). _Hint: you want SNPs that will unambiguously distinguish a locus as coming from IMB211 or R500._

**b** Subset the `vcf.data.good` data frame so that you only have these SNPs. Place the results in `vcf.data.good.F2`

**c** How many SNPS are retained?

**Exercise 5**
**a** Using the high quality F2 SNP list from Exercise 4 (`vcf.data.good.F2`), for each SNP plot its position on the chromosome (x axis), and total read depth (R500 and IMB211 combined) (y axis).

**Optional**: color each SNP based on the percentage of reads that are R500. (optional part not graded).

**b** Use the help function to learn about `xlim()`. Use this function to plot only the region between 20,000,000 and 25,000,000 bp. Why might there be gaps with no SNPs?

**For Fun (??)–not graded–**
Plot the number of each type of base change (A->G, etc). Are there differences? Is this expected?

## Additional options

There is a an R package [vcfR](https://cran.r-project.org/web/packages/vcfR/vignettes/intro_to_vcfR.html) that can help with import and analysis of vcf files in R. This provides an alternative to the the tasks we did above such as adding the header info and extracting the genotype information. It may be worth exploring this package if you are doing a lot of this type of work, although I personally have found using the tidyverse functions here to be more intuitive (and quicker) than the vcfR package.
