---
layout: post
title: "Introducing Citron"
description: "An introduction to Citron, an LALR parser generator for Swift"
tagline: "An LALR parser generator for Swift"
category: posts

---

I'd like to introduce you to [Citron], an LALR parser generator for Swift
that I've been working on. For a given input grammar, Citron creates a
reentrant type-safe parser in Swift.

Citron is based on the [Lemon] parser generator by [Richard Hipp] that's
used in SQLite. The parser generation engine of Citron is written in C,
as is the case with Lemon, but the output parser code is all Swift.

[Citron]: http://roopc.net/citron/

### What's different

Citron uses the same grammar specification syntax used by Lemon, which
is a little different from what Bison and Yacc use. In addition, Citron
is stricter with code blocks and types than Bison, Yacc or Lemon.

In Citron:

  - All terminals and non-terminals in the grammar must have explicitly
    defined types

  - Every grammar rule should be accompanied by an associated code block
    that will be invoked when the rule gets used during parsing

  - The code block associated with a rule should take in values of the
    correct type as input, and should return a value of the correct
    type

### Writing a rule

Let's say you have a grammar rule for Swift's `let` statement that can
be expressed in [BNF] like this:

[Lemon]: https://www.hwaci.com/sw/lemon/lemon.html
[Richard Hipp]: http://www.hwaci.com/drh/
[BNF]: https://en.wikipedia.org/wiki/Backus–Naur_form

~~~ Text
<let-decl> ::= "let" <id> ":" <type-decl>
~~~

An equivalent rule in a Citron grammar file could look like this:

~~~ Text
let_decl ::= LET ID COLON type_decl.
~~~

In Citron, terminals (a.k.a. tokens) start with an uppercase letter (like
`LET`, `ID` and `COLON`), and non-terminals start with a lowecase letter (like
`let_decl` and `type_decl`). "let", ":" and the identifier are expected to be
recognized at tokenization stage and converted to tokens `LET`,
`COLON` and `ID` respectively, so the parser only sees these tokens.

If one were to create a parse tree from the input, there would be
semantic types representing the terminals and non-terminals present in
the grammar. A `let` node of the parse tree could be represented with
something like:

~~~ Swift
class LetDeclaration {
    var id: String
    var type: TypeDeclaration
    init(id: String, type: TypeDeclaration) {
        self.id = id
        self.type = type
    }
}
~~~

If that's the case, you should tell Citron about these backing types.
It's a good practice to declare the type of the terminals before the
rules, and the declare the types of the non-terminals close to where the
they occur in a rule, like this:

~~~ Text
%token_type String // type representing LET, ID and COLON

%nonterminal_type let_decl LetDeclaration
%nonterminal_type type_decl TypeDeclaration

let_decl ::= LET ID COLON type_decl.
~~~

Citron expects a code block immediately after a rule. The code block
should form the body of a function that returns a value representing the
LHS symbol of the rule. The LHS symbol of this rule is `let_decl`, so
our code block should return something of type `LetDeclaration`.

We can add alias names to the RHS symbols as shown below, and
they will be made available as named function parameters within the code
block with the appropriate type.

~~~ Text
%token_type String
%nonterminal_type let_decl LetDeclaration
%nonterminal_type type_decl TypeDeclaration

let_decl ::= LET ID(n) COLON type_decl(t). {
    // 'n' of type String and 't' of type TypeDeclaration
    // are available for use in this code block
    return LetDeclaration(id: n, type: t)
}
~~~

Now we have a valid Citron rule -- the rule is immediately followed by a
code block, and the code block takes in and returns values of correct
types.

You can find an example grammar file [here][eg_grammar], which has a set
of rules that describe simple arithmetic expressions.

[eg_grammar]: https://github.com/roop/citron/blob/master/examples/calc/ArithmeticExpressionParser.y

In the Citron-generated code, the code block is used as the body of a
function. The type signature of the function is based on the symbol on
the LHS and the aliased symbols on the RHS. For the above code block,
the Citron-generated function would look something like this:

~~~ Swift
func codeBlockForRule1(n: String, t: TypeDeclaration) -> LetDeclaration {
    // 'n' of type String and 't' of type TypeDeclaration
    // are available for use in this code block
    return LetDeclaration(name: n, type: t)
}
~~~

