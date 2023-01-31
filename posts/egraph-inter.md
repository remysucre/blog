# E-Graph Intersection

A Rust implementation of the algorithm based on 
[egg](https://egraphs-good.github.io/)
can be found [here](https://github.com/remysucre/eggegg).

**Definition** [e-graph intersection]
An e-graph represents a (potentially infinite) set of terms and
an equivalence relation over the terms.
The intersection of two e-graphs G1, G2 
is an e-graph G that satisfies the following: 
1. the set of terms it represents 
is the intersection of the sets of terms represented by G1, G2, and
2. two terms are equal (according to the equivalence relation) in G 
if and only if they are equal in both G1 and G2. 

To be even more formal, define $G = (T, E)$ 
where T is the set of terms and E the equivalence relation,
and similarly $G_1 = (T_1, E_1), G_2 = (T_2, E_2)$.
Then $T = T_1 \cap T_2$ and 
$(t_1, t_2) \in E \Leftrightarrow (t_1, t_2) \in E_1 \wedge (t_1, t_2) \in E_2$. 

**Algorithm** [intersecting e-graphs]
Given two e-graphs G1, G2, compute their intersection G. 

Observe that G’s equivalence relation E must be a refinement of E1 and E2, 
because if $(t_1, t_2) \in E$ we must have $(t_1, t_2) \in E_1$ 
and $(t_1, t_2) \in E_2$, 
but if $(t_a, t_b) \in E_1$ we might not have $(t_a, t_b) \in E$, 
and similar for E2.
Given this observation, we can define the following maps: 

- $E \rightarrow E_2$ will map each class in E to a corresponding class in E2, 
- $E_1 \rightarrow \{E\}$ will map each class in E1 to a (possibly empty) set of classes in E. 

A class in E1 may map to the empty set 
if the terms it contains are not represented in G.
The main purpose of these maps is to relate eclasses across G, G1 and G2,
so that we can for example check if two enodes in G are equal by asking G1 and G2.
We initialize all maps to be empty at the beginning.
The intersection algorithm proceeds as follows: 

```
while G changes:
  for class in G1.classes:
    for op(c1,...,cn) in class.nodes:
      // map child classes to classes in G
      for op(c1’,...,cn’) where c1’∈E1→E[c1],...,cn’∈E1→E[cn]
        // only add the node if it’s also in G2
        if let Some(c2) = G2.get(op(E→E2[c1’],...,E→E2[cn’]))
          cnew = G.add(op(c1’,...,cn’))
          E→E2[cnew] = c2
          E1→E[class].insert(cnew)
          // this loop is expensive but seems necessary, because the 
          // new node may only be equiv. to some nodes & not others
          for c in E1→E[class]
            if G2.find(E→E2[c]) = G2.find(c2): G.union(cnew,c)
```

In English, we repeatedly scan the nodes in G1 and try to add them to G.
Nodes will get added bottom-up starting with the leaves 
(we can’t add a node without adding its children first),
and a node only gets added if it’s in G2 as well.
But one node in G1 may become multiple nodes in G,
because any of its child class may have been split into multiple classes in G.
So we follow the map $E_1 \rightarrow \{E\}$ and create the new nodes accordingly.
After adding a new node to G, we update the maps.
Because all nodes we added from the same class in G1 
($E_1 \rightarrow \{E\}[\text{class}]$)
are equivalent according to G1,
we only need to check if they are also equivalent under G2.
If any pair are equivalent, we union them in G.
We repeat this process until G stops changing, much like equality saturation. 
In fact, we can roughly think of the process as an equality saturation 
where the rewrite rules are given by G1 and G2. 

**Correctness Theorem**
When the algorithm terminates (it always does),
G is the intersection of G1 and G2. 

*Termination*: we first prove G must be finite.
Assume the contrary, i.e. G has finitely many classes but infinite enodes,
or infinitely many classes.
The first case is impossible,
because with finite function symbols and finite classes 
we can only construct a finite number of enodes 
(multiple copies of identical enodes are hash-cons’ed).
If there are infinite classes in G,
then G must have infinite classes that correspond to the same class in G1.
But the fact that they are not unioned in G means 
they must each correspond to a different class in G2,
which is impossible because the latter is finite.

Now it is easy to show the algorithm terminates.
Because union always reduces the number of classes,
the only way for G to keep changing is to keep adding new enodes.
But this will result in an infinite number of enodes, which is impossible. 

*Correctness*: the correctness of the algorithm consists of the following: 
- *Representation soundness*: every term represented in G is represented in G1 and G2, 
- *Representation completeness*: if a term is represented in both G1 and G2, it is in G, 
- *Equality soundness*: every two term equal in G are equal in both G1 and G2, 
- *Equality completeness*: if two terms are equal in both G1, G2, they are equal in G. 

The following needs work - 
these reply on some invariants about the maps $E \rightarrow E_2$ and  $E_1 \rightarrow \{E\}$: 
- If $c_2 = E \rightarrow E_2[c]$, then the terms represented by c are all in c2. 
- If $c \in E_1 \rightarrow \{E\}[c_1]$, then the terms represented by c1 are all in c.

*Representation soundness*: every enode added to G is constructed from an enode in G1,
and we only add it if it’s also found in G2. 

*Equality Soundness*: union’s arguments come from the same eclass in G1,
and we only union if they also correspond to the same eclass in G2. 

Completeness should be proved by induction on the terms. 

**Acknowledgement**
The idea of intersecting e-graphs is due to 
[Altan](https://altanh.com/) and [Josh](https://joshmpollock.com/);
they [implemented it](https://github.com/uwplse/unscramble) before I did.
I wrote my implementation from scratch,
but I suspect it is equivalent to theirs.
Altan and Josh were inspired by the [work](https://doi.org/10.1145/3133886)
using tree automata for synthesis.
The paper below by Gulwani, Tiwari \& Necula is also relevant; 
I suspect it achieves percisely e-graph interesection. 

**References**

Burghardt, Jochen. "E-generalization using grammars." Artificial intelligence 165.1 (2005): 1-35.

Gulwani, Sumit, Ashish Tiwari, and George C. Necula. "Join algorithms for the theory of uninterpreted functions." International Conference on Foundations of Software Technology and Theoretical Computer Science. Springer, Berlin, Heidelberg, 2004.

Xinyu Wang, Isil Dillig, and Rishabh Singh. 2017. Synthesis of data completion scripts using finite tree automata. Proc. ACM Program. Lang. 1, OOPSLA, Article 62 (October 2017), 26 pages. DOI:https://doi.org/10.1145/3133886
