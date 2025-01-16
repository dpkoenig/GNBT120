# BLAST Part II

## Orientation
This is a continuation of the BLAST lab focused on identifying the likely cause of a respiratory illness. In the previous lab you learned how to run blast and identified a candidate for the disease agent. Here we will explore this candidate in more detail and learn how to use for loops to automate running multiple blast searches.

## Preliminaries
Make sure that you have logged into the hpcc and have started an interactive job as shown in part one of this three part lab!

## Part 5: Focus on best candidate
We now want to do some more analyses on the best candidate identified in Exercise 7. Looking only at one or two BLAST hits may be misleading. In addition, to understand where this virus came from we should build a phylogenetic tree. To accomplish these tasks we want to optimize our blast search.

*First, open the patient_viral.txt file in RStudio or nano and copy the best candidate sequence from Exercise 7 into a new file.* We will use this new file with a single sequence as the query for the rest of the exercises.

### Word Size
There are many parameters that control the speed of BLAST. Among the most important of these are the seeding parameters. While `blastp` uses both word size and a score threshold to determine seeds, `blastn` only uses word size and requires a perfect match between the query and the subject to seed a search. By default `blastn` is optimized to find very similar sequences using an algorithm called `megablast` that has a default word size of 28; more traditional `blastn` searches use a word_size of 11. Let’s experiment with word sizes ranging from 9 to 19, odd values only, and also include 28

**Exercise 9:** Predict how changing the word size will affect search sensitivity and speed. Explain your reasoning.

*End of Exercise 9*

Now test your predictions: figure out how to change the word-size by getting help on blastn:
```
blastn -help | more
```
Now, instead of typing the command 7 times, lets use a **for loop**. You may need to review for loops from the previous lab.

I have started the code required for the for loop below. You will need to complete it to get this to run. Be sure to have each loop save the results to a **separate** file that **includes the word size** in the file name. Keep the evalue cutoff of 1e-3 but set the max_target_seqs filter to 10000 so we can see how many hits we are getting. What directory should the output be saved in? Make sure it goes there.
```
for WS in {9..19..2} 28
  do
    echo "Starting blastn with word size $WS"
    #time blastn [YOUR CODE HERE]
  done
```
**Exercise 10:** Provide the code used to test different word sizes and then record the time each wordsize blast search took in a markdown table. Use the “real” time reported by the `time` command (as opposed to “user” or “sys”).
*End of exercise 10*

How did word size affect sensitivity? To find out we need to count the number of hits in each file. The challenge is that there can be multiple hits returned for a given subject sequence. So, we need a way to count the number of unique hits.

We will first use the command `cut` to extract the sequence ID of the subject, then sort these with `sort` and finally use `uniq` which removes duplicates so long as there are in order. With this down we can count the number of rows returned.

let’s build this up slowly
```
cut -f 2 blastout.WS9.tsv | head # return the second field; use head to limit output
```
Note: you may need to change the file name above to match your naming scheme

good start but this also includes the info lines that we don’t want. A quick look at the `man` entry for `cut` reveals that `-s` will help us:

```
cut -f 2 -s blastout.WS9.tsv | head
```
Good! They appear to already be grouped together by name, but just in case, let’s sort before using unique
```
cut -f 2 -s blastout.WS9.tsv | sort | uniq | head
```
**Exercise 11:**

*Exercise 11a:* Modify the code above so that it counts the number of unique hits.

*Exercise 11b:* Use a for loop that builds on the command from 11a to count the number of unique hits in each file. Add these results to the table in Exercise 10. Did word size affect time and sensitivity in the direction you predicted from Exercise 9? (FYI a word size of 7 takes 16 minutes to run, but I decided to spare you that pain). If you didn’t already do this above, explain why word size is affecting the results in the way that it does.

For an evolutionary analysis we want to cast a broad net. Let’s explore a couple of additional changes we can make to get more distantly related sequences. There are two things to try: 1) Further optimize blastn; 2) switch to a protein based search. We’ll try both and see how they each work.

Other blastn flavors
It turns out that `blastn` actually has four “flavors”:

1. ‘megablast’, for very similar sequences (e.g, sequencing errors)
2. ‘dc-megablast’, typically used for inter-species comparisons
3. ‘blastn’, the traditional program used for inter-species comparisons
4. ‘blastn-short’, optimized for sequences less than 30 nucleotides.
These different “flavors” change the search parameters such as word-size, gap opening and extension penalties, etc. dc-megablast also allows for *discontiguous* word matches. If you want to see how the default parameters change with the different searches, check out [this table](https://www.ncbi.nlm.nih.gov/books/NBK279684/table/appendices.T.blastn_application_options/)

You can specify which one you want using the `-task` flag, e.g. `-task megablast`.

By default `blastn` runs `megablast`. This was probably good for our first task (finding very similar sequences in order to ID the disease virus) but isn’t good for finding a broader spectrum of related sequences.

**Exercise 12:**

Repeat the `blastn` searches but now comparing `megablast`, `dc-megablast`, and `blastn`. To focus on how task options other than word_size affected these searches, specify a word_size of 11. Use an evalue limit of 1e-3. Add the time and number of (unique) hits to your table. Comment on the differences. For these sequences is word_size or algorithm (task) more important for the number of sequences found?

If you want to use a loop for this, you can loop through the algorithms with:
```
for task in megablast dc-megablast blastn
  do
    echo $task
    # your blastn code here
    # your word count code here
  done
```
##Optional (not graded):## modify the code for exercise 12 to use a nested for loop to test 3 different word sizes for each type of blastn (try 11, 17, 28). You will have an outer loop going though the different tasks and an inner loop that loops through each word size.

Protein-based searches are generally better than nucleotide-based searches for finding distantly related sequences (why?). However, we have nucleotide based viral sequences. Will a protein-based search work? `tblastx` translates the query and the database in all 6 frames and then searches all frames against one another. Viral genomes tend to be very compact (not very much non-coding) so perhaps this is reasonable; let’s give it a shot.

Also, `tblastx` is much slower than `blastn` because it is doing 36 times more work (6 query frames against 6 database frames). If we have requested 4 cpus when we started our interactive job; we can have blast use multiple cpus with the -num_threads option. In the code below we set -num_threads 3 to use 3 CPUs. (It is always a good practice to leave at least 1 CPU free for other tasks).

# note: this will take ~ 6 minutes to run
```
time tblastx -db ../input/ncbi_virus_110119_2.txt -query ../input/candidate_seq.fa -evalue 1e-3 -outfmt '7 std stitle' -max_target_seqs 10000 -num_threads 3 > blastout.tblastx.tsv
```
Note: you may need to change the file name and path of the query

##Exercise 13:## How many unique sequences were found by `tblastx`? Add the `tblastx` result to your table and comment here.

Congratulations, in this lab you have:

identified the likely causative disease agent
learned and gotten practice with bash for loops
delved into some of the intricacies of BLAST searches
found homologous sequences that can be used for subsequent phylogenetic analyses (coming soon to a lab session near you).
