# Succession

_See also: [Transforming Declarations](./rw-dispatch.md#transforming-declarations)_

The succession system is a pure convenience: it accounts when there is exactly one canonical successor of a declaration, then uses that successor to rewrite references to the declaration. When rewriting a declaration is left to the default dispatch implementation, there is always one canonical successor: the declaration it is rewritten into. This can be witnessed in the default implementation of `dispatch` for declarations, which is defined in `NonLatchingRewriter`:

```scala
override def dispatch(decl: Declaration[Pre]): Unit =
  allScopes.anySucceed(decl, decl.rewriteDefault())
```

The method `anySucceed` both declares the new declaration, and accounts it as the successor of the old declaration. In the rare case that you need to only account the successor of the declaration, but not declare it, you can call `anySucceedOnly` instead.

Note it is typically not necessary to use `allScopes`: it is simply a dispatcher for the scopes by declaration kind, like `globalDeclarations`, `classDeclarations`, etc. For example, you can declare and account the successor of a procedure as such:

```scala
globalDeclarations.succeed(oldProc, newProc)
```

## Rewriting references

The default way of rewriting any reference is to query the successor of the declaration it refers to. Note that this is by default always done lazily: the succession accounting may be delayed until the first time the reference is queried with `_.decl`.

There is no obligation to always use an instance of `Scopes` to account successors. For convenience you can opt to use `SuccessionMap`, which is a thin layer over a thread-safe regular map that allows the construction of `Ref`s that query the map lazily, and provide some useful error otherwise. Even this is not mandatory: you can choose to use e.g. a `mutable.Map` instead if you want. It is however important to remember the law of succession:

> When the successor of a declaration is not accounted, you must consider __all__ positions that can reference that kind of declaration.

Luckily for most kinds of declarations there are not that many nodes that can refer to it. It is helpful to consider that references to a broad type are extremely sparse. In particular, nearly all references at least stay within the type of its kind (e.g. `ClassDeclaration[G]`, `GlobalDeclaration[G]`) and the vast majority refer to a leaf type.

## Scoping

The successor of a declaration is visible only in the scope that the declaration itself is visible. For example: it does not make sense to ask about the successor of a local variable, if the variable belongs to a different method. What this enables is the simple-sounding property that it is usually safe to rewrite a node twice: the successors accounted in the second rewrite do not clobber the successors accounted in the first rewrite, because they are scoped correctly. We call nodes for which it is safe to rewrite them twice duplicable.

Most structural nodes are duplicable, and declarations are themselves not duplicable: you are not allowed to overwrite the successor of a declaration within one scope. Note that structural nodes that _contain_ declarations (e.g. `Program`, `Forall`) are usually duplicable. The property that determines whether a node is duplicable is as such: whenever a declaration can (transitively) occur in a node, it must have a parent Node that `@scopes` that kind of declaration.

