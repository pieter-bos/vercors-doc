# Importing

Importing a library definition from a file is a fairly straightforward affair. Similar to regular user input, a file is parsed, resolved and somewhat rewritten, after which it is placed in the subject tree.

The `vct.importer.Util` object defines `loadPVLLibraryFile`, which parses, resolves and disambiguates an input file. It also caches the result of this chain for later use. It is currently used for datatype definitions and simplification rules.

Locating a declaration for reference in a library file is done by filtering on the `SourceName` of declarations. As opposed to its preferred name, the `SourceName` is the formal name exactly as it appeared in the input. As a matter of style, this is only used when immediately placing the library in the AST, after which we no longer consider the `SourceName` reliable. For example, it is perfectly valid that user code contains an overlapping `SourceName`.