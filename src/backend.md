# Backend and Errors

Our backend accepts a vastly reduced input language, and must call the appropriate blame when a verification goal fails:

```scala
trait Backend[P] {
  def transform(program: Program[_], output: Option[Path]): P
  def submit(intermediateProgram: P): Boolean
}
```

The `Backend` trait presumes that the input program first must be translated to the internal language of the backend. The boolean returned by `submit` indicates whether there were any verification failures at all This is used because verification failures can potentially be spurious. That is: it is possible that with additional help or different random heuristics the proof might succeed. It is thus not safe to cache the result of a program verification when there are failures.

## Viper

Our current backend is [Viper](http://viper.ethz.ch/tutorial/). It is an [intermediate language](https://github.com/viperproject/silver) and verification tool that accepts simple imperative programs with annotation in first-order permission-based separation logic. The input is straightforwardly translated to the Viper language, then submitted to one of Viper's two backends:

* [Silicon](https://github.com/viperproject/silicon/), which uses a symbolic execution approach, and interrogates [z3](https://github.com/z3prover/z3). Silicon also has flags that enable it to talk to cvc5, but so far this seems of limited use.
* [Carbon](https://github.com/viperproject/carbon), which uses verification condition generation, and submits a translated program to [Boogie](https://github.com/boogie-org/boogie).

On balance Silicon is almost always faster, so most support in VerCors is geared towards that backend. It is also the default.

## History

Earlier versions of VerCors encoded to [Chalice](https://www.pm.inf.ethz.ch/research/chalice.html), [Boogie](https://github.com/boogie-org/boogie) and [Dafny](https://dafny.org/). These approaches are in the git history of VerCors, but are otherwise lost to time. Some details as to what fits onto what these days:

* The carbon verifier of Viper still encodes to Boogie
* The original version of Chalice encodes to Boogie
* A [newer implementation](https://github.com/viperproject/chalice2silver) of Chalice encodes to Viper instead, though it is unclear whether this is maintained.
* Silver is an internal name for the Viper language, and should no longer be used. Using it in code is fine, but prefer using "Viper language" in writing.