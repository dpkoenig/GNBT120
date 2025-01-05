# Preliminaries
This course will introduce you to interacting with you computer using the command line. Why use the command line when you can use a graphical user interface (GUI) to do some of the same things? 

The command line provides substantially more flexibility to modify a program's behavior in a number of ways with minimal effort. It also makes it easy to iterate a process thousands of times very easily, which is crucial when dealing with large datasets found in bioinformatic studies.

# Using a terminal emmulator
In this course we will interact with the Unix/Linux operating system using a terminal emmulator. 

1. On macintosh this software is preinstalled and called `Terminal`.
2. On Linux operating systems common terminal editors include `GNOME` and `xterm`.
3. On Windows, you will install a software to work in a Unix/Linux environment. There are a number of ways to accomplish this, but I suggest installing [cygwin](https://www.cygwin.com) for this course.

# Connecting to the HPCC
The High Preformance Computing Center Cluster is a shared research computing system availiable at UCR to researchers and to students in classes that require computing resources. I will discuss this resource in more detail in lecture, and you can also read an [introduction to the HPCC](https://girke.bioinformatics.ucr.edu/GEN242/tutorials/linux/linux/) and explore the detailed [HPCC manuals](https://hpcc.ucr.edu) for more information.

We have already created accounts for you on the HPCC, and we want to make sure you can connect using your UCR netID using the following steps.
1. Open you terminal application. You should see something that looks a bit like this:

2. Enter the following substituting your netID in for NETID:
```
ssh -X NETID@cluster.hpcc.ucr.edu
```
3. You will be prompted to enter your password. When you type the cursor will not move and you will not see anything printed.
4. You will then be asked to authenticate via Duo by entering `1` and `return` (like when you connect to other UCR resources).
5. If login was successful you will see something like this:

6. Finally, if this is your first time logging in, you need to reset your password. Type `passwd`, and then follow the instructions. **MAKE SURE TO KEEP TRACK OF YOUR PASSWORD!**
7. Once you are done, you can log off by typing `exit`.
