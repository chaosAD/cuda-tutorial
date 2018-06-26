# Lab 02: CUDA in Actions

## Introduction 

In [tutorial 01](../lab01/), we implemented vector addition in CUDA. However, the program runs using only one GPU thread. The strength of GPU lies in its massive parallelism. In this tutorial, we will explore how to implement parallel program for GPUs. 

## Going parallel

CUDA use a kernel execution configuration `<<<...>>>` to tell CUDA runtime how many threads to launch on GPU. Threads are organized in a group called "*thread block*", where all threads inside the same thread block will run on the same multiprocessor. Kernel can launch multiple thread blocks, organized into a "*grid*" structure. 

The syntax of kernel execution configuration is as follows 

```C
<<< M , T >>>
```

Which indicate that a kernel launches with a grid of `M` thread blocks. Each thread block has `T` parallel threads. 

## Exercise 1: Parallelizing vector addition using thread block level parallelism

In this exercise, we will parallelize vector addition from tutorial 1 ([`vector_add.cu`](./vector_add.cu)) using a thread block with 256 threads. The new kernel execution configuration is shown below. 

```C
vector_add <<< 1 , 256 >>> (out, d_a, b, N);
```

CUDA provides built-in variables for accessing thread information. In this exercise, we will use two of them: `threadIdx.x` and `blockIdx.x`. `threadIdx.x` stores the index of current thread in the block and `blockDim.x` stores the size of thread block (number of threads in the thread block). 

For the `vector_add()` configuration, the value of `blockDim.x` is 256. For `i`-th threads, the value of `threadIdx.x` is `i`. The value of `i` ranges from 0 to 255. 

### Parallelizing idea

Recalls the kernel of single thread version in [`vector_add.cu`](./vector_add.cu)

```C
__global__ void vector_add(float *out, float *a, float *b, int n) {
    for(int i = 0; i < n; i++){
        out[i] = a[i] + b[i];
    }
}
```

 In this implementation, one thread iterates through and computes the element-wise summation of the whole array. With a thread block of 256 threads, we can perform the summation of 256 elements simultaneously instead of one by one. 

The main idea is to virtually split the array into subarrays, where each subarray contains 256 elements. Instead of iterate through each element, the loop iterates through each subarray. The summation of the subarray is computed simultaneously by assigning the `i`-th thread to the `i`-th element of the subarray. 

![parallel thread](./01_parallel_thread.png "parallel thread")

<!--
When converting this idea back to the whole array, we get the following 
* In the `0`-th iteration, the `i`-th thread computes the summation of `a[i]` and `b[i]`. 
* In the `1`-st iteration, the `i`-th thread computes the summation of `a[i + 256]` and `b[i + 256]`. 
* ...
* In the `n`-th iteration, the `i`-th thread computes the summation of `a[i + 256 * n]` and `b[i + 256 * n]`. 
-->

**EXERCISE: Try to implement this in `vector_add_thread.cu`**

1. Copy `vector_add.cu` to `vector_add_thread.cu`

```bash
$> cp vector_add.cu vector_add_thread.cu
```

2. Parallelize `vector_add()`. 

3. Compile and profile the program

```bash
$> nvcc vector_add_thread.cu -o vector_add_thread
$> nvprof ./vector_add_thread
```

See the solution in [`solutions/vector_add_thread.cu`](./solutions/vector_add_thread.cu)


## Exercise 2: Using whole GPU





## Wrap up


## Acknowledgments

* Contents are adopted from [An Even Easier Introduction to CUDA](https://devblogs.nvidia.com/even-easier-introduction-cuda/) by Mark Harris, NVIDIA and [CUDA C/C++ Basics](http://www.int.washington.edu/PROGRAMS/12-2c/week3/clark_01.pdf) by Cyril Zeller, NVIDIA. 

