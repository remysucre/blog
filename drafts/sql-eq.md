# How to Check 2 SQL Tables are the Same

Today Stanley asked me a simple question: 
 how can we check if the contents of 2 SQL tables are the same?
Well, you just do `SELECT * FROM t1 = t2`... wait, that's wrong, comparison
 doesn't work on entire tables in SQL.
My second attempt is a bit better: if we take the difference of the table both ways, 
 and end up with empty results, then they must be the same, right?
In SQL: `SELECT * FROM t1 EXCEPT SELECT * FROM t2` (and the other way).
Wrong again! Because `EXCEPT` takes the *set* difference, 
 it will be empty if, say, `t1` contains 2 copies of a tuple, but `t2` contains only one.

I gave up a little bit and started searching online,
 but surprisingly there was not a single satisfying answer!
The solutions online either suffer from the same issue as the `EXCEPT` query, 
 or use some obscure features that are not standard SQL (e.g. `CHECKSUM` which doesn't really work anyways).
How hard can it be to compare two tables in SQL?!

Intrigued, I posted the problem as a challenge to my colleagues:
 **Write a query, using only standard SQL features, to check if two tables are the same**.
Here "same" means the two table contains the same set of distinct tuples,
 and every tuple has the same number of copies in each table.
Formally, they are the same bag/multiset.

If you've read the SQL standard (and every "non-standard") cover to cover,
 you'll come with the following query after a few campari drinks: 
 `SELECT * FROM t1 EXCEPT ALL SELECT * FROM t2`{:.SQL}.
The key is `EXCEPT ALL` which takes the "bag difference"
 similar to how `UNION ALL` takes the "bag union".
Alas, `EXCEPT ALL` is not implemented by SQLite!
And probably for good reasons:
 whereas `EXCEPT` can be compiled to just an anti-join,
 executing `EXCEPT ALL` probably requires keeping track of 
 which copy of the same tuple we've seen,
 or keeping a count per distinct tuple.

A more "vanilla SQL" solution looks like this:
```SQL
select *, count(*) 
  from r 
 group by [ALL ATTRIBUTES OF R] 
EXCEPT
select *, count(*) 
  from s 
 group by [ALL ATTRIBUTES OF S];
[and the other direction...]
```

Why isn't this a feature? Every test needs this!