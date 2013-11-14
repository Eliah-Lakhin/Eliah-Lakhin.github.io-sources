#### Problem

Most of the PL compilers usually process the whole set of source codes at once.
And optimized for fast performance. Even though compilation time in this
approach is proportional to the overall source code size. And in practice
compilation time may grow to seconds and even minutes as the code base is
growing.

This approach is not suitable in writing language support plugins for code
editors and IDEs by two reasons:

 * The editor must always be in touch with source code's syntax and semantic.
 * Small and frequent changes end user makes to the source code should be
   indexed immediately, without any significant time delays.

Another problem is that PL compilers and appropriate IDE plugins are usually
developed separately for each code editor.

The solution for this problem might be designing compiler's front-end to work
as a compilation server in incremental fashion:

 * It should be able to provide code editor through it's API with metadata on
   source code's syntax structure, semantic references and possible actual
   errors in the code. So the external editor can utilise this metadata to
   provide end user with a set of conventional IDE tools such as code
   refactoring, jump-to-definition, variable renaming, syntax and error
   highlighting.
 * The compilation server should work continuously by caching all the metadata
   including AST and semantic references. And being able to reuse this caching
   state to resume compilation process when source code is changing. Therefore
   compilation performance will be relative not to the overall codebase size,
   but the size of the changes end user makes to the code.

In other words the idea is to move code editor's conventional responsibilities
to compiler's side. So the compiler becomes universal reusable component for
a set of code editors and IDEs. Whereas the editor works as a thin client that
depends on compilation server. The advantage of this approach is simplifying
development of PL plugins for code editors and IDEs, and support unification of
their support.

The heart of the compiler's front-end is a source code parser. And the parser of
incremental compiler should work in incremental fashion as well.

And this is an issue. There are a lot of tools to build ordinary parsers based
on predefined programming language grammar. Including generators such as
[YACC](http://www.quut.com/c/ANSI-C-grammar-y.html) and
[ANTLR](http://www.antlr.org/) or combinators such as
[Parsec](http://www.haskell.org/haskellwiki/Parsec) or
[Parboiled](https://github.com/sirthias/parboiled/wiki). But too few of them
provide a way to build incremental parsers. Incremental parsing feature
support is usually have experimental status in these projects. And hard to be
used by the almost developers.

The goal of my project is eliminating the lack of such instruments, by providing
easy to use library for constructing incremental parsers with a number of
handy features that compiler and PL support plugin developers usually need.