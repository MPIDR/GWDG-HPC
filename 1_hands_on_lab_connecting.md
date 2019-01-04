Hands-on Lab part 1
================

-   [Connecting to the cluster using Windows](#connecting-to-the-cluster-using-windows)
    -   [Connect to the GWDG network](#connect-to-the-gwdg-network)
    -   [ssh to a frontend](#ssh-to-a-frontend)
    -   [Basic Linux commans - a refresher](#basic-linux-commans---a-refresher)
    -   [Exploring and modifying directories](#exploring-and-modifying-directories)
    -   [Permission settings](#permission-settings)
    -   [Working with files](#working-with-files)
    -   [Changing language to English](#changing-language-to-english)
    -   [Editing text files from the terminal](#editing-text-files-from-the-terminal)
    -   [File transfer with WinSCP](#file-transfer-with-winscp)
-   [The queue system](#the-queue-system)
    -   [Submitting jobs to the cluster](#submitting-jobs-to-the-cluster)
    -   [Monitoring and evaluating jobs](#monitoring-and-evaluating-jobs)
-   [Sources](#sources)

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

### Basic Linux commans - a refresher

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
cd per_test
touch file
cd ..
ls -l
```

This will show the current permission settings, giving full control over the directory to the user:

``` bash
...
drwx------ 2 d.alburezgutierrez MRDF  1 Jan  4 12:59 per_test
...
```

To remove the 'write' permissions, simply type:

``` bash
chmod u-w per_test
ls -l

...
dr-------- 2 d.alburezgutierrez MRDF  1 Jan  4 12:59 per_test
...
```

Note that the 'execute' permission is needed to access the directory, as we will do now. By displaying the content of the directory `per_test` we can see that the permission changes were not inherited to `file1`. This behaviour can be changed with the `-R` (recursive) option, although this is generally not advised.

``` bash
ls -l per_test

-rw------- 1 d.alburezgutierrez MRDF 0 Jan  4 12:59 file1
```

### Working with files

In Unix and Linux OS, everything is a file.

Some useful commands are:

| command       | description                              |
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

``` bash
export LANG=en_US.UTF-8
```

### Editing text files from the terminal

The terminal is the only way of interacting with the GWDG cluste. This means that Graphical User Interface (GUI) are (with a few exceptions) not available. These text editors that can be launched within the terminal: `vi`, `mcedit`, `joe`, `nano`.

In this tutorial we will use `nano`. To run it, simply type `nano {filename}` on the terminal. The `{filename}` can be omitted if you wnat to create a new file from within nano. Let's try it out.

``` bash
touch test
nano test
```

Write something to save in your file, for example the unimaginative: 'Hello world!', and press Ctrl+x to exit (and save). We can now check that the changes were saved by printing the file:

``` bash
cat test
Hello World!
```

### File transfer with WinSCP

In this section we will learn how to upload files to the cluster with an SCP client for Windows. WinSCP isan open-source client available from the internet and MPIDR intranet.

This is a two-step processs - please note that if you skip any of the steps, it will not work. \#\#\#\# In Putty

We need to establish a new ssh connectino to the cluster that will allow us to transfer files.

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

The queue system
----------------

``` bash
```

``` bash
```

``` bash
```

### Submitting jobs to the cluster

### Monitoring and evaluating jobs

Sources
-------

-   Boehme, C. and Ehlers, T. (2018) Using the GWDG Scientiﬁc Compute Cluster - An Introduction. Goettingen: GWDG. [\[link\]](https://info.gwdg.de/docs/lib/exe/fetch.php?media=en:services:scientific_compute_cluster:parallelkurs.pdf)
-   MPIDR IT Wiki: *how to access the GWDG compute cluster*
-   MPIDR IT Wiki: *Parallel computing examples in R at GWDG*
