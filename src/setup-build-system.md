# Build System

The default build system for Scala, and hybrid Scala/Java project, is SBT. However, our current build system is [mill](https://mill-build.com/). For a general overview of why we don't use SBT anymore, see [here](http://www.lihaoyi.com/post/SowhatswrongwithSBT.html). 

For us specifically we considered these things:

- For meta-build alterations in ColHelper, it was annoying to remember that you need to reload.
- You needed to alter the IntelliJ settings immediately, otherwise the project does not build by default.
- The run script broke for almost every user, because we use a custom classpath caching solution, because sbt "run example.pvl" starts up way too slow (~7 seconds when vercors is already entirely compiled).
- Most people had encountered a situation where the project just breaks, and you need to re-import everything.
- Anything custom was hard to implement, because we didn't understand enough about how sbt works.

Mill on the other hand does not have these drawbacks:

- It is almost just scala.
- Tasks are a straightforward and easy abstraction:
	- Custom stuff is easy to implement, even when interacting with mill internals.
	- Caching is essentially free due to the data model.
	- Tooling around mill is easy, because the hierarchy in the build definition corresponds to the hierarchy in `out`.
	- You can chain tasks entirely as you like and mill is fine with it: e.g. `col.generatedSources` depends on `colMeta.meta.runClasspath`.
- Mill boots with the vercors build in under one second, e.g. `time ./mill -j 0 vercors.runScript` -> `real 0m0,753s`
	- This means we can finally get rid of our own classpath abstraction: we can be sure vercors runs normally every time.
- Mill seems to work well with IntelliJ via the BSP integration, which does not need any altered settings in IntelliJ.