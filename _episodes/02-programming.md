---
title: "Programming in C, Fortran and Python"
teaching: 45
exercises: 15
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

## Programming in C, Fotran and Python

The purpose of this lesson is not learning 3 programming languages in two hours.
The objective is to become familiarize with the syntax of those three languages
and how you compile code that is written with them.

We take this oportunitity to also introduce several important libraries used in
scientific computing. Across the examples we will present here you will encounter
the use of LAPACK/BLAS, GSL and HDF5.
Those libraries are commonly used to perform operations that require linear
algebra (BLAS and LAPACK), general scientific operations (GSL) and robust and
portable storage of data (HDF5).

First, see a <a href="https://julialang.org/benchmarks/">Comparison between
languages</a>


There are more specialized languages, such as R, Julia. But we will try to keep the things simple here by showing how the same task is programmed in the 3 languages we have selected.
See for example \ref{bench} for a comparison on the performance of those different languages.

I am taking these examples from http://rosettacode.org.
To give you a flavor of what is the feeling writing code in those 3 languages I have selected 2 tasks and showing how the solution is express in those languages.

### Sieve of Eratosthenes

The Sieve of Eratosthenes is a simple algorithm that finds the prime numbers up to a given integer.

Lets start with the implementation in C. I am selecting not the most optimized version, but the simplest implementation for pedagogical purposes. The idea is to get a flavor of the language.

~~~
#include <stdio.h>
#include <stdlib.h>

void sieve(int *, int);

int main(int argc, char *argv[])
{
  int *array, n;

  if ( argc != 2 ) /* argc should be 2 for correct execution */
    {
      /* We print argv[0] assuming it is the program name */
      printf( "usage: %s max_number\n", argv[0] );
    }
  else
    {
      n=atoi(argv[1]);
      array =(int *)malloc(sizeof(int));
      sieve(array,n);
    }
  return 0;
}

void sieve(int *a, int n)
{
  int i=0, j=0;

  for(i=2; i<=n; i++) {
    a[i] = 1;
  }

  for(i=2; i<=n; i++) {
    printf("\ni:%d", i);
    if(a[i] == 1) {
      for(j=i; (i*j)<=n; j++) {
	printf ("\nj:%d", j);
	printf("\nBefore a[%d*%d]: %d", i, j, a[i*j]);
	a[(i*j)] = 0;
	printf("\nAfter a[%d*%d]: %d", i, j, a[i*j]);
      }
    }
  }

  printf("\nPrimes numbers from 1 to %d are : ", n);
  for(i=2; i<=n; i++) {
    if(a[i] == 1)
      printf("%d, ", i);
  }
  printf("\n\n");
}
~~~
{: .source}

This example shows the basic elements from the c language, the creation of variables, loops and conditionals. The inclusion of libraries and the printing on screen.

You can compile this code using the code at

~~~
Day3_AdvancedTopics/2.Programming
~~~
{: .source}

~~~
gcc sieve.c -o sieve
~~~
{: .source}

and execute like this

~~~
./sieve 100
~~~
{: .source}

Now lets consider the Fortran version of the same problem.

~~~
module str2int_mod
contains

  elemental subroutine str2int(str,int,stat)
    implicit none
    ! Arguments
    character(len=*),intent(in) :: str
    integer,intent(out)         :: int
    integer,intent(out)         :: stat

    read(str,*,iostat=stat)  int
  end subroutine str2int

end module

program sieve

  use str2int_mod
  implicit none

  integer :: i, stat, i_max=0
  logical, dimension(:), allocatable :: is_prime
  character(len=32) :: arg

  i = 0
  do
    call get_command_argument(i, arg)
    if (len_trim(arg) == 0) exit

    i = i+1
    if ( i == 2 ) then
       call str2int(trim(arg), i_max, stat)
       write(*,*) "Sieve for prime numbers up to", i_max
    end if

  end do

  if (i_max .lt. 1) then
     write (*,*) "Enter the maximum number to search for primes"
     call exit(1)
  end if

  allocate(is_prime(i_max))

  is_prime = .true.
  is_prime (1) = .false.
  do i = 2, int (sqrt (real (i_max)))
    if (is_prime (i)) is_prime (i * i : i_max : i) = .false.
  end do
  do i = 1, i_max
    if (is_prime (i)) write (*, '(i0, 1x)', advance = 'no') i
  end do
  write (*, *)

