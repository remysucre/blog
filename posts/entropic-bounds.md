# Fundamental Entropic Bounds

When my advisor Dan Suciu taught me entropy, he said everyone should know 3
inequalities: the "entropy range", monotonicity, and submodularity. Luckily I
don't have to memorize the bounds as each inequality has a very simple intuition.
First, the "entropy range" simply bounds the value of any entropy function: 

```math
0 \leq H(X) \leq \log(|X|)
```

On one hand, the entropy is 0 if $X$ takes a certain value $A$ with probabilty
$1$. In that case, we know $X$'s value ($A$) without communicating a single bit.
On the other hand, we can always simply use full binary encoding with
$\log(|X|)$ bits to encode $X$, ignoring the probability distribution. 

**Monotonicity** says that the entropy of a string of random variables is no less than
the entropy of any substring: 

```math
H(X) \leq H(XY)
```

Here $XY$ simply "concatenates" $X$ and $Y$, in that a value of $XY$ concatenates
a value of $X$ with a value of $Y$. The entropy $H(XY)$ is the number of bits
necessary to transmit a string in $XY$. With this in mind, monotonicity simpy says
that transmitting more information requires more bits. 

Finally, our last inequality, submodularity, conveys the intuition of
"diminishing returns": 10 dollars matter less to a millionaire than to a PhD
student. More concretely, suppose we have a function $f$ from wealth to
quality of life. Then $f$ is submodular because $f(x + \delta) - f(x)$ gets
smaller and smaller as $x$ increases. In the context of information theory, $H$
is submodular because adding additional information to a long message takes
little effort. For example, suppose a submarine needs to send reports describing
the fish it finds, and the description includes weight, length and species. Then
if it says the fish is 80 feet long, you'll know it's a blue whale without
looking at the species field. In general, we can save some bits by inferring
facts from the correlation of data; if all variables are independent we can save
nothing. With this intuition, let's look at the formal statement of
**submodularity**:

```math
H(X) + H(Y) \geq H(X \cup Y) + H(X \cap Y)
```

Rearranging, we get $H(X \cup Y) - H(X) \leq H(Y) - H(X \cap Y)$. Note $(X
\cup Y) - X = Y - (X\cap Y)$, and if we define $\delta = Y - (X\cap Y)$, the
inequality becomes $H(X + \delta) - H(X) \leq H((X\cap Y) + \delta) - H(X\cap
Y)$, which states precisly "the law of diminishing returns" because $X \geq
(X\cap Y)$!
