# Blames

One of the core tenets of VerCors is that technical verification errors are translated as precisely as possible to a problem in the input to the tool.

We submit a large query with many verification goals to our backend, Viper, and receive back a list of errors. This list is translated backwards to relate to the input through the blame system: each node that induces a verification goal must explain what to do with the error. The most common way to explain away an error is to just blame somebody else! Each rewriter gets an input tree, along with blames for each goal-inducing node. On the output side, we have to provide the blame instead, but for simpler rewrites there usually is an apparent blame we can use from the input side.

## Blaming somebody else

If the type of error on the input and output side are the same, we can simply place the blame in the output tree without rewriting it — blames are special in this sense. A slight complication occurs when the type does not match. For example, we might be tranlating a well-formedness condition like `Blame[MapKeyError]` to a function precondition blame of type `Blame[InvocationFailure]`.

In this case we have to indicate how the error is translated:

```scala
case class MapKeyPreconditionFailed(get: MapGet[_]) extends Blame[InvocationFailure] {
  override def blame(error: InvocationFailure): Unit =
    error match {
      case PreconditionFailed(_, _, _) =>
        get.blame.blame(MapKeyError(get))
      case ContextEverywhereFailedInPre(_, _) =>
        PanicBlame("This rewriter does not generate context_everywhere clauses for map get functions")
          .blame(error)
    }
}
```

If the precondition of the function fails in some position, we translate the error to be the original well-formedness conditions on a map get operation.

## Blaming the system

We declare that it is impossible that `ContextEverywhereFailedInPre` occurs, because we guarantee in the rewriter that the `context_everywhere` clause of the function we generate is `true`. It is important that we do not just silence the error.

Note: Although this style of blame transformation works, it is too easy to silence errors this way. In fact: this snippet would still compile if the `PanicBlame` line was omitted entirely. We should consider making blames functional: it should return a (list of) errors instead of invoking a further blame explicitly.

## Blaming everybody else

For some positions where a contract is specified, you can also depend on what part of the contract is false. You can find these positions through the fact that they contain an `AccountedPredicate[G]` instead of an `Expr[G]`. They additionally report a sequence of `AccountedDirection` in the appropriate failure, which you can match on in the blame. In fact: you need to be careful to ensure that when the shape of an `AccountedPredicate` changes, the matching `blame` needs to also account for the changed shape.

Typically a contract expression is added to the start or end of an accounted predicate. For example, `context_everywhere` is transated to be a regular pre- and postcondition as such:

```scala
contract.rewrite(
  contextEverywhere = tt,
  requires = SplitAccountedPredicate(
    UnitAccountedPredicate(freshInvariants()),
    dispatch(contract.requires),
  ),
  ensures = SplitAccountedPredicate(
    UnitAccountedPredicate(freshInvariants()),
    dispatch(contract.ensures),
  ),
)
```

Here the `context_everywhere` specifications are prepended, so we have to call the original blame when the accounted predicate path starts with `FailRight`:

```scala
inv.rewrite(blame =
  PreBlameSplit
    .left(ContextEverywherePreconditionFailed(inv), inv.blame)
)
```

Note that although there is only a few places where `AccountedPredicate` is used, this is really only for convenience: there is no strong reason to keep e.g. loop invariants an `Expr` other than convenience.

## Blaming the user

So far we have looked at blames that blame other blames, but they do come from somewhere. All the transformations of parse trees to COL trees keep around a `blameProvider`, which explains how to create the initial blame. Note: although the constructed blame can optionally depend on the position, this is not usually used to determine the location at the source. Instead, the origin of the relevant node in the `VerificationFailure` is typically used. This design is a bit shaky and should be reviewed.

This process of constructing an initial blame is __only__ done in the parse tree stage: from then on new blames are not created. The typical way to provide blame is to pass a `ConstantBlameProvider` that gives out a constant `BlameCollector` for every blame: a facility that simply stores a list of `VerificationFailure`s in a buffer.