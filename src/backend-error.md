# Error Translation

Viper returns errors of type `AbstractVerificationError`, which have to be translated to an appropriate `VerificationFailure` and given to an appropriate blame.

Nearly all errors contain at least an `offendingNode` and an `errorReason`, where the error reason again contains an `offendingNode`. To find an appropriate blame we first find the `NodeInfo` associated with the `offendingNode`, or the `NodeInfo` of the error reason's `offendingNode`. In some cases the node contained in the `NodeInfo` immediately is the appropriate node with a blame, otherwise one of the fields denote the nearby node that has the appropriate blame.

The translation of errors is trickier than you might expect. This is primarily due to the fact VerCors makes a distinction between errors on fallible statements, and well-formedness problems that happen to occur in a fallible statement. For example, VerCors considers these different kinds of error: 

* `fold p()` may fail because you do not have all the permissions in `p()`: this is a problem with the `fold` statement.
* `fold p(x / y)` may fail because `y` may be zero: this is a well-formedness problem, and is orthogonal to the `fold` statement.

Nevertheless Viper considers both these errors a `FoldFailed`. To determine what's what we have to first inspect the type of the `errorReason`:

* `AssertionFalse` is always a problem with the node viper is reporting on.
* `DivisionByZero`, `LabelledStatementNotReached`, `SeqIndexNegative`, `SeqIndexExceedsLength` and `MapKeyNotContained` are always well-formedness problems, and are reported on the node related to the `errorReason`.
* `InsufficientPermission` and `MagicWandChunkNotFound` depend on context. E.g. in `x.f := y.f` when the insufficient permission is `x.f`, the assignment is the problem, whereas for `y.f` the value expression is just not well-formed.

Finally there are two reasons that are a bit special in VerCors and Viper:

* `NegativePermission` and recently `NonPositivePermission`: this reason used to be embedded in the truth value of a contract, so `Perm(_, -1)` previously was simply equivalent to `false`. This was changed, and it is now a well-formedness problem: it belongs in the same category as `DivisionByZero` etc.
* `QPAssertionNotInjective` is currently a well-formedness condition by default, but this also used to be part of the truth value of a contract. This means that similarly an expression like `(\forall int i=0..2; Perm(x.f, 1\4))` would be equivalent to `false`. A difference with `NegativePermission` is that this behaviour is still recoverable with a backend flag: `--assumeInjectivityOnInhale`.

`NegativePermission` and `NonPositivePermission` are improperly supported in VerCors, and should be looked at. VerCors does not support `--assumeInjectivityOnInhale` and error translation will likely break if you try to use it.

If you've been keeping count, there are between 3 and 5 error reasons that make a contract false: `AssertionFalse`, `InsufficientPermission`, `MagicWandChunkNotFound`, and depending on flags and Viper version `NegativePermission` and `QPAssertionNotInjective`. In VerCors we call these a `ContractFailure`.

All this can be summarized in the following table:

| Reason | Error Node | Reason Node | Contract Failure
| - | - | - | - |
| DivisionByZero | ❌ | ✅ | ❌ |
| LabelledStatementNotReached | ❌ | ✅ | ❌ |
| SeqIndexNegative | ❌ | ✅ | ❌ |
| SeqIndexExceedsLength | ❌ | ✅ | ❌ |
| MapKeyNotContained | ❌ | ✅ | ❌ |
| AssertionFalse | ✅ | ❌ | ✅ |
| InsufficientPermission | ✅ | ✅ | ✅ |
| MagicWandChunkNotFound | ✅ | ✅ | ✅ |
| NegativePermission | ❌ | ✅<sup>1</sup> | ❌<sup>1</sup> |
| NonPositivePermission | ❌ | ✅<sup>2</sup> | ❌ |
| QPAssertionNotInjective | ✅<sup>3</sup> | ✅ | ✅<sup>3</sup> |

> <sup>1</sup>Currently still reported on the error node and treated as a contract failure
>
> <sup>2</sup>This error reason is not matched at all, and will likely crash VerCors if encountered
>
> <sup>3</sup>Treating this as a contract failure with `--assumeInjectivityOnInhale` is as yet unsupported