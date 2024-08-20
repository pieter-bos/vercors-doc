# Utilities

This chapter describes a few utilities included with VerCors. A fair few utilities are implemented as a middleware:

## Middleware

A middleware for VerCors is a concern that is cross-cutting, needs installing and potentially cleanup after finishing, and needs to be accessible globally for convenience. Examples of middlewares in VerCors are: logging, profiling, and progress printing. All have a start and an end with installation logic and cleanup, and it would be wildly inconvenient if we had to pass around a profiling object everywhere in the tool.

The motivation for making middleware an explicit abstraction is twofold:

### LSP

Middleware will enable us to swap in different implementaitons for logging, progress, etc. when we implement VerCors as a language server. Currently middlewares are immediately implemented as a class, but we can easily fit an interface inbetween, and install different middlewares when in console mode or LSP mode.

### Crashing and all that

It is helpful that certain clean-up tasks happen irrespective of whether VerCors finishes normally, the process is interrupted via Ctrl+C, or VerCors crashes. Finishing normally and crashing could harmonize middleware cleanup via a `finally` block, but this is not the case for stopping VerCors with Ctrl+C (and other signals). The only way to have clean-up behaviour in that case is to register a shutdown hook. The middleware abstraction simply makes this uniform: uninstall tasks always run, and they run in reverse order of how they were installed.