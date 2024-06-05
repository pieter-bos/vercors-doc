# Resolution

Resolution is the process of assigning a meaning to a text name. Resolution occurs in two stages:

* `vct.col.resolve.ResolveTypes`: Resolves the meaning of types, and loads any library types that are mentioned but not in the input. Declaration with a type can be important in resoving other references, such as field dereferences.
* `vct.col.resolve.ResolveReferences`: Resolves everything else.

Both resolution stages have the same structure. At each node we receive a context that decides how we resolve that particular node. Before resolving any node, we fist resolve all of its children. This is because the type and well-formdness of children is more often relevant to the parent, than the well-formdness of the parent is to the children. The context is expanded upon by a node before passing it to its children. The recipe is thus as such:

1. `enterContext` decides how to expand the context from a node
2. `resolve` recureses into the node's children with the updated context
3. `resolveOne` is called for the node, which may assume the children are well-formed.

`ResolveTypes` is followed by its companion rewriter `LangTypesToCol`, which sets the resolved references in mutable state in stone. Dually the `ResolveReferences` resolver is followed by `LangSpecificToCol`.

## From parse tree to COL tree
VerCors has two ways of representing a text name:

* A node contains a `Ref` that is of type `UnresolvedRef`. This is appropriate for places where the name is exactly one label (as opposed to a fully qualified name like `java.lang.String`), and will clearly resolve to a category of declaration. This is unfortunately quite often not the case.
* A node remembers the text name temporarily, and contains mutable state to receive the result of resolution. An example of this is `JavaLocal`.

This is an area of VerCors that should be refactored: there should be one clear way to represent an unresolved name.

## `UnresolvedRef`
This class contains mutable state in its field `resolvedDecl`, which can be filled by calling `UnresolvedRef.resolve`. Most occurences of `Ref` have an entry in `ResolveReferences`, but it is not clear that there is always a sensible way to implement this.

## `Referrable`
Any other situation should just remember the text name from the input in a node, and contain mutable state that receives the result of resolving it. By convention, this is a `var ref: Option[?] = None`. Other code like `check` and `t` may assume the `ref` is filled.

Usually the resolution result points to something in the AST, in which case you should use `Referrable`. It can point to the following kinds of thing:

* Exactly one `Declaration`;
* A portion of a declaration, such as field #2 in the multi-declaration `int x, y, z;`;
* A reference to an intrinsic part of the language that cannot be directly parsed, such as cuda's `blockDim`;
* A reference to a VerCors-built-in concept, like the `.size` of a sequence;
* A reference to a construct that exists implicitly, such as Java's default constructor.

It is a bit frustrating that all declarations are essentially duplicated under `Referrable`, but it does give us the flexibility to make names point to whatever we want.

## From COL tree to resolved tree
When using an `UnresolvedRef` or adding a `Option[Referrable]` variable, we should take care that an appropriate entry is added in `Resolve{References,Types}.resolveOne`. We split out the resolution by language (including the specification language), which usually derives an appropriate `Referrable` from the `{Type,Reference}ResolutionContext`. Note again that we are allowed to e.g. query the type of child nodes, which is very relevant in resolving e.g. instance methods.

## From resolved tree to `Ref`
The companion rewriters `LangTypesToCol` and `LangSpecificToCol` translate resolved references into a `Ref` (that is not an `UnresolvedRef`). From then on references are defined only by pointing at the declaration it resolved. A consequence of this is that past the language-specific pass pointing to anything other than exactly one declaration as above is not possible.

Note: the `LangSpecificToCol` pass has accumulated far too much logic. The ideal path to implement a feature in VerCors is that you design new concepts in the core of COL that you need, and figure out a straightforward way to map to it in `LangSpecificToCol`. The task of `LangSpecificToCol` is not to lower language features into low-level concepts immediately, and VerCors often benefits from well-designed features in the core of COL across front-ends.