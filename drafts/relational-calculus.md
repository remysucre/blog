# Relative Safety in Relational Calculus

In classical model theory, there are two important questions:

1. **(Finite) Satisfiability**: Given a first-order sentence $\phi$, 
   is there a (finite) structure $M$ such that $M \models \phi$? 

2. **Model Checking**: Given a first-order sentence $\phi$ and 
   a structure $M$, is it true that $M \models \phi$?

In fact, the history of the first question goes back to the inception of computer science. 
In 1936, Alonzo Church and Alan Turing independently proved that Satisfiability for 
first-order logic is undecidable, resolving David Hilbert's Entscheidungsproblem.
Their proofs also invented the two pillars supporting modern computer science: 
the lambda calculus and the Turing machine.

A less well-known result is that the *Finite* satisfiability is also undecidable.
Specifically, if a sentence $\phi$ contains a relation of arity at least 2, 
then it is undecidable to determine if $\phi$ has a finite model. 
This result is due to Boris Trakhtenbrot.

On the other hand, the Model Checking problem is very easy.
The data complexity (over the size of the model) of Model Checking over FOL 
(extended with $\times$ and $+$) is $AC^0$, one of the lowest classes 
in the polynomial hierarchy.
And even if we extend FOL with a least-fixpoint operator (i.e. Datalog), 
the problem is still in PTIME.

But in databases, we care about a slightly different question from Satisfiability 
and Model Checking:

3. **Relation Calculus Query**: Fix a finite structure $M$, 
and a first-order sentence $\phi(\mathbf{x})$ over $M$,
where $\mathbf{x}$ are the free variables of $\phi$.
Let $Q$ be a relation symbol.
Find a model for $Q$ such that $\forall \mathbf{x} : Q(\mathbf{x}) \Leftrightarrow \phi(\mathbf{x})$. 

Note that $Q$ may not always have a finite model. For example, if $\phi(x) := x = x$ then $Q$
must contain all values in the domain which can be infinite.
We may therefore want to ask a question similar to Finite Satisfiability: 

4. **Relative Safety**: Given a Relational Calculus query and a fixed database,
 does the query have a finite answer on that database?
That is, is there a finite model for $Q$, defined as above?

This problem sits in between the Finite Satisfiability problem and the Model Checking problem: 
it asks for a model for $Q$, but we already have the models for all other relations.
Relative Safety was proposed by [AGSS86], which is written in Russian. 
Other papers cite it for a proof that Relative Safety is decidable, 
but I don't read Russian, so I had to do the proofs myself. 
Luckily, the proof is actually not very complicated.

**Theorem** Consider Relational Calculus with the following syntax:

```math
Q := \bot \mid \top \mid x = t \mid R(t_1, \ldots, t_k) \mid \neg Q \mid Q \vee Q \mid Q \wedge Q \mid \exists x .Q
```

In short, it is the standard FOL where the only interpreted predicate is equality.
The Relative Safety is decidable.

**Proof**
Because the database contains only a finite set of finite relations,
any infinite query output must contain some value that is not in the database.
The converse is also true: if the query output contains *any* value not in the database, 
the output must be infinite.
This is because the query cannot *distinguish* values not in the database, 
because the only predicates are relation symbols or =.
Therefore, if we see one output tuple containing some $v$ not in the database, 
we can always change $v$ to another value not in the database to get another tuple.

With the above insight, we can easily test if a query is infinite using a finite 
number of values not in the database.
We can't use just one, because the sentence may contain a predicate $v = v'$ where
both $v$ and $v'$ are free, in which case we may want to use two different values for them.
The algorithm is as follows:

Let $C$ be the (finite) set of values appearing in the database.
Let $V = \{ v_1, \ldots, v_n \}$ be a set of $n$ values not in the database, 
where $n$ is the number of free variables of $Q$.
For each tuple $t=(t_1, t_2, \ldots)$ where each $t_i \in C \cup V$,
check if $\phi(t)$ is valid (model checking).
If it is, and if $t$ contains a value not in $C$, then the query is infinite.
If we exhaust all tuples ($C \cup V$ is finite) without finding such a $t$,
the query is finite. 

[AGSS86] Alfred K. Ailamazyan, Mikhail M. Gilula, Alexei P. Stolboushkin, and Grigorii F. Schwartz. Reduction of a relational model with infinite domains to the case of finite domains. Doklady Akademii Nauk SSSR, 286(2):308–311, 1986. URL: http://mi.mathnet.ru/dan47310.