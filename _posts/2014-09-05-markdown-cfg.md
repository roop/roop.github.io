---
layout: post
title: "Why isn't there a formal grammar for Markdown?"
description: "On the impossibility of writing a CFG for Markdown"
category: posts
readthisnext: /posts/2014/eval-stmd
---

The idea of writing a [context-free grammar][CFG] (or CFG) for
[Markdown] keeps coming up. The assumption is that once it's formalized
with a grammar in [BNF] or [EBNF], it serves as a formal specification
of the syntax, and it also becomes a source of parsers, possibly using
parser generators.

That idea doesn't get far once you really take a look at the Markdown
syntax, which is not like the syntax of a programming language at all.
The Markdown syntax is inherently ambiguous. (Yes, [the syntax of C++ is
ambiguous too][c++-ambiguity], but the points of ambiguity are known and
limited.)

To illustrate, consider the case of emphasis. `*a*` is em and `**a**` is
strong. If we wrote a (very simplified) CFG in an EBNF-like format for
Markdown text runs (i.e. contents of Markdown paragraphs), it would look
like this:

~~~
text-run    = span text-run | span
span        = strong | em | normal-text
strong      = "**" text-run "**"
em          = "*" text-run "*"
normal-text = [a-zA-Z0-9 ]
~~~

This already has ambiguities in that `**a**` can be interpreted by this
grammar in two different ways: Just a strong run, or an em run inside
another em run.

But it gets worse. The grammar we saw above would reject the input `*a`,
which in Markdown should be interpreted as normal text. Same case for
`a*`, `**a` and `a**`. If we wanted to handle these too, we need these
expansions as well:

~~~
normal-text = "*" text-run | "**" text-run |
              text-run "*" | text-run "**"
~~~

And with that addition, the amount of ambiguity shoots up significantly
and unmanageably.

If we take a closer look at why this happens, we can see that when a `*`
token is encountered, the grammar cannot decide whether it should be
part of an em-qualifier or normal text (or sometimes, a
strong-qualifier). That decision can be made only after scanning the
rest of the input till we find a matching closing `*`. If there is no
matching closing `*`, we will know that only if we scan until the end of
the text-run's string (which would be the end of the paragraph). So, for
the parser to know which rule to choose at a point, it might have to
look ahead an arbitrary number of tokens.

If Markdown was a format in which `*a` without a closing `*` is a syntax
error, this wouldn't be a problem to the parser - in that case, a single
`*` can always be interpreted as part of an em-qualifier. But that's not
how Markdown is, and if we change that, it wouldn't be a very useful
writing format.

So, that's the reason why we won't see a practical unambiguous CFG for
Markdown. [The best we can do is just ".\*" ][trevorjim].

I've seen some suggestions that the indenting-based syntax for some
constructs in Markdown is what makes it hard to write a CFG for it. I
don't think that's the case because indentation can be detected at
tokenization stage and the grammar can refer them as INDENT and DEINDENT
tokens. This, per my understanding, is how Python CFGs handle
indentation.

[Markdown]: http://daringfireball.net/projects/markdown/
[CFG]: http://en.wikipedia.org/wiki/Context-free_grammar
[BNF]: http://en.wikipedia.org/wiki/Backus–Naur_Form
[EBNF]: http://en.wikipedia.org/wiki/Extended_Backus–Naur_Form
[c++-ambiguity]: http://en.wikipedia.org/wiki/Most_vexing_parse
[trevorjim]: http://trevorjim.com/a-specification-for-markdown/
