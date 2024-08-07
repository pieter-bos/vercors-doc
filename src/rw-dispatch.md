# Dispatch

A rewriter describes how a COL tree is transformed. The root of the tree is passed to one of the `dispatch` methods: each `dispatch` method decides what should happen next. By default we always recurse into the tree, querying `dispatch` for the child nodes of our current node. You might imagine that the default implementation of `dispatch` for a `Plus` node is structured like this:

```scala
def dispatch(e: Expr[Pre]): Expr[Post] =
  e match {
    case Plus(left, right) =>
      Plus(dispatch(left), dispatch(right))(e.o)

    /* ... other alternatives ... */
  }
```

## Structural transformations

The idea is now that you override dispatch methods for the nodes you would like to rewrite. For example, we might want to make an optimization that simplifies away additions of `0`:

```scala
val ZERO = BigInt(0)

override def dispatch(e: Expr[Pre]): Expr[Post] =
  e match {
    case Plus(IntegerValue(ZERO), e) => dispatch(e)
    case Plus(e, IntegerValue(ZERO)) => dispatch(e)

    case other => other.rewriteDefault()
  }
```

A new concept here is `rewriteDefault`: it is the mechanism used to peel off one layer of a program tree and recurse into its children using dispatch. It is automatically generated for each node, here in `PlusRewrite`. It takes an implicit value of type `AbstractRewriter`, which is the rewriter used to call `dispatch` again. By default it picks the `implicit val rewriter` which is defined for each `AbstractRewriter`. Note that Scala requires that a match statement is complete. A default case as above also makes the match complete.

A common mistake is to erroneously call `rewriteDefault` too often: when recursing into nodes yourself you should almost always call `dispatch`, which will decide to do with the sub-node.

## Transforming declarations

Declarations are special because it is not required that a declaration is transformed into a declaration. This is also represented in the signature of the `dispatch` method for `Declaration[_]`:

```scala
def dispatch(decl: Declaration[Pre]): Unit
```

Nevertheless declarations have to be placed in the tree, and references to declarations need to be rewritten. Both are arranged through an instance of the `Scopes` class — one for each kind of declaration. Rewriting references is tricky, so this is explained later on in the section on [succession](./rw-succession.md).

A node that contains declaration, such as `Class[G](decls: Seq[ClassDeclaration[G]], ...)`, needs to come up with a new list of `ClassDeclaration[G]`. We first have to consider the appropriate `Scopes` instance: in this case `classDeclarations`. We call `classDeclaration.collect`, which pushes a collection buffer onto a stack. A lambda is passed to `collect` in which we can take any actions, chiefly to declare declarations of kind `ClassDeclaration`. By convention we call `dispatch` on all the `Pre`-state declarations that we are interested in, and (optionally) we can declare one or more other `ClassDeclaration`s we need to add. All this is also allowed to happen recursively: we can be in a statement somewhere in the class and still declare a helper method to the `classDeclarations` scope.

Combining this information we can obtain the new declarations of a class as such:

```scala
val newDeclarations = classDeclarations.collect {
  cls.decls.foreach(dispatch)
}._1
```

Collect returns a tuple: a list of new declarations, and whatever the result of the lambda is (here: `Unit`). On the other side of the fence, the old class declarations are rewritten and declared to the scope. Reproducing the default behaviour for `ClassDeclaration`s might look like this, skipping over succession for the moment:

```scala
override def dispatch(decl: Declaration[Pre]): Unit =
  decl match {
    case decl: ClassDeclaration[Pre] =>
      classDeclarations.declare(decl.rewriteDefault())

    case other =>
      allScopes.anyDeclare(other.rewriteDefault())
  }
```

Our friend `rewriteDefault` makes another appearence, which peels off the `ClassDeclaration` constructor, and recurses into its components. In the case that a `ClassDeclaration` contains further declarations, it will call `collect` again in a similar way. For example, an `InstanceMethod` is a `ClassDeclaration` and contains declarations of kind `Variable` for its arguments.

After rewriting the declaration fully by opting to rewrite it in a one-to-one manner, it is declared to the buffer of `ClassDeclaration`s that we are currently collecting.

Lastly we must always complete the match, which is done by also rewriting other declarations one-to-one, and using `allScopes` to find the correct scope to put the declaration in.