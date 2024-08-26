# New specifications

To add a specification to VerCors, you should consider whether your specification is language-specific, or whether it can be implemented as a language-agnostic feature.

For language-agnostic specifications:

* Add grammar rules to `SpecParser.g4`
* Add new keywords in `SpecLexer.g4`. Consider prefixing your keyword with a backslash.
* Add translation logic in all instances of `vct.parsers.XToCol`. An unfortunate fact is that the specification translation is kept the same between languages by copy-pasting all specification translation manually. This is because the specification grammar types are different by frontend, but structurally identical.
* Nodes are cheap! Instead of bolting on new behaviour to existing nodes, just make your own.
* Add rewriters in `vct.rewrite` that compile away your new features to existing features.
	* They should be added somewhere in the rewrite chain of `vct.main.stages.SilverTransformation`.
	* Prefer writing rewriters that do not care where they are in the chain.

We have few specifications that are language-specific on the parsing level. A notable exception is `valClassDeclaration`, which contains rules for declarations that only make sense when they belong to a Java-style class.