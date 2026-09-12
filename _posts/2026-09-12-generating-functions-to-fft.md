---
layout: post
title: "From Generating Functions to the Fast Fourier Transform"
date: 2026-09-12
description: "How roots of unity, divide-and-conquer, and butterfly computation turn the DFT into the FFT."
author: Kehan Wu
tags:
  - Signal Processing
  - FFT
  - DFT
categories:
  - Technical Notes
real_post: true
related_posts: false
giscus_comments: false
disqus_comments: false
---

The fast Fourier transform (FFT) can initially look like a collection of clever implementation tricks. Its speed, however, comes from a deeper algebraic structure: the symmetry of roots of unity and the way that symmetry repeats at progressively smaller scales. Starting with generating functions makes this structure easier to see and leads naturally from polynomial evaluation to the discrete Fourier transform (DFT), then to the radix-2 FFT.

## 1. A Generating-Function View

A finite sequence of numbers can be encoded as the coefficients of a polynomial:

$$
f(x) = \sum_{i=0}^{N-1} a_i x^i.
$$

The coefficient $a_i$ may represent a count, a measurement, or any other discrete quantity indexed by $i$. This viewpoint is useful because algebraic operations on $f(x)$ manipulate all coefficients at once. For example, multiplying two such polynomials combines their coefficient sequences through convolution.

Evaluation provides another way to extract structure. At an arbitrary value of $x$, every coefficient contributes to the result with a different weight $x^i$. At carefully chosen complex values, these weights follow symmetric patterns: selected contributions reinforce one another, while others cancel. Generating functions therefore motivate an important change of representation—from storing a polynomial by its coefficients to describing it by values sampled at special points.

The two representations emphasize different operations. Coefficients make addition straightforward and expose individual samples; point values make multiplication straightforward because corresponding evaluations can be multiplied independently. Efficiently moving between these views is therefore valuable well beyond spectral plots. The Fourier transform supplies exactly such a structured set of evaluation points, and the FFT makes the conversion computationally practical.

## 2. Roots of Unity

Define the primitive $N$-th root of unity as

$$
W_N = e^{-j2\pi/N}.
$$

The values $1, W_N, W_N^2, \ldots, W_N^{N-1}$ are equally spaced around the complex unit circle. Rotating by $W_N$ exactly $N$ times returns to the starting point, so $W_N^N=1$. Their geometric symmetry produces the cancellation identity

$$
\sum_{k=0}^{N-1} W_N^{km}=0,
$$

whenever $m$ is not a multiple of $N$. The terms form a regular polygon in the complex plane, and their vector sum vanishes. When the phases align instead, the contributions reinforce.

This cancellation-and-reinforcement mechanism is central to Fourier analysis. Sampling a coefficient polynomial at successive roots of unity tests how strongly the sequence aligns with different complex oscillations. Each evaluation isolates one frequency pattern from the others.

## 3. The Discrete Fourier Transform

For an input sequence $x[n]$ of length $N$, the DFT is

$$
X[k] = \sum_{n=0}^{N-1} x[n]e^{-j2\pi kn/N}
     = \sum_{n=0}^{N-1} x[n]W_N^{kn},
$$

for $k=0,1,\ldots,N-1$. Here, $x[n]$ is the input sequence, $X[k]$ is its $k$-th frequency-domain component, and $W_N^{kn}$ is the twiddle factor that sets the phase used for that component.

This formula is also polynomial evaluation: if $f(z)=\sum_n x[n]z^n$, then $X[k]=f(W_N^k)$. The DFT represents the same information as the original sequence, but in a basis of discrete complex frequencies.

A direct implementation computes $N$ output values. Each output requires a sum over approximately $N$ input values, giving about $N^2$ multiply-add operations overall. Its computational complexity is therefore

$$
O(N^2).
$$

That cost becomes prohibitive for large signals or repeated real-time analysis.

## 4. From DFT to FFT

Assume that $N$ is even. Split the DFT sum into samples with even and odd indices:

$$
X[k]
= \sum_{r=0}^{N/2-1} x[2r]W_N^{2rk}
+ W_N^k\sum_{r=0}^{N/2-1}x[2r+1]W_N^{2rk}.
$$

Because $W_N^2=W_{N/2}$, both sums are themselves $N/2$-point DFTs. Define

$$
E[k]=\sum_{r=0}^{N/2-1}x[2r]W_{N/2}^{rk},
\qquad
O[k]=\sum_{r=0}^{N/2-1}x[2r+1]W_{N/2}^{rk}.
$$

The first half of the output follows immediately:

$$
X[k]=E[k]+W_N^kO[k].
$$

The periodicity of the smaller transforms and the identity $W_N^{k+N/2}=-W_N^k$ give the second half:

$$
X[k+N/2]=E[k]-W_N^kO[k],
$$

where $k=0,1,\ldots,N/2-1$. The important point is reuse: one pair $E[k]$ and $O[k]$ produces two output components. A direct DFT would calculate those outputs independently and repeat much of the same work.

## 5. Divide and Conquer

The even and odd transforms have exactly the same form as the original problem, only half the size. The decomposition can therefore be applied recursively:

```text
N
↓
N/2 + N/2
↓
N/4 + N/4 + N/4 + N/4
↓
...
↓
2-point DFTs
```

For a power-of-two length, there are $\log_2 N$ decomposition stages. Across any one stage, all subproblems together contain $N$ values, so combining their results requires $O(N)$ work. The total complexity becomes

$$
O(N)\times O(\log N)=O(N\log N).
$$

The transform has not changed; only the evaluation schedule has. The speedup comes from exposing repeated subproblems and avoiding redundant computation.

## 6. Butterfly Computation

The radix-2 butterfly is the basic combination step. Given outputs $A$ and $B$ from two smaller transforms, it computes

$$
A+W_N^kB
\qquad\text{and}\qquad
A-W_N^kB.
$$

These are the two equations derived above for $X[k]$ and $X[k+N/2]$. Repeating this compact operation combines small Fourier transforms into progressively larger ones. The name “butterfly” comes from the crossed shape made by the data paths in a signal-flow diagram.

## 7. Bit-Reversal Ordering

In a radix-2 decimation-in-time FFT, recursive even/odd splitting groups indices according to their binary digits. An iterative implementation commonly arranges the inputs in bit-reversed order so that the butterfly stages can then proceed regularly. For $N=8$:

```text
0 → 000 → 000 → 0
1 → 001 → 100 → 4
2 → 010 → 010 → 2
3 → 011 → 110 → 6
4 → 100 → 001 → 1
5 → 101 → 101 → 5
6 → 110 → 011 → 3
7 → 111 → 111 → 7
```

The resulting order is $0,4,2,6,1,5,3,7$. Bit reversal is not an unrelated indexing trick; it is the iterative trace of the same recursive even/odd decomposition.

## 8. DFT vs. FFT

The distinction is conceptual:

- **DFT:** the mathematical transform that maps a finite sequence to its discrete frequency coefficients. Direct evaluation costs $O(N^2)$.
- **FFT:** a family of efficient algorithms for computing that DFT by exploiting the symmetry and periodicity of roots of unity. A radix-2 FFT costs $O(N\log N)$.

The FFT is not a different transform from the DFT. Given the same input and numerical precision, it produces the same Fourier coefficients more efficiently.

## 9. Takeaway

The FFT is more than a low-level optimization. Roots of unity make the DFT highly symmetric, while even/odd decomposition turns that symmetry into reusable subproblems. Butterfly operations combine those subproblems across roughly $\log_2N$ stages, reducing the cost from quadratic to quasilinear time. This algebraic structure is what makes large-scale and real-time spectral analysis practical.
