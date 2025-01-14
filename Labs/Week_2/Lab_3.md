# BLAST Part I
## Attribution
This lab was developed by Julin Maloof and modified by Daniel Koenig.

## Scenario
Let’s turn the clock back to December, 2019. There is a strange new infectious disease causing flu-like symptoms and respiratory distress in humans and it is spreading rapidly. Based on the disease etiology it seems most likely to be caused by a virus. Your collaborators have isolated viral particles from an infected patient, purified the viral genomes and sequenced them. They are asking for your help in analyzing the sequences.

Of the sequenced viral genomes from the patient there are two immediate questions:

1. Which of the viral sequences from the patient is most likely to be the one responsible for the disease?
What is the origin of this virus?
2. We will focus on those questions over the next few days, explore how BLAST parameters affect search outcomes, and practice our Linux skills along the way.

## Part 1: Getting Organized
### Open the repo in Rstudio as a project
On the assignments page, click on the assignment 2 link to create your repo, then clone it into Rstudio.

For this part of assignment 2:

* Graded: answer the questions in the `Assignment_2_BLAST_template.md` file located in the `scripts` folder.
* Not graded, but recommended: keep notes on what you are learning in the `Assignment_2_lab_notebook.md` file.
cd into the `assignment_2...` directory.

A common way to organize projects is to create the following three directories:

* input this will contain any input data
* output this will contain any results coming from analysis
* scripts scripts or markdown files used for the analysis
These should have been created when you cloned your repository.