Unless the aliased symbols' types and the return type used in the code
block are consistent with the rule, the generated parser code will not
compile. This type safety helps catch code block errors at build time
rather than at run time.

### The parser interface

The Citron-generated code contains a parser class, named `Parser` by
default, with two parsing methods:

 1. `consume(token:code:)`

    This methods makes the parser consume one token. This should be
    called multiple times to pass a sequence of tokens to the parser.
    Typically, a separate tokenization stage would generate this
    sequence of tokens. As the tokens get generated, they can be passed
    to the parser through this method.

    The first argument, `token:`, is the semantic value of the token, as
    seen by the code blocks in the grammar. The type of this argument
    is the `%token_type` type specified in the grammar.

    The second argument, `code:`, is the token code that Citron knows
    this token by. The Citron-generated class contains an enum of all
    terminals in the grammar, and this argument should be a value of
    that enum.

    For example, the "n" in "let n: Int" would be passed to the
    parser like this: `parser.consume(token: "n", code: .ID)`.

 2. `endParsing()`

    This method tells the parser that there are no more tokens to consume, and
    signifies the end of input.

    This method takes no arguments, and returns the parse result. The
    parse result is the value returned by the code block of the start
    rule (a.k.a. root rule) of the grammar. If we're building a parse
    tree using the code blocks as illustrated above, the parse result
    would typically be the parse tree.

    The return type of this method is the `%nonterminal_type` of the
    start symbol (a.k.a. root symbol) of the grammar.

The string "let n: Int" could get tokenized into four tokens and passed
to Citron as:

~~~ Swift
let parser = Parser()
do {
    ...
    try parser.consume(token: "let", code: .LET)
    try parser.consume(token: "n", code: .ID)
    try parser.consume(token: ":", code: .COLON)
    try parser.consume(token: "Int", code: .BUILTIN_TYPE)
    ...
    let parseTree = try parser.endParsing()
} catch {
    // Handle errors
}
~~~

### Error handling

Both `consume(token:code:)` and `endParsing()` are throwing methods, so
they should be invoked with a preceding `try`. They throw when an input
token at a certain position is inconsistent with the grammar, or when the
input ends prematurely.

Moreover, we can throw errors from within a rule's code block and those
throws would propagate up to one of the two parsing methods.

### The lexer interface

Citron offers a simple lexer that can tokenize an input string.

We give the lexer a series of rules with either string or regular
expression patterns. The token data that the lexer should output can be
of an arbitrary type, so the lexer is defined as a generic type, with
the token data as a type parameter. For each string pattern, we give it
the token data that should be output, and for each regular expression
pattern, we give it a closure that returns the token data.

For example, you could say:

~~~ Swift

// A tuple with all the data we need for a token
typealias TokenData = (String, Parser.CitronTokenCode)

// A lexer that can output tokens as the tuple we just defined
typealias Lexer = CitronLexer<TokenData>

// Create the lexer with tokenization rules
let lexer = Lexer(rules: [

        // Keywords

        .string("let", ("", .LET)),
        .string("var", ("", .VAR)),
        .string("class", ("", .CLASS)),
        ...

        // Punctuation

        .string("{", ("", .OPEN_FLOWER_BRACKET)),
        .string("}", ("", .CLOSE_FLOWER_BRACKET)),
        .string(":", ("", .COLON)),
        ...

        // Built-in types

        .string("Int", ("", code: .INT)),
        .string("String", ("", code: .STRING)),
        ...

        // Identifiers

        .regexPattern("[A-Za-z][0-9A-Za-z]*", { str in (str, .ID) }),

        // Whitespace

        .regexPattern("\\s", { _ in nil }) // Ignored
    ])
~~~

We can ask the lexer to tokenize a string and give it a closure to run
on each token it identifies. The token data gets passed into the closure
as a parameter. Inside the closure, we can pass on the token data to the
parser.

~~~ Swift
do {
    try lexer.tokenize(inputString) { (t, c) in
        try parser.consume(token: t, code: c)
    }
    let parseTree = try parser.endParsing()
    print("\(parseTree)")
} catch {
    print("Error during parsing")
}
~~~

### Try it

Citron is [on Github][Citron], along with a few [examples].

I'd be happy to answer questions or take feedback [on Twitter] or [by
email].

[examples]: https://github.com/roop/citron/tree/master/examples/
[on Twitter]: https://twitter.com/roopeshchander
[by email]: mailto:roop@roopc.net

