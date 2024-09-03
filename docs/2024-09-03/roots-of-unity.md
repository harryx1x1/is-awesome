# Roots of unity

Roots of unity is the $z$ values to make this equation satisfied:

$z^n = 1$

For $n > 0$
## Roots of unity in complex field

For complex number, we have Euler's formula:
$e^{xi} = cos(x) + isin(x)$

If $x = 2 \pi$:
$e^{xi} = e^{2 \pi i} = cos(2 \pi) + isin(2 \pi) = 1$

If $x = 2 * 2 \pi$:
$e^{xi} = e^{2 * 2 \pi i} = cos(2 * 2 \pi) + isin(2 * 2 \pi) = 1$

So for any integer $k = 1, 2, 3...$, if $x = 2k \pi$:
$e^{xi} = e^{2k \pi i} = cos(2k \pi) + isin(2k \pi) = 1$

Use Euler's formula to get all roots of unity:

$z^n = 1$

$z = 1^{1/n} = (e^{2k \pi i})^{1/n} = e^{2k \pi i / n} = cos(2k \pi / n) + isin(2k \pi / n)$

There will be n roots for $k = 0, 1, 2, ..., n-1$

Because for $k > n-1$, it will cycle back to  $k = 0, 1, 2, ..., n-1$, for example:

If $k = 0$, $2 \pi k / n = 0$

If $k = 1$, $2 \pi k / n = 2 \pi / n$

If $k = n$, $2 \pi k / n = 2 \pi n / n = 2 \pi = 0$

If $k = n + 1$, $2 \pi k / n = 2 \pi (n + 1) / n = 2 \pi + 2 \pi / n = 2 \pi / n$

You can continue to calculate for $k > 1$ and $k > n + 1$, you will get the same result.

Each root divide the circle by a degree of $2 \pi k / n$

For example if $n = 4$, then the roots will be for $k = 0, 1, 2, 3$, and the degree on the circle between 2 roots will be $2 \pi /4 = \pi / 2$. Below image shows the 4 roots(z1, z2, z3, z4) of $z^4 = 1$
![[Drawing-root-of-unity-complex-n4.excalidraw|500]]

If we double n to $n = 8$, then the roots will be for $k = 0, 1, 2, 3, ..., 7$, there are 8 roots. The degree on the circle between 2 roots will be $2 \pi /8 = \pi / 4$. Below image shows the 8 roots of $z^8 = 1$

![[Drawing-root-of-unity-complex-n8.excalidraw|800]]

The formula(Euler's formula) is the same, what different is the degree, by continuing multiply the degree between 2 roots and calculate with Euler's formula, you will get all the roots.

Finite field has similar properties regarding this.
## Roots of unity in finite field

In finite field, each nonzero element is root of unity. For example $F_{17} = \{0, 1, 2, 3, ..., 16\}$, $1, 2, 3, ..., 16$ are the roots of unity for equation:

$z^{16} = 1 \mod 17$

![[Drawing-root-of-unity-group-n16.excalidraw|450]]



You can verify by calculate them one by one:
$$
\begin{align*} 
1^{16} \mod 17 &= 1 \\ 
2^{16} \mod 17 &= 1 \\ 
3^{16} \mod 17 &= 1 \\
... \\
16^{16} \mod 17 &= 1 
\end{align*}
$$
Primitive root of unity is the smallest number that is able to generate all elements of finite field, means if $a$ is the primitive root of unity, we can get all elements of finite field by:

$F_{17} = \{a^0, a^1, a^2, a^3, ..., a^{16}\} \mod 17$

We can try from 1 to find the primitive root of unity.

If $a = 1$, then $\{1^0, 1^1, 1^2, 1^3, ..., 1^{16}\} \mod 17 = \{1\} \textcolor{red}{\neq} F_{17}$

So 1 is not primitive root of unity.

If $a = 2$, then $\{2^0, 2^1, 2^2, 2^3, ..., 2^{16} \} \mod 17 = \{2, 4, 8, 16, 15, 13, 7, 1 \} \textcolor{red}{\neq} F_{17}$

So 2 is also not primitive root of unity.

If $a = 3$, then $\{3^0, 3^1, 3^2, 3^3, ..., 3^{16}\} \mod 17 = \{3, 6, 9, 10, 13, 5, 15, 11, 16, 14, 8, 7, 4, 12, 2, 6 \} \textcolor{red}{=} F_{17}$

So 3 is primitive root of unity.
