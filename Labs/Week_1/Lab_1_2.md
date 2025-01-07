# Connecting to RStudio and turning in projects
## Attribution
This lab was originally developed by Julin Maloof and modified by Daniel Koenig

## Overview
We will use git to turn in assignments. I will post a link for each assignment. The link will ask you to logon to GitHub and join the GNBT 120 classroom.

For each assignment click on the link and follow instructions below to create your own repository for the assignment.

Remember to use Rstudio or the command line to `git add` files that you create or change, `git commit` your changes and `git push`. **DO THIS VERY OFTEN!** That way if something goes wrong you won’t have lost any work.

Make your final pushes before the assignment due date.

You can check your repo on GitHub.com to make sure that your latest changes have been uploaded.

## Projects in RStudio
Rstudio allows you to work in projects to help you keep your files organized. Furthermore it allows these projects to be associated with a github repository. This is how we will work on each assignment in this class. Below I show you how to create an Rstudio project for Assignment 01 by creating and cloning your Assignment 01 repo.

### Example: Assignment 1
#### Accept the assignment link to create a Gihub repo
1. On the [assignments page](https://dpkoenig.github.io/GNBT120/assignments), click on the assignment 1 link, click on the `Accept This Assignment` button
2. Wait until you see a link to your repo then click on the link to go to your repository for this assignment.
![](/GNBT120/assets/images/Lab_1_2_1.png)

## Clone your Assignment repo to the HPCC using Rstudio
The next step is to clone it to your instance using Rstudio.
1. Open a new browser window and then navigate to [Rstudio on the HPCC](https://rstudio.hpcc.ucr.edu) and log in as described in `Lab_1_1`.
2. Click on `Project` on the upper right-hand side and then `New Project`
![](/GNBT120/assets/images/Lab_1_2_2.png)
3. Next click on `Version Control`
![](/GNBT120/assets/images/Lab_1_2_3.png)
4. Click on `Git`
![](/GNBT120/assets/images/Lab_1_2_4.png)
5. Return to the GitHub assignment page. Click on the `Code` button.
![](/GNBT120/assets/images/Lab_1_2_5.png)
6. Make sure that you click the SSH tab, then copy the link below.
![](/GNBT120/assets/images/Lab_1_2_6.png)
7. Return to the RStudio tab, then paste in the SSH url into the `Repository URL` spot. Then click the `Browse` button.
![](/GNBT120/assets/images/Lab_1_2_7.png)
8. Navigate to `bigdata`
![](/GNBT120/assets/images/Lab_1_2_8.png)
9. Click `New Folder` and create a new folder (I will call this folder Assignment_1)
10. Click `Choose`
11. Now click `Create Project`

Done! You now have an Rstudio Project for Assignment 01 and it is linked to your github repository.

Note: The next time that you open RStudio you may have to tell it to open your project. Click on the `Project` button on the upper right corner and then choose your project.

## Git from Rstudio
Now that RStudio knows that you are working with a git repository you can use its built-in tools to add, commit, push, and pull.

Summary:

* Click on the “Git” tab in the upper right-hand pane
* Untracked files are shown with a “?”
* Files that have been modified since the last commit are shown with a “M”
* Files that are staged to be added are shown with an “A”
* If you want to stage a new or changed file for a commit click on the checkbox
* You can then click on “commit” to open up a new window that shows you the changes
* Type a commit message in the box and press commit!
* Now press the push button to push you changes up to github.
* There is also a pull button to pull changes down to your instance or local computer.