## Connect to the cluster and learn how to create an interactive job
1. Connect to the server as you learned [here](https://dpkoenig.github.io/GNBT120/Labs/Week_1/Lab_1_Preliminaries)
2. Start an interactive job on the cluster to conduct your analysis. I will discuss this further in a future class but for now use the following to connect. You can find more information about initiating jobs on the cluster [here](https://hpcc.ucr.edu/manuals/hpc_cluster/jobs/). Enter the following to generate an interactive job.

```
srun --mem=1gb --cpus-per-task 1 --ntasks 1 --time 5:00:00 --pty bash -l
```
3. You will need to use the blast software to conduct the following lab. BLAST is installed on the cluster, but you need to load it to use it. The cluster has a software management system where specific software can be loaded using `module`. you can see all the software that can be loaded by entering the command `module avail`. Enter the following to load blast.
```
module load ncbi-blast/2.16.0+
```
4. Change directories into your project directory (inside the bigdata directory)

## Setup
Please find a template for answering the exercises in the `scripts` folder. Open `Assignment_2_BLAST_template.md` in the editor of your choice (RStudio or nano, for example)

There are two sequence files that you will be working with in this lab, you will need to download them both (see instructions below).

`ncbi_virus_110119_2.txt.gz` This contains all complete genomic viral sequences available at [NCBI](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/#/) on November 01, 2019. I will refer to this file as the “refseq” file below.
The second file contains the sequences obtained from the patient. I will refer to this as the “patientseq” file below.
We will go over how to download them into the proper `Assignment_2` directory with some commands in a moment.

Now let’s download the files.

What directory should you place these in? Please decide whether they should go into `input`, `output`, `scripts`, or the top-level `assignment-02` directory, and then `cd` there

Download the files with `wget` as follow. `wget` gets things from the Web…

(OK to cut and paste this…)
```
wget https://cluster.hpcc.ucr.edu/~dkoenig/COURSE_DATA/ncbi_virus_110119_2.txt.gz
wget https://cluster.hpcc.ucr.edu/~dkoenig/COURSE_DATA/patient_viral.txt.gz
```

Let’s begin by practicing our Linux skills by summarizing the file content. Start by taking a look at each file using `zless` or `zmore`.

`zless` and `zmore` are shortcuts for typing `gunzip -c 'your_file.gz' | less` and `gunzip -c 'your_file.gz' | more`

## Start of Exercises
*When in doubt always provide the code you used to obtain your answers. This will help in both remembering what you did and troubleshooting problems*

**Exercise 1:** What is the [sequence format](https://www.animalgenome.org/bioinfo/resources/manuals/seqformats) for these sequences? Do the files contain RNA or DNA sequences? How do you know?

**Exercise 2:** For the refseq file, the header line contains a number of fields, each separated by a `|`.

*Exercise 2a:* Use `zgrep` and `head` to display the first set of header lines (without printing the sequences themselves). What command did you use?

*Exercise 2b:* For the first entry in the refseq file (starts with `>MH781015`), try to figure out what each item in the header is. List each field and what you think it is. You should have fields for

* sequence completeness
* viral species
* country where the virus was isolated
* sequence title
* accession number (a unique identifier for the sequence).
* mystery (figure out what the unnamed field is)

**Exercise 3:** How many of the viruses come from a domesticated cat (*Felis catus*) host? How many come from a human host? How many were isolated in the United States? Show the commands used to answer these questions.

*Hint 1: Look at the `man` entry for `grep` to figure out how to count things*

*Hint 2: You can do the whole thing with `grep`, or you might want to use `cut`. `cut` will give you “fields” from a text file. For example: (try this in the terminal to see what happens)*
```
# create a simple example to illustrate cut
## first create a text file
echo "Koenig,Daniel,dkoenig@ucr.edu" > email.txt
echo "Seymour,Danelle,dseymour@ucr.edu" >> email.txt

## look at the file
cat email.txt

## extract the third field
cut -f 3 -d "," email.txt
```
Stop and Think: what do the `-f` and `-d` options do? You can look at `man cut` if you aren’t sure

*Hint 3: Be aware when searching for viruses isolated in the United States, the United States can appear in an entry even when it was not isolated there. Make sure you are using the “country where virus isolated” field when obtaining your count*

**Exercise 4**: Create a simple (markdown formatted) table with the following information for the refseq and patientseq files:

1. Size of the file (in kilobytes, megabytes, or gigabytes. Indicate your units).
2. Number of sequences in the file.
3. Total number of basepairs or amino acids in the file (see hint 2 below!)
4. Average sequence length.

*Hint 1: Review the “Just enough Unix” tutorial to determine the correct commands needed to answer these questions. You may need to use the `man` pages to learn about options to give the commands.*

*Hint 2: The total number of basepairs will be the number of nucleotide letters. This includes letters other than the usual “ACGT”. If you are curious in the meaning of these new letters you can look up their definitions here. Finding the size of the genome is a little complicated because it’s* **not** *the size of the file. The files also have newline characters and FASTA headers. You need to subtract these. Fortunately, you can do all of this with the Unix skills you already have. Use the `man` pages!*

## Part 3: Build BLAST database
Now that we have explored the files some, let’s get BLASTING!

In order for BLAST to search efficiently if needs to build a database of words for each sequence in the reference that we want to search. This only needs to be done once.

You only need to build a database for `ncbi_virus_110119_2.txt`

The command is `makeblastdb`. Take a look at the help file to see how to run it:
```
makeblastdb -help
```
Can you figure out how to specify the input file and the sequence type? If you get stuck, use the text underneath this sentence for a hint.
```
makeblastdb -in ncbi_virus_110119_2.txt -dbtype nucl
```
Getting an error? Maybe the file needs to be `gunzipped`?

## Part 4: Run BLAST, explore output parameters, find disease agent
First, `cd` to the output directory in your project folder.
```
cd ../output
```
**At the command line you can run blast as follows:**
```
time blastn -db ../input/ncbi_virus_110119_2.txt -query ../input/patient_viral.txt -evalue 1e-3 > blastout.default
```
Where the `evalue` option limits the returned results to those with an evalue < 1e-3

**Note:** time is optional, it just tells bash to report the amount of time that a command takes.

**Note:** the command above is written assuming that you are in the `output` directory when you issue the command and that your sequence files and blast database are in `input`. You may need to modify the paths depending on how you organized things.

**Exercise 5:** How long did it take BLAST to run?

**Exercise 6:** Take a look at the beginning of the output file. What type of virus did “Seq_A” come from?

## Change format of the BLAST output
If you scroll through the blast output you will see that for each query sequence there are a list of hits followed by alignments for the top hits. This might be useful if you have a single query but is not so great for summarizing the results of multiple query sequences as we have here.

One thing that would be helpful would be to have each “hit” summarized on a single line. We can do this by specifying `-outfmt 7`. We’ll modify that table by adding subject title (`stitle`) to the standard (`std`) fields. (Look at blastn help to see all options).
```
time blastn  -db ../input/ncbi_virus_110119_2.txt -query ../input/patient_viral.txt -evalue 1e-3 -outfmt '7 std stitle' > blastout.default.tsv
```
### Limit the number of results returned
Later in the quarter we will learn how to easily load and analyze these sequences in R. But in the meantime it would be helpful to have only the single or a few hits reported for each query.

We can do this with `-max_target_seqs`. `-max_target_seqs` limits the number of subject sequences returned. Due to some quirks with the BLAST algorithm these may not be the absolute best hits but they should be pretty similar. For more info, click [here](https://blastedbio.blogspot.com/2018/11/blast-max-alignment-limits-repartee-two.html).

**Exercise 7:** Run a new BLAST but limit the results to 2 subjects per query sequence. (Show your code). Which patient sequence do you think comes from the virus that causes the respiratory disease? If you are unsure, do some literature/web searches on the different viruses returned. Explain your reasoning.

**Note:** You may get more than 2 hits per a query sequence, if you look closely at columns 2, 7, and 8 of the output you may figure out why.

**Exercise 8:** Consider the host species listed for the hits. Remembering that our samples came from a human patient, which three hosts are most evolutionarily distant from humans? Could the viruses generating these hits still have come from the patient sample or does this represent contamination from another source? (Again some web or literature searches will be helpful). Explain

OK that is it for part 1 of BLAST. Next we will learn how to automate repetitive tasks with for loops, and then apply that technique to running BLAST.
