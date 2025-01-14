# Bash for loops

## Setup
Make sure you are working in your Assignment_02 respository in RStudio. There should be text at the upper right hand side of the Rstudio window that says `assignment-02-`[your-githubusername].

For this part of the assingment:

* Graded: answer the questions in the `Assignment_2_FORLOOP_template.md` file located in the `scripts` folder.
* Not graded, but recommended: keep notes on what you are learning in the `Assignment_2_lab_notebook.md` file.
## Background
I have claimed that there are tools at the command line that make automating tasks easier. You are probably wondering when you will get to see that. Now is the time.

## Linux For Loops
**Open a Linux terminal and connect to the HPCC cluster**

A **for loop** is a method of iterating over a series of values and performing an operation on each value. All programming languages have some version of this, although the syntax varies.

First a toy example. Let’s say that we have a list of fruit and we want to write a short script that will automatically convert the fruit names to plurals.

Let’s create our items. (Type these commands yourself to get practice)

Pay close attention to the format of the command, there are no spaces on either side of = this is important and required
```
fruit_list="banana apple orange grape plum pear durian pineapple"
```
This creates the variable `fruit_list`

Remember to refer to a variable after it is defined we place a “$” in front of it (and optionally curly brackets)
```
echo "${fruit_list}"
```
**Stop and Think** (not graded) in your own words, write down what the two previous commands have done. Talk it over with your lab mates.

In unix-bash a for loop has four parts

1. A statement saying what items we want to loop over
2. do to note the beginning of the commands that we want to loop through
3. The commands to repeat
4. done to indicate the end of the loop
So if we want to loop through each fruit in $fruit_list and print them one at a time we would write
```
for fruit in ${fruit_list}
  do
    echo "${fruit}"
  done
```
Translation: for each fruit in the list of fruits `$fruit_list` take one value at a time and place it in a new variable `$fruit`. Then run the command `echo $fruit` to print the current fruit. Go back to the top, place the next fruit in the list into `$fruit` and repeat the `echo` command. This will continue until there are no more fruits.

What if we want to add an “s” to make these fruit plural?
```
for fruit in ${fruit_list}
  do
    echo "${fruit}s"
  done
```
The curly brackets are used to help bash distinguish between the variable name `fruit` and the text we want to add `s`

Exercise One: Write a for loop to pluralize peach, tomato, potato (remember that these end in “es” when plural). Put the code you use (formatted as a code block) into your `Assignment_2_FORLOOP_template.md`

*Hint:* I **strongly** recommend typing your code in your markdown file in an editor (e.g. Rstudio) **first**. Then cut and paste into the terminal and see if it works. If it does not work, then you can make a change in Rstudio without having to retype the whole thing.

Confused? There are a bunch of tutorials online. I like [this one](http://www.cyberciti.biz/faq/bash-for-loop/) and [this one](http://www.tutorialspoint.com/unix/for-loop.htm)

## Interacting with files.
I most commonly use for loops when I want to process a list of files.

We can use file names as input or output in `for` loops. Here is another silly example, but hopefully it illustrates the point. Let’s say the goal was to print the output of every file in a directory.

First set it up:
```
mkdir for_example
cd for_example
echo "this" > file1.txt
echo "is" > file2.txt
echo "silly" > file3.txt
```
**Stop and Think:**(not graded) what did that code block do? If you are unsure, what commands can you use to figure it out? (Hint what directory are you in, what files are in the directory, what are the contents of those files?)

Now to use these files in a for loop:
```
myfiles=$(ls)
```
This runs the commands `ls` and places the results in the variable `myfiles`. The `$()` is important and indicates that the `ls` command is to be executed. Note that those are smooth brackets, not curly brackets.
```
echo ${myfiles}         # make sure it really does have a list of files

for file in ${myfiles}
  do
    cat ${file}
  done
```
If you haven’t already done so, run the code above to see what happens.

**Exercise Two** In your own words provide a human “translation” of the above loop.

Note that we could be more shorthand about this and not bother defining `$myfiles`
```
for file in $(ls)
  do
    cat ${file}
  done
```
**Exercise Three** Modify the above loop to produce the following output
`
file file1.txt contains: this
file file2.txt contains: is
file file3.txt contains: silly
`
*hint 1:* you will want the command inside the loop to start with echo. THINK about the difference between `echo` and `cat`
*hint 2:* you will need to use the `$()` inside the for loop, placing a command inside of those parentheses
*hint 3:* if you aren’t sure where to start, try writing it out in pseudo-code. That is, describe what you want to happen.

If you want a bit more of a challenge, try this (optional)
`
file “file1.txt” contains: “this”
file “file2.txt” contains: “is”
file “file3.txt” contains: “silly”
`
## Nested for Loops
Sometimes we want to nest one loop inside of another…

### hour and day example
What if we wanted to print out the working hours of each day in the work week. We would want to loop through the days of the week, and for each day we want to loop through the hours of the day.

Let’s set up our days and hours
```
days="mon tue wed thur fri"
hours="09 10 11 12 01 02 03 04 05"
```
**Exercise Four**

Write code that produces the following output. Be sure to separate each day with an extra line-feed as shown below.
```
It is 09:00 on mon
It is 10:00 on mon
It is 11:00 on mon
It is 12:00 on mon
It is 01:00 on mon
It is 02:00 on mon
It is 03:00 on mon
It is 04:00 on mon
It is 05:00 on mon

It is 09:00 on tue
It is 10:00 on tue
It is 11:00 on tue
```
And so forth, through 5:00 on fri

*Hint:* First write a loop that prints each day of the week once. Then think about how you can add another loop inside of your first one to print the hours for each day.

You will get practice this in the next lab.
