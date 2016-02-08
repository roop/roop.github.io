---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Feb 1, 2016"
description: "Review of Swift API guidelines ends. <code>#line</code>
accepted. Native regexes discussed."
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
---

Here's a summary of selected updates from last
week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Review discussions

  - **API Design guidelines**

    _Previously:_ Weeks of
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#api-design-guidelines),
    [Jan 18](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-18/#api-design-guidelines),
    [Jan 25](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-25/#api-design-guidelines).


    In Swift 3, the look of the language is getting a redesign, which is
    governed by three interlinked proposals:

      - [API Design Guidelines](https://github.com/apple/swift-evolution/blob/master/proposals/0023-api-guidelines.md)

        Based on the discussions, Dave Abrahams [started
        afresh](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/008838.html)
        on reasoning about using function argument labels
        (`add(thingie:)`) vs. merging the label with the function base
        name (`addThingie()`) vs. dropping the label altogether
        (`add()`), and
        [revised](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/009206.html)
        it again.

        Douglas Gregor made an importer with the revised scheme, and the
        results look [like this](https://github.com/apple/swift-3-api-guidelines-review/pull/10).

      - [Objective-C name translation](https://github.com/apple/swift-evolution/blob/master/proposals/0005-objective-c-name-translation.md)

        Drew Crawford, Nate Cook and others strongly disapproved of dropping the
        `NS` prefix for Foundation classes.

        [Nate Cook](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/008675.html):

        > The Swift standard library defines Array, Set, and Dictionary
        > as collection types with value semantics that operate quite
        > differently from their peers in Foundation. This change will
        > introduce OrderedSet, CountedSet, HashTable, MapTable, and
        > others, all of which use reference semantics and therefore
        > don't provide the same set of guarantees about ownership and
        > immutability.

        > As an example, the seemingly similar Set and CountedSet types
        > produce different results from nearly identical code.

        [Dave Abrahams](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/008760.html):

        > I think the juxtaposition of Array, String, Dictionary, and
        > Set (mutable, with value semantics) and MutableArray,
        > MutableString, MutableDictionary, and MutableSet (mutable,
        > with reference semantics) is one of the most obvious problems
        > with this part of the change.

        Erica Sadun [wrote about this proposal](http://ericasadun.com/2016/02/02/se-0005-the-one-swift-evolution-proposal-youll-want-to-know-about/) in general in her blog:

        > I’m not convinced that developers will welcome widespread API
        > adjustments that may incur costs in code review, error
        > detection, maintenance as well as the production of new code.

      - [Swift stdlib API changes](https://github.com/apple/swift-evolution/blob/master/proposals/0006-apply-api-guidelines-to-the-standard-library.md)

        Dave Abrahams [explained](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/008793.html) the reasons for proposing that "Generator"s be changed to "Iterators":

         1. Iterator is the more well known concept
         2. The method name “generate()” was obviously wrong ... Once
            you start thinking about other names for that, it leads
            naturally to considering other names for the things it
            returns.

### Accepted proposals

  - [Modernizing \_\_LINE\_\_, etc.](https://github.com/apple/swift-evolution/blob/master/proposals/0028-modernizing-debug-identifiers.md)

    _Previously_: Week of
    [Jan 25](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-25/#modern-debug-identifiers).

    This proposal was [accepted](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/008982.html), but this means that `#line` means different things based on its position in a line.


    If the `#line` is in the middle of a line, it would be a debug identifier
    for the current line, like in:

    ```
    print("At line \(#line) of file \(#file)")
    ```

    If the `#line` is at the start of a line, it's a [line control
    statement](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/zzSummaryOfTheGrammar.html#//apple_ref/doc/uid/TP40014097-CH38-NoLink_925)
    that tells the compiler to use that line number and filename for
    error messages (useful for in Swift code that is
    automatically-generated based on some other directive file).

    ```
    #line 10 "foo.grammar"
    ```

    Chris Lattner says:

    > The core team isn’t thrilled with the magic “first token on a
    > line” whitespace behavior that #line will be getting, and would
    > like someone to start a discussion about renaming the old #line
    > directive to something more specific and tailored to its purpose.

    The alternatives being explored for the line control statement
    include renaming it to `#setline` and `#sourceline`, or using a
    different syntax altogether, like `#reset line=50,
    file="foo.swift"`.

### Up for review

  - [Remove passing function arguments as tuples](https://github.com/apple/swift-evolution/blob/master/proposals/0029-remove-implicit-tuple-splat.md)

    Implicit tuple-splatting for function calls is a little-known and
    hardly-ever-used feature in Swift, and Chris Lattner proposes that it
    be removed.

### Discussions

  - <a name="regex"></a>
    **Regular expressions**

    Patrick Gili
    [made](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/008544.html)
    a draft proposal to have regex literals in Swift, by introducing a
    Perl-like syntax for creating `NSRegularExpression` instances.

    But it turned out that Chris Lattner's plans for regexes in Swift
    are different and much more ambitious. He
    [said](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/008593.html):

    > Instead of introducing regex literals, I’d suggest
    > that you investigate introducing regex’s to the pattern grammar,
    > which is what Swift uses for matching already.  Regex’s should be
    > usable in the cases of a switch, for example.  Similarly, they
    > should be able to bind variables directly to subpatterns.
    >
    > Further, I highly recommend checking out Perl 6’s regular
    > expressions.  They are a community that has had an obsessive
    > passion for regular expressions, and in Perl 6 they were given the
    > chance to reinvent the wheel based on what they learned.  What
    > they came up with is very powerful, and pretty good all around.

    And then, [in another mail](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/008866.html):

    > I think it would make a lot of sense for primitive types to
    > support “default” regex rules (e.g. integers would default to
    > /[0-9]+/ ) and then have modifier characters that support other
    > standard modes for them (e.g. x for hexadecimal).  This would
    > obviously want to be extensible to arbitrary types, so that (e.g.)
    > NSDate could support the format families that make sense.

### Other stuff

  - There's now an app to follow the Swift-related mailing lists:
    [Hirundo](https://stylemac.com/hirundo/)
    
    I find it very useful, and if you'd like to follow the discussions
    in more detail, you should definitely give this app a try.

### Other weeklies

  - Jesse Squires' [Swift Weekly Brief Issue #8](http://swiftweekly.github.io/issue-8/)
    talks about open-source Swift happenings last week
