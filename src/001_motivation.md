# Motivation

Databases offer a *powerful abstraction* over *complex mechanisms*.
But this comes with a cost. In order to enforce the abstraction boundary with maximum efficiency,
these systems are by design *closed*, *opaque* and *monolithic*.

This is not an issue as long as an application uses a single database storing all the data.
However, data kinds and workloads are nowadays so diverse
that both the [industry](https://www.allthingsdistributed.com/2018/06/purpose-built-databases-in-aws.html)
and the [academics](https://dl.acm.org/doi/10.1109/ICDE.2005.1)
recommend a multi-specialized-databases approach to cope with conflicting requirements,
say to offer both sophisticated analytics over large volumes of data and instant reactions over high-velocity event streams.

In this context of data diversity and engine specialization, the close and opaque design of the databases makes things really complex to get right.
The application software has to move data around systems and to solve consistency and availability issues across boundaries.
Somehow the abstraction is broken and one has to rebuild it around incompatible engines.

## Can databases be less monolithic but still efficient?

The abstraction implemented by databases let the users manipulate the data independently of the mechanisms actually used to process them.
The user is given query expression power over data with a simple, uniform and natural shape;
while the engine can use the most appropriate representations even if complex, ad-hoc or opaque.

However, this abstraction establishes a barrier that is more than inconvenient.

* Data outside the database are not first-class citizens and have first to be imported even for a basic query.
  Query processing is independent of the actual data layout, but this layout is chosen by the engine's designers, not the users.
  Exploratory of large datasets would be easier with the ability to directly query these datasets using the original formats.
  This is also the core issues when you have to combine two specialized databases in your application: each excludes the data of the other.
* Computations inside the database are more constrained than outside.
  And this is made worse by SQL being not extensible with libraries.
  To manipulate new data kinds, one has to wait for these to be added by the database editors.
  In practice, many data transformations are done outside the database, introducing unnecessary movements and inefficiency.
* There is a chasm between the internal and external concepts.
  What is visible at the surface of a database are essentially abstractions:
  tables manipulated with operators of the relational algebra.
  Under the hood, components are of a wholly different nature and level of abstraction:
  files, pages, caches, indexes, redo logs, locks ...
  Having intermediate abstraction levels would help advanced users
  to encapsulate external data sources or to introduce new database operators.

On the other side, database engines are monolithic for a reason.
The essential properties that a database must guarantee are transversal. 
Durability, consistency, availability and performance must be considered system-wide.
For example, a query plan optimizer combines aspects related to query semantics,
with operator performance data as well as statistics on the data distribution. 
How can a query optimizer work in a more open setting with user defined data layouts and operators?

Different approaches can be taken.

1. The first one is to stay in the continuity of what has been done so far:
   to accept the close nature of databases as a necessity for performance and reliability.
   It is then necessary to extend the domain of applicability as done
   by [HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing) databases handling both OLTP and OLAP workloads,
   or by [FaunaDB unifying major data models](https://fauna.com/blog/unifying-relational-document-graph-and-temporal-data-models).

2. By contrast, one can embrace the diversity of data sources and organize data movements. 
   It can be around a system like Kafka which acts as a transactional message hub across databases and applications.
   One can also use a system like Spark or Flink to distribute computations over independent data sources.  
   
3. Finally, one can also build a database engine specifically designed for the application at hand. 
   For that to be realistic, such a bespoke database needs to be built using building blocks components for key aspects as storage, query optimization, query processing, transactions ...
   But, how can be such components designed and composed, given the transversal nature of the key database properties?  

Despite the apparent obstacles, this is that third direction that I propose to explore.

## Database à la carte

What I want to explore is the idea to use modules à la SML/OCaml to build a database as an assemblage of data modules.

An OCaml module is a compilation unit that encapsulates types and values behind a signature that defines what is accessible from the outside,
so these types and values can be used from other modules and the modules combined into larger ones.
Notably, a signature can abstract the actual type of values, by not providing the actual representations,
still letting the other modules use these values consistently.
More specifically, in the context of a database, one can imagine a module that contains indexed values and exposes functions to efficiently access these values
without providing the internal representation of the dataset and indexes.


The idea is to __abstract a dataset__ as a module that provides not only the data but also the mechanisms to process these data as well as type and cost information.
In this setting, a data module contains:

* inter-related values forming the __actual content__ of the dataset including indexes,
* a __schema__ describing the types of these values and their relationships,
* __accessor functions__ to efficiently retrieve specific subsets of the values,
* __cost information__ for the accessor functions.

The challenge is then to design these modules so:

* datasets can be combined into larger ones,
* queries can be written over a combination of datasets using the individual schemas to check the soundness of these queries,
* queries can be written independently of the actual storage layout of the data,
* query plans can be built as a combination of accessor functions,
* query plans can be optimized using individual cost information.

The roadmap I propose is:

* to use the OCaml type system to represent a database schema using abstract types for the entities
  and binary-relations to navigate between entities and properties,
  i.e. using a statically typed data model akin to the 
 [entity–relationship model](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model),
* to use a combination of collection values, indexes, and functions to implement binary-relations
  with different constraints, capabilities, and efficiency but the same external interpretation
* to leverage the OCaml module system to build applications from modules independently written
  and defining the schema, the storage layer, the application queries, and the query engine.

These ideas are explored in more detail in this [prototype](https://github.com/didier-wenzek/data_module)
