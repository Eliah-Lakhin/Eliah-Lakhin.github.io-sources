### Problem

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
development of PL plugins for code editors and IDEs, and unification of their
support.

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

### Papa Carlo

I am pleased to introduce you my project
[Papa Carlo](http://localhost:8000/projects/papa-carlo/). This is a Scala
library to construct recursive descent incremental parsers based on PEG grammar.
The project is published on [GitHub](https://github.com/Eliah-Lakhin/papa-carlo)
under the APL2 license.

The library provides the following features:

 * Parser's grammar defined right in the Scala code using library's API. No
   generation explicit generation steps.
 * The API represents Parsing Expression Grammar operators. PEG is probably the
   most popular and easy to understand PL grammar for a nowadays.
 * Resulting parser builds and incrementally updates Abstract Syntax Tree out of
   the box. And there is no intermediate Parsing Tree build step.
 * Efficient error recovery. Result parser can build or update AST even from
   the source code that contains syntax errors.
 * Expression grammar with infix operators can be defined with special prepared
   operators based on Pratt algorithm.

The simple example of a full-feature expression parser can be defined with Papa
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

    terminals("(", ")", "%", "+", "-", "*", "/")

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
      postfix(rule, "%", 1)
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
`1 - -2 / 3 * +(4 / 5) + 6 + 7% + 8`, build and update AST, and recovery syntax
errors.

The more interesting example is
[JSON parser](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/main/scala/name.lakhin.eliah.projects/papacarlo/examples/Json.scala),
that incrementally parses large input files with over 600 lines of source code.
In one of the included tests these files has been changed and parsed several
times:

| File | Difference (lines) |  Syntax errors | Number of AST nodes | Parse time (seconds) |
|:----:|:------------------:|:--------------:|:-------------------:|:--------------------:|
| [Version 1](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/test/resources/fixtures/json/large/input/step0.txt) | - | 0 | 918 | 0.270 |
| [Version 2](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/test/resources/fixtures/json/large/input/step1.txt) | 1 | 1 | 923 | 0.007 |
| [Version 3](https://github.com/Eliah-Lakhin/papa-carlo/blob/master/src/test/resources/fixtures/json/large/input/step2.txt) | 2 | 3 | 924 | 0.008 |

This is where incremental parsing approach advantages take place.

Also Papa Carlo includes API to log and debug various aspects of developing
parser. This technique was used to cover the parser with a number of functional
tests.