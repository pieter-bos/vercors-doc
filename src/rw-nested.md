# Nested Rewriters

Occasionally it can be useful to nest rewriters. It essentially divorces the rewriting rules entirely from the parent rewriter when you invoke the nested rewriter. This means that any match arms or other logic in `dispatch` implementations in the parent rewriter are not used when inside the nested rewriter.

It is rare that a nested rewriter is actually what you want: usually it is more explicit and clear to account the desired behaviour with a piece of state in one rewriter. As an example we can consider the rewriter that collects inline patterns of quantifiers: its task is to scan for any patterns that occur on the inside of a quantifier marked with `{: ... :}`, and apply them as triggers for the quantifier. It may be tempting to do this with a rewriter: we ignore patterns while recursing into the tree until we encounter e.g. a `Forall`, and then our behaviour changes entirely: we instead remove patterns and collect them into some buffer — why not rewrite the body of the quantifier with a nested rewriter? 

The problem becomes apparent when you consider that a `Forall` can also contain a nested `Forall`. When scanning for patterns we encounter a nested `Forall` while in the nested rewriter. Do we call to the outer rewriter again? We can apply another nesting level in the rewriter, but where do we save patterns that are in the nested `Forall`? What if we add support for triggers for `Exists`, do we add the pattern in both places?

Instead of implementing this behaviour with a nested rewriter it is more straightforward to maintain a stack of buffers that collect patterns, and have the match arm for `Forall` and `InlinePattern` on the same level. `Forall` puts a collection buffer on the stack, whereas `InlinePattern` adds itself to the buffer on the top of the stack. It is a bit more explicit and stateful, but in actuality it is a better admission of how your rewriter is structured: a nested rewriter just thinly veils this state.

## Declaring a nested rewriter

By convention a nested rewriter is usually declared inside the parent rewriter it is used in:

```scala
case class OuterRewriter[Pre <: Generation]() extends Rewriter[Pre] {
	case class NestedRewriter[Pre <: Generation]() extends Rewriter[Pre] {
		// ...
	}
}
```

The only thing that is immediately necessary to consider is that of scopes. When communicating a tree from the parent to the nested rewriter, or vice versa, it is important that the rewriters agree on how to arrange declarations, and whether succession records in the nested rewriter are visible to the parent rewriter.

The most straightforward way to arrange scopes and succession is to just let the nested rewriter inherit the `allScopes` field of the parent rewriter:

```scala
case class OuterRewriter[Pre <: Generation]() extends Rewriter[Pre] {
	outer =>

	case class NestedRewriter[Pre <: Generation]() extends Rewriter[Pre] {
		override val allScopes = outer.allScopes
	}
}
```

This is the only reasonable option if it is required that the nested rewriter returns a tree of the `Post` generation. There is more flexibility if you can accept that the nested rewriter returns a `Pre`-tree. For example, the `Substitute` utility has an entirely fresh `allScopes` and uses it of declarations inside the tree that is being substituted, but returns references to declarations as-is when the successor is not known inside the substitution:


```scala
case class Substitute[G](/* ... */) extends NonLatchingRewriter[G, G] {

  case class SuccOrIdentity() extends SuccessorsProviderTrafo[G, G](allScopes) {
    override def postTransform[T <: Declaration[G]](
        pre: Declaration[G],
        post: Option[T],
    ): Option[T] = Some(post.getOrElse(pre.asInstanceOf[T]))
  }

  override def succProvider: SuccessorsProvider[G, G] = SuccOrIdentity()

  /* ... */
}
```

Notice that `Substitute` is thus not suitable when subtituting a tree that contains declarations that must be scoped _outside_ the tree that is being substituted. This is explained further in [Substitution](./rw-substitute.md).

Another option is to recognize that the declarations of the tree being rewritten are entirely disjoint from the parent tree. This means that there are no references from the parent to the subtree, or from the subtree to the parent. An example of this is the simplifier: it loads rules from a file, which of course cannot contain a reference to the tree being simplified, nor can the subject tree refer to simplification rules it has not loaded. Thus: rewriting a rule in a nested rewriter with fresh scopes is perfectly fine.