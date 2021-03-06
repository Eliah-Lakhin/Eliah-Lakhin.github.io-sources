### Problem

Most programming language compilers usually process source code files at once.
Compilation time using this approach is proportional to the overall source code
size in best case. In practice, the time may grow to seconds, and even minutes,
as the code base grows.

This approach is not suitable for writing language support plugins for code
editors and IDEs for two reasons:

 * The editor must always be in touch with source code's syntax and semantics.
 * Small and frequent changes that end users make to the source code should be
   indexed immediately, without any significant time delays.

Another problem is that PL compilers and appropriate IDE plugins are usually
developed separately for each code editor.

The solution to these problems might be designing compiler's front-end to work
as a compilation server in an incremental fashion:

 * It should provide external code editors with an API to obtain metadata on its
   internal compilation state. Such as the source code's syntax structure,
   semantic references and errors in the code. So the editor can utilise this
   information to provide end user with a set of conventional IDE tools such as
   code refactoring, jump-to-definition, variable renaming, syntax and error
   highlighting.
 * The compilation server should work continuously by caching all the metadata
   including AST and semantic references. This information can be reused to
   resume the compilation process when the source code is changed. Therefore,
   compilation performance will be relative not to the overall codebase size,
   but to the size of the changes that the end user makes to the code.

In other words, the idea is to move the code editor's conventional
responsibilities to the compiler's side. So the compiler becomes universal
reusable component for a set of code editors and IDEs. Whereas the editor works
as a thin client that depends on compilation server. The advantage of this
approach is simplifying development of PL plugins for code editors and IDEs,
and unification of their support.

The heart of the compiler's front-end is a source code parser. And the parser of
the incremental compiler should work in incremental fashion as well.

Here is an issue: there are a lot of tools to build ordinary parsers based
on predefined programming language grammar. These include generators such as
[YACC](http://www.quut.com/c/ANSI-C-grammar-y.html) and
[ANTLR](http://www.antlr.org/) or combinators such as
[Parsec](http://www.haskell.org/haskellwiki/Parsec) or
[Parboiled](https://github.com/sirthias/parboiled/wiki). But too few of them
provide a way to build incremental parsers. Incremental parsing feature
support usually has experimental status in these projects. They are interesting
from research and academic point of view, but usually are not ready for use in
real products.

The goal of my project is to eliminate the lack of such instruments, by
providing an easy to use library for constructing incremental parsers with a
number of handy features that compiler and PL support plugin developers usually
need.

### Papa Carlo

I am pleased to introduce you to my project,
[Papa Carlo](/projects/papa-carlo/). This is a Scala
library to construct recursive descent incremental parsers based on PEG grammar.
The project is published on [GitHub](https://github.com/Eliah-Lakhin/papa-carlo)
under the APL2 license.

The library provides the following features:

 * The parser's grammar is defined in the Scala code using the library's API. No
   explicit generation steps.
 * The API represents Parsing Expression Grammar operators. PEG is probably the
   most popular and easy to understand PL grammar nowadays.
 * Resulting parser builds and incrementally updates Abstract Syntax Tree out of
   the box. And there is no intermediate Parsing Tree build step.
 * Efficient error recovery. Resulting parser can build or update AST even from
   the source code that contains syntax errors.
 * Expression grammar with unary and binary precedence operators can be easily
   defined based on the Pratt algorithm using dedicated functions like
   ``infix()`` or ``group()``.

The simple example of a full-featured expression parser can be defined with Papa
Carlo in one short
[Scala file](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/main/scala/name.lakhin.eliah.projects/papacarlo/examples/Calculator.scala):

```scala
object Calculator {
  private def tokenizer = {
    val tokenizer = new Tokenizer()

    import tokenizer._
    import Matcher._

    tokenCategory("whitespace", oneOrMore(anyOf(" \t\f\n"))).skip

    tokenCategory("number", choice(chunk("0"), sequence(rangeOf('1', '9'),
      zeroOrMore(rangeOf('0', '9')))))

    terminals("(", ")", "!", "+", "-", "*", "/")

    tokenizer
  }

  def lexer = new Lexer(tokenizer, new Contextualizer)

  def syntax(lexer: Lexer) = new {
    val syntax = new Syntax(lexer)

    import syntax._
    import Rule._
    import Expressions._

    mainRule("expression") {
      val rule =
        expression(branch("operand", recover(number, "operand required")))

      group(rule, "(", ")")
      postfix(rule, "!", 1)
      prefix(rule, "+", 2)
      prefix(rule, "-", 2)
      infix(rule, "*", 3)
      infix(rule, "/", 3, rightAssociativity = true)
      infix(rule, "+", 4)
      infix(rule, "-", 4)

      rule
    }

    val number = rule("number") {capture("value", token("number"))}
  }.syntax
}

```

The parser will be able to parse precedence expressions like this
`1 - -2 / 3 * +(4 / 5) + 6 + 7! + 8`, build and update AST, and recover syntax
errors.

A more interesting example is a
[JSON parser](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/main/scala/name.lakhin.eliah.projects/papacarlo/examples/Json.scala),
which incrementally parses large input files with over 600 lines of source code.
In one of the included tests, these files have been changed and parsed several
times:

| File | Difference (lines) |  Syntax errors | Number of AST nodes | Parse time (seconds) |
|:----:|:------------------:|:--------------:|:-------------------:|:--------------------:|
| [Version 1](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/test/resources/fixtures/json/large/input/step0.txt) | - | 0 | 918 | 0.270 |
| [Version 2](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/test/resources/fixtures/json/large/input/step1.txt) | 1 | 1 | 923 | 0.007 |
| [Version 3](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/test/resources/fixtures/json/large/input/step2.txt) | 2 | 3 | 924 | 0.008 |

This is where the incremental parsing approach shows its advantage.

Also Papa Carlo includes an API to log and debug various aspects while
developing a parser. This technique was used to cover the library with a number
of functional tests.
