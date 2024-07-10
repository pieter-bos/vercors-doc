# Overview

Were you linked here, but not sure where to start? Maybe try a quick reference guide:

* [Adding a new input language to VerCors](start-language.md)
* [Add a new type of specification to VerCors](start-spec.md)
* [Making VerCors do something other than verifying files](start-mode.md)

## Introduction

VerCors is a tool that tries to show programs to be correct. Correct may mean that the program finishes, that it does not crash, that it never throws an exception, or returns the correct result subject to some specification.

VerCors accepts programs in a variety of languages. For each language (also: _frontend_) the grammar is altered slightly, maintaining compatiblity with the source langauge grammar, so that we may add various annotations to the program. These annotations aid in verifying the program by specifying behaviour about it. This is crucial in automating the proof of the program.

As a tool VerCors works by passing the program through a large number of transformations. In general each transformation reduces the number of _features_ in the program. You may think about features as different kinds of statements and operators. Various concerns are maintained in each element of the transformation. The most important guarded invariant is that of error transformations. As statement \\(P\\) is compiled into a different statement \\(Q\\), we have to remember how to translate errors about statement \\(Q\\) back into errors about statement \\(P\\).

An intermediate program is stored in VerCors as an _Abstract Syntax Tree_ (short: AST). The internal language is also referred to as the _Common Object Language_ (short: COL). Each part of the transformation is structured as a _Rewriter_, which visits each element in the AST. After visiting a node in the tree, the rewriter must specify what that node is replaced with. By default, the rewriter will recurse further into the tree, keeping the kind of node the same. For example, if we ask a default rewriter what \\(p + q\\) must be rewritten to, we will then ask what \\(p\\) and \\(q\\) will rewrite to. Supposing they rewrite to \\(p'\\) and \\(q'\\) respectively, the rewriter will by default restore the plus operator and return \\(p' + q'\\).

Around the transformation chain there are several steps. Unfortunately, syntax trees do not fall out of thin air, so we must first parse our text input into a tree. In VerCors, this is largely handled by by the ANTLR parser generator. We translate parse trees into COL trees as a part of the parsing stage, but there are no surprises there: as a matter of principle there is a corresponding COL node for each parse node. Next there is resolution: the process of associating each name in the tree with a referrable node. For example, variable names 'point' to the declaration of the variable. This stage usually presupposes that the file compiles at all.

After all the transformations we have a very simple program. On the other hand, proving it to be correct is no simple matter. We delegate this responsibility to Viper, a tool designed to automatically verify this dialect of program. Finally, a (hopefully empty) list of errors from Viper is translated into appropriate errors for the initial input to VerCors.

This completes the intended cycle of usage for VerCors: submitting a program to VerCors leads to a list of suggested improvements, which leads to editing the program, which leads to a new submission to VerCors. We have some qualitative wishes here: errors should be clear, specific and not a false negative. It should include context where appropriate, such as an example of how the error might occur. Ideally, VerCors is performant and can quickly establish whether or not there is an error.

The central goal is that we can encourage writers of critical software to prove their programs to be correct, so that their software is reliable.