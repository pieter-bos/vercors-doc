# Adding a Language

Adding a new language (also: frontend) to VerCors is a large task. To start, consider the following list:

* Add a language and extension mapping in `vct.main.stages.Parsing.Language` and `vct.options.types.ReadLanguage`
* Add a grammar for your language.
    * Our default stance is to parse languages ourselves with ANTLR v4. See if yours is available [here](https://github.com/antlr/grammars-v4).
    * Place your lexer and parser as `LangXLexer.g4` and `LangXParser.g4` in `src/parsers/antlr4`.
    * `LangXLexer.g4` should import `SpecLexer.g4`. Make sensible lexer rules to define specifications inside comments. See the bottom of `LangJavaLexer` for an example.
    * Make `XParser.g4` which imports `SpecParser` and `LangXParser`. This should connect the specification and language by defining the missing grammar rules. See `JavaParser.g4` for an example.
    * Change `LangXParser.g4` to include specification constructs like contracts, `with`/`then` specifications, and more. All specification rules start with `val`, so try searching for `val` in `LangJavaParser.g4`.
* Implement a parser in `src/parsers/vct/parsers/parser/ColXParser` that extends `AntlrParser`.
* Implement a converter in `src/parser/vct/parsers/transform/XToCol` that extends `ToCol`. The general process is to start with the root rule of your grammar, match its alternatives, and implement further `convert` methods as you encounter parse rules. Anything that is not supported can be put aside by calling `??(rule)`.
* Connect your `Parsing.Language` and `ColXParser` parser in `vct.main.stages.Parsing.run`.

Congrats, you added a frontend! Most likely the real work only starts from here, though. Consider the following:

* If your language adds features that did not exist in VerCors before, consider adding them to the "core" of COL by adding a nice abstraction of your new language feature. For example, we started supporting GPU kernels, for which we added an abstract `ParBlock`.
* Your converter should contain no logic. Instead, implement any logic prefacing the normal rewriter chain in a language-specific rewriter. These rules are called in `LangSpecificToCol`, then deferred to specific rewriters like `LangXToCol`.
* When possible, prefer simple logic in `LangSpecificToCol` connecting to a nice abstraction in COL. It is helpful to future developers of VerCors that the rewriting logic occurs in the main rewriting chain when possible.
* Nodes are cheap! Instead of bolting on new behaviour to existing nodes, just make your own.
* Add rewriters in `vct.rewrite` that compile away your new features to existing features.
    * They should be added somewhere in the rewrite chain of `vct.main.stages.SilverTransformation`.
    * Prefer writing rewriters that do not care where they are in the chain.
* Unexpected results? Try using flags like `--output-after-pass`, `--trace-col`, `--backend-file-base`.
* Unacceptably slow parsing? See [here](https://github.com/utwente-fmt/vercors/wiki/Parser-Analysis).
