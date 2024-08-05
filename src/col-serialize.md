# Serialization

A lightly used feature of COL is serialization and deserialization: the ability to store a COL tree as an array of bytes, and the ability to translate the bytes back into a COL tree.

## Serialization on the VerCors side

Each node defines two serialization methods:

```
def serialize(decls: Map[Declaration[G], Long]): vct.col.ast.serialize.<Node>

def serializeFamily(decls: Map[Declaration[G], Long]): vct.col.ast.serialize.<NodeFamily>
```

The `decls` parameter is used to store the representation of references in the AST. There is also a utility method `vct.col.ast.Serialize.serialize` that computes a standard assignment of declaration to `Long` for a `Program`.

For deserialization objects are generated for each node in `vct.col.ast.ops.deserialize`, but again the `vct.col.ast.Deserialize.deserialize` method is there for conveniently deserializing `Program` nodes.

### Usage of serialization

Currently the only usage of serialization internal to VerCors is to cache library files and user inputs. Library files are cached by default, which saves us from having to continually parse them on each run of VerCors. User inputs are currently only cached when a development flag is enabled.

## Serialization outside VerCors

(De)serialization takes an extra step between COL trees and bytes through Protobuf. For those familiar with protobuf, we break with two important standards that are common for protobuf definitions:

* We do not promise backward or forward compatiblility: we are still in a stage where we extensively experiment with node representation, so we are careful that communication of serialized programs only occurs between identical versions of the serialization format. We may want to have some form of backward compatibility in the future.
* The protobuf definition is not the source of truth. Instead we derive a protobuf definition from `Node.scala`. It may be sensible to explicitly maintain a version of the protobuf definition in tandem with switching to a notion of backwards compatible nodes.

On to the good parts: for any VerCors version you can find its protobuf definitions at `out/vercors/col/helpers/global/generate.dest/vct/col/ast/col.proto`. You can use this file to generate bindings for all common programming languages. This effectively makes it so that any programming language can be used to generate an input to VerCors, e.g. by extending a compiler in whatever language that is written.