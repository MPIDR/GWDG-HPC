# How to use the GWDG Scientific Compute Cluster
**Note 2026-01-08**: the next section is from the 2019 workshop on how to use the GWDG SCC. Some information in part one is out of date. We have highlighted where this is the case and have updated some of the information. Part two is largely still relevant and applicable. Only the specific commands and parameters for submitting jobs to the cluster have changed.

## Tutorial for lab session (January 2019)

Welcome to the practical lab of the **MPIDR Introduction to the GWDG computer cluster** workshop! In this repository, you will find:

1.  [First part: Connecting to the cluster and submitting jobs](part1.md)
2.  [Second part: A practical introduction to parallelization](part2.md)

You can get additional help from the [GWDG documentation](https://info.gwdg.de/dokuwiki/doku.php?id=en:services:application_services:high_performance_computing:start) and the [MPIDR Intranet IT FAQ](https://intranet.demogr.mpg.de/home/support/it/faq).

**Survey**

You can complete a survey for the practical lab here: https://www.surveymonkey.de/r/337FFL7

**Contact**

Diego Alburez: alburezgutierrez\[at\]demogr.mpg.de

Sofia Gil: gil\[at\]demogr.mpg.de

## 2026 Update
If you are interested in using the GWDG Scientific Compute Cluster (SCC) for the
first time or if you have used it before but are wondering what changed during 
2025, read on.

We will cover these basic steps:

1. How do I get an account?
2. How do I set up an SSH connection?
3. What do I need to do before I can do real work?
4. How do I do real work?

**New users**: check if GWDG offers the courses "Using the GWDG Scientific Compute Cluster"
or "Supercomputing for Every Scientist" anytime soon [I think they're the same course under different
labels]: [GWDG courses](https://academy.gwdg.de/). They offer them every other month. It is a one-day
introduction to everything covered here. Their courses are very well structured and hands-on.

**Returning** ("legacy") users: all sections contain some new information. We tried
to make it easy to scan for new information but since this is already a short 
document, we recommend that you read all of it. 

The GWDG HPC documentation is here: <https://docs.hpc.gwdg.de>. However, it is
still incomplete. The old documentation still exists: [https://docs.gwdg.de](https://docs.gwdg.de/doku.php?id=en:services:application_services:high_performance_computing:start).
It contains some information that is still relevant and is not included in the
new documentation but it also contains outdated information. This document is
a condensed version of relevant information for MPIDR researchers.


**Note on the 2024/25 transition period**: Substantial changes to the compute
infrastructure are under way and both the infrastructure and documentation are
currently in a state of flux (2025-02-12). In the future, only the `scc-cpu` and
`scc-gpu` partitions will be relevant to MPIDR users and all documentation will
be consolidated under <https://docs.hpc.gwdg.de>.


### How do I get an account?
**Returning users**: you need a new account too!

**Returning users**: your old account will continue to work. It is called
a "legacy" account in the GWDG documentation. You will be restricted to the
"medium" partition. To get access to more resources, you need to use your new
user profile that is part of the MPIDR project. You can find more information
[here](https://docs.hpc.gwdg.de/start_here/user_disambiguation/index.html).

Whether you are a new or returning user, you need to email
<ithelpdesk@demogr.mpg.de> and ask to be added to the GWDG HPC Project.

You will receive an email from "GWDG HPC Project Portal" (or something similar).
That email will tell you your (new) user name.

If you are a new users, you should check whether you can connect to Academic
Cloud: <https://academiccloud.de/>. The credentials should be the same as for 
your MPIDR accounts. You may have to set up multi-factor authentication.

### How do I set up an SSH connection?
You will usually connect to the SCC via the command line. At the very least, the
initial setup requires you to do so.

If you are on Windows, you should install PuTTY from the intranet and refer to 
the relevant section in the GWDG/Github documentation linked below. But what you
should really do is install Cygwin or the Windows subsystem for Linux and get
familiar with basic command line tools.

For more detail on SSH (esp. under Windows): [GWDG documentation](https://docs.hpc.gwdg.de/start_here/connecting/generate_ssh_key/index.html), [Github documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

#### How do I create/upload SSH keys?
New users should do the following: 

- on your local machine, create an SSH key: `ssh-keygen -t ed25519 -f path_to_key_file`;
- go to `path_to_key_file` and copy the contents of the file ending in `.pub`;
- add the public key to your Academic Cloud account: log in, go to your profile
(three vertical dots next to your name), go to security, scroll down to "SSH 
public keys" ([GWDG docs with pictures](https://docs.hpc.gwdg.de/start_here/connecting/upload_ssh_key/index.html)).

For more detailed instructions, refer to the GWDG documentation: [Connect to GWDG with SSH](https://docs.hpc.gwdg.de/start_here/connecting/index.html).

#### What is the correct SSH incantation/configuration?
You could type out the following:

```shell
ssh your_username@login-mdc.hpc.gwdg.de -i path_to_key_file -J your_username@glogin.hpc.gwdg.de
# In some cases (WSL, PuTTY, old SSH version), you may need to add:
# -o ProxyCommand="ssh -i path_to_key_file -W %h:%p your_username@glogin.hpc.gwdg.de
# If that also does not work, try 'ssh.exe' instead of 'ssh'
```

Or you could add the following to your SSH configuration file (Linux 
`/home/your_username/.ssh/config`, MacOS `/User/your_username/.ssh/config`, Windows
`C:\Users\your_username\.ssh\config`, Cygwin `your_cygwin_directory/home/your_username/.ssh/config`):

```shell
# You can change the string after 'Host' to whatever you like. It is just a
# label.
Host jumphost
	Hostname glogin.hpc.gwdg.de
	User your_username
	IdentityFile path_to_key_file

Host SCC
	Hostname login-mdc.hpc.gwdg.de
	User your_username
	IdentityFile path_to_key_file
	ProxyJump jumphost
	# If you use WSL/PuTTY, uncomment the line below:
	#ProxyCommand ssh.exe -i path_to_key_file -W %h:%p jumphost

Host SCC-transfer
	Hostname transfer-scc.hpc.gwdg.de
	User your_username
	IdentityFile path_to_key_file
	ProxyJump jumphost
	# If you use WSL/PuTTY, uncomment the line below:
	#ProxyCommand ssh.exe -i path_to_key_file -W %h:%p jumphost

# SSH parses its config file top-to-bottom and later variable declarations
# do not override prior declarations. Therefore, leave the generic block at
# the end of the file.
Host *
    ForwardAgent no
    ForwardX11 yes
    ForwardX11Trusted yes
    ServerAliveInterval 120
```

Replace `your_username` with the user name you received after signing up for the
MPIDR GWDG HPC project and `path_to_key_file` with the appropriate file path.
This should be the path to your private SSH key, not the public one.

Now you should be able to type `ssh SCC` (beware capitalization) and connect to
the cluster. If encrypted your SSH key with a passphrase, you will have to enter
it twice. You will see a bunch of messages. If you want to know what they mean,
click
[here](https://docs.hpc.gwdg.de/start_here/connecting/login_nodes_and_example_commands/index.html).

You should see a prompt containing `your_username@gwdu10[1-2]` (and maybe some
extra info). `gwdu101` and `gwdu102` are the actual login nodes on which we, as
part of MPIDR, work. We need the "jump" through the `glogin` node, however, for
our login for technical GWDG-internal reasons.

If you want to connect directly to `gwdu101` or `gwdu102` in order to continue a
tmux/screen/IDE session, I trust that you know how to adapt the above.

**Returning users**: you may want to keep your previous GWDG entries until you are
confident that you have copied over/backed up all the important files. See below
on how to do so.

### What do I need to do before I can do real work?
#### How do I get my old files?
**Returning users**: Log in directly to `glogin.hpc.gwdg.de`, create a new SSH
key, copy the public key to Academic Cloud, and transfer your files:

```shell
rsync -e 'ssh -i path_to_new_key_file' your_OLD_username@transfer-scc.gwdg.de:/usr/users/your_OLD_username/old_directory .
```

If that doesn't work, check out the instructions here: [GWDG data migration guide](https://docs.hpc.gwdg.de/project_management/migrate_user_data/index.html#using-shared-academicid-group)

#### How do I use R/Python/...?
In most cases, you have to _load_ a software package (GWDG calls them "modules")
before you can use it. This is because the hardware behind your login node or
the interactive node(s) or the batch node(s) is not necessarily the same. 

If you are lucky, the software that you want to use has been pre-compiled for
the different hardware configurations in use at the SCC. In this case, the
module system will make sure to load the version that is appropriate to the node
_from where it is called_.

You can list all available modules like this (there aren't that many): `module
avail`.

If you are looking for a particular software, try `module avail some_string` to
list all available modules with `some_string` in their name.

To load a module: `module load my_module`.

To unload a module: `module unload my_module`.

To list all loaded modules: `module list`.

For more information about a particular module, you can try `module verb
my_module` where `verb` is one of `help`, `whatis`, or `show`.

For more detail on the module system: [GWDG module basics](https://docs.hpc.gwdg.de/software_stacks/module_basics/index.html).

#### I need to install a programm/package/something.
So you want to manually install some software, because it is not available as a
module or you need a different version or something else. Meet
[Spack](https://docs.hpc.gwdg.de/how_to_use/spack/index.html):

> Spack is a package manager designed for high-performance computing (HPC)
applications. It simplifies the process of building and installing multiple
versions of software on various platforms.

*Python tip*: try the `anaconda3` module before manually installing packages.

1. load Spack: `module load spack`; 
    - optional: run `source $SPACK_USER_ROOT/share/spack/setup-env.sh` (it takes a little while but makes using Spack nicer);
2. check whether Spack knows about the software you want to install: `spack list some_string`;
3. install the software: `spack install some_string`;
4. if Spack suggests multiple packages that could be installed (see example below):
    - if one of the suggested options looks good to you, make note of its 7-letter/digit hash and run the install command again with `/the_seven_letterdigit_hash` appended to the name of the software;
    - if you do not understand the differences between the versions, read the longer explanation below;
    - if none of the options look good, try the method outlined below and maybe you can coax Spack into installing the software to your specification.
    
```shell
# If you ask Spack to install a software for which there are multiple versions 
# (software versions, potential compilers, target architectures), you will get
# an error like this one.
==> Error: r matches multiple packages.
  Matching packages:
    bolvmc3 r@4.2.2%gcc@11.4.0 arch=linux-scientific7-haswell
    trugvwu r@4.3.0%gcc@11.4.0 arch=linux-scientific7-haswell
    5d7wuqz r@4.2.2%gcc@9.5.0 arch=linux-rocky8-cascadelake
    6hafays r@4.2.2%gcc@11.4.0 arch=linux-rocky8-cascadelake
    766rnwo r@4.4.0%gcc@11.4.0 arch=linux-scientific7-haswell
  Use a more specific spec (e.g., prepend '/' to the hash).
```
_Note_: `spack list some_string` searchers whether `some_string` is _available
to be installed_; `spack find some_string` searches whether `some_string` (any
version) is _already installed_.

_Note_: many Python and R packages are available in Spack as `py-packagename` or
`r-packagename`.

### How do I do real work?
#### Interactive use (for prototyping and quick tests)
You should not use the login nodes for actual work. Not even prototyping or
test runs. Use an "interactive session" instead. Interactive session means
that you request actual resources from the system but with an interactive 
shell instead of submitting a script to be run in the background without 
your input. Doing it this way makes you a good citizen and avoids angry 
emails by the GWDG admins (they are never really angry, just disappointed).

```shell
srun -p medium -c 1 -N 1 --pty bash
```
After a short wait, you will see that your command line prompt has changed from
`your_userid@gwdu_some-digits` to `your_userid@some_nodeID`. You are now on
a "compute" node instead of a "login node". You can now run an interactive
R or Python (or whatever) session or run scripts. You need to load the 
appropriate modules first, of course. Once you're done, you can exit the
session by typing `exit` or pressing `Ctrl-D`.

`-c` determines the number of cores and `-N` the number of nodes. If you want to
run a minimal job to quickly test something, `-N` is probably extraneous and you
should just use `-c` with a small number of cores. You can also add `-t hh:mm:ss`
to indicate for how long you want the session to exist. The fewer resources you
request and the shorter the session, the shorter your waiting time.

#### The batch system (for bigger analyses)
If you want to run parallel or interdependent analyses, you probably want to use
the batch system. GWDG uses "Slurm". Other HPC providers may use other systems. 

The GWDG Slurm documentation: [GWDG Slurm](https://docs.hpc.gwdg.de/how_to_use/slurm/index.html)

The Slurm project document: [Slurm documentation](https://slurm.schedmd.com/documentation.html)

If you are new to HPC and batch systems, you might want to take the GWDG
courses "Using the GWDG Scientific Compute Cluster" or "Supercomputing for Every 
Scientist" (see the beginning of this document).

The Slurm system requires you to submit "jobs" by running `sbatch <jobscript>`.
`jobscript` is any script that contains Slurm instructions at the top. Usually,
this is a shell script. But other scripting languages are possible too. We only
give examples of shell scripts here.

```shell
#!/bin/bash

#SBATCH -p medium
#SBATCH -c 10
#SBATCH -t 12:00:00

cd $HOME/my_project
module load R R_packages
Rscript my_analysis.R
```
Lines starting with `#SBATCH` are instructions to Slurm. For the different
parameters, see the GWDG Slurm documentation.

Since a job runs in its own instance, you need to load any libraries necessary
for the rest of the script from within the script. 

Other important commands:

- `squeue --me`: show your currently waiting or running jobs
- `scancel <job ID>`: cancel job with ID `job ID`

#### Using your own container
Ask Egor.

### Miscellaneous
#### How do I run an interactive job? {#subsec-interactive}
```shell
srun -p medium -c 1 -N 1 --pty bash
```

#### What types of resources (partitions/queues) do we have access to?
We should have access to the `scc-cpu`, `scc-gpu`, `medium`, and `jupyter` partitions. As members of the MPIDR project, we are part of the "SCC Legacy" island (a subset of the resources at GWDG HPC). For more information, [here](https://docs.hpc.gwdg.de/how_to_use/compute_partitions/index.html) is the relevant part of the GWDG HPC documentation.

#### What is the difference between nodes, partitions, queues, CPUs, and cores?
A partition contains nodes contains CPUs contains cores.

A core is a subunit of a CPU. You cannot directly request a core. However, your
software may be able to use different cores of the same CPU independently of and
in parallel to each other (e.g. R data.table).

A CPU is the smallest unit of work that you can request. There are different CPU
architectures in the SCC and you may need to tell Spack on which one you plan on
running your software.

A node is the server equivalent to your desktop or laptop; it is an independent
physical machine with CPU, memory, and everything that makes a computer goo
vrooom. All SCC nodes have 2 CPUs. These CPUs have varying numbers of cores.

A queue is a concept from the SLURM batch system. When you submit a job, you
must tell SLURM to which job queue to submit this job. In our case, a queue is
synonymous with a partition.

A partition is a logical/organizational unit that GWDG uses to refer to a set
of nodes. This is for the purposes of the batch system but also in order to
collect nodes with identical architectures into homogeneous sets. Of course, we 
are not so lucky to have access to "homogeneous" partitions.

TODO: the documentation is confused between CPUs and cores: 
https://docs.hpc.gwdg.de/start_here/accounting/index.html vs https://docs.hpc.gwdg.de/how_to_use/slurm/index.html.

#### What does `766rnwo r@4.4.0%gcc@11.4.0 arch=linux-scientific7-haswell` mean?
If you do not know what the strings after the first `@` or `%` above mean or if
you know that you will run your software on a specific
node/partition/architecture or if you want to tell Spack to (try to) install
some very specific specification of a software, read on.

Spack offers different specifications of software according to (often) at least
three aspects:

- the version of the software itself: this is determined by the `@` suffix;
- the compiler: this is determined by the `%` suffix (note that there can be different versions of the compiler itself);
- the target architecture (what kind of CPU/OS will it run on/under): this is determined by the argument `arch=something-something`.

The title of this section contains the specification for R version 4.4.0, 
compiled with gcc version 11.4.0 for Haswell CPUs and Scientific Linux 7.

#### How do I know which compiler or architecture I want?
**Compiler**: if you don't have a reason to prefer a specific compiler (or
version), just `module load gcc` and let Spack handle the rest. But sometimes to
you need a specific compiler (version) when installing external software. It
will usually tell you so either in the requirements or the error messages during
the installation.

**Architecture**: run a minimal interactive job on the partition/node that you want
to work on and run `spack arch`. If you are reading this document instead of the
original GWDG/Spack/whatever documentation, you probably do not have a good
reason to prefer one CPU architecture over another and should just go with
whatever the above command returns. Most probably, your job will run on the
`medium` partition and the lowest common denominator architecture for nodes in 
the `medium` partition is `linux-scientific7-haswell`(2025-02-04).

_Note_: if you know you want to use a specific architecture but still want/have
to use the `medium` partition, you can pass the `-C` or `--constraint` option
(see section "The CPUs"
[here](https://docs.hpc.gwdg.de/how_to_use/compute_partitions/cpu_partitions/index.html)).

_Note_: According to the documentation, SCC users should have access to the
`medium`, `int`, `fat`, and `fat+` partitions but I have only been able to run
jobs on the `medium` queue (2025-02-04). Maybe there are different groups of SCC
users with access to different resources.

**Other dependencies**: if you have other requirements (or the install process
failed because some software was unavailable or the wrong version), you can
pass more specific dependency requirements to Spack. In which case you should
probably read the [actual Spack documentation](https://spack.readthedocs.io/en/v0.21.2/features.html#custom-versions-configurations).

#### Should I request cores or nodes or a mix?
If you are using the SCC, you are probably (hopefully) running parallel jobs. If
there is very little communication between your processes, you can spread them
across cores and do not have to worry about nodes. Ignore the `-N` parameter and
request as many cores as seems appropriate. This should help your job advance 
more quickly in the waiting queue.

If your processes need to talk to each other, you want them to happen on the
same node as much as possible. 

#### R/gcc/make is complaining that some system library is missing.
Try `module spider '*glob*'` or `spack list '*glob*'`. Spack may have multiple
versions of the library ready to be installed. Pick the one compatible with the
 desired compiler and architecture and `spack install` it.

For example, in order to install vim from Github, you need to `module load ncurses libiconv`
or  Spack install and load ncurses and libiconv.
