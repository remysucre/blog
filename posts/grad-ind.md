# A Canonicity Proof via Gradient Induction

A canonical form for a language defines a representative among an equivalent class of terms.
It can help identify equivalent terms in the language.
Here I present a proof for the canonicity of the sum-over-product normal form for arithmetics,
to demonstrate an interesting technique that I call induction over derivatives.
A more catchy name I thought of is gradient induction.

First let's clarify what we mean by canonical form:
when two expressions in a language, considered as programs,
evaluate to the same result on all possible inputs,
we say they are semantically equivalent.
We therefore hope to find a "standard way" to write such expressions,
so that when we rewrite any two expressions to the standard form,
we can immediately tell if they are semantically equivalent by just looking at them.
Such a standard form is considered canonical -
we say that two terms, e1 and e2,
share the same canonical form if and only if they are semantically equivalent,
where semantically equivalent means the terms always compute the same result given same inputs.

Formally:

```math
\text{canonicalize}(e_1) \equiv \text{canonicalize}(e_2) \Leftrightarrow \forall x . \text{eval}(e_1, x) = \text{eval}(e_2, x)
```

Here $\equiv$ denotes syntactic equality, 
and = denotes semantic (value) equality.
In our case, the expressions are in the language of arithmetics $(+, \times, x, \mathbb{R})$:

**Definition** an *arithmetic expression* is either a variable,
a constant, the sum of two expressions, or the product of two expressions.

And our normal form is essentially the standard polynomials:

**Definition** the sum-over-product normal form of an arithmetic expression 
is the sum of products of literals,
where a literal is either a variable or a constant.
Furthermore, we combine monomials that only differ in their coefficients,
e.g. 2xy + 3xy is rewritten into 5xy.

One can rewrite an expression to SoP with the standard laws 
(associativity, commutativity & distributivity).
That is, we keep pulling + up and merge terms.
For example, the SoP canonical form of (x + z) (x + y) is x2 + xy + xz + yz.

**Proposition** the sum-over-product normal form is canonical:

```math
C_{sop}(e_1) \equiv C_{sop}(e_2) \Leftrightarrow \forall x . \text{eval}(e_1, x) = \text{eval}(e_2, x)
```

**Proof** the left-to-right direction can be proven by structural induction 
over the SoP normal form syntax,
together with the fact that 
the standard rewrite rules we use preserve the semantics of arithmetics.

I now prove the contrapositive of the backward direction:

```math
C_{sop}(e_1) \not\equiv C_{sop}(e_2) \Rightarrow \exists x . \text{eval}(e_1, x) \neq \text{eval}(e_2, x)
```

There are two cases for $C_{sop}(e_1) \not\equiv C_{sop}(e_2)$:
1. e1 and e2 differ in their constant term 
(e.g. e1 = 2xy + 4 and e2 = 3yz + 7), and
2. otherwise (e.g. e1 = 2xy + 4 and e2 = 3yz + 4).

Note that we only look at the lonely constants,
not the coefficients in other terms.

Case 1 is simple,
since if two expressions have different constants 
they evaluate to different results (i.e the constants themselves) on all-zero inputs.

To prove case 2, I break down the goal into two steps:

```math
C_{sop}(e_1) \not\equiv C_{sop}(e_2) \Rightarrow \exists y . \frac{\partial e_1}{\partial y} \neq \frac{\partial e_2}{\partial y}
```

and

```math
\exists y . \frac{\partial e_1}{\partial y} \neq \frac{\partial e_2}{\partial y} \Rightarrow \exists x . \text{eval}(e_1, x) \neq \text{eval}(e_2, x)
```

Recall that $\neq$ is semantic inequivalence.

The latter is simple:
pick x1 and x2 that only differ in the y variable (from $\partial y$ above).
Since the derivatives differ,
we can always find a pair of x1 and x2 such that 
either $\text{eval}(e_1, x_1) \neq \text{eval}(e_2, x_1)$ 
or $\text{eval}(e_1, x_2) \neq \text{eval}(e_2, x_2)$.

To prove $C_{sop}(e_1) \not\equiv C_{sop}(e_2) \Rightarrow \exists y . \partial e_1 / \partial y \neq \partial e_2 / \partial y$,
we perform induction over the derivatives of the expressions,
with our original proof goal as the inductive hypothesis:

```math
C_{sop}(e_1) \equiv C_{sop}(e_2) \Leftrightarrow \forall x . \text{eval}(e_1, x) = \text_{eval}(e_2, x)
```

Since $C_{sop}(e_1) \not\equiv C_{sop}(e_2)$,
we know $\exists y . \partial e_1 / \partial y \not\equiv \partial e_2 / \partial y$ (syntactically).
Since the derivative of a canonical form is also canonical 
(not that obvious, but you'll see it after thinking a little harder),
by our inductive hypothesis,
$\exists y . \partial e_1 / \partial y \neq \partial e_2 / \partial y$ (semantically).

The preceding induction is sound because taking the derivative makes any expression simpler,
eventually bringing it to a constant. $\blacksquare$

The main takeaway here is that,
when we want to perform an inductive proof,
we need every inductive step to make the terms simpler.
Usually this is done by structural induction over the language definition;
but when the language is differentiable,
the derivative is another tool for inductively simplifying the terms.
This can come handy for the PL researcher,
since we now know a wide range of languages are differentiable – not just polynomials!

p.s. [Haotian Jiang](https://jhtdavid96.wixsite.com/jianghaotian)
& [Sorawee Porncharoenwase](https://homes.cs.washington.edu/~sorawee/en/)
pointed out a much simpler proof:
given two semantically equivalent arithmetic expressions,
their difference is always zero.
Therefore, the expression that represents the difference has infinitely many roots.
According to the fundamental theorem of algebra,
the two expressions must be the same polynomial,
since otherwise their difference would be a none-zero polynomial and has finitely many roots.
Nevertheless, the main point of this post isn't to prove normalization, 
but to showcase the technique of gradient induction.
