# Outline

When implementing a new rewriter you should make a class that extends `Rewriter`. Additionally it should have a companion object that extends `RewriterBuilder`: other than giving a `key` and `desc` to present to the user, it describes how to construct a rewriter with `apply`. The `apply` method is defined implicitly if you make your rewriter a `case class`.

The outline of an empty rewriter then looks like this:

```scala
package vct.col.rewrite

import vct.col.ast._
import vct.col.rewrite.{Generation, Rewriter, RewriterBuilder}
import vct.col.util.AstBuildHelpers._

case object MyRewriter extends RewriterBuilder {
  override def key: String = "my"
  override def desc: String = "Apply a transformation to the COL tree"
}

case class MyRewriter[Pre <: Generation]() extends Rewriter[Pre] {
  
}

```