end program sieve
~~~
{: .source}


You can notice the particular differences of this language compared with C, working with arrays is in general easier with Fortran.

You can compile this code using the code at

~~~
Day3_AdvancedTopics/2.Programming
~~~
{: .source}

~~~
gfortran sieve.f90 -o sieve
~~~
{: .source}

and execute like this

~~~
./sieve 100
~~~
{: .source}

Finally, this is the version of the Sieve written in python

~~~
#!/usr/bin/env python

from __future__ import print_function
import sys

def primes_upto(limit):
    is_prime = [False] * 2 + [True] * (limit - 1)
    for n in range(int(limit**0.5 + 1.5)): # stop at ``sqrt(limit)``
        if is_prime[n]:
            for i in range(n*n, limit+1, n):
                is_prime[i] = False
    return [i for i, prime in enumerate(is_prime) if prime]

if __name__=='__main__':

    if len(sys.argv)==1:
        print("Enter the maximum number to search for primes")
        sys.exit(1)
    limit = int(sys.argv[1])
    primes = primes_upto(limit)
    for i in primes:
        print(i, end=' ')
    print()
~~~
{: .source}

Python is an interpreted language so you do not need to compile it, instead directly execute the code at:

~~~
Day3_AdvancedTopics/2.Programming
~~~
{: .source}

using the command line:

~~~
python sieve.py 100
~~~
{: .source}

### Matrix inversion

The purpose here is not to show the algorithm behind matrix inversion but to show how that could be achieve in several programming languages using external libraries, in particular we will show you the problem solved using LAPACK for Fortran, GSL for C and Numpy for python

Lets start with the Fortran version. BLAS and LAPACK are a set of well known libraries to perform Linear Algebra calculations. This is a simple example of inverting a real matrix.

~~~
program inverse_matrix
  implicit none
  double precision, allocatable, dimension(:,:) :: a, ainv
  double precision, allocatable, dimension(:) :: work
  integer :: i,j, lwork

  integer :: info, lda,	m, n
  integer, allocatable, dimension(:) :: ipiv

  integer deallocatestatus
  character(len=15) :: mformat='(100(E14.6,1x))'

  external dgetrf
  external dgetri

  n = 4
  lda = n
  lwork = n*n
  allocate (a(lda,n))
  allocate (ainv(lda,n))
  allocate (work(lwork))
  allocate (ipiv(n))

  call random_seed()
  call random_number(a)

  print '(" ")'
  print*,"LU matrix:"  
  do i = 1, n
     write(*,mformat) (a(i,j), j = 1, n)
  end do

  print '(" ")'
  ! dgetrf computes an lu factorization of a general m-by-n matrix a
  ! using partial pivoting with row interchanges.

  m=n
  lda=n

  ! store a in ainv to prevent it from being overwritten by lapack
  ainv = a

  call dgetrf( m, n, ainv, lda, ipiv, info )

  if(info.eq.0)then
     print '(" LU decomposition successful ")'
  endif
  if(info.lt.0)then
     print '(" LU decomposition:  illegal value ")'
     stop
  endif
  if(info.gt.0)then
     write(*,'(a,i4)') 'LU decomposition return',info
  endif

  print '(" ")'
  print*,"LU matrix:"
  do i = 1, n
     write(*,mformat) (ainv(i,j), j = 1, n)
  end do

  !  dgetri computes the inverse of a matrix using the lu factorization
  !  computed by dgetrf.
  call dgetri(n, ainv, n, ipiv, work, lwork, info)

  print '(" ")'
  if (info.ne.0) then
     stop 'Matrix inversion failed!'
  else
     print '(" Inverse successful ")'
  endif

  print '(" ")'
  print*,"Inverse matrix:"
  do i = 1, n
     write(*,mformat)(ainv(i,j), j = 1, n)
  end do

  print '(" ")'

  deallocate (a, stat = deallocatestatus)
  deallocate (ainv, stat = deallocatestatus)
  deallocate (ipiv, stat = deallocatestatus)
  deallocate (work, stat = deallocatestatus)

  print '(" done ")'
  print '(" ")'

  stop
