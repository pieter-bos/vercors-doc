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