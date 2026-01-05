# Preliminaries
This course will introduce you to interacting with you computer using the command line. Why use the command line when you can use a graphical user interface (GUI) to do some of the same things?

The command line provides substantially more flexibility to modify a program's behavior in a number of ways with minimal effort. It also makes it easy to iterate a process thousands of times very easily, which is crucial when dealing with large datasets found in bioinformatic studies.

# Connecting to the HPCC
The High Preformance Computing Center Cluster is a shared research computing system availiable at UCR to researchers and to students in classes that require computing resources. I will discuss this resource in more detail in lecture, and you can also read an [introduction to the HPCC](https://girke.bioinformatics.ucr.edu/GEN242/tutorials/linux/linux/) and explore the detailed [HPCC manuals](https://hpcc.ucr.edu) for more information.

I have already created accounts for you on the HPCC, and we want to make sure you can connect using your UCR netID. There are several ways to do this outlined below, we will usually use the HPCC OnDemand method but others are put here for your information.

# Connect to HPCC OnDemand
Most of the work you will do for this course will be mediated by the [HPCC OnDemand web interface](https://ondemand.hpcc.ucr.edu/). Using this interface you can start a virtual desktop and and Rstudio instance on the cluster.

1. Follow the link and log in with your UCR credentials.
2. Once logged in you should see something that looks like this:
![](/GNBT120/assets/images/prelim1.png)
3. Click on the `Interactive Apps` dropdown
![](/GNBT120/assets/images/prelim2.png)
4. Select `HPCC Desktop`
![](/GNBT120/assets/images/prelim3.png)
5. You will now see a page that allows you to request computational resouces. The more resouces you request, the longer it will take to start your session.
![](/GNBT120/assets/images/prelim4.png)
6. `Number of cores` requests the number of computers you will be using for the session. Much of the time one core will work for your HPCC Desktop sessions. However, you will use [multithreading](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)) in some of your work and you will need to request more cores depending on the number of threads you will use. `Change this value to 1`
![](/GNBT120/assets/images/prelim5.png)
7. `Memory in GB` requests the amount of memory to allocate to the job. Usually 8GB will be fine.
![](/GNBT120/assets/images/prelim6.png)
8. `Job runtime` is how long you want to keep this session running. Once the time runs out, the session will end and any unsave work will be deleted. Since the format is: Days-hours:minutes:seconds. Let's set it for 6 hours:
![](/GNBT120/assets/images/prelim7.png)
9. The `Partition` drop down sets which sets of computational nodes we would like to work on. Different parts of the cluster can be optimized for different purposes or can give different priorities for users. You can think of these as different lines in the check out stand. For example, very short jobs with low resources (less than 2 hours) will get priority in the queue if you choose the `short` partition. Today, choose `batch`.
![](/GNBT120/assets/images/prelim8.png)
10. The `Additional Slurm Arguements` section can alter the behavior of the virtual desktop in many ways. Don't worry about this for now.
11. Click the `Launch` button
![](/GNBT120/assets/images/prelim9.png)
12. You should now be directed to the `My Interactive Sessions` view. Click `Launch HPCC Desktop`.
![](/GNBT120/assets/images/prelim10.png)

# Contratulations you are now connected to the HPCC!
You should see something like this in your browser:
![](/GNBT120/assets/images/prelim11.png)

This is a virtual desktop that you have started on the HPCC. 

# Using a terminal emmulator
In this course we will interact with the Unix/Linux operating system using a terminal emmulator.

Start a terminal that will allow you to interact with the `command line`. 
1. Click on the `Terminal` app on the bottom of the desktop:
![](/GNBT120/assets/images/prelim12.png)
2. You should see something like this:
![](/GNBT120/assets/images/prelim12.png)
3. If this is your first time logging in, you need to reset your password. Type `passwd`, and then follow the instructions. **MAKE SURE TO KEEP TRACK OF YOUR PASSWORD!**
4. Once you are done, you can log off by typing `exit`.

# Using a terminal emmulator on your own computer *not necessary for lab 1*
It is possible to  emmulate the terminal from your computer directly rather than from the virtual desktop on the HPCC. If you need to do this follow these instructions.

## Find your terminal emmulator application
1. On macintosh this software is preinstalled and called `Terminal`.
2. On Linux operating systems common terminal editors include `GNOME` and `xterm`.
3. On Windows, you will install a software to work in a Unix/Linux environment. There are a number of ways to accomplish this, but I suggest installing [cygwin](https://www.cygwin.com) for this course.

## Connect to the HPCC
1. Open you terminal application. You should see something that looks a bit like this:
![](/GNBT120/assets/images/prelim1.png)
2. Enter the following substituting your netID in for NETID:
```
ssh -X NETID@cluster.hpcc.ucr.edu
```
3. You will be prompted to enter your password. When you type the cursor will not move and you will not see anything printed.
4. You will then be asked to authenticate via Duo by entering `1` and `return` (like when you connect to other UCR resources).
5. If login was successful you will see something like this:
![](/GNBT120/assets/images/prelim2.png)

6. Finally, if this is your first time logging in, you need to reset your password. Type `passwd`, and then follow the instructions. **MAKE SURE TO KEEP TRACK OF YOUR PASSWORD!**
7. Once you are done, you can log off by typing `exit`.
