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

