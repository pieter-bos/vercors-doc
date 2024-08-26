# New modes

The different tools that are built with or on top of VerCors, such as Alpinist, VeSUV and VeyMont, are internally implemented as modes of VerCors. To start:

* Add a mode in `vct.options.types.Mode`
* Add a flag that sets your mode in `vct.options.Options`
	* Also add any additional mode-specific options under your flag as children of that flag.
* Match your mode in `vct.main.Main.runMode`
* The outer shell of your mode should typically be an assembly of `Stages`, which is helpful in progress printing.
* Try to re-use stages in vct.main.stages if possible, otherwise implement your own logic as a `hre.stages.Stage`.