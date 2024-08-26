# Substitution

`vct.col.util.Subsitute` is a utility that allows you to replace expressions and types according to a map. As noted in [Nested Rewriters](./rw-nested.md) it is just a rewriter, but it is a bit more convenient to use safely.

> __Rule 1__: Substitution can only occur in the `Pre` generation.

> __Rule 2__: Substitution can only be applied to structural nodes.

Since Substitute needs to be able to rewrite declarations to rewrite types and expressions that occur in declarations, `Substitute` cannot work on `Post` trees when it contains references that are not known yet. Additionally, you can not perform substitutions in a tree that contain declarations that need to be declared outside the substitution. This means that generally you can apply `Substitute` to structural nodes, but not declarations. It is perfectly possible to design nodes in such a way that a structural node (transitively) contains a declaration, but does not scope it. This is pretty rare, though an example is `IterVariable`.

This limitation is a reflection of the fact that substituting a declaration that needs to be accessible outside the substitution necessarily imposes an obligation on the parent rewriter to explain what the successor of the declaration is. An alternative is that `Substitute` instead inherits the `allScopes` of the parent rewriter, but this would make `Substitute` a `Pre`-to-`Post` rewriter: we could no longer substitute again in the tree, and it cannot be further processed in the parent rewriter, limiting its use.