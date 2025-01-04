# Introduction to Markdown
[Markdown](https://en.wikipedia.org/wiki/Markdown) is a simple system for formatting text. It is easy to type, human readable, and easy to format to html or PDF. Almost all of the documents generated for this lab class are written in markdown.

We will require that your lab reports be generated in markdown or the variant [Rmarkdown](https://rmarkdown.rstudio.com)

Why are we so excited about markdown? Markdown makes it fantastically easy to generate good looking documents that adhere to **three important principles for good coding**:

1. Work should be well documented
2. Work should be reproducible
3. Files should be saved in open (non-proprietary) formats

## Exercise 1: A markdown tutorial
Begin by completing this [tutorial](https://commonmark.org/help/tutorial/)

You may find this [quick reference](https://commonmark.org/help/) handy.

## A few extras
The tutorial covers almost all of the most important markdown features but leaves out one important feature and an additional nice feature.

### Extra 1 (important): Tables
You can make tables in markdown:

The basic format is like this:
```
| Organism |  Genome Size | Number of Chromosomes|
|:---------|:----------:|--------:|
| Arabidopsis | 135 million bp | 5      |
|  _Homo sapiens_ | 3.2 billion bp  |  23  |
```
which produces
| Organism |  Genome Size | Number of Chromosomes|
|:---------|:----------:|--------:|
| Arabidopsis | 135 million bp | 5      |
|  _Homo sapiens_ | 3.2 billion bp  |  23  |

You use pipes `|` to denote the columns, and separate the headers from the data with a row of `|--|`

You can use colons in the header separating row to left or right (or center) justify. Look at columns 1 vs 2 vs 3 in the table.

Note that even if the spacing in your table is uneven (pipes don’t line up) the table still looks good in the output. (But try to make it look nice even in the unformatted text)

For more details, see [here](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#tables). For a tool to help you generate a table without all of that finicking typing, see [here](https://www.tablesgenerator.com/markdown_tables)

### Extra #2: syntax highlighting
As you know from the tutorial, if you want to show a block of code, then you should “fence” it with three backticks ```

like this:

\`\`\`
```
code goes here
```
\`\`\`

Doing so turns:
for v in ls -p /Volumes | grep BIS180L do echo “copying to $v” cp -r ~/Desktop/BIS180L2022 “/Volumes/$v” & done

(ugly!)

into

```
for v in `ls -p /Volumes | grep BIS180L`
  do
    echo "copying to $v"
    cp -r ~/Desktop/BIS180L2022 "/Volumes/$v" &
  done
```
I know that both look like gibberish to you now, but you have to admit that the second way of displaying it results in much nicer gibberish!

The **extra** is that (at least in some editors), if you indicate the language that your code is written in, you can get syntax highlighting. Do this by giving the language after the first set of back ticks like so:

\`\`\`bash
```bash
for v in `ls -p /Volumes | grep BIS180L`
  do
    echo "copying to $v"
    cp -r ~/Desktop/BIS180L2022 "/Volumes/$v" &
  done
```
\`\`\`

## Exercise Two
Use the Rstudio editor to write a brief biography of a lab partner in Markdown.

Although Rstudio is designed for R, we can also use it to edit simple markdown files.

We will give you a proper introduction to Rstudio in a couple of weeks. For markdown files, do the following.

* Open [Rstudio on the HPCC](https://rstudio.hpcc.ucr.edu)
* In the lower-right pane, click on +Folder and then create a new folder “Assignment_01” New Folder
* Click on the the Assignment_01 folder that you just created so that the Rstudio file browser is now in that folder.
* Cick on + Blank File and then select text file. New File
* When prompted for a name, name it Biography.md. By adding .md to the end, Rstudio knows to treat this as a Markdown file. New File2
* Now you can edit your file in the upper-left pane. If you want to take a look at your formatting, click on the Preview button. (If you click on it now, you will get a blank page since you haven’t typed anything yet) New Bio
Include:

* a header/title
* his or her name
* their major (in bold type)
* their year (in italics)
* A table. This could be of schools attended or places lived. You would have columns for start year, end year, and location. Or make a different table
* a bulleted list of likes
* a link that to a webpage that is relevant to something in their biography.
* Save the final file. You will be asked to turn it in a later class.

Check the markdown formatted file to make sure that it looks as you intend

