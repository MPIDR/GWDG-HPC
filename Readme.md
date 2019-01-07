Hands-on lab (First Part)
================

-   [Connecting to the cluster using Windows](#connecting-to-the-cluster-using-windows)
    -   [Connect to the GWDG network](#connect-to-the-gwdg-network)
    -   [ssh to a frontend](#ssh-to-a-frontend)
-   [Working with files and directories](#working-with-files-and-directories)
    -   [Basic Linux commans - a refresher](#basic-linux-commans---a-refresher)
    -   [Changing language to English](#changing-language-to-english)
    -   [Exploring and modifying directories](#exploring-and-modifying-directories)
    -   [Permission settings](#permission-settings)
    -   [Editing text files from the terminal](#editing-text-files-from-the-terminal)
-   [File transfer with WinSCP](#file-transfer-with-winscp)
    -   [In Putty](#in-putty)
    -   [In WinSCP](#in-winscp)
-   [The cluster and queue system](#the-cluster-and-queue-system)
    -   [File storage in the cluster](#file-storage-in-the-cluster)
    -   [Processing jobs on the cluster](#processing-jobs-on-the-cluster)
    -   [a. The job script](#a.-the-job-script)
    -   [Example: submitting jobs to the queue](#example-submitting-jobs-to-the-queue)
-   [Sources](#sources)
-   [Next on...](#next-on...)

Connecting to the cluster using Windows
---------------------------------------

### Connect to the GWDG network

Connecting to a GWDG frontend is a two-step process if you are working from outside the GWDG network (e.g. from the MPIDR or from home). First, you need to first establish a connection to the GWDG network. For this

1.  Open Putty (available from the intranet)
2.  Host Name: login.gwdg.de; Port: 22; Connection Type: SSH
3.  Acknowledge Putty's security alert by clicking on 'Yes'
4.  Enter your GWDG username (usually **firstname.lastname**, but you can also find it by logging into [your GWDG account](https://www.gwdg.de/))
5.  Enter your MPIDR password (no asterix will appear as you type) and press enter

You are now logged into the GWDG network. Congrats! However, you are not yet connected to any of the three frontends (gwdu101.gwdg.de/, gwdu102.gwdg.de/, or gwdu103.gwdg.de/). To do this we must open a new ssh connection.

### ssh to a frontend

Since the GWDG server uses Ubuntu (16.04.5 LTS), we cannot use Putty for this. We must establish the ssh connection "the Linux way".

1.  In the the Putty terminal, type:

``` bash
ssh gwdu102.gwdg.de

The authenticity of host ’gwdu101.gwdg.de (134.76.8.101)’ can’t be established.
ECDSA key fingerprint is SHA256:sIJNEepmILeEq/7Zqq4HCtpTM8L98arWTny5EiAX+gI.
or ECDSA key fingerprint is 7c:52:2b:17:f8:ba:29:bd:c5:45:d1:1a:9e:8d:d6:f0.
or RSA key fingerprint is b9:f9:46:0f:23:c8:8d:76:b9:83:b9:1b:f6:5e:d5:6b.
Are you sure you want to continue connecting (yes/no)?
```

1.  Accept the connection by typing 'yes'
2.  Enter your GWDG userID (if prompted) and your password

Welcome to the the gwdu102.gwdg.de frontend of the GWDG cluster. If you see something like this, you are now ready to write Linux commands in the terminal:

``` bash
gwdu102:5 10:37:08 ~ >
```

Working with files and directories
----------------------------------

### Basic Linux commans - a refresher

These are some useful Linux commands that we will use in the next section to create, explore, and edit files from the terminal:

| Command       | Description                              |
|---------------|------------------------------------------|
| pwd           | show current directory                   |
| cd            | change directory                         |
| top           | display processes as a sorted list       |
| ps            | display current process                  |
| touch         | create file                              |
| cat           | print file content                       |
| cp            | copy file                                |
| rm            | remove file                              |
| mv            | move file                                |
| mkdir         | new directory                            |
| rmdir         | remove directory                         |
| ln            | create (hard or symbolic) link to a file |
| df (-h) (-hl) | show disk space                          |
| chmod         | change file attributes                   |

### Changing language to English

By default, the cluster operates in German. You can check this by typing:

``` bash
echo $LANG
```

You can change this for the current session:

``` bash
export LANG=en_US.UTF-8
```

For a permanent solution, add the command to the `.profile` file with:

``` bash
echo ’export LANG=en_US.UTF-8’ >> ~/.profile
```

### Exploring and modifying directories

To list the files in your current directory, type `ls`

``` bash
ls
bin exercises familinx intel Mail
```

To see more details, type \`ls -la'

``` bash
ls -la

drwx------   2 d.alburezgutierrez MRDF     0 Sep 25 10:42 bin
drwx------   3 d.alburezgutierrez MRDF     0 Oct 23 14:09 exercises
drwx------   4 d.alburezgutierrez MRDF     0 Oct 23 14:15 familinx
drwx------   3 d.alburezgutierrez MRDF     0 Oct 23 12:02 intel
drwx------   2 d.alburezgutierrez MRDF     0 Sep 25 10:42 Mail
```

### Permission settings

The previous command showed a series of *persmission flags* associated with each file in our current directory.

The first character indicates the special permission flag, that can either be `d` for directory, `-` for a normal ﬁle, or `l` for a symlink. We will not focus on this now. More important are the other positions:

| Position | Permission | User type         |
|----------|------------|-------------------|
| 2        | read       | User (file owner) |
| 3        | write      | User (file owner) |
| 4        | execute    | User (file owner) |
| 5        | read       | Group             |
| 6        | write      | Group             |
| 7        | execute    | Group             |
| 8        | read       | Others            |
| 9        | write      | Others            |
| 10       | execute    | Others            |

New files and directories are created by default with the following permission settings: `-rw-------`, meaning that only the owner can read and write. To alter file permissions we can use the `chmod` command. The basic usage is:

``` bash
chmod {options} filename
```

There are two ways of changing permissions: by specifying the `{options}` with numbers or with letters. In this tutorial we will only cover letters, as this is more intuitive. The GWDG presentation referenced at the end of this document provides a more detailed description of the use of `chmod` for alterning permission settings.

| chmod Options | Definition        |
|---------------|-------------------|
| u             | owner             |
| g             | group             |
| o             | other             |
| a             | all               |
| x             | execute           |
| w             | write             |
| r             | read              |
| +             | add permission    |
| -             | remove permission |
| =             | set permission    |

Let's see an exmaple. First we will create an empty directory, check it's current permission settings, and then edit these to make the directory 'ready-only' (i.e. remove 'write' permissions).

``` bash
mkdir per_test
ls -l
```

This will show the current permission settings, giving full control over the directory to the user:

``` bash
...
drwx------ 2 d.alburezgutierrez MRDF  1 Jan  4 12:59 per_test
...
```

As an exercise, we will create `file1` within our new directory `per_test` and save some text to it:

``` bash
cd per_test
touch file1
echo GWDG cluster workshop >> file1
ls -l
cd ..
```

To remove the 'write' permissions of the directory, simply type:

``` bash
chmod u-w per_test
ls -l

...
dr-x------ 2 d.alburezgutierrez MRDF  1 Jan  4 12:59 per_test
...
```

It is now impossible to create new files within `per_test` as we did before:

``` bash
touch file_error /per_test

touch: cannot touch ‘/per_test’: Permission denied
```

However, we can still edit the existing `file1`:

``` bash
echo This is a new line >> per_test/file1
cat per_test/file1

GWDG cluster workshop
This is a new line
```

By displaying the content of the directory `per_test` we can see that the permission changes were not inherited to `file1`. This behaviour can be changed with the `-R` (recursive) option, but this is generally not advised. Note that the 'execute' (`x`) permission is needed to access the directory, as we will do now.

``` bash
ls -l per_test

-rw------- 1 d.alburezgutierrez MRDF 0 Jan  4 12:59 file1
```

Permissions are useful for granting and restricting access to specific directories and files in a way that will be familiar to users of the MPIDR's `N:` drive. However, note that the cluster administrators have access to all directories stored in the cluster. Please do not store sensitive information in the cluster without first getting in touch with the GDGW security team.

### Editing text files from the terminal

The terminal is the only way of interacting with the GWDG cluster. Graphical User Interface (GUI) are, with a few exceptions, not available. Text files can be edited from within the terminal using editors such as: `vi`, `mcedit`, `joe`, or `nano`.

In this tutorial we will use `nano`. To run it, simply type `nano {filename}` on the terminal. The `{filename}` can be omitted if you want to create a new file from within nano. Let's try it out.

``` bash
touch test
nano test
```

Write some text to save in your file and press `Ctrl`+`x` to exit (and save). We can now check that the changes were saved by printing the file:

``` bash
cat test
Hello World!
```

You can see all the `nano` keyboard shortcuts by pressing `Ctrl`+`G`.

![](resources/nano_help.png)

File transfer with WinSCP
-------------------------

In this section we will learn how to upload files to the cluster with an SCP client for Windows. WinSCP isan open-source client available from the internet and MPIDR intranet.

This is a two-step processs - please note that if you skip any of the steps, it will not work.

#### In Putty

We first need to establish a new ssh connectino to the cluster that will allow us to transfer files.

1.  Open Putty and change the following settings
2.  Host Name: `transfer.gwdg.de`; Port: `22`; Connection Type: `SSH`
3.  **Connection** | **Data**: Auto-login username: **firstname.lastname**
4.  **Connection** | **SSH** | **Tunnels**: Source port `4022`, Destination: `transfer-scc.gwdg.de:22`
5.  Save session and open it (MPIDR password)

#### In WinSCP

Create a new connection with the following settings:

1.  File protocol `SCP`, Host name `localhost`, Port number `4022`, User name **firstname.lastname**
2.  Login with your MPIDR password

To change to the /scratch file system, click in the remote windows (right), press Ctrl+O, enter /scratch and press Enter.

To create a subfolder, click in the remote windows (right), press F7, enter the folder name (e.g. mickey.mouse) and set the permissions as necessary.

The cluster and queue system
----------------------------

For submitting jobs to the cluster it is necessary to create a `shell file` using a `batch system` through LSF commands. So, the GWDG cluster is operated by the LSF platform, which is operated by shell commands on the frontends. The **frontends** are special nodes (gwdu101, gwudu102, and gwdu103) provided to interact with the cluster via shell commands.

The batch system distributes the processes across job slots, and matches the job's requirements to the capabilities of the job slots. Once sufficient suitable job slots are found, the job is started. LSF considers jobs to be started in the order of their priority.

It is also necessary to know how the cluster is structured:

![](resources/GWDGarchitecture.PNG)

Source: \[GWDG documentation)(<https://info.gwdg.de/dokuwiki/doku.php?id=en:services:application_services:high_performance_computing:running_jobs>)

### File storage in the cluster

-   $HOME: our home directory is available everywhere, permanent, and comes with backup, but it is comparatively slow.

-   /scratch: this is the shared scratch space, available on *gwda, gwdc*, and *gwdd* nodes and on the frontends *gwdu101* and *gwdu102*. For being sure of having a node with access to shared */scratch* the command `-R scratch` must be written in the shell file.

-   /scratch2: this is the shared scratch space, available on *dfa, dsu, dge*, and *dmp* nodes, and on the frontend *gwdu103*. For being sure of having a node with access to shared */scratch2* the command `-R scratch2` must be written in the shell file.

### Processing jobs on the cluster

Jobs are usually not run *interactively* in the GWDG cluster. Compute-intensive jobs are submitted to specific *queues* of the cluster. These jobs are not processed immediately, but rather *stand in the queue* until the necessary resources are made available for the cluster to run them (for more info, see the [official documentation](https://info.gwdg.de/dokuwiki/doku.php?id=en:services:application_services:high_performance_computing:running_jobs)).

Submitting a job is a two-step process consisting of

1.  Creating a job script
2.  Submitting the job script to a GWDG queue for execution

### a. The job script

Job scripts are shell scripts with special comment sections (`#BSUB`) that force each line to be interpreted as an option of `bsub`. The job files can be created through either the linux terminal using the editor *nano* or Notepad++ in Windows.

A typical job script looks something like this:

``` bash
#!/bin/sh 
#BSUB -N 
#BSUB -u <email address>
#BSUB -q <queue to submit job to> 
#BSUB -W <max runtime in hh:mm>
#BSUB -o <file path> 
 
<Command to execute>
```

#### b. Submitting job scripts with the `bsub` command

The `bsub` command submits information regarding your job to the batch system. The basic syntax:

``` bash
bsub <bsub options> [mpirun.lsf] <path to program> <program parameters>
```

#### Some `bsub` parameters

-   -N: sends the job report to you by email when the job finishes.
-   -u: sends the email to the specified email destination.
-   -W: sets the maximum runtime limit of the job. If this time is exceeded the job is killed.
-   -o: appends the standard output of the job to the specified file path.

All the parameters of `bsub` can be specified via the terminal but this results in very long commands. Job scripts can be used insted to specify all relevant parameters of `bsub`.

#### Specifying node properties with *-R*

`-R` runs the job on a host that meets the specified resource requirements. Some of its basic parameters are:

-   span\[hosts=1\]: this puts all processes on one host.
-   span\[ptile=&lt; x &gt;\]: *x* denotes the exact number of job slots to be used on each host. If the total process number is not divisible by *x*.
-   scratch(2): the node must have access to *scratch(2)*.

### Example: submitting jobs to the queue

In this example, we will submit a simple R script to the `XX` queue of the cluster. We first create the `sample_code.R` using `nano` (copy and paste the R commands).

``` bash
nano sample_code.R

--Copy and paste this text-------
#!/usr/bin/env r 
data("airquality")
print(head(airquality, 6))
-------------------------------
```

We now create a simple job script. Edit the text below before copying and pasting it. Note that the `sample_code.R` is preceded by the `Rscript` command, which is used to run R scripts non-interactively.

``` bash
nano job_script.sh

--Copy and paste this text-------
#!/bin/sh 
#BSUB -N 
#BSUB -q mpi 
#BSUB -o NameOutput.%J.txt 
 
Rscript sample_code.R
-------------------------------
```

Once the file has been created, we submit the job to the `XX` quede using the `bsub` command:

``` bash
bsub < job_script.sh
```

#### Other job-processing LSF commands

-   bjobs: lists currents jobs.
-   bhist: lists older jobs.
-   lsload: status of cluster nodes.
-   bqueues: status of cluster nodes.
-   bhpart: shows current user priorities.
-   bkill <jobid>: stops the current job.

Sources
-------

-   Boehme, C. and Ehlers, T. (2018) Using the GWDG Scientiﬁc Compute Cluster - An Introduction. Goettingen: GWDG. [\[link\]](https://info.gwdg.de/docs/lib/exe/fetch.php?media=en:services:scientific_compute_cluster:parallelkurs.pdf)
-   GWDG/DOCS: Running Jobs [\[link\]](https://info.gwdg.de/dokuwiki/doku.php?id=en:services:application_services:high_performance_computing:running_jobs)
-   MPIDR IT Wiki: *how to access the GWDG compute cluster*
-   MPIDR IT Wiki: *Parallel computing examples in R at GWDG*

Next on...
----------

Click [here](https://github.com/Parimucyeran/Parallel_Course/blob/master/Github.md) for the Second Part of the Hands-on lab!
