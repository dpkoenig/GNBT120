# Rice SNPs
## Getting Started
* I have created a new repository called `Assignment_4` file there.
* Clone the repository into a new Rstudio Project
* Download the file [RiceSNPData.tar.gz](https://cluster.hpcc.ucr.edu/~dkoenig/COURSE_DATA/RiceSNPData.tar.gz), move the file to your input directory, and inflate the tar-ball (see your notes from “Just Enough Unix” if you don’t remember how to do this.) . You may or may not need to create the `input` folder.
* Open the `Assignment_5.Rmd` file and use that for your answers

## Preliminaries
Let’s load the libraries we need:
```
library(tidyverse)
```
## Some new tools
We begin by learning a few new tools to help us with the upcoming data sets. There will be a couple of more tutorials showing up later in this lab.

### GGPlot
GGplot is a powerful graphing package for R and is part of the Tidyverse.

**Learn some basics by doing [Julin Maloof's tutorial](https://malooflab.ucdavis.edu/apps/ggplot-tutorial/)**

More information on ggplot is available at

* [R for Data Science Chapter 3](https://r4ds.had.co.nz/data-visualisation.html)
* [Cookbook for R](http://www.cookbook-r.com) has many “how-tos” like how to change axis labels, etc.
* [ggplot cheatsheet](https://rstudio.github.io/cheatsheets/data-visualization.pdf)
### Applying functions across rows or columns
It is very common to want to apply a function to each row. We can use the `apply()` function for this. `apply` takes at least 3 arguments.

`apply(X,MARGIN,FUN)` where

* X is a data frame or matrix
* MARGIN is whether to apply a function to each row (1) or each column (2)
* FUN is the function that you want to use

For example
```
set.seed(430) # This command will make it so you get the same matrix
m <- matrix(round(rnorm(24),2),
            ncol=6,
            dimnames=list(NULL, LETTERS[1:6])) #create a matrix of numbers to illustrate apply
m
```
```
##          A     B     C     D     E     F
## [1,] -1.05  0.41  1.52 -0.47  0.41 -0.72
## [2,] -0.69 -0.87 -0.51  0.97  1.01  1.80
## [3,]  2.36  0.20 -0.32  0.81 -0.42 -0.47
## [4,]  0.93 -0.64 -0.08 -1.25  0.34  0.33
```
```
cat("\nrow minimums: ", apply(m,1,min)) # find the minimum value of each row
```
```
##
## row minimums:  -1.05 -0.87 -0.47 -1.25
```
**PRACTICE (not graded)** find the mean of each column of m using apply

### Alternatives
For some key functions there are pre-defined convenience functions
```
rowMeans(m)
colMeans(m)
rowSums(m)
colSums(m)
```
### Tidyverse Alternatives
To summarize all columns in the tidyverse way, use `across()`, which by default applies a function to each column.
```
m %>% as_tibble() %>%
  summarize(across(.cols=everything(),.fns = mean))
```
```
## # A tibble: 1 × 6
##       A      B     C      D     E     F
##   <dbl>  <dbl> <dbl>  <dbl> <dbl> <dbl>
## 1 0.388 -0.225 0.152 0.0150 0.335 0.235
```
Tidyverse row summaries are more cumbersome, but if you want to do it, first specify that you want to be operating on each row by `rowwise()` and then use `mutate()` to create a new column with the row means. The `c_across()` function combines each column specified (all columns by default) into a vector.
```
m %>% as_tibble() %>%
  rowwise() %>%
  mutate(rowmean = mean(c_across(cols=everything())))
```
```
## # A tibble: 4 × 7
## # Rowwise:
##       A     B     C     D     E     F rowmean
##   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>   <dbl>
## 1 -1.05  0.41  1.52 -0.47  0.41 -0.72  0.0167
## 2 -0.69 -0.87 -0.51  0.97  1.01  1.8   0.285
## 3  2.36  0.2  -0.32  0.81 -0.42 -0.47  0.36
## 4  0.93 -0.64 -0.08 -1.25  0.34  0.33 -0.0617
```
## Lets get started with the real data
### Data Import
You learned how to import data last week using `read_csv()`. Note that read_csv can read in a .gzipped file directly. Today we will provide an additional argument to the read_csv function:

* This will take a minute or two to load
```
Sys.setenv(VROOM_CONNECTION_SIZE="500000") # needed because the lines in this file are _extremely_long.
data.geno <- read_csv("../input/Rice_44K_genotypes.csv.gz",
                      na=c("NA","00")) #this tells R that missing data is denoted as "NA" or "00"
```
```
## New names:
## Rows: 413 Columns: 36902
## ── Column specification
## ────────────────────────────────────────────────────────────────────────────── Delimiter: "," chr
## (36902): ...1, 1_13147, 1_73192, 1_74969, 1_75852, 1_75953, 1_91016, 1_146625, 1_149005, 1_149...
## ℹ Use `spec()` to retrieve the full column specification for this data. ℹ Specify the column types or
## set `show_col_types = FALSE` to quiet this message.
## • `` -> `...1`
## • `6_17160794` -> `6_17160794...22252`
## • `6_17160794` -> `6_17160794...22253`
```
Use `dim()` to determine the numbers of rows and columns. (You can also look at the info in the right-hand pane). There are 413 rows and 36,902 columns. Generally I recommend looking at files after they have been read in with the `head()` and `summary()` functions but here we must limit ourselves to looking at a subset of what we read in.
```
head(data.geno[,1:10]) #first six rows of first 10 columns
```
```
summary(data.geno[,1:10]) #summarizes the first 10 columns
```
In this file each row represents a different rice variety and each column a different SNP. The column `...1` gives the ID of each variety. `...1` is not a very informative name (that column did not have a name in the data file and R assigned it the name `...1`.) Let’s rename it:
```
data.geno <- data.geno %>% rename(ID=...1)

head(data.geno[,1:10]) #first six rows of first 10 columns
```
```
## # A tibble: 6 × 10
##   ID     `1_13147` `1_73192` `1_74969` `1_75852` `1_75953` `1_91016` `1_146625` `1_149005` `1_149754`
##   <chr>  <chr>     <chr>     <chr>     <chr>     <chr>     <chr>     <chr>      <chr>      <chr>
## 1 NSFTV1 TT        TT        CC        GG        TT        AA        CC         TT         AA
## 2 NSFTV3 CC        CC        CC        AA        GG        <NA>      CC         GG         TT
## 3 NSFTV4 CC        CC        CC        AA        GG        GG        CC         GG         TT
## 4 NSFTV5 CC        CC        TT        GG        GG        AA        TT         GG         TT
## 5 NSFTV6 CC        CC        CC        AA        GG        GG        CC         GG         TT
## 6 NSFTV7 TT        TT        CC        GG        TT        AA        CC         TT         AA
```
The column names give the chromosome and locations of each SNP. For example, “1_151492” is a SNP on chromosome 1, base position 151492.

## A PCA Plot.
Principal Components Analysis (PCA) and Multi-Dimensional Scaling (MDS) plots can be used to display the genetic diversity in a 2D space. The full 36,901 SNPs take too long to run for this class so we will subset the data.

To do this we take advantage of the sample() function. Sample takes a random sample of the items it is given.
```
sample(x=1:10, # x is that we want to sample from
       size=5) # size is the number of samples we want to take from x
```
```
## [1] 2 6 7 9 1
```
```
# so the above takes 5 random samples from the numbers 1:10 (by default without replacement)
```
We can use this to randomly sample a data frame by rows or columns.
```
# first make a demo matrix
m2 <- matrix(1:100,nrow=10)
m2
```
```
##       [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
##  [1,]    1   11   21   31   41   51   61   71   81    91
##  [2,]    2   12   22   32   42   52   62   72   82    92
##  [3,]    3   13   23   33   43   53   63   73   83    93
##  [4,]    4   14   24   34   44   54   64   74   84    94
##  [5,]    5   15   25   35   45   55   65   75   85    95
##  [6,]    6   16   26   36   46   56   66   76   86    96
##  [7,]    7   17   27   37   47   57   67   77   87    97
##  [8,]    8   18   28   38   48   58   68   78   88    98
##  [9,]    9   19   29   39   49   59   69   79   89    99
## [10,]   10   20   30   40   50   60   70   80   90   100
```
if we want to randomly sample 3 rows, then:
```
m2[sample(1:nrow(m2),3),]
```
```
##      [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## [1,]    6   16   26   36   46   56   66   76   86    96
## [2,]    2   12   22   32   42   52   62   72   82    92
## [3,]    3   13   23   33   43   53   63   73   83    93
```
### Random number seeds
Computers generate [psuedo-random](https://en.wikipedia.org/wiki/Random_number_generation) numbers. Sometimes it is useful to have the computer generate the same set of “random” numbers for reproducibility. This is accomplished by setting the random number “seed” which determines the starting point for the random number generator. We will do this below so that we all are working with the same samples.

**Exercise 1**

Now, create a data subset that contains a random sample of 10000 SNPs from the full data set. Place the smaller data set in an object called `data.geno.10000`. **Very important: you want to keep the first column, the one with the variety IDs, and you want it to be the first column in `data.geno.10000`. AND You do not want it to show up randomly later on in the data set.** Think about how to achieve this. Hint: `[ ]` will help

Make the first line of you code block `set.seed(0421)`

Check the dimensions of your subset. If you don’t get the output below, you did something wrong:
```
dim(data.geno.10000)
```
```
## [1]   413 10001
```
Make sure that “ID” is in the first column and nowhere else (if not, you did something wrong)
```
colnames(data.geno.10000) %>% str_which("ID") #returns the position of entries that match "ID".
```
```
## [1] 1
```
*End of Exercise 1*

Now that we have our smaller subset we can create the PCA plot

First convert the genotype data to a numeric form.
```
#convert the data matrix to numbers
geno.numeric <- data.geno.10000[,-1] %>% # -1 to remove the first column, with names.
  lapply(factor) %>% # convert characters to "factors", where each category is internally represented as a number
  as.data.frame() %>% # reformat
  data.matrix() #  convert to numeric

head(geno.numeric[,1:10])
```
In order to calculate the principal components we need to get rid of NAs. A quick and dirty method is to replace the missing data with the average genotype for the SNP. More sophisticated would be to impute the genotype using linkage data and the neighboring SNPs. Will use quick and dirty here.

Fill in missing data (remind me to explain this in a Friday discussion):
```
geno.numeric.fill <-
  apply(geno.numeric, 2, function(x) {
    x[is.na(x)] <- mean(x, na.rm=T)
    x})
```
Compute the principal components.
```
geno.pca <- prcomp(geno.numeric.fill,
            rank.=10) # We really only need the first few, this tells prcomp to only return the first 10

str(geno.pca)
```
```
## List of 5
##  $ sdev    : num [1:413] 32.3 13.47 12.8 7.42 6.26 ...
##  $ rotation: num [1:10000, 1:10] -0.01263 0.01099 0.00136 -0.00671 0.01351 ...
##   ..- attr(*, "dimnames")=List of 2
##   .. ..$ : chr [1:10000] "X2_29698250" "X6_1585457" "X3_27978687" "X6_25054910" ...
##   .. ..$ : chr [1:10] "PC1" "PC2" "PC3" "PC4" ...
##  $ center  : Named num [1:10000] 1.65 1.4 1.04 1.37 1.38 ...
##   ..- attr(*, "names")= chr [1:10000] "X2_29698250" "X6_1585457" "X3_27978687" "X6_25054910" ...
##  $ scale   : logi FALSE
##  $ x       : num [1:413, 1:10] -31.82 46.71 36.6 -5.28 34.89 ...
##   ..- attr(*, "dimnames")=List of 2
##   .. ..$ : NULL
##   .. ..$ : chr [1:10] "PC1" "PC2" "PC3" "PC4" ...
##  - attr(*, "class")= chr "prcomp"
```
`str()` is a good way to take a quick look at complex objects. Geno.pca has 5 items

* `x`: the principal components
* `sdev`: the standard deviation of the principal components
* `rotation`: the loadings
* `center and scale`: centering and scaling factors or FALSE if not used
```
#The PCs themselves are here
head(geno.pca$x)[,1:5]
```
```
##             PC1        PC2        PC3        PC4        PC5
## [1,] -31.816332  -3.683880  19.608941 -0.8600434  0.6719670
## [2,]  46.709019  19.330302   5.215883  0.9866784 -4.0609646
## [3,]  36.601600 -21.143653  -4.445832 -6.8283539  0.2118635
## [4,]  -5.283928  -7.599221  -4.681273 40.4523248 -2.1894743
## [5,]  34.894337 -20.935916  -4.642829 -6.5149579  1.4025620
## [6,] -19.210544   3.859785 -17.933528 -0.9314848 -0.7210319
```
each column is a PC and each row is one of the 413 rice samples.
```
#you can relate back to the original SNPs by:
head(geno.pca$rotation)[,1:5] #tells you how much each SNP contributed to each PC
```
```
##                       PC1           PC2          PC3           PC4          PC5
## X2_29698250  -0.012634195  0.0006115434 -0.001759956  0.0067432225  0.003213385
## X6_1585457    0.010991928 -0.0027230438 -0.005572183 -0.0102837893 -0.017947735
## X3_27978687   0.001359511  0.0031429310  0.001171402 -0.0002198166  0.002617432
## X6_25054910  -0.006712620  0.0009538070 -0.005407723  0.0080698479  0.000800433
## X8_27369483   0.013514845 -0.0011350776  0.002373342 -0.0063878390 -0.001589261
## X11_25310834 -0.004878862  0.0043541442 -0.000538531  0.0027339308 -0.006100854
```
```
#The std. deviation explained by each PC is at:
head(geno.pca$sdev)
```
```
## [1] 32.298809 13.473447 12.803174  7.421557  6.263168  5.211491
```
If we want to determine the proportion of variance explained by each PC we can use the info in sdev:
```
pcvar <- geno.pca$sdev^2 # square std dev to get variance
pcvar.pct <- tibble(pctvar=pcvar/sum(pcvar) * 100,
                    PC=1:length(pcvar))
```
**Exercise 2**: plot the variance explained by the first 10 PCs (that is plot the first 10 rows of `pcvar.pct`); it should look something like plot of chunk PC_pct_var So, we have collapsed the majority of the variation in our 36,900 SNPs into 3 PCs!

Let’s pull out the PCs into their own tibble:
```
PCs <- as_tibble(geno.pca$x) %>% # the principal components
  mutate(ID=data.geno.10000$ID) %>%
  select(ID, everything())
head(PCs)
```
```
## # A tibble: 6 × 11
##   ID        PC1    PC2    PC3    PC4    PC5     PC6     PC7     PC8     PC9  PC10
##   <chr>   <dbl>  <dbl>  <dbl>  <dbl>  <dbl>   <dbl>   <dbl>   <dbl>   <dbl> <dbl>
## 1 NSFTV1 -31.8   -3.68  19.6  -0.860  0.672  -0.101   0.224 -0.0914   0.157 -2.05
## 2 NSFTV3  46.7   19.3    5.22  0.987 -4.06  -13.8     4.58  -1.03     1.39  -1.59
## 3 NSFTV4  36.6  -21.1   -4.45 -6.83   0.212  -5.02  -12.9    6.24   -17.3   -2.95
## 4 NSFTV5  -5.28  -7.60  -4.68 40.5   -2.19    0.958  -1.21   4.42    -0.160  1.21
## 5 NSFTV6  34.9  -20.9   -4.64 -6.51   1.40   -6.34  -14.9    7.13   -19.1   -3.85
## 6 NSFTV7 -19.2    3.86 -17.9  -0.931 -0.721   0.136  -0.390  0.739   -1.73   2.46
```
**Exercise 3:** Make 2 scatter plots, the first of PC1 vs PC2, and second PC2 vs PC3 (keep PC2 on the y-axis for both plots). Is there any evidence for populations structure (different sub populations)? If so, how many sub populations do you think the PCA plot reveals? What do you make of the individuals that are between the major groups?

### Alternative: MDS
How this would compare to an MDS analysis? You can compute the MDS as follows:
```
#calculate the Euclidian distance between each rice variety
genDist <- as.matrix(dist(geno.numeric))

#perform the multi-dimensional scaling
geno.mds <- as_tibble(cmdscale(genDist))
```
```
## Warning: The `x` argument of `as_tibble.matrix()` must have unique column names if `.name_repair` is omitted
## as of tibble 2.0.0.
## ℹ Using compatibility `.name_repair`.
```
```
#Add the variety ID back into this
geno.mds$ID <- data.geno.10000$ID
head(geno.mds) #now we have 2 dimensions + the ID
```
`geno.mds` contains the genotypic information rescaled to display in two dimensions. Now lets plot it. Make a x-y scatter plot of the data, with “V1” on one axis and “V2” on the other axis.

### Adding phenotypes
The file “RiceDiversity.44K.MSU6.Phenotypes.csv” contains information about the country of origin and phenotypes of most of these varieties.

We are going to need combine the data in the phenotype file with the genotype PCA data. In the tidyverse this is known as **joining** two data frames.

Please learn about joins by working through [Julin Maloof's join tutorial](http://malooflab.phytonetworks.org/apps/joins-tutorial/)

**EXERCISE 4:**

* Use the `read_csv()` `head()` and `summary()` (or `skim()`…look it up) functions that you learned earlier to import and look at this file. Import the file into an object called “data.pheno”.
* Use a `join` function to merge the PC genotype data (in the object `PCs`) with the phenotype data into a new object called `data.pheno.pca`. Use summary and head to look at the new object and make sure that it is as you expect.
* It (`data.pheno.pca`) should have 413 rows and 49 columns.
* Include your code in the .Rmd
```
## Rows: 413 Columns: 39
## ── Column specification ──────────────────────────────────────────────────────────────────────────────
## Delimiter: ","
## chr  (6): NSFTVID, Accession_Name, Country_of_Origin, Region, Seed color, Pericarp color
## dbl (33): Alu.Tol, Flowering time at Arkansas, Flowering time at Faridpur, Flowering time at Aberd...
##
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
We can now color points on our PCA plot by characteristics in this data set.

**EXERCISE 5**: Prepare three different PCA plots to explore if subgroups vary by 1) Amylose content; 2) Pericarp color; 3) Region. That is make a scatter plot of PC1 vs PC2 and color the points by the above characteristics. Do any of these seem to be associated with the different population groups? Briefly discuss. (optionally repeat the plots plotting PC2 vs PC3)

*Hint 1* to refer to column names that contain spaces in the name you can surround the name with backticks.

*Hint 2* use `color=` argument ggplot to color the point by the different traits

*Hint 3* when plotting the Region data, the colors will be easier to interpret if you include the following in your ggplot code: `+ scale_color_brewer(type="qual", palette = "Set1")` This specifies an opimized set of colors is used. (Try plotting with and without this).

## Assign populations with fastStructure
From the PCA plot it looks like there is structure in our population. But how do we know which individual belongs in which population? We can take an alternative approach and assign individuals to specific population classes with fastStructure, a python program for accomplishing this task.

We have to create three files in order to run fastStructure

### Genotype file
First we have to convert our genotypes to the form that fastStructure expects. fastStructure wants a separate row for each allele (two rows for each variety). A couple of the commands below are a bit complex. I can explain them in lab or discussion if desired. **OK** to **CUT** and **PASTE**.
```
#Create a new Matrix to hold reformatted data
data.geno.10000.fs <- matrix("",nrow=nrow(data.geno.10000)*2,ncol=ncol(data.geno.10000)-1+6)

#for each row of genotypes, create 2 rows, one with the first allele and one with the second allele.
#additionally, fill the first 6 columns with the strain ID (could also be blank but we need something there)
for (i in 1:nrow(data.geno.10000)) {
  data.geno.10000.fs[(i-1)*2+1,1:6] <- data.geno.10000[[i,1]]
  data.geno.10000.fs[(i-1)*2+2,1:6] <- data.geno.10000[[i,1]]
  data.geno.10000.fs[(i-1)*2+1,-1:-6] <- substr(data.geno.10000[i,-1],1,1)
  data.geno.10000.fs[(i-1)*2+2,-1:-6] <- substr(data.geno.10000[i,-1],2,2)
}

data.geno.10000.fs[is.na(data.geno.10000.fs)] <- -9 # fastStructure's code for missing data

dim(data.geno.10000.fs)

#take a look
data.geno.10000.fs[1:10,1:10]

write.table(data.geno.10000.fs,file="../output/rice.data.fastStructure.input.str", col.names = FALSE, row.names = FALSE, quote = FALSE)
```
### Create the .fam file.
The .fam file describes the individuals in the study. It has a number of columns designed for human studies which do not exactly apply here.

The fields are:

1. Family ID (‘FID’)
2. Within-family ID (‘IID’; cannot be ‘0’)
3. Within-family ID of father (‘0’ if father isn’t in data set)
4. Within-family ID of mother (‘0’ if mother isn’t in data set)
5. Sex code (‘1’ = male, ‘2’ = female, ‘0’ = unknown)
6. Phenotype value (‘1’ = control, ‘2’ = case, ‘-9’/’0’/non-numeric = missing data if case/control)

Since our samples are inbred strains we use the same ID for columns 1 and 2, and 0 for columns 3 and 4. Rice plants are hermaphroditic, so we use “0” for the sex. We don’t have a phenotype for column 6 for we will put -9 there.

**OK** to **CUT** and **PASTE**.
```
fam <- tibble(
  FID=data.geno.10000$ID,
  IID=data.geno.10000$ID,
  PID=0,
  MID=0,
  Sex=0,
  Ptype=-9)

head(fam)
```
```
## # A tibble: 6 × 6
##   FID    IID      PID   MID   Sex Ptype
##   <chr>  <chr>  <dbl> <dbl> <dbl> <dbl>
## 1 NSFTV1 NSFTV1     0     0     0    -9
## 2 NSFTV3 NSFTV3     0     0     0    -9
## 3 NSFTV4 NSFTV4     0     0     0    -9
## 4 NSFTV5 NSFTV5     0     0     0    -9
## 5 NSFTV6 NSFTV6     0     0     0    -9
## 6 NSFTV7 NSFTV7     0     0     0    -9
```
```
write.table(fam,file="../output/rice.data.fastStructure.input.fam",col.names = FALSE, row.names = FALSE, quote = FALSE)
```
### Create the bim
The bim file provides information on the alleles.

It contains 6 fields:

1. Chromosome code (either an integer, or ‘X’/’Y’/’XY’/’MT’; ‘0’ indicates unknown) or name
2. Variant identifier
3. Position in Morgans or centiMorgans (safe to use dummy value of ‘0’)
4. Base-pair coordinate (normally 1-based, but 0 OK; limited to 231-2)
5. Allele 1 (corresponding to clear bits in .bed; usually minor)
6. Allele 2 (corresponding to set bits in .bed; usually major)
**OK to CUT and PASTE.**
```
bim <- data.geno.10000.fs[,-1:-6]

colnames(bim) <- colnames(data.geno.10000)[-1]

bim[bim=="-9"] <- NA

bim <- apply(bim,2,function(x) unique(na.omit(x)))

bim[,1:5]
```
```
##      2_29698250 6_1585457 3_27978687 6_25054910 8_27369483
## [1,] "G"        "A"       "C"        "T"        "C"
## [2,] "C"        "G"       "T"        "A"        "G"
```
```
bim <- t(bim) %>%  # t transposes the matrix
  as_tibble() %>%
  mutate(SNP_ID=colnames(bim), cM=0)

head(bim) # now we have a table where each row is a SNP
```
```
## # A tibble: 6 × 4
##   V1    V2    SNP_ID         cM
##   <chr> <chr> <chr>       <dbl>
## 1 G     C     2_29698250      0
## 2 A     G     6_1585457       0
## 3 C     T     3_27978687      0
## 4 T     A     6_25054910      0
## 5 C     G     8_27369483      0
## 6 A     G     11_25310834     0
```
Next step is to separate the SNP_ID to get chromosome and position

**OK to CUT and PASTE.**
```
bim <- bim %>%
  separate(SNP_ID,into = c("chromosome","position"),sep="_",remove=FALSE) %>% # create a column for chromosome and position
  select(chromosome, SNP_ID, cM, position, allele1=V1, allele2=V2) # get columns in right order

head(bim)
```
```
## # A tibble: 6 × 6
##   chromosome SNP_ID         cM position allele1 allele2
##   <chr>      <chr>       <dbl> <chr>    <chr>   <chr>
## 1 2          2_29698250      0 29698250 G       C
## 2 6          6_1585457       0 1585457  A       G
## 3 3          3_27978687      0 27978687 C       T
## 4 6          6_25054910      0 25054910 T       A
## 5 8          8_27369483      0 27369483 C       G
## 6 11         11_25310834     0 25310834 A       G
```
```
write.table(bim,file="../output/rice.data.fastStructure.input.bim",col.names = FALSE, row.names = FALSE, quote = FALSE)
```
### Run fastStructure
Now we can run fastStructure. fastStructure will determine the proportion of each individual’s genome that came from one of K ancestral populations.

First log in to the hpcc from the terminal and use `srun` to connect start an interactive job. **Do not run on the head node!**
Then load the faststructure module

fastStructure is run from the Linux command line.
```
structure.py -K 4 --input=rice.data.fastStructure.input --output=rice.fastStructure.out --format=str
(this will take about 2 minutes to run)
```
In the above command, `-K 4` specifies the number of ancestral populations that fastStructure should create

### Load the fastStructure results
The file `rice.fastStructure.out.4.meanQ` contains 1 row for each sample, and 1 column for each ancestral population. The numbers give the proportion of the genome inferred to have come from the ancestral population.

Back in R:
```
fs_results <- read_delim("../output/rice.fastStructure.out.4.meanQ", delim="  ", col_names = FALSE, col_types = 'nnnn')
head(fs_results)
```
Now lets add sample IDs back and give our populations names
```
fs_results <- fs_results %>%
  mutate(ID=data.geno.10000$ID) %>%
  select(ID, pop1=X1, pop2=X2, pop3=X3, pop4=X4)
head(fs_results)
```
```
## # A tibble: 6 × 5
##   ID         pop1     pop2     pop3     pop4
##   <chr>     <dbl>    <dbl>    <dbl>    <dbl>
## 1 NSFTV1 0.000013 1.00     0.000014 0.000013
## 2 NSFTV3 0.000014 0.000014 0.000014 1.00
## 3 NSFTV4 0.880    0.000014 0.000014 0.120
## 4 NSFTV5 0.339    0.259    0.402    0.000014
## 5 NSFTV6 0.887    0.000014 0.000014 0.113
## 6 NSFTV7 0.0431   0.0116   0.944    0.00166
```
In the fs_results table, each row is an individual and each column represents one of the hypothesized populations.
Genomes with substantial contributions from two ancestral genomes are said to be admixed

Let’s use these proportions to assign each individual to a particular population. The code below assigns an individual to a population based on whichever ancestral genome has the highest proportion. Of course this can be somewhat problematic for heavily admixed individuals.
```
fs_results$assignedPop <- apply(fs_results[,-1], 1, which.max)
head(fs_results)
```
If you want to know how many individuals were assigned to each population, you can use table()
```
table(fs_results$assignedPop)
```
The fastStructure output must be reformatted in order to plot it well. Not all of the commands are fully explained below. If you have questions we can go over this in Friday’s discussion.

For plotting it will be helpful to order the samples based on population identity and composition. To do this, let’s make a new column that has the maximum proportion for each sample and then rank them accordingly
```
fs_results$maxPr <- apply(fs_results[,2:5],1,max)
fs_results <- fs_results %>%
  arrange(assignedPop,desc(maxPr)) %>%
  mutate(plot.order=row_number())
```
### Reshaping data.
Spreadsheet data is often organized in a “wide” format where there is one row per individual and then multiple traits measured on that individual are in separate columns. This is a convenient format for data entry, but R often wants data in a “long” format with one observation per row. For example, the FastStructure data is in wide format but we need it in long format. Please work through [Julin Maloof's Pivot tutorial](http://malooflab.phytonetworks.org/apps/tidyverse-pivot-tutorial/) to learn more about these formats and how to convert.
```
fs_results_long <- fs_results %>% pivot_longer(pop1:pop4,
                                               names_to="population",
                                               values_to="proportion")
head(fs_results_long)
```
```
## # A tibble: 6 × 6
##   ID      assignedPop maxPr plot.order population proportion
##   <chr>         <int> <dbl>      <int> <chr>           <dbl>
## 1 NSFTV13           1  1.00          1 pop1         1.00
## 2 NSFTV13           1  1.00          1 pop2         0.000014
## 3 NSFTV13           1  1.00          1 pop3         0.000014
## 4 NSFTV13           1  1.00          1 pop4         0.000014
## 5 NSFTV33           1  1.00          2 pop1         1.00
## 6 NSFTV33           1  1.00          2 pop2         0.000014
```
Finally we are ready to plot the results. In the plot produced below, each column is a single rice variety. The colors correspond to ancestral genomes. Do you see any evidence of admixture?
```
fs_results_long %>%
  ggplot(aes(x=plot.order, y=proportion, color=population, fill=population)) +
  geom_col()  +
  ylab("genome proportion") +
  scale_color_brewer(type="div") + scale_fill_brewer(type="div")
```
## How do these population assignments relate to the PCA plot?
**Exercise 6**: First, use a join function to combine the PCA data (in object `PCs`) with the population assignments (in `fs_results`) and place the result in `geno.pca.pop` Then re plot the PCA data, but use the population assignment to color the points. Make one plot for PC1 vs PC2 and a second for PC3 vs PC2. How do the populations assignments relate to the PCA plots?

**Hint** convert the `assignedPop` variable to a character type before starting, with:
```
fs_results <- fs_results %>% mutate(assignedPop=as.character(assignedPop))
```
We will use some of the objects that you have created today in next week’s lab, so lets save them in an .Rdata file for easy loading on next week
```
save(data.pheno,geno.pca, PCs, geno.pca.pop,fs_results,file="../output/data_from_SNP_lab.Rdata")
```
