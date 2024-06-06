# Checking

In general we attempt to design the AST in such a way that it is not possible to represent invalid situations. When that is not possible, we implement structural checks on the AST when it is convenient.

# The `Ambiguous` pattern

To avoid duplicating code we implement a rigid structure first in a lot of cases, that does not yet connect to the frontends. An example of this is `Location`, which has rigid alternatives per type of heap location. Since `Perm(x.f, _)` simply contains an expression `x.f` rather than something that is immediately identifiable as a location, we punch a hole in `Location` named `AmbiguousLocation` that can contain that expression for so long as it is not yet resolved. We then implement a rewriter that scans for the allowed forms of expression in `AmbiguousLocation` and translates them to a proper location. If the form is not proper, we can simply throw a `UserError` in the rewriter.

The alternative is that we implement `check` for `AmbiguousLocation` and ensure that it contains a correct location. It is not that clear when a location is valid. For example, a `JavaDeref` can be resolved to a built-in like `seqn.size`. That does not resolve to a correct location, but it is more convenient to wait until the `JavaDeref` becomes a `Size`, rather than needing to be aware of every possible resolution of `JavaDeref` in `AmbiguousLocation`.

# Type checks

One dimension along which we cannot escape checking is that of type constraints. Every node that **contains** an expression defines whether it imposes a type constraint on it in `CoercingRewriter`. This coercion is computed in the implementation of `check` for every node.

# Structural checks

As said before the amount of structural checks implemented in VerCors currently is very limited. Nevertheless we are still missing some important ones at time of writing, so we should look into this more.

The ones that do exist are implemented by extending `CheckError`, and then overriding `check` in the Impl-trait of the node you want to implement a check for. Note that it is typically wanted behaviour that the `super`-implementation of `check` is included in the result.