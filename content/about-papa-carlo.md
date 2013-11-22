Constructor of incremental parsers in Scala using PEG grammars.

Papa Carlo was designed to construct programming language parsers that may be
used in a wide range of code editors from simple syntax highlighters to
full-featured IDEs like Eclipse or IntelliJ Idea.

Key features:

 * **Incremental parsing**. Parsing performance is always proportional to the
   changes end users make to the source code. Even if the source code consists
   of thousands lines.
 * **Automatic AST construction**. No rule action needed to construct Abstract
   Syntax Tree that represents source code's structure. Parser builds AST out of
   the box.
 * **Error-recovery mechanism**. The parser can recognise source code even if
   the code contains syntax errors.
 * **Language definition using DSL in Scala.** No external files needed. The
   developer can define parser using the library's API in the Scala code.
 * **PEG grammars**. Modern and easy to understand language definition grammar.
 * **Expression parsing with Pratt algorithm.** Complicated expression syntax
   with a number of unary/binary precedence operators can be defined in a few
   lines of source code.

The project is [published on GitHub](https://github.com/Eliah-Lakhin/papa-carlo)
under the terms of
[Apache Public License 2](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/LICENSE).

Library's build artifacts are hosted on
[Sonatype](http://oss.sonatype.org/content/groups/public/name/lakhin/eliah/projects/papacarlo/)
and [Maven Central](http://central.maven.org/maven2/name/lakhin/eliah/projects/papacarlo/)
remote repositories. They are ready to use in the SBT based projects.
