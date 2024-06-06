# Pretty-Printing

The pretty printer of VerCors is derived from "A prettier printer" by Philip Wadler, accessible [here](https://homepages.inf.ed.ac.uk/wadler/papers/prettier/prettier.pdf). Typesetting a COL AST is done in three stages: it is first translated into a document tree, then a list of elements, then text.

# COL
Each node implements `layout`, which is externally accessible via `show`. The only purpose of `show` is to wrap the document tree with `NodeDoc`, so that we may later recall what document tree corresponds to what COL tree. If you forget to implement `layout` a debugging representation of the node is printed that does not look entirely terrible.

`Expr` is a bit more special: it demands an implementation of `def precedence: Int` for each expression. You can use `bind`, `assoc`, `nassoc`, `lassoc` and `rassoc` as conveniences to see whether sub-expressions require parentheses:

 - `bind` adds parentheses if the precedence of the subexpression is lower than the specified precedence
 - `assoc` adds parentheses if the precedence of the subexpression is strictly lower than our precedence (i.e. we "associate" with the subexpression)
 - `nassoc` adds parentheses if the precedence of the subexpression is less or equal than our precedence (i.e. we do not "associate" with the subexpression)
 - `lassoc` is the correct implementation for a left-associative binary operator (e.g. `+`, `*`)
 - `rassoc` is the correct implementation for a right-associative binary operator (e.g. `::`, `==>`)

# Document tree
The elements of a document tree are as follows:

 - `Empty`, which renders as nothing
 - `Text(String)`, which simply represents a piece of text. Must not contain newlines.
 - `Cons(Doc, Doc)` (or `Doc <> Doc`), which represents the unspaced concatenation of two trees.
 - `Nest(Doc)`, which indicates the tree should be nested (i.e. indented)
 - `Group(Doc)`, which indicates that newlines should preferrably be removed from the tree.
 - `Line` and `NonWsLine`, which are both newlines. If collapsed in a `Group`, only `Line` is replaced with one space.
 - `NodeDoc(Node[_], Doc)`, which has no effect on typesetting, but indicates this document tree corresponds to this node.

The deviations from Wadler's algorithm are:

 - We have no `<|>` to indicate alternatives. Instead our very opinionated `Group` has in effect two alternatives: we prefer for it to be printed on one line, but it may keep the `Line`s and `NonWsLine`s if necessary to reduce its horizontal occupancy. When `Group` is nested we first peel off the outer `Group`, keeping the inner `Group` if possible.
 - The `flatten` of `NonWsLine` is `Empty` instead of `Text(" ")`
 - `NodeDoc` is there for post-processing

Otherwise the typesetting algorithm is a straightforward transliteration of the paper.

`Doc` has several convenient operators to make trees:

 - On the right hand side of operators, you can write anything that implements `Show`, such as `Node[_]`
 - `x <> y` is `Cons(x, y)`
 - `x <+> y` is `x <> " " <> y`
 - `x </> y` is `x <> NonWsLine <> y`
 - `x <+/> y` is `x <> Line <> y`
 - `x <>> y` is `x <> Nest(Line <> y)`
 - `Doc.fold` applies a function on pairs of `Doc` until there is only one `Doc`. If the sequence of `Doc`s is empty, the result is `Empty`.
 - `Doc.spread`, `Doc.lspread` and `Doc.rspread` place spaces between `Doc`s. `r` has an extra space on the right, `l` has an extra space on the left.
 - `Doc.stack` places newlines between `Doc`s that reduce to a space in a `Group`.
 - `Doc.arg` indents a `Doc` on a new line, then appends another newline. Both newlines reduce to nothing in a `Group`. Appropriate for an argument to a function or control structure like `while`, when wrapped in a `Group`.
 - `Doc.args` combines comma-separated arguments with newlines that reduce to a space.
 - `Doc.spec` and `Doc.inlineSpec` surround the `Doc` with `/*@ @*/` if necessary. Note: the context only indicates to the tree argument that we are now in a spec if the argument to `Doc.spec` is not yet a `Doc`, but something yet to be showed. This can be delayed with `Show.lazily` if necessary.

# Elements
After typesetting the document we are left with a list of elements, which are one of two variants:

- `ELine(indent: Int)`, which renders a new line with a given amount of indentation. The context contains an indentation string, that is repeated `indent` amount of times
- `EText(text: String)`, which is rendered as its constitutent text.

Additionally, we again remember what corresponds to which node, for post-processing:

- `EStart(Node[_])` indicates the start of a region that represents that node.
- `EEnd(Node[_])` indicates the end of that region.

The metadata is used in `Doc.highlight`, which can point to a node in a rendered tree.

Any node that implements `Show` has convenience functions `write(Appendable)` and `toStringWithContext(Ctx)` to convert the `Show` to text.

# Context
The class `vct.col.print.Ctx` is available when rendering nodes and can be updated when the parent context is relevant (such as being inside a specification). Otherwise it contains information like declaration names, the horizontal width, etc.