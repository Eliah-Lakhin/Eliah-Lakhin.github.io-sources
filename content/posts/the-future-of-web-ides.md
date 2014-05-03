[![Click to open Demo web application](http://i.imgur.com/XGSswoh.png?1)](/projects/papa-carlo/demo/)

In this post I would like to explain why having an incremental compiler is
important for the browser based IDEs and text editors.

To understand what an incremental compiler is, please take a look at the
[Demo web application](/projects/papa-carlo/demo/) I have published a few days
ago.

### Web IDE

These days popularity of text editors made completely on top of web
technologies is growing fast. Probably, you have heard about such products as
Cloud9, Koding or even Atom.io from GitHub released recently. Tens if not
hundreds of new instances entering the stage.

These solutions are especially useful, because they provide user with the ready
to use remote development environment, and there are no special steps in
deployment. User just opening web-browser and voila, he/she has a ready to use
workstation. At first glance, it seems like a good solution for web developers
who are working with dynamic typing languages such as JavaScript, Ruby, PHP,
Python, etc.

However, we should admit they don't shine so much when we are talking about
static typing languages like Java, C# or Objective-C. Especially, when we are
comparing web editors with the major well known solutions for desktops such as
Visual Studio, IntelliJ Idea or Eclipse. The key distinguishing feature of
desktop based "old-school" editors "old-school" editors is the ability of deep
syntax and semantic understanding of the programming languages. Finally, they
provide end users with a number of handy and smart code manipulation
instruments, including Code Completion, Jump To Definition,
Refactoring/Renaming. Having these features is very important when we are
working with the large codebase.

Internally, these editors accomplish their abilities by continuous indexing
source codes of the entire project files. They perform indexing using so called
["incremental parsers/compilers"](http://en.wikipedia.org/wiki/Incremental_compiler#Definition).
Incremental parsers are responsible for continuous scanning the code for
methods, functions and variable definitions, and their application sites in the
project file's.

An incremental parser is differ from ordinary parser by a number of properties:

1. **Continuous indexing the code in real time**. An ordinary compiler scans
entire source code once, and then it stops. If the end user changes something in
the code, the compiler should be run over and over. This approach is
inappropriate for IDEs, since complete compilation of the project may takes
seconds if not minutes.

2. **Syntax error tolerance**. When the developer writing a code, the program is
frequently stands in the syntactic incorrect states. However, IDE should be able
to recognize the rest of the code. For example, to be able to provide the user
with a Code-Completion feature in any place of the code.

3. **Mapping between the structure of the Parse Tree and the code**. All of the
nodes in the Parse Tree should keep track of the code segments they were
produced of. This property provides with a Jump-To-Definition feature.

Unfortunately, there is a lack of development tools for the incremental
compilers nowadays. Ordinary tools such as parser generators don't fit well. So,
developers usually have to implement these core parsers for the IDEs more or
less manually.

That's why I have developed my own parsing library -
[Papa Carlo](/projects/papa-carlo/) - designed especially for development of
incremental compilers. The API of the library should be familiar for
developers who have worked with ordinary parser combinators and generators.
But the resulting parser has all of the features mentioned above out of the box.

The Demo example I referred to in the beginning of the post is written on top of
the Papa Carlo library. It doesn't use server-side to process computations,
the parser works completely in the web browser. I believe this approach may
be scaled up to develop language support plugins for the modern Web Editors to
turn them into full-featured IDEs comparable to the desktop based IDEs.

### A few words about the Demo

The library Papa Carlo is written in Scala and compiled to JavaScript using
the ScalaJS compiler. So all of the computations performing completely on the
client-side. The server hosted only the static content: JS and HTML sources.

The parser is running in the WebWorker (if possible). Since that, application
loading is distributed between two threads: one of them is responsible for UI
management, and another one performs compilation.

All of the visual features are done with SVG using the d3.js library. The code
editor on the left side of the screen is a Code Mirror widget.

---

If you like this project and the Demo, please, give it a star on GitHub:
[papa-carlo@Eliah-Lakgin](https://github.com/Eliah-Lakhin/papa-carlo).

