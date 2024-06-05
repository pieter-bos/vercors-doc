# References
The normal way to create references to a declaration in a program is by naming it. This creates a few obligations, for example:

* Declarations must have a consistent and unique name with regards to its scope
* References to a declaration must name an existing declaration
* Renaming declaration must consider all its references
  (Otherwise, we may silently refer to the wrong declaration)
* Moving declarations or references across scopes must consider what declaration we are pointing to
  (Otherwise, we may silently refer to the wrong declaration)
* Importing internal definitions and generating side conditions must generate fresh unique names
  (Otherwise, we may silently refer to the wrong declaration)

We had this approach in earlier versions of VerCors, leading to declarations with a lot of underscores, and `VERCORS`/`SYSTEM`/`INTERNAL` prefixes. Even still this would probably be pretty manageable as long as the number of rewriters is not too big. As of writing we have 67 different rewriting steps, so this is no longer the case.

## No names!
We have instead elected to get rid of names entirely, replacing them with explicit references instead. This is a lie, because the initial input to VerCors of course *does* contain names, but it is a convenient lie: in all other passes the names are not relevant to rewriters. For debug output the preferred name for declarations is stored nevertheless in the `preferredName` of the origin of the declaration.

## Tying the knot
For simple declaration types like variables, we need to do nothing special to rewrite them. We can think of the following order to rewrite them:

* Rewrite all the variables of a `Scope`
* Rewrite the body of the `Scope`, which may or may not contain references to the variables.

We can then account which `Pre`-variable is rewritten to which `Post`-variable, and substitute references accordingly. Note that the variables of the `Scope` may themselves contain references to variables (e.g. type variables), but we can argue that by the structure of the scopes they have already been rewritten.

This convenient structure manifestly does not work for e.g. method invocations. Consider this simple program:

```
void p() {
	q();
}

void q() {
	p();
}
```

There is no way to order the rewrites correctly here:
1. We cannot name the succesor of `p` before rewriting `p`
2. We cannot rewrite the body of `p` before knowing the successor of `q`
3.  We cannot name the succesor of `q` before rewriting `q`
4. We cannot rewrite the body of `q` before knowing the successor of `p`
5. `goto 1`

The way we resolve this by making references lazy in general. We can still refer directly to declaration that we know of in context, such as temporary variables, by wrapping them in `DirectRef`. If not, we supply the computation that will resolve the reference to `LazyRef`. Importantly, this computation need not be valid immediately, it must be correct only after the `Rewriter`.

By default, we store one-to-one succesors in a map. References then by default construct a `LazyRef` that looks up the successor in this map. The actual mechanics of this are explained further in [Resolution](resolution.md) and [Succession](rewriting-succession.md)