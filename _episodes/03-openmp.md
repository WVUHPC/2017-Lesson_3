---
title: "Parallel Programming with OpenMP"
teaching: 45
exercises: 15
questions:
- "When OpenMP can be the right choice for accelerating an algorithm"
- "What is the difference between OpenMP and MPI"
objectives:
- "Introduce OpenMP and API for parallel computing on SMP machines."
keypoints:
- "First key point."
---

# Parallel Programming with OpenMP

OpenMP is an Application Program Interface (API) that provides a portable, scalable model for developers of shared memory parallel applications.
OpenMP works with Symmetric Multiprocessing (SMP)
The API supports C/C++ and Fortran on a wide variety of architectures.
This tutorial covers some of the major features of OpenMP 4.0, including its various constructs and directives for specifying parallel regions, work sharing, synchronization and data environment.
Runtime library functions and environment variables are also covered.
This tutorial includes both C and Fortran example codes and a lab exercise.

## What is a Symmetric Multiprocessing (SMP)?

Most computers today have several cores, that is also true for the individual nodes on a cluster.
The several cores in a node have access to the main memory of the machine.
When several processing units see the same memory we say that that machine is a SMP system.
Most multiprocessor systems today use an SMP architecture.
Symmetric multiprocessing (SMP) involves a multiprocessor computer hardware and software architecture where two or more identical processors are connected to a single, shared main memory, have full access to all I/O devices, and are controlled by a single operating system instance that treats all processors equally.

<a href="https://upload.wikimedia.org/wikipedia/commons/1/1c/SMP_-_Symmetric_Multiprocessor_System.svg">
  <img src="https://upload.wikimedia.org/wikipedia/commons/1/1c/SMP_-_Symmetric_Multiprocessor_System.svg" alt="SMP" />
</a>

## What is OpenMP

OpenMP is an implementation of multithreading, a method of parallelizing whereby a master thread (a series of instructions executed consecutively) forks a specified number of slave threads and the system divides a task among them. The threads then run concurrently, with the runtime environment allocating threads to different processors.

The core elements of OpenMP are the constructs for thread creation, workload distribution (work sharing), data-environment management, thread synchronization, user-level runtime routines and environment variables.

In C/C++, OpenMP uses #pragmas. Those instructions are comments to the non openmp compiler but can be understood if the compiler support them and if the compilation line includes instructions to include openmp in the final object.
In Fortran OpenMP uses comments like "!$omp", they are also comments in Fortran 90+ but can be used by the compiler if the compilation arguments are included.

<a href="https://upload.wikimedia.org/wikipedia/commons/9/9b/OpenMP_language_extensions.svg">
  <img src="https://upload.wikimedia.org/wikipedia/commons/9/9b/OpenMP_language_extensions.svg" alt="OpenMP" />
</a>

## First example

Lets create a very simple example in C. Lets call it "omp_simple.c"

~~~
#include <stdio.h>

int main(int argc, char *argv[])
{
#pragma omp parallel
  printf("This is a thread.\n");
  return 0;
}
~~~
{: .source}

~~~
gcc -fopenmp omp_simple.c
~~~
{: .bash}

Executing the executable "a.out" will produce:

~~~
This is a thread.
This is a thread.
This is a thread.
This is a thread.
...
~~~
{: .output}

The actual number of threads is based on the number of cores available on the
machine. However, you can control that number changing the environmental variable
"OMP_NUM_THREADS"

~~~
OMP_NUM_THREADS=3 ./a.out
~~~
{: .bash}

Similarly, the fortran version of the same program is

~~~
program hello

  !$omp parallel
  print *, 'This is a thread'
  !$omp end parallel

end program hello
~~~
{: .source}

~~~
gfortran -fopenmp omp_simple.f90
~~~
{: .source}

And executed like
~~~
OMP_NUM_THREADS=3 ./a.out
~~~
{: .bash}

Now its time to a more useful and complex example.
One of the typical cases where OpenMP offers advantages is doing
Single Instruction Multiple Data (SIMD) operations.
For example, if you want to compute the dot product of two vectors, you are
doing the same operation, multiplying two numbers, for all the indices of
the vectors. As both vectors are on the same memory space that all cores can see,
the operation can be easily compute concurrently, by giving each core a chunk
of the vector and storing the result on another one also shared by all cores.
Lets see the example of that.
The C version looks like this:

~~~
#include <omp.h>
#include <stdio.h>
#define N 10000
#define CHUNKSIZE 100

int main(int argc, char *argv[]) {

  int i, chunk;
  float a[N], b[N], c[N];

  /* Initializing vectors */
  for (i=0; i < N; i++)
    {
      a[i] = i;
      b[i] = i * N;
    }
  chunk = CHUNKSIZE;

#pragma omp parallel shared(a,b,c,chunk) private(i)
  {
#pragma omp for schedule(dynamic,chunk)
    for (i=0; i < N; i++)
      c[i] = a[i] + b[i];
  }   /* end of parallel region */

  for (i=0; i < 10; i++) printf("%17.1f %17.1f %17.1f\n", a[i], b[i], c[i]);
  printf("...\n");
  for (i=N-10; i < N; i++)  printf("%17.1f %17.1f %17.1f\n", a[i], b[i], c[i]);
}
~~~
{: .source}

And the fortran version is:

~~~
program omp_vec_addition

  integer n, chunksize, chunk, i
  parameter (n=10000)
  parameter (chunksize=100)
  real a(n), b(n), c(n)

  !initialization of vectors
  do i = 1, n
     a(i) = i
     b(i) = a(i) * n
  enddo
  chunk = chunksize

  !$omp parallel shared(a,b,c,chunk) private(i)

  !$omp do schedule(dynamic,chunk)
  do i = 1, n
     c(i) = a(i) + b(i)
  enddo
  !$omp end do

  !$omp end parallel

  do i = 1, 10
     write(*,'(3F17.1)') a(i), b(i), c(i)
  end do
  write(*,*) '...'
  do i = 1, 10
      write(*,'(3F17.1)') a(n-10+i), b(n-10+i), c(n-10+i)
  end do

end program omp_vec_addition
~~~
{: .source}



{% include links.md %}
