---
title: "Parallel Programming with MPI"
teaching: 45
exercises: 15
questions:
- "Key question"
objectives:
- "Understand the basic concepts behind Message Passing Interface"
keypoints:
- "First key point."
---

## Introduction

MPI is a specification for the developers and users of message passing libraries.
By itself, it is NOT a library - but rather the specification of what such a
library should be.
There are several implementations of MPI, the most popular today are OpenMPI and
Intel MPI and MPICH and MVAPICH.

## Compiling MPI programs

The different implementations of MPI offer wrappers to the compilers.
The following table shows how to compile applications with several
implementations.

Using OpenMPI the compile line is as follows

| Language |	Wrapper | Command	Example |
|:---------|:---------|-----------------|
| C        | mpicc	  | mpicc [options] main.c |
| C++	     | mpicxx	  | mpicxx [options] main.cpp |
| Fortran	 | mpif90	  | mpif90 [options] main.f90 |

Using Intel MPI you can use the commands above if using GCC as underlying compiler,
or using the following commands for the intel compilers.

| Language |	Wrapper | Command	Example |
|:---------|:---------|-----------------|
| C        | mpiicc	  | mpiicc [options] main.c |
| C++	     | mpiicpc	| mpiicpc [options] main.cpp |
| Fortran	 | mpiifort	| mpiifort [options] main.f90 |

## Getting started

Lets start with one simple example showing the basic elements of the interface.
The program do actually nothing useful but illustrate the basic functions that
are common to basically any program that uses MPI.
The version in C is:

~~~
// required MPI include file  
   #include "mpi.h"
   #include <stdio.h>

   int main(int argc, char *argv[]) {
   int  numtasks, rank, len, rc;
   char hostname[MPI_MAX_PROCESSOR_NAME];

   // initialize MPI  
   MPI_Init(&argc,&argv);

   // get number of tasks
   MPI_Comm_size(MPI_COMM_WORLD,&numtasks);

   // get my rank  
   MPI_Comm_rank(MPI_COMM_WORLD,&rank);

   // this one is obvious  
   MPI_Get_processor_name(hostname, &len);
   printf ("Number of tasks= %d My rank= %d Running on %s\n", numtasks,rank,hostname);

        // do some work with message passing

   // done with MPI  
   MPI_Finalize();
   }
~~~
{: .source}

~~~
program mpi_env

  ! required MPI include file                                                                                                                
  include 'mpif.h'

  integer numtasks, rank, len, ierr  
  character(MPI_MAX_PROCESSOR_NAME) hostname

  ! initialize MPI      
  call MPI_INIT(ierr)

  ! get number of tasks   
  call MPI_COMM_SIZE(MPI_COMM_WORLD, numtasks, ierr)

  ! get my rank          
  call MPI_COMM_RANK(MPI_COMM_WORLD, rank, ierr)

  ! this one is obvious    
  call MPI_GET_PROCESSOR_NAME(hostname, len, ierr)
  print *, 'Number of tasks=',numtasks,' My rank=',rank,' Running on=',hostname

  ! do some work with message passing

  ! done with MPI       
  call MPI_FINALIZE(ierr)

end program mpi_env
~~~
{: .source}

For the purpose of this tutorial, we will use OpenMPI, the compilation line
is as follows, for the C version:

~~~
mpicc mpi_env.c -lmpi
~~~
{: .source}

For the Fortran version:

~~~
mpif90 mpi_env.f90 -lmpi
~~~
{: .source}

Lets review the different calls.
The basic difference between the functions C and Fortran subroutines is the
integer returned.
In C the return is from the function. In Fortran the equivalent subroutine
usually contains and extra output only argument to store the value.

### MPI_Init

Initializes the MPI execution environment.
This function must be called in every MPI program, must be called before any
other MPI functions and must be called only once in an MPI program.
For C programs, MPI_Init may be used to pass the command line arguments to all
processes, although this is not required by the standard and is implementation
dependent.

~~~
MPI_Init (&argc, &argv)
MPI_INIT (ierr)
~~~
{: .source}

### MPI_Comm_size

Returns the total number of MPI processes in the specified communicator,
such as MPI_COMM_WORLD.
If the communicator is MPI_COMM_WORLD, then it represents the number of
MPI tasks available to your application.

~~~
MPI_Comm_size (comm, &size)
MPI_COMM_SIZE (comm, size, ierr)
~~~
{: .source}

### MPI_Comm_rank

Returns the rank of the calling MPI process within the specified communicator.
Initially, each process will be assigned a unique integer rank between 0
and number of tasks - 1 within the communicator MPI_COMM_WORLD.
This rank is often referred to as a task ID.
If a process becomes associated with other communicators, it will have a
unique rank within each of these as well.

~~~
MPI_Comm_rank (comm,&rank)
MPI_COMM_RANK (comm,rank,ierr)
~~~
{: .source}


### MPI_Get_processor_name

Returns the processor name.
Also returns the length of the name.
The buffer for "name" must be at least MPI_MAX_PROCESSOR_NAME characters in size.
What is returned into "name" is implementation dependent - may not be the
same as the output of the "hostname" or "host" shell commands.

~~~
MPI_Get_processor_name (&name,&resultlength)
MPI_GET_PROCESSOR_NAME (name,resultlength,ierr)
~~~
{: .source}

### MPI_Finalize

Terminates the MPI execution environment. This function should be the last MPI routine called in every MPI program - no other MPI routines may be called after it.

~~~
MPI_Finalize ()
MPI_FINALIZE (ierr)
~~~
{: .source}

## Compiling and executing a MPI program on Spruce

To compile the program you need first to load the modules

~~~
module load compilers/gcc/6.3.0 mpi/openmpi/2.0.2_gcc63
~~~
{: .bash}

To execute a MPI program is strongly suggested to run it using the queue system.
Assuming that the executable is a.out.
A minimal submission script could be:

~~~
#!/bin/sh

#PBS -N MPI_JOB
#PBS -l nodes=1:ppn=4
#PBS -l walltime=00:05:00
#PBS -m ae
#PBS -q debug
#PBS -n

module load compilers/gcc/6.3.0 mpi/openmpi/2.0.2_gcc63

cd $PBS_O_WORKDIR
mpirun -np 4 ./a.out
~~~
{: .bash}

This submission script is requesting 4 cores for running your job in one node.
Using MPI you are not constrain to use a single node, but it is always a good
idea to run the program using cores as close as possible.



{% include links.md %}
