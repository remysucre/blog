# Entropy

Entropy can be understood as the minimum amount of data needed to be transmitted
in order to communicate a piece of information. Concretely, this is the average
number of bits needed to encode a message. For example, imagine a spaceship
sends a status code every minute to indicate if it has found any alien
civilization. The code is any letter from the English alphabet, with $A$ meaning
"nothing new", and some other letter describing the alien. For example $F$ means
the alien is friendly, and $B$ means the alien is blue, etc. For simplicity
assume every code is only 1-letter long. Then we can simply encode this status
code with $\log(26)$ bits, where $A = 0 \dots 0, B = 0\dots 1, \dots$. However,
we can be a little clever because we know most of the time the spaceship won't
be finding new civilizations. In that case, the satellite can remain silent to
indicate "nothing new"; otherwise it sends the status code with our original
encoding [^1].

Then we only need to send on average little more than 0 bit per minute. In
general, we can save some bits if we know certain messages occur with high/low
probability. In other words, the minimum commucation cost depends on the
probability distribution of the data. Entropy precisely formalizes this
intuition. Formally, if $X$ is a random variable with outcomes $x_1, \dots, x_N$
each of probabilities $p_1, \dots, p_N$, then its **entropy** is defined as:

```math
H(X) = \sum_i p_i \log \frac{1}{p_i}
```

This matches our intuition: when $X$ is uniform and $|X| = N$, $H(X)=N(1/N
\log N)=\log N$; when $X$ is almost always a certain message, say $A$, then
$H(X)= p_A \log \frac{1}{p_A} + \sum_{i \not = A} p_i \log \frac{1}{p_i} =
0.99999 \log \frac{1}{0.99999} + \delta \approx 0$. For a more general case,
suppose message $A$ occurs half of the time, $B$ one quarter of the time,
$C$ one eighth and so on. Then we can use one bit $1$ to indiate that $A$
occurs; otherwise we first send one bit $0$ to indicate it's not $A$, then
send one bit $1$ if $B$ occurs and $0$ otherwise, and so on. On average we
need $p_A \times 1 + p_B \times 2 + \dots = p_A \times \log(\frac{1}{p_A}) +
p_B \times \log(\frac{1}{p_B}) + \dots = H(X)$ bits.

[^1]: This works fine because we assume the spaceship is not dead. Otherwise we can
have the spacehip send a single bit $0$ every minute to signal it's alive; when it
finds an alien, we could also prefix the status code with $1$ so that the message 
doesn't get mangled with the alive-bits. 
