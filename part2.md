Hands-on lab (Second Part)
================
Sofia Gil
January 15, 2019

-   [First steps for parallelizing in R](#first-steps-for-parallelizing-in-r)
    -   [The basics](#the-basics)
    -   [Divide and Conquer](#divide-and-conquer)
-   [Submitting R scripts into the cluster](#submitting-r-scripts-into-the-cluster)
    -   [The basics](#the-basics-1)
    -   [Parallel jobs](#parallel-jobs)
    -   [Other important LSF commands](#other-important-lsf-commands)
-   [References](#references)

First steps for parallelizing in R
==================================

The basics
----------

It is well-known that when we are using *R* we always have to avoid **for** loops. Due to the calls that *R* does to *C* functions, therefore it is better to vectorize everything.

``` r

rm(list=ls())
gc()
#>           used (Mb) gc trigger (Mb) max used (Mb)
#> Ncells  625898 33.5    1207107 64.5  1207107 64.5
#> Vcells 1137928  8.7    8388608 64.0  1663533 12.7

len<-5000000

a<-runif(len)
b<-runif(len)

c1=0

system.time({
  for(i in 1:length(a)){
    c1<-a[i]*b[i]+c1
  }
})['elapsed']
#> elapsed 
#>    0.41

system.time({c2<-a%*%b})['elapsed']
#> elapsed 
#>    0.05
```

But... what should we do when the problem we are facing needs a loop? There are two options:

1.  Write the loop and wait.(*Easy way*)
2.  Paralellize. (*Fun way*)

Imagine we want to do the Kronecker Product of *A* ⊗ *B*, that means:

<img src="Github_files/figure-markdown_github/Kronecker.png" width="300" />

Where *A* and *B* are matrixes.

#### Easy way

**Memory access**: It is important to know how *R* stores matrices, i.e., by row or by column. In the specific case of *R*, all matrices (also data frames) are stored by column, given that *R* is a statistical software where the columns almost always represent variable names. This is crucial since each variable created in *R* has an **ID number**, so the compiler or the interpreter will assign specific addresses in memory at which every element of the variables (e.g. vector, matrices, data frames, lists,...) will be stored. Given that these addresses are consecutive, it will always decrease memory access time if we call them in a consecutive way.

<img src="Github_files/figure-markdown_github/ColumnOriented.png" width="300" />

``` r

(A<-matrix(runif(9),nrow = 3,ncol = 3))
#>           [,1]      [,2]      [,3]
#> [1,] 0.9611183 0.8296429 0.4385134
#> [2,] 0.8282748 0.1126998 0.1259877
#> [3,] 0.5347326 0.6106551 0.6310976

for(i in 1:length(A))
  print(paste(i,A[i]))
#> [1] "1 0.96111828298308"
#> [1] "2 0.828274838859215"
#> [1] "3 0.534732625121251"
#> [1] "4 0.829642858589068"
#> [1] "5 0.112699778983369"
#> [1] "6 0.610655125696212"
#> [1] "7 0.438513382570818"
#> [1] "8 0.125987676205114"
#> [1] "9 0.631097618956119"
```

For solving with a loop this basic knowledge can help to increase the pace of the algorithm.

``` r

library(foreach)

rm(list=ls())
gc()
#>           used (Mb) gc trigger  (Mb) max used (Mb)
#> Ncells  631020 33.8    1207107  64.5  1207107 64.5
#> Vcells 1146328  8.8   14905332 113.8 11333422 86.5

row=100
col=100

A<-matrix(runif(row*col),nrow = row,ncol = col)
B<-matrix(runif(row*col),nrow = row,ncol = col)

CR<-matrix(A,nrow = row^2,ncol = col^2)
CC<-matrix(A,nrow = row^2,ncol = col^2)

### By row 
system.time({
  CR<-foreach(j=1:row,.combine = 'rbind',.packages = 'foreach')%do%{
        foreach(i=1:length(A[j,]),.combine = 'cbind')%do%{
            A[j,i]*B
        }
  }
})['elapsed']
#> elapsed 
#>    6.37


### By column 
system.time({
  CC<-foreach(A2=A,.combine = 'cbind',.packages = 'foreach')%do%{
        foreach(i=1:length(A2),.combine = 'rbind')%do%{
            A2[i]*B
        }
  }
})['elapsed']
#> elapsed 
#>    4.42


sum(CC-CR)
#> [1] 0
```

#### Fun way

There are several packages for parallelizing:

-   doParallel
-   Parallel
-   snow
-   multicore

Some of those packages are required by the others, so lets start with **doParallel**. Once we open this package in our session it will require **parallel**, which will automatically open the rest.

The easiest way to parallelize the loop above is using the package **foreach** in conjunction with **doParallel**.

``` r

library(doParallel)
#> Loading required package: iterators
#> Loading required package: parallel

(no_cores<-detectCores()-1)
#> [1] 35
```

Now we proceed to parallelize, for this it's necessary to call the *registerParallel* function.:

``` r

registerDoParallel()

getDoParWorkers()
#> [1] 3

system.time({
  C<-foreach(A2=A,.combine = 'cbind',.packages = 'foreach')%dopar%{
        foreach(i=1:length(A2),.combine = 'rbind')%do%{
            A2[i]*B
        }
  }
})['elapsed']
#> elapsed 
#>    2.93


stopImplicitCluster()

sum(CC-C)
#> [1] 0
```

Anytime we initialize the cores using **registerDoParallel** there is no need to close the connections, they will be stopped automatically once the program detects they aren't used anymore.But anytime we want to explicitly require R to release them we can use **stopImplicitCluster**.

### Life is always a tradeoff

You might be wondering why we are not using more than 3 cores. Let's do it!

First, we need to introduce some new functions:

-   **makeCluster**: initializes all the cores or clusters that we require.
-   **stopCluster**: stops and releases the cores that we have initialized.

``` r

Times<-matrix(0,nrow=no_cores,ncol=20)

for(cols in 1:20){
  for(h in 1:no_cores){
    cls<-makeCluster(h)
    registerDoParallel(cls)
    Times[h,cols]<-system.time({
        foreach(A2=A,.combine = 'cbind',.packages = 'foreach')%dopar%{
          foreach(i=1:length(A2),.combine = 'rbind')%do%{
              A2[i]*B
          }
    }
    })['elapsed']
    stopCluster(cls)
  }
}

write.csv(Times2,'Times_Time_graph_DoPar_Hydra.csv',row.names = FALSE)
```

The next plot shows the boxplots of the time Hydra required to finish computing the Kronecker product depending the number of cores that we ask.

<img src="Github_files/figure-markdown_github/Time_graph_DoPar_Hydra.png" width="500" />

Given that we are sharing memory, i.e., in every iteration we are calling *B*, the way *R* in Windows handles this is by making the different cores wait until the others finish using *B*. Therefore, more cores doesn't always mean less processing time.

Divide and Conquer
------------------

Now we will divide the task into subtasks that will be processed by different nodes (called workers) and that at the end will be managed by a master.

``` r

# Checking cores
detectCores()
#> [1] 36


doichunk<-function(chunk){
  require(doParallel)
  MatC<-foreach(A2=A[,chunk],.combine = 'cbind',.packages = 'foreach')%dopar%{
    foreach(i=1:length(A2),.combine = 'rbind')%do%{
        A2[i]*B
    }
  }
  return(MatC)
}


ChunkKronecker<-function(cls,A,B){
  clusterExport(cls,c('A','B'))
  ichunks<-clusterSplit(cls,1:ncol(A)) # Split jobs
  tots<-clusterApply(cls,ichunks,doichunk)
  return(Reduce(cbind,tots))
}


cls<-makeCluster(4)
system.time(valor<-ChunkKronecker(cls,A,B))['elapsed']
#> elapsed 
#>    4.63
stopCluster(cls)


sum(CC-valor)
#> [1] 0
```

``` r

Tiempos<-matrix(0,nrow = no_cores,ncol = 20)

for(j in 1:20){
  for(i in 1:no_cores){
    cls<-makeCluster(i)
    Tiempos[i,j]<-system.time(valor<-ChunkKronecker(cls,A,B))['elapsed']
    stopCluster(cls)
  }
}
```

The next plot shows the boxplots of the times per number of cores for finishing the jobs.

<img src="Github_files/figure-markdown_github/Time_graph_Workers_Hydra.png" width="500" />

Submitting R scripts into the cluster
=====================================

From now on, the operating system will be Linux. Linux and Windows have significant differences from each other, not only in the graphic user interface, but also in their architectural level. One of the main differences is that Windows doesn't fork and Linux does. That means that Linux make copies of the needed variables in all the cores, instead of forcing the cores to queue and wait for using the shared variables.

The basics
----------

First, open *R* in the terminal, install the packages and close *R*.

``` r

> R

install.packages("doParallel", dependencies=TRUE) ## CRAN Mirror 34 (Germany)


library(doParallel)

(no_cores<-detectCores()-1)

registerDoParallel()

getDoParWorkers()

q()
```

Let's check what happens if instead of **makeCluster** we use **makeForkCluster** (attention: this function is only available for Linux) for the Kronecker product.

``` r

rm(list=ls())
gc()

row=100
col=100

A<-matrix(runif(row*col),nrow = row,ncol = col)
B<-matrix(runif(row*col),nrow = row,ncol = col)



library(doParallel)

detectCores()

no_cores<-5 # Practical reasons

#### Forking ####
#### Normal ####

TimesN<-rep(0,no_cores)

for(h in 1:no_cores){
  cls<-makeForkCluster(h)
  registerDoParallel(cls)
  TimesN[h]<-system.time({
    foreach(A2=A,.combine = 'cbind',.packages = 'foreach')%dopar%{
      foreach(i=1:length(A2),.combine = 'rbind')%do%{
        A2[i]*B
      }
    }
  })['elapsed']
  stopCluster(cls)
}


#### Master - Workers ####

doichunk<-function(chunk){
  require(foreach)
  MatC<-foreach(A2=A[,chunk],.combine = 'cbind')%:%
    foreach(i=1:length(A2),.combine = 'rbind')%do%{
      A2[i]*B
    }
  return(MatC)
}


ChunkKronecker<-function(cls,A,B){
  clusterExport(cls,c('A','B'))
  ichunks<-clusterSplit(cls,1:ncol(A)) # Split jobs
  tots<-clusterApply(cls,ichunks,doichunk)
  return(Reduce(cbind,tots))
}



TimesC<-rep(0,no_cores)

for(i in 1:no_cores){
  cls<-makeForkCluster(i)
  TimesC[i]<-system.time(valor<-ChunkKronecker(cls,A,B))['elapsed']
  stopCluster(cls)
}


Times<-data.frame(NoCores=1:no_cores,Normal=TimesN, Workers=TimesC)

write.csv(Times,'TimesFork.csv')


#### No Forking ####
#### Normal ####

TimesN<-rep(0,no_cores)

for(h in 1:no_cores){
  cls<-makeCluster(h)
  registerDoParallel(cls)
  TimesN[h]<-system.time({
    foreach(A2=A,.combine = 'cbind',.packages = 'foreach')%dopar%{
      foreach(i=1:length(A2),.combine = 'rbind')%do%{
        A2[i]*B
      }
    }
  })['elapsed']
  stopCluster(cls)
}


#### Master - Workers ####

doichunk<-function(chunk){
  library(foreach)
  MatC<-foreach(A2=A[,chunk],.combine = 'cbind')%:%
    foreach(i=1:length(A2),.combine = 'rbind')%do%{
      A2[i]*B
    }
  return(MatC)
}


ChunkKronecker<-function(cls,A,B){
  clusterExport(cls,c('A','B'))
  ichunks<-clusterSplit(cls,1:ncol(A)) # Split jobs
  tots<-clusterApply(cls,ichunks,doichunk)
  return(Reduce(cbind,tots))
}



TimesC<-rep(0,no_cores)

for(i in 1:no_cores){
  cls<-makeCluster(i)
  TimesC[i]<-system.time(valor<-ChunkKronecker(cls,A,B))['elapsed']
  stopCluster(cls)
}


Times<-data.frame(NoCores=1:no_cores,Normal=TimesN, Workers=TimesC)

write.csv(Times,'TimesNoFork.csv')

```

For running and R code in terminal, without opening *R*, just write **Rscript** followed by the name of your *R* file.

``` r

> Rscript KroneckerLinux.R
```

### Specifying node properties with *-R*

**-R** runs the job on a host that meets the specified resource requirements. Some of its basic parameters are:

-   span\[hosts=1\]: this puts all processes on one host.
-   span\[ptile=&lt; x &gt;\]: *x* denotes the exact number of job slots to be used on each host. If the total process number is not divisible by *x* then the rest is submitted in the last core.
-   scratch(2): the node must have access to *scratch(2)*.

### Last but not less important parameters

-   -n: submits a parallel job and specifies the number of tasks in the job.
-   -a: this option denotes a wrapper script required to run SMP or MPI jobs. The most important wrappers are[1]:
    -   Shared Memory (**openmp**). This is a type of parallel job that runs multiple threads or processes on a single multi-core machine. OpenMP programs are a type of shared memory parallel program.
    -   Distributed Memory (**intelmpi**). This type of parallel job runs multiple processes over multiple processors with communication between them. This can be on a single machine but is typically thought of as going across multiple machines. There are several methods of achieving this via a message passing protocol but the most common, by far, is MPI (Message Passing Interface).
    -   Hybrid Shared/Distributed Memory (**openmpi**). This type of parallel job uses distributed memory parallelism across compute nodes, and shared memory parallelism within each compute node. There are several methods for achieving this but the most common is OpenMP/MPI.

### The **bsub** command: Submitting jobs to the cluster

**bsub** submits information regarding your job to the batch system. Instead of writing a large *bsub* command in the terminal, we will create a shell file specifying all the *bsub* requirements.

The shell files can be created through either the linux terminal using the editor *nano* or Notepad++ in Windows.

Parallel jobs
-------------

``` r

> nano NameShell.sh

------------------------
#!/bin/sh 
#BSUB -N 
#BSUB -u <YourEmail>
#BSUB -q mpi 
#BSUB -W <max runtime in hh:mm>
#BSUB -o NameOutput.%J.txt 
#BSUB -n <min>,<max> or <exact number> 
#BSUB -a openmp 
#BSUB -R span[hosts=1] 
 
R CMD BATCH NameCode.R
--------------------------
```

``` r
> bsub < NameShell.sh
```

So, for submiting our job the shell file would look like the next one:

``` r

> nano KroneckerLinux.sh

------------------------
#!/bin/sh 
#BSUB -N 
#BSUB -u gil@demogr.mpg.de
#BSUB -q mpi 
#BSUB -W 00:30
#BSUB -n 8 
#BSUB -a openmp 
#BSUB -R span[hosts=1] 
 
R CMD BATCH KroneckerLinux.R
--------------------------
```

``` r
> bsub < KroneckerLinux.sh
```

Once we submit the job in the cluster it will look like:

<img src="Github_files/figure-markdown_github/SUBMIT.PNG" width="800" />

Once

<img src="Github_files/figure-markdown_github/email.PNG" width="300" />

<img src="Github_files/figure-markdown_github/ForkVSnoFork.png" width="500" />

Other important LSF commands
----------------------------

-   bjobs: lists currents jobs.

While not having a job slot:

<img src="Github_files/figure-markdown_github/WAITING.PNG" width="800" />

Once having a job slot:

<img src="Github_files/figure-markdown_github/WORKING.PNG" width="800" />

-   bhist: lists older jobs.
-   lsload: status of cluster nodes.
-   bqueues: status of cluster nodes.
-   bhpart: shows current user priorities.
-   bkill <jobid>: stops the current job.

<img src="Github_files/figure-markdown_github/bKill.PNG" width="800" />

References
==========

<img src="Github_files/figure-markdown_github/Book.jpg" width="200" />

<https://info.gwdg.de/dokuwiki/lib/exe/fetch.php?media=en:services:scientific_compute_cluster:parallelkurs.pdf>

[1] <https://wiki.uiowa.edu/display/hpcdocs/Advanced+Job+Submission>