end program inverse_matrix
~~~
{: .source}

The C version is implemented using GSL:

~~~
#include <stdio.h>
#include <gsl/gsl_matrix.h>
#include <gsl/gsl_linalg.h>

int main(int argc, const char * argv[])
{
  // Declare pointer variables for a gsl matrix
  gsl_matrix *A, *Ainverse;
  gsl_permutation *p;
  int i, j, s, status, N=4;

  // Create the matrix
  A = gsl_matrix_alloc(N, N);
  Ainverse = gsl_matrix_alloc(N, N);
  p = gsl_permutation_alloc (N);

  for(i=0; i<N; i++) {
      for(j=0; j<N; j++) {
        gsl_matrix_set(A,i,j,drand48());
    }
  }

  // Print the initial matrix
  printf("Initial Matrix\n");
  for (i=0;i<N;i++)
    {
      for (j=0;j<N;j++)
	     {
	        printf("%16.4f ",gsl_matrix_get(A,i,j));
        }
      printf("\n");
    }
  printf("\n");

  status = gsl_linalg_LU_decomp(A, p, &s);
  printf("Status of decomposition %d\n", status);

  status = gsl_linalg_LU_invert (A, p, Ainverse);
  printf("Status of inversion %d\n", status);

  // Print the initial matrix
  printf("Inverse Matrix\n");
  for (i=0;i<N;i++)
    {
      for (j=0;j<N;j++)
	     {
	        printf("%16.4f ",gsl_matrix_get(Ainverse,i,j));
        }
      printf("\n");
    }
  printf("\n");

  // Clean up
  gsl_permutation_free (p);
  gsl_matrix_free(A);
  gsl_matrix_free(Ainverse);

  return 0;
}
~~~
{: .source}

### Storing data with HDF5

HDF5 is a library to store data in a binary file.
Lets see one example:

~~~

/*
 *  This example illustrates how to create a dataset
 *  array.  It is used in the HDF5 Tutorial.
 */

#include <stdio.h>
#include <stdlib.h>
#include "hdf5.h"

#define H5FILE "dset.h5"
#define NX     100000                      /* dataset dimensions */
#define NY     3


int main() {

  FILE        *fp;
  hid_t       file_id, dataset_id, dataspace_id;  /* identifiers */
  hsize_t     dims[2];
  herr_t      status;
  double      data[NX][NY];          /* data to write */
  int         i, j;

  /*
   * Data  and output buffer initialization.
   */
  for (j = 0; j < NX; j++) {
    for (i = 0; i < NY; i++)
      data[j][i] = drand48();
  }

  /* Create a new file using default properties. */
  file_id = H5Fcreate(H5FILE, H5F_ACC_TRUNC, H5P_DEFAULT, H5P_DEFAULT);

  /* Create the data space for the dataset. */
  dims[0] = NX;
  dims[1] = NY;
  dataspace_id = H5Screate_simple(2, dims, NULL);

  /* Create the dataset. */
  dataset_id = H5Dcreate2(file_id, "/dset", H5T_NATIVE_DOUBLE, dataspace_id,
                          H5P_DEFAULT, H5P_DEFAULT, H5P_DEFAULT);

  /*
   * Write the data to the dataset using default transfer properties.
   */
  status = H5Dwrite(dataset_id, H5T_NATIVE_DOUBLE, H5S_ALL, H5S_ALL,
		    H5P_DEFAULT, data);

  /* End access to the dataset and release resources used by it. */
  status = H5Dclose(dataset_id);

  /* Terminate access to the data space. */
  status = H5Sclose(dataspace_id);

  /* Close the file. */
  status = H5Fclose(file_id);

  fp=fopen("dset.txt","w");
  for (j = 0; j < NX; j++) {
    for (i = 0; i < NY; i++)
      fprintf(fp, "%.16e ", data[j][i]);
    fprintf(fp, "\n");
  }
  fclose(fp);

}
~~~
{: .source}







{% include links.md %}
