Constructor of incremental parsers in Scala using PEG grammars.

Papa Carlo was designed to construct programming language parsers that may be
used in a wide range of code editors from simple syntax highlighters to
full-feature IDEs like Eclipse or IntelliJ Idea.

Key features:

 * **Incremental parsing**. Parsing performance is always proportional to the
   changes end user makes to the source code. Even if the source code consists
   of thousands lines.
 * **Automatic AST construction**. No rule action needed to construct Abstract
   Syntax Tree that represents source code's structure. Parser builds AST out of
   the box.
 * **Error-recovery mechanism**. The parser can recognise source code even if
   the code contains syntactical errors.
 * **Language definition using Scala DSL.** No external files needed. Developer
   defines parser using Papa Carlo's API right in the Scala code.
 * **PEG grammars**. Modern and easy to understand language definition grammar.
 * **Expression parsing with Pratt algorithm.** Complicated expression syntax
   with a number of infix precedence operators can be defined in a few lines
   of Scala code.

The project is [published on GitHub](https://github.com/Eliah-Lakhin/papa-carlo)
under the terms of
[Apache Public License 2](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/LICENSE).