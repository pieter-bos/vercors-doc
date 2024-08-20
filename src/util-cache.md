# Caching

The `hre.cache.Cache` utility allows you to create an on-disk cache directory that is determined by a specified list of keys. There is also a facility to tie the cache directory only to the process that the cache is being requested in, essentially reducing the cache directory to a temporary directory.


## Usages

There are a few usages of caching:

* In `vct.cache.Caches` a directory is made to cache verification results from carbon and silicon. This behaviour is guarded behind the experimental option `--dev-cache`. An entry is stored in the cache if a program has no verification failures, in which case the program is serialized to disk.
* In `vct.cache.Caches` there is a directory that caches the result of parsing, resolving and normalizing a library file writting in PVL. This is used for e.g. simplification rules and datatype definitions. The result is serialized to disk.
