# Field Extensions

![[Pasted image 20240914173210.png]]

Field extensions refer to the process of adding new elements to an existing field, known as the base field. This not only expands the field but allows us to perform arithmetic operations with the new elements. For example, consider adding the element $\sqrt{2}$ to the field of rational numbers $\mathbb{Q}$. Once $\sqrt{2}$ is introduced, we can perform various calculations such as:

$$
-\sqrt{2}, \frac{3}{\sqrt{2}}, 5 + \sqrt{2}, \left(\sqrt{2} + \frac{3}{2}\right)^3, \frac{\sqrt{2}}{3 + \sqrt{2}}
$$

These expressions can be simplified as follows:

$$
\frac{3}{\sqrt{2}} = \frac{3}{2} \sqrt{2}
$$

$$
\left(\sqrt{2} + \frac{3}{2}\right)^3 = \left(\sqrt{2} + \frac{3}{2}\right)^2 \left(\sqrt{2} + \frac{3}{2}\right) = \frac{99}{8} + \frac{35}{4} \sqrt{2}
$$

$$
\frac{\sqrt{2}}{3 + \sqrt{2}} = \frac{\sqrt{2}(3 - \sqrt{2})}{(3 + \sqrt{2})(3 - \sqrt{2})} = -\frac{2}{7} + \frac{3}{7} \sqrt{2}
$$

Notice that the results all follow the structure $a + b\sqrt{2}$, where $a, b \in \mathbb{Q}$. Hence, this extension field can be represented as:

$$
E = \{a + b\sqrt{2} \mid a, b \in \mathbb{Q}\}
$$

Thus, $\mathbb{Q} \subset E \subset \mathbb{R} \subset \mathbb{C}$.

You might be more familiar with the field of complex numbers $\mathbb{C}$, where each element can be written as $a + bi$, with $a, b \in \mathbb{R}$, and $i$ being a new element introduced into the base field $\mathbb{R}$. Similarly, $\sqrt{2}$ and $i$ play analogous roles, each introducing a new symbol to their respective base fields, forming extension fields. This process is at the heart of field extensions.

## Finite Fields and Their Extensions

Next, consider how to extend finite fields. Take the finite field $\mathbb{Z}_2 = \{0, 1\}$. If we introduce $\sqrt{2}$ into this field, using the same method as above, the resulting extension field would be:

$$
E = \{a + b\sqrt{2} \mid a, b \in \mathbb{Z}_2\}
$$

Since $a$ and $b$ can only be $0$ or $1$, we get:

$$
E = \{0, 1, \sqrt{2}, 1 + \sqrt{2}\}
$$

Similarly, if we introduce an element like $i$ or any element of symbol $t$, the extension becomes:

$$
E = \{0, 1, t, 1 + t\}
$$

This method of extending fields works, but there is a more general approach that always succeeds: using irreducible polynomials.

### Field Extensions Using Irreducible Polynomials

An irreducible polynomial is one that has no roots in the base field. For instance, $x^2 - 2 = 0$ has no roots in $\mathbb{Q}$, but it does have roots in $\mathbb{R}$ and $\mathbb{C}$, where the roots are $\pm \sqrt{2}$. This is precisely the process of introducing new elements into the base field.

The reason for using irreducible polynomials is that they ensure the introduction of a new element. If a polynomial were reducible, its roots would already exist in the base field, failing to extend it.

To extend a field using an irreducible polynomial, we use a quotient:

$$
E = F[t]/\langle p(t) \rangle
$$

Where $p(t)$ is an irreducible polynomial over $F$. $F[t]$ is all possible polynomial rings over $F$. This quotient construction ensures that $p(t)$ has a root in the new field $E$.

You can think $p(t)$ here as number 2 in $\mathbb{Z}_2$, which is 0, so over $E$ we will get $p(t) = 0$.

We will use an example to explain how this works.

#### Example: Extending $\mathbb{Z}_2$

Let’s take $F = \mathbb{Z}_2 = \{0, 1\}$, and consider the irreducible polynomial $p(t) = t^2 + t + 1$. We check whether $p(t)$ has roots in $\mathbb{Z}_2$:

$$
p(0) = 1 \neq 0, \quad p(1) = 3 \mod 2 = 1 \neq 0
$$

Since $p(t)$ has no roots in $\mathbb{Z}_2$, we claim that the quotient $E = \mathbb{Z}_2[t]/\langle p(t) \rangle$ is an extension of $\mathbb{Z}_2$. This extension field $E$ includes a root of $p(t)$, because $p(t) = 0$ in this quotient.

In $E$, we have the relation:

$$
1 + t + t^2 = 0
$$

We can now express powers of $t$ in terms of lower powers:

$$
t^2 = 1 + t
$$

This allows us to simplify higher powers, such as:

$$
t^3 = t^2 \cdot t = (1 + t) \cdot t = t + t^2 = t + (1 + t) = 1
$$

Thus, the elements of $E$ can be represented as:

$$
E = \{0, 1, t, 1 + t\}
$$

This matches the result we derived using the first method.

### Introducing More Elements

Consider the irreducible polynomial $p(t) = t^3 + t + 1$ over $\mathbb{Z}_2$. Checking for roots:

$$

  

p(0) = 1 \neq 0, \quad p(1) = 3 \mod 2 = 1 \neq 0

  

$$

Since $p(t)$ has no roots in $\mathbb{Z}_2$, we again form the quotient $E = \mathbb{Z}_2[t]/\langle p(t) \rangle$, which introduces a root of $p(t)$.

In $E$, the relation becomes:

$$
1 + t + t^3 = 0
$$

Thus, we can simplify powers of $t$:

$$
t^3 = 1 + t
$$

By reducing higher powers of $t$, we eventually obtain the eight elements of the extension field:

$$
E = \{0, 1, t, 1 + t, t^2, 1 + t^2, t + t^2, 1 + t + t^2\}
$$

![[Pasted image 20240911211754.png]]
Each element can be written as a linear combination of $\{1, t, t^2\}$. So these examples demonstrate the usefulness of irreducible polynomials in constructing extension field.

## Conclusion

The quotient $E = F[t]/\langle p(t) \rangle$ defines a field extension of $F$ that contains a root of the irreducible polynomial $p(t)$. This method provides a systematic way to extend fields.

