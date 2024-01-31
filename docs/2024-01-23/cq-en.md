# An Introduction to Cached Quotients Lookup Arguments

This article is supported by the [PSE Acceleration Program](https://github.com/privacy-scaling-explorations/acceleration-program).

Code: [pylookup](https://github.com/luckyyang/pylookup/tree/cq_fk/src/cq)

This code is an implementation based on the CQ paper. Please see `README.md` to learn how to run it.

Additionally, it includes a file `cq.ipynb`, which is a Python notebook version of the implementation. It's a simplified version of the CQ implementation and is quite similar to this tutorial.
## The Problem Lookup Arguments Solve

In zero-knowledge proof systems, it's common to use arithmetic processes to convert a computational problem into an equivalent mathematical expression (usually a polynomial). This allows the use of mathematical properties to transform, compute, and verify polynomials. This is a key step in constructing zero-knowledge proofs because it enables complex computational processes to be precisely expressed and verified in mathematics.

However, some computations cannot be effectively arithmetized, or their arithmetization may produce more constraints (polynomials). For example, proving a value is within a range ($0<x<5$) might require more than 10 constraints. In such cases, using lookup arguments can reduce the number of constraints and the size of the generated proof.

Lookup arguments allow the prover to demonstrate that a value belongs to a predefined set.

Taking the previous example, to prove $0<x<5$, we only need one lookup constraint: $x \in \{1, 2, 3, 4\}$, that is, proving $x$ is in the table $\{1, 2, 3, 4\}$.

| Table |
|-------|
| 1     |
| 2     |
| 3     |
| 4     |

The above example is a single-column table. Further, a table can express any relationship, as long as the corresponding input and output can be found in the table, we can say that the lookup proof is valid.

| Input 1 |  Input 2 | Input 3 | Output |
|-------|--------|-------|-------|
| 1     |  2      |   3   |   6   |
| 2     | 3       | 4     |   24   |
| 2     | 3      | 8     |   48   |
| 4     | 8      | 3    |   13   |
| 13    | 6     |    4   |   33   |

For example, the first row of the table can be defined as an addition: $1+2+3=6$. As long as the given lookup is $\{1, 2, 3, 6\}$, it satisfies the lookup constraint; the second row can be defined as multiplication: $2 * 3 * 4 = 24$, and as long as the given lookup is $\{2, 3, 4, 24\}$, it meets the lookup constraint.

## The Contribution of CQ

As seen in the comparison from the paper (illustrated below), CQ (Cached Quotients Lookup Arguments) currently has the lowest computational complexity for the prover among lookup arguments, standing at $O(n \log n)$. This is achieved through certain preprocessing steps. Compared to Baloo, CQ offers fewer group operations and a smaller proof size.

![[Pasted image 20240116145259.png]]

The main difference between CQ and the four lookup arguments mentioned in the above table is that CQ utilizes the technique of logarithmic derivatives. This allows the prover's computations to be based on the original large table, without needing to extract a subtable from the large table, thereby making the prover's computations more efficient.

Below, we explain the core principle of CQ. If you're interested in more details and optimizations, please refer to the original paper.
## Logarithmic Derivatives

Lookup arguments is about proving that elements of a vector/table $\vec f$ are all contained within $\vec t$. This is consistent with other lookup arguments.

Assume:

- $\vec f = \{f_1,  f_2,  f_3,  f_4,  ...,  f_n\}$
- $\vec t = \{t_1,  t_2,  t_3,  t_4, ..,  t_n\}$

For example:

-  $\vec f = \{1,  2,  2,  3,  3,  3\}$
-  $\vec t = \{1,  2,  3,  4\}$

The above problem can be transformed into a new one: construct a polynomial using these elements, which have the same roots:
- $f(x) = (x-1)(x-2)(x-2)(x-3)(x-3)(x-3)$
- $t(x) = (x-1)^1(x-2)^2(x-3)^3(x-4)^0$

The advantage here is that we can utilize the mathematical properties of polynomials for subsequent calculations and proofs.

Note that in $t(x)$, the times of each element in $\vec f$ needs to be counted as the power of the corresponding root in the polynomial $t(x)$. The times of elements not present is 0. As seen, the times of elements of $\vec f$ in $\vec t$ is:

- 1 appears 1 time
- 2 appears 2 times
- 3 appears 3 times
- 4 appears 0 time

This can be represented by a vector $\vec m$:

$\vec m = \{1,  2,  3,  0\}$

Hence, $t(x)$ is constructed as: $t(x) = (x-1)^1(x-2)^2(x-3)^3(x-4)^0$.

If $f(x)$ equals to $t(x)$, it means that the elements of $\vec f$ are all in $\vec t$, satisfying the lookup argument proof. 

Constructing a multiplicative polynomial has another benefit: the order of elements in $\vec f$ and $\vec t$ does not need to be considered.

However, in practice, we usually do not directly compare in this way because:
1. $\vec f$ is usually a secret value and not public, so a direct comparison is not possible.
2. The size of $\vec f$ and $\vec t$ is generally large, and the polynomials constructed have high degrees, making them relatively difficult to handle.

Is there a better way to prove $f(x) = t(x)$?

Yes. CQ uses the method of "logarithmic derivatives" mentioned in the paper [Multivariate lookups based on logarithmic derivatives](https://eprint.iacr.org/2022/1530.pdf), transforming the multiplication relationship into an addition relationship, making the proof much more concise.

What is logarithmic derivative?

In mathematics, logarithmic derivative is a technique for finding the derivative of a function/polynomial, especially useful when the function is a product of multiple functions. This method simplifies the derivation process by applying the natural logarithm ($ln$) and the chain rule.

Let's calculate the logarithmic derivative of $f(x) = (x-1)(x-2)^2(x-3)^3$. Following the steps of logarithmic derivative, we first take the logarithm of $f(x)$ and then differentiate the resulting expression. Here are the steps:

**Taking the logarithm**:

$\ln(f(x)) = \ln((x-1)(x-2)(x-2)(x-3)(x-3)(x-3))$

**Expanding the logarithm** (using logarithmic properties to convert multiplication to addition):

$\ln(f(x)) = \ln(x-1) + \ln(x-2)+ \ln(x-2)+ \ln(x-3)+ \ln(x-3) + \ln(x-3)$

**Differentiating both sides** (applying the chain rule and basic derivative rules):

$\frac{d}{dx}\ln(f(x)) = \frac{d}{dx}(\ln(x-1) + \ln(x-2)+ \ln(x-2)+ \ln(x-3)+ \ln(x-3) + \ln(x-3))$

Simplifying to:

$\frac{f'(x)}{f(x)} = \frac{1}{x-1} + \frac{1}{x-2}+ \frac{1}{x-2} + \frac{1}{x-3} + \frac{1}{x-3} + \frac{1}{x-3}$

So:

$\frac{f'(x)}{f(x)} = \sum_{j \in |\vec f|} \frac{1}{x-f_j}$

Similarly, the logarithmic derivative of $t(x)$ is:

$\frac{t'(x)}{t(x)} = \frac{1}{x-1} + \frac{2}{x-2}+ \frac{3}{x-3}$

So:

$\frac{t'(x)}{t(x)} = \sum_{i \in |\vec t|} \frac{m_i}{x-t_j}$

Therefore, we need to prove:

$\frac{f'(x)}{f(x)} = \frac{t'(x)}{t(x)}$

Or:

$\sum_{j \in |\vec f|} \frac{1}{x-f_j} = \sum_{i \in |\vec t|} \frac{m_i}{x-t_i}$

Simplifying further, define $a_i = m_i / (x - t_i)$ and $b_j = 1 / (x - f_j)$,

We need to prove:

$\sum_i a_i = \sum_j b_j$ 

This process transforms the product of multiple functions into a summation form. Ultimately, the lookup proof is converted to proving the equality of logarithmic derivatives ($\frac{f'(x)}{f(x)}$ and $\frac{t'(x)}{t(x)}$).

## Logarithmic Derivative Lookup Protocol

Through the above logarithmic derivatives, we can construct a simple lookup argument:

- The prover sends $\vec f, \vec t, \vec m$ to the verifier.
- The verifier sends a random number $\beta$ to the prover.
- The prover sends multiple values to the verifier: $a_i = m_i / (\beta - t_i)$ and $b_j = 1 / (\beta - f_j)$, where the ranges of $i$ and $j$ are the number of elements in $\vec t$ and $\vec f$, respectively, $i \in |\vec t|, j \in |\vec f|$.
- The verifier performs three types of checks:
	- Check of $a_i$ values: $a_i \times (\beta - t_i) = m_i$.
	- Check of $b_i$ values: $b_i \times (\beta - f_i) = 1$.
	- Sum check: $\sum_i a_i = \sum_j b_j$.

For the checks of $a_i$ and $b_i$, the above lookup protocol has an issue: the proof size is relatively large because all the $a_i$ and $b_i$ values need to be sent to the verifier. To resolve this issue, we can use the Schwartz-Zippel lemma, first encode all the $a_i$ and $b_i$ values into polynomials, then, the verifier only needs to send a random value for verification, which ensures that all the values of $a_i$ and $b_i$ provided by the prover are correct.

For the sum check, we can directly use the univariate sumcheck.

Next, let's see how to construct the corresponding proofs.
## Polynomial Construction and Pairing: Proving $a_i$ and $b_i$

To verify:

- The correctness of $a_i$: $a_i \times (\beta - t_i) = m_i$.
- The correctness of $b_i$: $b_i \times (\beta - f_i) = 1$.

We construct polynomials for $a_i$ and $b_i$ respectively and then use the properties of polynomials for verification.

This requires:

- Constructing polynomial $A(x)$ from $a_i$ and verifying its correctness.
- Constructing polynomial $B(x)$ from $b_i$ and verifying its correctness.

Polynomials are usually constructed using Lagrange interpolation. This requires constructing a series of $\{x_i, y_i\}$ value pairs to interpolate into a polynomial, usually $\{x_i\}$ values come from a multiplicative subgroup, the size of the group is the size of $\{y_i\}$, here respectively $|\vec t|$ and $|\vec f|$.

How to construct $A(x)$?

1. The verifier sends a random number $\beta$, let $x = \beta$.
2. Select a multiplicative subgroup $H_t = \{\omega_i\}$ of size $|\vec t|$, $i \in |\vec t|$.
3. $A(\omega_i) = a_i = m_i / (t_i - \beta)$. The $\{x_i, y_i\}$ value pairs are $\{\omega_i, a_i\}$.
4. Use Lagrange interpolation to construct the polynomial $A(x)$.

How to check if $A(x)$ is correctly constructed?

1. Interpolate $\vec m$ and $\vec t$ on the multiplicative subgroup to get polynomials $M(x), T(x)$.
2. On the multiplicative subgroup $H_t$, the polynomial satisfies $A(x) = M(x)/(T(x) - \beta)$.
3. Transform to $A(x)(T(x) - \beta) - M(x) = 0$.
4. By the polynomial remainder theorem, $A(x)(T(x) - \beta) - M(x) = Q_A(x) * Z_A(x)$. This equation holds for any value of $x$ on the entire group. $Q_A(x)$ is the quotient polynomial, $Z_A(x) = \prod_i(x - \omega_i)$ is the vanishing polynomial.
5. The prover sends:
	1. The commitment of $A(x)$: $[A(x)]_1$.
	2. The commitment of $Q_A(x)$: $[Q_A(x)]_1$.
	3. The commitment of $M(x)$: $[M(x)]_1$.
6. The verifier independently computes the commitments of $T(x)$ $[T(x)]_2$ and $Z_A(x)$ $[Z_A(x)]_2$, then verifies using elliptic curve pairing for KZG verification: $e([T(x)]_2, [A(x)]_1) = e([1]_2, [A(x)]_1 * \beta + [M(x)]_1) * e([Z_A(x)]_2, [Q_A(x)]_1)$.

Note: Different from the paper, for ease of explanation, some places in this article use a $-$ sign instead of a $+$ sign. This is because subtraction operations on the group are transformed into addition operations, e.g., for a group with $modulus = 5$, $\beta=-2\%5 = 3$, so $(x - 2)\%5 = x + 3$. We can always transform subtraction into addition representation, so the symbols are not distinguished in certain scenarios.

Construct $B(x)$ in the same way:

1. The verifier sends a random number $\beta$, let $x = \beta$.
2. Select a multiplicative subgroup $H_f = \{\omega_j\}$ of size $|\vec f|$, $j \in |\vec f|$.
3. $B(\omega_j) = b_j = 1 / (f_j - \beta)$. The $\{x_j, y_j\}$ value pairs are $\{\omega_j, b_j\}$.
4. Use Lagrange interpolation to construct the polynomial $B(x)$.

Check if $B(x)$ is correctly constructed:

1. Interpolate $\vec f$ on the multiplicative subgroup to get polynomial $F(x)$.
2. On the multiplicative subgroup $H_f$, the polynomial satisfies $B(x) = 1/(F(x) - \beta)$.
3. Transform to $B(x)(F(x) - \beta) - 1 = 0$.
4. By the polynomial remainder theorem, $B(x)(F(x) - \beta) - 1 = Q_B(x) * Z_B(x)$. This equation holds for any value of $x$ on the entire group. $Q_B(x)$ is the quotient polynomial, $Z_B(x) = \prod_j(x - \omega_j)$ is the vanishing polynomial.
5. The prover sends:
	1. The commitment of $B(x)$: $[B(x)]_1$.
	2. The commitment of $Q_B(x)$: $[Q_B(x)]_1$.
	3. The commitment of $F(x)$: $[F(x)]_1$.
6. The verifier independently computes the commitment of $Z_B(x)$ $[Z_B(x)]_2$, then verifies using elliptic curve pairing: $e([F(x)]_2, [B(x)]_1) = e([1]_2, [B(x)]_1 * \beta + [1]_1) * e([Z_B(x)]_2, [Q_B(x)]_1)$.

Note: The verification process in the paper has been optimized. For ease of explanation, we do not use the same verification method as in the paper, but the core principle is the same, which is KZG verification through pairing.

$B(x)$ also needs further check to verify the polynomial's degree is correct, specifically by:
- Let $N = |\vec t|, n = |\vec f|$
- The prover sends
	- The commitment of $B(x)$, $b = [B(x)]_1$
	- The commitment of $X^{N−1−(n−1)}$, $r = [X^{N−1−(n−1)}]_2$
	- The commitment of $P(x)$, $p = [P(x)]_2$, where $P(X) := B(X) * X^{N−1−(n−1)}$
- The verifier checks: $e(b, r) = e(p, [1]_2)$

The paper here makes a certain optimization, using $B_0(x)$ instead of $B(x)$, but the goal is the same: to check the degree of the polynomial.

## Sumcheck

Next, we need to verify that the sum holds: $\sum_i a_i = \sum_j b_j$.

After proving that each item on both sides is correct, we now need to prove that the sum of both sides is correct, i.e., they are equal to each other. The method to prove is univariate sumcheck. This can be demonstrated using the following lemma originally used in the Aurora construction ($[BCR+19]$, Remark 5.6):

$$
\sum_{a \in H} f(a) = t \cdot f(0)
$$

Where:

- $H$ is a multiplicative subgroup: $H=\{1, \omega, \omega^2, ..., \omega^{t-2}, \omega^{t-1} \}$, $t$ is the number of elements in $H$, $t=|H|$, $\omega$ is a primitive root.
- $f(x)$ is a polynomial, $f(x) = f_0 + f_1x + f_2x^2 + ... + f_{t-2}x^{t-2} + f_{t-1}x^{t-1}$.

Let's decompose the calculation process, substitute the values of $a$ from $H$ into $f(x)$:

$$
\begin{align*}
& \sum_{a \in H} f(a) = f(1) + f(\omega) + f(\omega^2) + ... + + f(\omega^{t-1}) \\
& \\
& \\
& = f_0 + f_1 + f_2 + ... + f_{t-2} + f_{t-1} \\
& + f_0 + f_1\omega + f_2\omega^2 + ... + f_{t-2}\omega^{t-2} + f_{t-1}\omega^{t-1} \\
& + f_0 + f_1(\omega^2) + f_2(\omega^2)^2 + ... + f_{t-2}(\omega^2)^{t-2} + f_{t-1}(\omega^2)^{t-1} \\
& . \\
& . \\
& . \\
& + f_0 + f_1(\omega^{t-2}) + f_2(\omega^{t-2})^2 + ... + f_{t-2}(\omega^{t-2})^{t-2} + f_{t-1}(\omega^{t-2})^{t-1} \\
& + f_0 + f_1(\omega^{t-1}) + f_2(\omega^{t-1})^2 + ... + f_{t-2}(\omega^{t-1})^{t-2} + f_{t-1}(\omega^{t-1})^{t-1} \\
& \\
& \\
& = t \cdot f_0 + f_1\sum_{i \in t}(\omega)^i + f_2\sum_{i \in t}(\omega^2)^i + ... + f_{t-2}\sum_{i \in t}(\omega^{t-2})^i + f_{t-1}\sum_{i \in t}(\omega^{t-1})^i \\
\end{align*}
$$

In the above formula, except for the first term, the sum of each of the last $t-1$ terms is 0. This is because each $\omega^i$ is an element of the multiplicative subgroup in the finite field, and each element can be the generator of the multiplicative subgroup, which means that the sum of each of the last $t-1$ terms is the sum of all elements in the multiplicative subgroup.

The multiplicative subgroup in the finite field is structurally similar to the unit circle in the complex field. On the unit circle:
- All elements are generated by the generator and are evenly distributed on the unit circle.
- Each element has another symmetric point based on the origin, the vectors of these two points in the complex plane cancel each other out, and their sum is 0.

Therefore, the sum of all elements of the unit circle is 0.

Since we need to do the interpolation with FFT(Fast Fourier Transform) algorithm, so the number of elements in the multiplicative subgroup need to be powers of 2. In the multiplicative subgroup, each element also has a symmetric point. For a multiplicative subgroup $H$ with $t$ elements: $H_i + H_{i+t/2} = 0$, these two symmetric points add up to 0, so the sum of all elements in the multiplicative subgroup is also 0. This is why the sum of each of the last $t-1$ terms is 0:

$$
\begin{align*}
& \sum_{i \in t}(\omega)^i = 0\\
& \sum_{i \in t}(\omega^2)^i = 0 \\
& . \\


& . \\
& . \\
& \sum_{i \in t}(\omega^{t-2})^i = 0 \\
& \sum_{i \in t}(\omega^{t-1})^i = 0 \\
\end{align*}
$$

So:

$$
\begin{align*}
& \sum_{a \in H} f(a) \\
& = t \cdot f_0 + f_1\sum_{i \in t}(\omega)^i + f_2\sum_{i \in t}(\omega^2)^i + ... + f_{t-2}\sum_{i \in t}(\omega^{t-2})^i + f_{t-1}\sum_{i \in t}(\omega^{t-1})^i \\
& = t \cdot f_0 + f_1 \cdot 0 + f_2 \cdot 0 + ... + f_{t-2} \cdot 0 + f_{t-1} \cdot 0 \\
& = t \cdot f_0
\end{align*}
$$

Thus:

$\sum_{x \in H_t} A(x) = |\vec t| \cdot A(0)$

$\sum_{x \in H_f} B(x) = |\vec f| \cdot B(0)$

We only need to verify if the following relation holds: 

$|\vec t| \cdot A(0) \stackrel{?}{=} |\vec f| \cdot B(0)$

Finally, with these three proofs($A(x), B(x), Sumcheck$), we complete the CQ lookup arguments.

## Acknowledgements
- Guo Yu
- Yu-Ming Hsu
- Jing-Jie Wang
- Paul Yu
- Even
- Xiaoxiong
## 参考
- [cq: Cached quotients for fast lookups](https://eprint.iacr.org/2022/1763)
- [A Close Look at a Lookup Argument - Mary Maller](https://www.youtube.com/@thebiuresearchcenteronappl8783)
- [理解 PLONK（七）：Lookup Gate](https://github.com/sec-bit/learning-zkp/blob/develop/plonk-intro-cn/7-plonk-lookup.md)
