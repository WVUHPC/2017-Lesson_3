---
title: "Parallel Programming with OpenMP"
teaching: 45
exercises: 15
questions:
- "Key question"
objectives:
- "Introduce OpenMP and API for parallel computing on SMP machines."
keypoints:
- "First key point."
---

# Parallel Programming with OpenMP

OpenMP is an Application Program Interface (API). 
OpenMP provides a portable, scalable model for developers of shared memory parallel applications. 
OpenMP works with Symmetric Multiprocessing (SMP)
The API supports C/C++ and Fortran on a wide variety of architectures. 
This tutorial covers some of the major features of OpenMP 4.0, including its various constructs and directives for specifying parallel regions, work sharing, synchronization and data environment. 
Runtime library functions and environment variables are also covered. 
This tutorial includes both C and Fortran example codes and a lab exercise.

## What is a Symmetric Multiprocessing (SMP)?

Symmetric multiprocessing (SMP) involves a multiprocessor computer hardware and software architecture where two or more identical processors are connected to a single, shared main memory, have full access to all I/O devices, and are controlled by a single operating system instance that treats all processors equally, reserving none for special purposes. Most multiprocessor systems today use an SMP architecture. In the case of multi-core processors, the SMP architecture applies to the cores, treating them as separate processors.

<a href="https://upload.wikimedia.org/wikipedia/commons/1/1c/SMP_-_Symmetric_Multiprocessor_System.svg">
  <img src="https://upload.wikimedia.org/wikipedia/commons/1/1c/SMP_-_Symmetric_Multiprocessor_System.svg" alt="SMP" />
</a>

{% include links.md %}
