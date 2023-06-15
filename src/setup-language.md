# Language

VerCors is a JVM project, and the current main language we use is Scala. We do accept contributions in both Scala and Java, but we highly encourage you to learn some Scala before contributing.

Since the main backend of VerCors is currently Viper, we enjoy the benefit of calling into it directly and being able to debug it directly, since VerCors is also a JVM application.

We primarily develop VerCors in Scala for several reasons:

* Scala has great support for functional programming paradigms. VerCors primarily deals with immutable data-structures and pushes out effects, like logging verification failures and writing out files.
* Scala supports pattern matching, which is very suitable for analysing ASTs.
* Scala supports metaprogramming, which is very handy for generating functionality that recurses into the ASTs: rewriters, comparators, etc.
* Other nice to haves: good collection api (sequences, sets), optional structural equality, sealed types.

On the surface it seems Kotlin has significant feature overlap with Scala, but some of the above points are not available and are important. We may revisit that stance in the future.