# Bounding the run time of recursive queries with the AGM bound

**Acknowledgements** All ideas in this post are due to @mabokhamis and @hung-q-ngo; 
I'm just writing them down so that I don't have to bother them when I forget.

In an [earlier post](wcoj.md) I talked about how to use the AGM bound 
  to bound the run time of the Generic Join algorithm.
It turns out we can sometimes bound the run time of recursive queries as well.
Consider the transitive closure program in Datalog: 

```
P(x, y) :- E(x, y).
P(x, y) :- E(x, z), P(z, y). 
```

During semi-naive evaluation, we will compute a delta relation $dP$ every iteration as 
$dP_{i+1} = E \bowtie dP_i - P_i$.
The set difference can be done in linear time, so we will focus on the join only.

If we look at *all* iterations, we'll be computing 
$E \bowtie dP_0 \cup E \bowtie dP_1 \cup \cdots \cup E \bowtie dP_{k-1}$.
Factoring out `E`, we get $E \bowtie (dP_0 \cup \cdots \cup dP_{k-1}) = E \bowtie P_k$, 
 where $P_k$ is the relation $P$ at iteration $k$. 
Since $P_k$ must be contained in the final output $O$, i.e. $|P_k| \leq |O|$,
 at this point we can say the whole semi-naive algorithm runs in $O(|E|\times|O|)$.
But turns out we can do better. 

To reduce clutter, I'll write $P$ for $P_k$. 
Now take a closer look at the join `Q(x, y, z) :- E(x, z), P(z, y)`.
For the moment, let's also add $O$ into the join to make a triangle `Q'(x, y, z) :- E(x, z), P(z, y), O(x, y)`.
With this, we can use the AGM bound to get $O(|E|^\frac{1}{2} |P|^\frac{1}{2} |O|^\frac{1}{2}) \leq O(|E|^\frac{1}{2} |O|)$, 
 which is a tighter bound than the above.
I now claim we can also use this bound for $Q$. 
The key is that the execution of Generic Join for $Q'$ is exactly the same as that for $Q$. 

Consider the variable ordering `z, x, y`. The GJ loop for $Q'$ is the following:

```
for z in E.z ^ P.z
  for x in E[z].x ^ O.x
    for y in P[z].y ^ O[x].y
      output(x, y, z)
```

Since $O$ is the final output of the transitive closure program, 
 we have the invariant $\forall x, y, z : (x, z) \in E \wedge (z, y) \in P \implies (x, y) \in O$.
Therefore, we can remove the intersections with $O$ on both inner loops, 
 and the run time would remain the same since $O$ does not filter out any value.
With $O$ removed, the nested loop now computes exactly $Q$, 
 taking the same time as $Q'$. 