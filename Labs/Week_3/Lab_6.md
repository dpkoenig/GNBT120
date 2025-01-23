# Introduction to R
Connect to Rstudio on the hpcc, download and open project for Assignment 3
## Clone the Assignment 3 Respository
On the assignments page, click on the [assignment 3](https://dpkoenig.github.io/GNBT120/assignments) link and then start a new project in R by cloning the repo.

## The swirl tutorial
Install the tutorial
```
install.packages("swirl")
```
Next tell R that you want to load the swirl package
```
library(swirl)
```
And start the swirl tutorial.
```
swirl()
```
When given the choice, choose `1: R Programming: The basics of programming in R`. There will be a delay while the tutorial is downloaded.

At the next prompt choose `1: R Programming`.

Keep a markdown (or RMarkdown) notebook to record what you learn. **You will be asked to turn your notebook in as part of Assignment 38**. Alternatively, if you learn better by writing rather than typing notes, you can write your notes and submit a photo.

Complete the following 8 tutorials within this course. Remember to **TAKE A BREAK** if you start to feel fuzzy-headed. This is a lot to take in.

When `swirl` asks you *“Would you like to receive credit for completing this course on Coursera.org?”* you can answer *“no”*

1) Basic Building Blocks
(You can Skip 2)
3) Sequences of Numbers
4) Vectors
5) Missing Values
6) Subsetting Vectors
7) Matrices and Data Frames
8) Logic (On this one you can stop after you get to the 52% completion point; **see note below**)
9) Functions (On this one you can stop after you get to the 51% completion point; **see note** below)
(You can skip 10-11 for now)
12) Looking at Data

**Note:** for tutorials 8 and 9, where you can stop at the half-way point, when you are ready to stop, then at an R command prompt(>) type:
```
delete_progress("YOURNAME")
main()
```
Where you replace YOURNAME with whatever name you gave swirl when you started

We will return to some of the other tutorials later.

Complete the `swirl()` tutorials 1, 3, 4, 5, 6, 7, 8 (to 52%), 9 (to 51%) and 12 before continuing

## More about RStudio
R commands are written in R-script (.R) or R-markdown (.Rmd) files. More on those differences below. To start a new R script file you can choose “File > New > R-Script”, or click on the new document icon a the upper left of the window, or type Ctrl-shift-N. This will create a new panel, upper left, in your RStudio window.

There are total of four panes in the RStudio window.

* Upper left: a text editor. You will write your scripts/commands here.
* Lower left: the command line interface, or console. While you can type commands here, I strongly encourage you instead to type them in the text editor mentioned above. Output will appear here.
* Upper Right: This shows objects in your workspace. Think of objects as containers or variables that hold things. More about this later.
* Lower Right: A multipurpose window that can show:
Help
Plots
Files in your directory
## R files types
### R script files
An old-school R analysis, or an R program, would be written as an `R script` (file extension .R). You have just started a blank R-script file above.

R-script files contain R commands and comments.

Begin your script file with some information about what it is. In this manual, code that you should type will generally be shown in a grey block such as:
```
#Simple R Script for GNBT120
#Author: Daniel Koenig
#Original Date: April 17, 2014
#Updated: January 23, 2025
```
Lines that begin with a pound sign # are comment lines and are ignored by R. Type the text above into the script window in the upper left pane (replacing “Daniel Koenig” with your name, changing the date info, and adding any additional information that you think is useful).

Now enter some simple R code into your script:
```
print("Hello World!")
```
Type the text above into the script (upper-left) window. Then with the cursor still on that line, hold down the control key (CMD key on Mac) and press the enter or return key. This causes RStudio to copy the text from your script file, paste it into the console and “execute” the code.

If you wanted to save your script file then you could press `Ctrl-S` or choose “File > Save”

### R Markdown files
Recommended reading: Chapter 27 in R for Data Science.

As an alternative file type you can use an R-flavored version of Markdown, [RMarkdown](http://rmarkdown.rstudio.com/), with the file extension “.Rmd”. Rmarkdown comes pre-installed in RStudio. R markdown is particularly nice for authoring reports, either for your scientific colleagues, or in this case, for *turning in your GNBT120 homework*. One of the nice things about Rmarkdown is that you can “knit” your file and R will run any code in the Rmarkdown and then embed the results in a html or PDF file for sharing. This allows reproducibility in your projects.

Lets try it.

Start a new R Markdown document. Choose “File > New File > R Markdown” or click on the new file icon and choose R Markdown.

When the box opens to ask you questions, add a title of your choosing, add your name as author, and keep “html” as the default output option. Click OK.

A document template opens with some embedded markdown and code.

As noted above, the cool thing about RMarkdown and R Notebooks is that embedded R code chunks are run and incorporated into the output. **This is an ideal way to create reproducible reports and analyses**

To see this for yourself, click the right arrow in the code chunk that contains `plot(pressure)`, or place your cursor in the text there and press `Ctrl+Shift+Enter`.

Voila!

To see how the fully formatted document would look, click the “Knit” button above your file (It may take 20 seconds for a new window to appear but you will then be prompted to enter a file name to save the Rmarkdown file). Note that the html file will only update if you knit your Rmarkdown file again.

You can now remove everything below the lower “—” (line 6) in the R markdown file and enter your own content.

Lets re-create the “Hello World” program above. Enter it into your .Rmd file.
```
# A silly R program
# This is the first program that Professor Maloof asked us to write in R

\\\{r hello_world2}

print("Hello World")

\\\
```
You can still execute R-code lines one at a time with Ctrl-Enter to check your code. But when each chunk is complete you should run the entire chunk with the right arrow or by typing Ctrl-Shift-Enter

I recommend writing separate chunks for each bit of output to be produced.

You can automatically create a new code chunk in Rstudio by typing “Ctrl-Alt-i” or choosing “New Chunk” from the “Chunks” menu (upper right hand side of the text-editor window). (“Ctrl-alt-i” isn’t working for Prof. Maloof from his mac to Rstudio on the instance but it is working for John on his PC).

If you want to produce a PDF or even a Word document (horrors) instead, click the down arrow next to “Knit” and choose an alternative format instead.

Lets review what we know about R Markdown.

* R Markdown is very similar to markdown
* To denote R code, start the code block with \`\`\`{r SomeTitle} and then end it with \`\`\`
* Anything outside of the code block is formatted using normal Markdown rules.
* R studio has a built in rendering engine that will produce html or PDF files.
* A handy cheatsheet and reference guide are available and go into many more options than I have covered here. You can also access these documents from the Help menu within Rstudio.

As noted above, Chapter 27 in R for Data Science is all about R markdown.

An even more [extensive](https://bookdown.org/yihui/rmarkdown/) book is available, written by the creator of Rmarkdown.

Now that you are done with the `swirl` tutorial you should **always** create a Rmarkdown file and type your commands in the upper left window, thereby keeping a record of what you have done.
