# Exploring functional programming and database design

The guiding theme of this project is to revisit an established technology — __databases__ —
and to explore it through the lens of __functional programming languages__ and __type systems__.

The idea is hardly new and has already been successfully explored along several dimensions.

* The most notable contribution has been to apply the concept of list comprehension
(now formalized as [monad comprehension](https://dl.acm.org/doi/abs/10.1145/91556.91592))
to [database queries](https://ieeexplore.ieee.org/document/176921), 
leading to numerous variants
and cumulating with the Microsoft's Language Integrated Query ([LINQ](https://dl.acm.org/doi/10.1145/1142473.1142552)) system.
LINQ leverages the concept of monad comprehensions as well as the type system
to let the developers compose well-behaved queries guided by the types of data at hand.

* More recently, the term of functional database is used to emphasise
that the values manipulated by the database are immutable, in particular the core datasets.
Snapshots of the database are accumulated over time, transactions after transactions,
using the functional programming legacy of persistent data structures to optimize the space required for all these versions.
This approach can be used to improve the performance of the system with simpler synchronisation mechanisms.
But the main benefits are from a user perspective.
Queries are reproducible, leading to the same result when applied to the same version of the database.
Versions can be compared. The history can be queried.
Several systems are built on this idea: [Irmin](https://irmin.org/), [Datomic](https://www.datomic.com/) or [NixOs](https://nixos.org/) to name a few.

On the strength of these successes, I'm convinced that many pearls are still to be discovered.
And I have many promising ideas I would like to share!

- [Motivation](src/001_motivation.md)
