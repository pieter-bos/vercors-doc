# Program Translation

To submit the program to Viper, the COL AST is transformed into a Viper AST. As opposed to the transformation steps, this is not done with a rewriter, but with explicit recursion. Like in the parser transformation stage we strive to maintain a standard of "no surprises" in the `ColToSilver` transformation: the COL ast should already nearly read like a Viper program before we translate it to Viper.

## Info

Each node in Viper carries an `info` field that can be used to store additional data unrelated to verification. We store a `NodeInfo` in this field, which remembers what COL node caused the generation of this Viper node. Additionally it remembers parent nodes of the node that may be relevant for blame accounting

## Position

Each Viper node also remembers a position. This is typically a file path, line number and column, but since the Viper position is normally not visible to the user of VerCors, we set it to something that is useful in debugging. It contains as text a file position, and a `unique_id` that can be programatically traced back to the node it corresponds to. This currently serves two purposes: 

* Identical errors about identical nodes are deduplicated, but when they occur in a different position in the tree (that is: they have distinct positions) they are not deduplicated. There is still a bug open about this [here](https://github.com/viperproject/silicon/issues/613), but there is presently no real issue, given that encoding different positions for different nodes is a valid solution.
* The position mostly survives when encoding to SMTLIB, e.g. when quantifiers are named. In quantifier reports we can thus recover the COL node related to the quantifier by matching on the `unique_id` portion of its name.