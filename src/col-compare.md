# Comparing

## Equality

All nodes already implement `==`, which corresponds to the property that the nodes are semantically equivalent. This means that the following properties of nodes are considered when checking them for equality:

- Descendants of `NodeFamily` are deemed equal by structural equality: they must be of the same type, and have the same fields recursively.
- Descendants of `Declaration` are deemed equal by referential equality: two separate instances of structurally equal `Declaration`s are *not* equal.
- The `blame` of nodes is *not* considered in equality: this is an unfortunate design mistake. Blames should be redesigned so that they are a structural component of the AST.
- The origin of nodes is not considered in equality: this is correct, because the origin only contains metadata not relevant for the boolean verification result of the node.

If this notion of equality does not fit, you can instead use `compare`.

## Comparison
Each node implements `compare` through code generation. Its only argument is the node to compare to, and it returns a lazy list of `CompareResult`. The idea is that `compare` recurses into the left and right argument simultaneously, reporting any differences along the way. The three alternatives for `CompareResult` are:

- `StructuralDifference`: in this subtree the kinds of node are different, or they have a different structure at that level. For example: a node that contains a `Seq` has a differing number of arguments.
- `MatchingReference`: so far the tree is structurally equal, and we have recursed to a point that left and right are a reference to a declaration. The references may or may not be equal.
- `MatchingDeclaration`: these declarations are structurally equivalent, but they may or may not be reference-equal.

From here several useful abstractions are implemented:

- `Compare.compare`: the same as `compare` above, but attempt to reconcile structural differences by bringing left and right into a specified normal form. This is useful to compare nodes e.g. under equivalence of associativity of `**`.
- `Compare.getIsomorphism`: attempts to match up `MatchingReference` and `MatchingDeclaration` that point to the same declarations. If this succeeds, we say that left and right are isomorphic by substituting left declaration for right declarations. In that case the result is `Right`: a map of left-declaration to right-declaration. If not, a list of differences is returned: either structural differences or declarations that cannot be reconciled.
- `ApplyTermRewriter`: uses `compare` to implement rewriting rules, by explicitly accepting a `StructuralDifference` in the case that the left term is a `Local` used in the rule pattern.