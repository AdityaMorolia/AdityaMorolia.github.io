---
layout: post
title: Fastest Known Algorithms for Lattice Problems
tags: [Lattices, Algorithms, Quantum Algorithms]
description: The fastest known algorithms for SVP, LWE, and CVP.
date: 2026-11-27
updated: 2026-11-27
---

I see a lot of confusion on this question because of the recent flurry of papers on the subject. So here is my attempt to clarify. 
I should mention that there are lots of parallel work that I have skipped, these are the fastest (asymptotically or in practice). 

## Shortest Vector Problem (SVP)

Given a basis of a rank-$n$ lattice $L$, exact SVP asks for a shortest nonzero vector. Write $\lambda_1(L)$ for its length. For $\mu$-SVP, any nonzero vector of length at most $\mu\lambda_1(L)$ will do. All lengths here are Euclidean.
The table gives time and space bounds as of November 2026, up to subexponential $2^{o(n)}$ factors.
Space includes the classical memory that the quantum algorithms access coherently.
The last row the the fastest *implementation* of an SVP algorithm for the [SVP Hall of Fame Challenge](#svp-in-practice). 

<style>
.table.svp-bounds {
  width: auto;
}

.svp-bounds td:nth-child(2),
.svp-bounds td:nth-child(3) {
  white-space: nowrap;
}
</style>

<div class="table-responsive" role="region" aria-label="SVP time and space bounds" tabindex="0" markdown="block">

| Problem and guarantee | Classical | Quantum | Reference |
|:---|:---|:---|:---|
| Exact SVP, worst case | Time $2^{0.4085n}$<br>Space $2^{0.2593n}$ | Time $2^{0.2469n}$<br>Space $2^{0.2469n}$ | [HKKL26](#hkkl26) |
| Exact SVP, worst case, under [kissing number assumption](#kissing-number-assumption) | Time $2^{0.2925n}$<br>Space $2^{0.2075n}$ | Time $2^{0.2469n}$<br>Space $2^{0.2469n}$ | [HKKL26](#hkkl26) |
| $\mu$-SVP, worst case, large constant $\mu$ | Time $2^{0.2925n+O(n/\mu^2)}$<br>Space $2^{0.2075n+O(n/\mu^2)}$ | Time $2^{0.2469n}$<br>Space $2^{0.2469n}$ | [HKKL26](#hkkl26) |
| Exact SVP, Haar-random lattice | Time $2^{0.2925n}$<br>Space $2^{0.2075n}$ | Time $2^{0.2469n}$<br>Space $2^{0.2469n}$ | [HKKL26](#hkkl26) |
| Exact SVP, random $q$-ary kernel, density at most one | Time $2^{0.2741n}$<br>Space $2^{0.2614n}$ | Time $2^{0.2469n}$<br>Space $2^{0.2469n}$ | [KKLL26](#kkll26), [HKKL26](#hkkl26) |
| SVP, heuristic sieving | Time $2^{0.2925n}$<br>Space $2^{0.2075n}$ | Time $2^{0.2563n}$<br>Space $2^{0.2075n}$ | [BDGL16](#bdgl16), [BCSS23](#bcss23) |
| [SVP Challenge](#svp-in-practice), dimension 210 | Time 78.36 days<br>Space IDK | — | [Ding and Zhao](#svp-challenge) |
{: .table .table-sm .svp-bounds }

</div>

The results concert random lattices with two different distributions. 
The first samples a determinant-one lattice from the Haar measure. 
The second samples a uniform $m\times n$ matrix $H$ modulo $q$ and take all integer vectors $x$ satisfying $Hx=0\bmod q$. The density condition is $m\log_2(q)\le n$.

### Kissing number assumption

How many unit balls can touch ("kiss") one unit ball without overlapping? This is the famous [*kissing number*](https://en.wikipedia.org/wiki/Kissing_number) problem. 
It is equivalent to the largest number of points one can have on an $n$ dimensional unit sphere such that the angular distance between every pair is at least $60^\circ$.

Write $A(n,\theta)$ for the largest number of points on the unit sphere in $\mathbb R^n$ with pairwise angles at least $\theta$. The conjecture says that for each fixed $0<\theta<\pi/2$, we have 

$$A(n,\theta)\le 2^{o(n)}(\sin\theta)^{-n}.$$ 

Here we only care about the $\theta = 60^\circ$ case. The conjecture says that the kissing number is at most $(4/3)^{n/2+o(n)} \approx 2^{0.2075n+o(n)}$, matching the known lower bound up to sub-exponential factors. 
This limits how many short lattice vectors the classical algorithm has to account for.
(Notice that this is where the space complexity numbers above come from: once any algorithm finds this many vectors of roughly the same norm, it can create shorter vectors for sure.) 

### SVP in practice

The biggest instance of SVP that is solved on the SVP Challenge leaderboard is of **dimension 210**. On January 1, 2026, Jintai Ding and Ziyu Zhao submitted a vector of length $1.04497$ times the Gaussian heuristic’s prediction. Their implementation of the [Becker–Gama–Joux sieve](https://eprint.iacr.org/2015/522), BGJ-Sieve-GPU, spent 56.05 days on preprocessing and 22.31 days on the final sieve and lifting. The largest sieve they actually ran had dimension 171. 

[SVP Challenge Hall of Fame](https://latticechallenge.org/svp-challenge/halloffame.php)

[Their GitHub Implementation](https://github.com/zhaoziyu0008/BGJ-Sieve-GPU)


## Learning With Errors (LWE)

Recall that in search LWE we get $m$ noisy linear equations of the form $b=\langle a, s \rangle+e\bmod q$ and want to recover the $n$-dimensional secret $s$. 
The secret $s$ is uniformly random in $\mathbb Z_q^n$, and so is each $a$. The errors are discrete Gaussian samples of width $\sigma$.
This one is tricky, since there is shit ton of parameter regimes people consider. But basically the hardness of the problem is linked to a parameter $\sigma$ that controls the noise distribution, and the modulus $q$ (and of course the number of samples $m$). 

**For a concrete parameters.** People have built the [Lattice Estimator](https://github.com/malb/lattice-estimator). It compares the costs of various algorithms for a given modulus, sample count, and choice of secret and error distributions. If the algorithm uses a sieve, it’s the *sieving dimension* that goes into the SVP exponent. 

The largest secret dimension on the LWE Challenge is **100**, at relative error $\alpha = 0.005$ (this is $\sigma / q$). Jintai Ding’s submission used a sieve of dimension **167** and consumed 419.7 RTX 4090 GPU-days.

[LWE Challenge submission](https://www.latticechallenge.org/lwe_challenge/details.php?alpha=0.005&n=100)

**Remark.** They don't mention how they go from SVP to LWE for this particular submission, but from [their github implementation](https://github.com/zhaoziyu0008/BGJ-Sieve-GPU) it seems that they embed the LWE samples in a lattice wehre a unique shortest vector encodes the error $e$. They then *probably* use the [BKZ algorithm](https://blog.simons.berkeley.edu/2020/04/lattice-blog-reduction-part-i-bkz/) to do [basis reduction](https://blog.simons.berkeley.edu/2020/04/lattice-blog-reduction-part-i-bkz/) using the SVP algorithm. Once you know $e$, you can find $s$ by solving a linear system. 
Who knows what they actually do? (Probably Astra, but I am too tired to go through the slop and make sense of it right now.)


## References

- **HKKL26**{: #hkkl26} · Minki Hhan, Jiseung Kim, Sunyeop Kim, and Thijs Laarhoven. [Superlattice methods for the shortest vector problem and discrete Gaussian sampling](https://eprint.iacr.org/2026/2761).
- **KKLL26**{: #kkll26} · Jiseung Kim, Junhyuk Kwon, Changmin Lee, and JeongHwan Lee. [Fast Provable Algorithms for Approximate CVP and Exact SVP on Random $q$-ary Lattices](https://opencryptology.org/2026/2711).
- **BDGL16**{: #bdgl16} · Anja Becker, Léo Ducas, Nicolas Gama, and Thijs Laarhoven. [New directions in nearest neighbor searching with applications to lattice sieving](https://eprint.iacr.org/2015/1128).
- **BCSS23**{: #bcss23} · Xavier Bonnetain, André Chailloux, André Schrottenloher, and Yixin Shen. [Finding Many Collisions via Reusable Quantum Walks](https://eprint.iacr.org/2022/676).
- **BGJ15** · Anja Becker, Nicolas Gama, and Antoine Joux. [Speeding-up lattice sieving without increasing the memory, using sub-quadratic nearest neighbor search](https://eprint.iacr.org/2015/522).
- **SVP Challenge**{: #svp-challenge} · [Rules](https://latticechallenge.org/svp-challenge/) · [Hall of Fame and submission timings](https://latticechallenge.org/svp-challenge/halloffame.php) · [Zhao and Ding’s paper on disk-based sieving](https://link.springer.com/chapter/10.1007/978-3-032-25327-9_6) · [BGJ-Sieve-GPU](https://github.com/zhaoziyu0008/BGJ-Sieve-GPU).
- **LWE resources**{: #lwe-resources} · [Lattice Estimator](https://github.com/malb/lattice-estimator) · [LWE Challenge Hall of Fame](https://www.latticechallenge.org/lwe_challenge/halloffame.php) · [Dimension-100 submission](https://www.latticechallenge.org/lwe_challenge/details.php?alpha=0.005&n=100).
