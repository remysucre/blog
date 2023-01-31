# Late materialization is (almost) worst-case optimal

In our SIGMOD'23 [paper](https://arxiv.org/abs/2301.10841) we proposed
 a new join algorithm called *free join* that generalizes and unifies
 both traditional binary joins and worst-case optimal joins (WCOJ).
This bridge can be very useful in practice, because it brings WCOJ closer
 to traditional algorithms, making it easier to adopt in existing systems.
Indeed, WCOJ has not seen wide adoption mainly because it seems so different
 from what's already implemented in databases. 
The surprise in this post is that your database is probably already almost 
 worst-case optimal, and only some small changes are needed to get the last mile.
In particular, if your system implements late materialization, then you're 
 in good shape.
You just need the following to get worst-case optimality:

1. Make the late materialization a little more aggressive
2. Change the query plan a bit
3. Add a very simple adaptive processing primitive

## A simple fact about AGM

This whole WCOJ line of work goes back to the AGM bound, 
 and I've written about it [before](wcoj.md).
A very simple and useful property of the AGM bound is *decomposability* (Lemma 4.1, [NRR'13](https://arxiv.org/abs/1310.3314)),
 demonstrated with the triangle query here:

```math
\sum_{(a, b) \in R} \text{AGM}\left( \texttt{Q(z) = S(a,z), T(z,b)} \right) 
\leq \text{AGM}\left( \texttt{Q(x,y,z) = R(x,y), S(x,z), T(z,y)} \right)
```
What this means is that instead of the standard Generic Join where we build a trie
 for each relation and do intersections, 
 we can skip trie building for one of the relations and simply iterate over it.
That is, the following is still worst-case optimal:

```
for a, b in R
  for c in S[a].z ^ T[b].z
    output(a, b, c)
```

This is the first step bringing us closer to binary join from WCOJ, 
 because in binary (hash) join we only build hash on one side and iterate over the other.
 
## Late materialization

Late materialization is one of the ideas with the most bang-for-the-buck:
 it's very simple yet can lead to dramatic speedup.
To illustrate with an example, consider the query `Q(x, u, v) :- R(x), S(x, u), T(x, v)`.
This is a simplified version of the "clover query" in our free join [paper](https://arxiv.org/abs/2301.10841).
Now imagine the `x` column contains book titles, i.e. it's short, 
 but the `u, v` columns contain the content of books, i.e. very long.
Naive binary join will first join `R` and `S`, and loop over each result to join with `T`.
That is:

```
# hash S on x, hash T on x

for x in R:
  us = S[x]? # ? means continue to enclosing loop if the lookup fails
  for u in us:
    vs = T[x]?
    for v in vs:
      output(x, u, v)
```

The second loop is a bit silly, because we are retrieving the `u`s even though 
 we don't need them to probe into `T`.
And getting each `u` is expensive, because they contain the entire content of a book.
The idea of late materialization is to delay actually retrieving each `u` until 
 we are ready to output in the innermost loop.
For example, we can simply iterate over an array of pointers to the `u`s, 
 and dereference only at the end.

## More aggressively late

To get more performance, we need to be more aggressive in being late.
Instead of delaying dereference during iteration, 
 we delay the entire iteration until it's needed.
Using our example, we'll push the second loop to run after the lookup on `T`:

```
# hash S on x, hash T on x

for x in R:
  us = S[x]? # ? means continue to enclosing loop if the lookup fails
  vs = T[x]?
  for u in us:
    for v in vs:
      output(x, u, v)
```

Probing into both `S` and `T` first will filter out some `u`s and `v`s 
 so we won't have to iterate over those.

Another limitation found in many systems is that they only delay the materialization
 of non-join attributes. 
In general, you might also want to delay materializing join attributes as well.
The triangle query is one example, where all attributes are joined on.
Furthermore, when building hash people usually hash on *all* join attributes.
To get worst-case optimality, we'll want to hash on some of the attributes first, 
 and delay the rest until later. 
For more details see the COLT data structure in our [paper](https://arxiv.org/abs/2301.10841).

## Changing the plan

In the above when we pushed down the second loop, we already broke away from the original 
 binary plan that computes the join of `R` and `S` first.
A simple way to guarantee worst-case optimality now is to go all the way and
 use a generic join plan.
A generic join plan is simply a total ordering of all the attributes. 
For example, `[x, y, z]` for the triangle query and `[x, u, v]` for our second example query above.
During execution, we'll join all the relations that have the first variable first, then
 go to those with the second variable, and so on.
While joining on a variable, we delay the materialization of all other variables, 
 and this essentially implements the "intersection" from generic join.

However, it's debatable if we *should* go all the way to a generic join plan just because we can.
If the optimizer picks a particular binary plan, it's probably because it thinks the plan is fast.
In the paper we take a more conservative approach and greedily transform the binary plan 
 towards WCOJ, while avoiding accidental slowdowns.

## Adaptive processing

A subtle detail of WCOJ is that every intersection must take time linear
 to the size of the smallest relation; 
 otherwise the algorithm won't be worst-case optimal.
In hash join, we intersect by iterating over the keys of one relation, 
 and probing into the others.
A simple way to satisfy the requirement is to simply pick the smallest 
 relation to iterate over at run time. 
This will guarantee worst-case optimality, but there's an interesting trade-off:
 iterating over the smallest relation means we now have to build 
 hash on the larger relations, and this is expensive.
On the other hand, if we iterate over the largest one, 
 we save time hashing it.
In practice, the time saved can be significant, and may outweigh the 
 warm and fuzzy feeling of worst-case optimality.

## Conclusion

At this point, even if you're not convinced you can massage your join 
 implementation into one that is worst-case optimal, 
 I hope your suspicion is strong enough that you'll try it.
I think the COLT data structure is really central to all of this, 
 and if you see you can implement COLT with late materialization, 
 the rest is simple.