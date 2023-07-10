# Coercion
Various nodes contain expressions, such as other expressions (e.g. `Plus` contains its two arguments) and non-expressions (e.g. `Branch` contains the conditions for its branches). Each expression position in the tree may induce a typing constraint on the expression in it. For example, the conditions in `Branch` have to be booleans:

```scala
case Branch(branches) => 
	Branch(branches.map { case (cond, effect) => (bool(cond), effect) })
```

and the arguments of `Plus` may be integers or rationals:

```scala
case Plus(left, right) =>
	firstOk(e, s"Expected both operands to be numeric, but got ${left.t} and ${right.t}.",
		Plus(int(left), int(right)),
		Plus(rat(left), rat(right)),
	)
```

Here we coerce each `cond : Expr[G]` into a boolean, and do nothing with `effect`. For `Plus` we pick the first of two transformations that succeeds: either both `left` and `right` can be coerced with `int`, or both can be coerced with `rat`.

Coercions serve two purposes: they check that the AST is well-typed, and can be used in `Rewriter`s to enact transformations on the AST.

## Type checking
The coercions in `CoercingRewriter` are run for every node during a `check` in the implementation of `DeclarationImpl` and `NodeFamilyImpl` (which together span all nodes). They use a special descendant of `CoercingRewriter` called `NopCoercingRewriter` that applies no transformtions along coercions.

All nodes that contain expressions should have their coercions defined in `CoercingRewriter`. If the expression should be in a simple type (class), you can use `coerce(expression : Expr[Pre], type : Type[Pre])`. Shorthands for several common types are defined, such as `rat`, `bool` and `int`.

If your node can be interpreted in multiple ways, determined by the type of its arguments, you can use the `firstOk` helper to select the first set of coercions that suceed on the arguments. Only if all alternatives fail an error is raised.

## Transformations
It is occasionally useful to implement transformation along the description of a coercion. Coercions are described by the node family `Coercion`. For example, coercing a `seq<int>` into a `seq<rational>` is described by `CoerceMapSeq(CoerceIntRat(), TInt(), TRational())`. Rewriters that extend `CoercingRewriter` have access to this coercion object by overriding `applyCoercion`. 

One example of this behaviour is in the overloaded interpretation of `null`. In VerCors the array types are entirely separate from the class types. Thus, `null` can be coerced into either an array or a class type. In the rewriter that encodes arrays, `ImportArray`, we filter for usages of `null` for arrays by translating a `CoerceNullArray` coercion to the appropriate value for a null array.

## Safety
Note that very nearly all defined/permissible coercions are promoting coercions. That is, they only admit coercions when a value definitely fits into the target type of the coercion. This is defined programatically by defining `isPromoting` in `CoercionImpl`. Currently the only exception is rationals into `zfrac` into `frac`. We have not investigated yet whether it is a good idea to define more implicit coercions to represent situations like demoting a `uint64` into a `uint32` - perhaps they are better suited as explicit non-`ApplyCoercion` nodes at an early point. Failures in coercion are reported to the blame in `Program`, mostly because we have no other good place to put it.

The other safety concern related to promoting coercions is that sets only map over promoting coercions due to the fact that the default coercion is defined as \\(x \in \textsf{input} \leftrightarrow \textsf{coerce}(x) \in \textsf{output}\\). Alternative encodings we investigated are cumbersome because they add additional necessary proof steps.