# Nodes

COL programs are stored in a tree-like immutable data structure. All nodes are defined in `src/col/vct/col/ast/Node.scala`. Other than some primitive types and collection types like `Int`, `String`, `Seq[_]` and tuples `(_, _)`, all types in the tree are descendants of `Node`.

Nodes are split into two kinds:

* **Descendants of `NodeFamily`**. These are regular nodes, like expressions and statements. 
	* They have structural equality, meaning comparing them constitutes comparing their fields. 
	* It makes sense to say that each family of nodes should rewrite to itself: one `Statement` should always be rewritten to another `Statement`.
* **Descendants of `Declaration`**. These are nodes to which we can refer.
	* They have reference equality, which means a `new Variable(TInt) != new Variable(TInt)`.
	* It make sense to ask in which scope the declaration can be referred to. For example, a reference to an argument makes no sense outside the method it is declared in.
	* We should be able to rewrite a declaration to zero, one or more successors of itself.

We will not exhaustively describe all kinds of node here, but we highlight the ones that have special Properties.

## Expr
All expressions have a type:

```scala
trait ExprImpl[G] { this: Expr[G] =>
	def t: Type[G]
}
```

You may presume that the node is `check`ed before its type is queried, and may crash otherwise. The type of expressions is closely related to coercion.

## Type
The relations between types must be defined in `Types.leastCommonSuperType` and `CoercionUtils.getCoercion`.