# Connect to RStudio and setup git authentication
You will use Git to store your work and to turn in your assignments. Git does not allow you to use your password to push/pull your repositories. Instead, we will use an [ssh public key](https://help.ubuntu.com/community/SSH/OpenSSH/Keys) to authenticate your identity.

In this method of authentication you generate a public key and a private key. The private key stays on your computer (or on the compute cluster) and the public key is given to 3rd parties who will want to verify your identity (in this case GitHub).

When you attempt to login to GitHub a program called [SSH](https://help.ubuntu.com/community/SSH) tests to see if your computer has the matching private key.

## Create a GitHub Account
If you do not have an account, please go to [GitHub](https://github.com) and create an account.

## Generate a ssh key pair
You can either generate a ssh key pair at the command line or in Rstudio. For this class, we will use Rstudio.

## Open Rstudio on the HPCC
You will conduct a good portion of your analysis using an RStudio instance using UCR's High-Preformance Computing Center (HPCC). You can find a lot of additional information about the HPCC [here](https://hpcc.ucr.edu). We will discuss the HPCC a lot more later in the course so don't worry to much about the details at this point. 

1. Begin by connecting to [RStudio](https://rstudio.hpcc.ucr.edu) on the HPCC. Log-in with your HPCC credentials.
2. Choose `Tools` > Global options from the pull-down menu.
3. From the options box, click on `Git/Svn` on the left hand tab side
4. Click `Create SSH Key`.
5. There is no need to setup a passphrase. Newer versions of OS might try to enter a passphrase here, click on `Other Options` and select `Choose My Own Password` and leave it blank.
6. Click `Create`
7. Click `Close`
8. Click `View Public Key`
9. Press ctrl+c to copy the key to your clipboard.

## Add your public key to github
Go to github.com and login to your account

Click on the your profile icon near the upper right hand side and then select settings.

Click on `SSH and GPG` keys on the left hand side

Click on `New SSH Key`, upper right hand side

Enter a name for your key, paste in your public key, then press `add SSH key`

## Test the connection
Open a linux terminal (not R) and type:
```
ssh -T git@github.com
```
You may get a warning. Go ahead and type yes. You should then get a message that you have successfully authenticated.

## Clone repositories using ssh
In order to use the public key / private key authentication you must clone your repositories using `ssh` instead of `https`.

I will provide an example of what this means in the next document “Turning in Assignments”

