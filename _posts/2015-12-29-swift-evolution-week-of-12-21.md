---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Dec 21, 2015"
description: "Lazy <code>flatMap</code> for sequences of optionals and tuple comparison operators passed review. Coming up next week for review: <code>associated</code> and <code>memberwise init</code>." 
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
modified: Dec 30, 2015
---

A summary of selected updates from last week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Upcoming reviews

  - We declare associated types currently using the `typealias` keyword,
    and the same keyword is used to create an alias for a type. Loïc
    Lecrenier's [proposal][associated] to use `associated` to declare
    associated types will be coming up for review next week.

  - The idea of having [flexible memberwise
    initialization][flexible_init] was selected for review last week.

    Swift currently offers an automatic [memberwise
    initializer][memberwise_swift_book] for structs (unless they
    have an explicit initializer).
    Matthew Johnson proposed a way for this feature to be made available
    for classes (under certain conditions), and also allows us to
    specify values for private variables in the memberwise initializer
    (among other stuff). The proposed syntax looks like this:

        class MyClass {
            var i: Int
            private var j: Int
            memberwise init(...) { // '...' is part of the syntax
                j = 42
            }
        }

        let m = MyClass(i: 2)

    This proposal is likely to be up for review next week.

[associated]: https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md
[flexible_init]: https://github.com/apple/swift-evolution/blob/master/proposals/0018-flexible-memberwise-initialization.md
[memberwise_swift_book]: https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID214

### Accepted proposals

  - Currently, `flatMap` works lazily only for sequences of
    non-optionals. Oisin Kidney's proposal to [make flatMap work lazily
    for sequences of optionals][lazy_flatmap] as well was **accepted**
    for inclusion in Swift 2.2.

  - The proposal for [tuple comparison operators][tuple_comparison] was
    **accepted** for inclusion in Swift 2.2, and
    [merged](https://github.com/apple/swift/pull/408) into the Swift
    codebase.

    At present, comparing tuples requires the relevant equality or
    comparison operations to be written explicitly. Kevin Ballard
    proposed that the Swift standard library should include equality
    (`==`, `!=`) and comparison (`<`, `<=`, `>`, `>=`) operators for
    2-tuples, 3-tuples, 4-tuples, 5-tuples and 6-tuples. This is
    implemented through a [gyb][] file: [Tuple.swift.gyb](https://github.com/apple/swift/blob/3576996543e122637437a17c79d10bee81323a79/stdlib/public/core/Tuple.swift.gyb).

  - The implementation for the previously accepted proposal for
    [allowing language keywords as argument labels][keyword_as_label]
    was
    [merged](https://github.com/apple/swift/commit/c8dd8d066132683aa32c2a5740b291d057937367)
    into the Swift codebase.

    To use a Swift keyword (like `in`, for example) as an argument label
    in a Swift function, we currently have to escape it in backticks
    (like ``indexOf(value, `in`: collection)`` &ndash; because `in` is a
    keyword). The backtick-escaping won't be necessary in Swift 2.2.

[keyword_as_label]: https://github.com/apple/swift-evolution/blob/master/proposals/0001-keywords-as-argument-labels.md
[lazy_flatmap]: https://github.com/apple/swift-evolution/blob/master/proposals/0008-lazy-flatmap-for-optionals.md
[tuple_comparison]: https://github.com/apple/swift-evolution/blob/master/proposals/0015-tuple-comparison-operators.md 
[gyb]: https://lists.swift.org/pipermail/swift-users/Week-of-Mon-20151207/000226.html

### Discussions

A few discussions that caught my eye last week:

 1. <a name="inout"></a>
    Replace 'inout' with '&'
    ([earlier](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151214/003483.html),
    [last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151221/003862.html))
 
    The use of `inout` in function declarations looks very similar to
    argument labels, especially now that keywords will be allowed as
    labels. For example, `func foo(inout x: Int)` looks very similar to
    `func foo(label x: Int)`. Joe Groff proposed that we could use `&`
    instead to make it easier to perceive the difference when reading
    code (one way to do that would be `func foo(&label x: Int)`). Erica
    Sadun proposed a few other syntaxes, the most popular of which seems
    to be `func foo(x: inout Int)`.

 2. <a name="final-by-default"></a>
    Final by default for classes and methods
    ([earlier](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151207/000871.html),
    [last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151221/003815.html))
 
    Swift classes are at present non-final unless marked as final.
    There's a discussion on making them final by default, or atleast
    "sealing the classes" by default across module boundaries, and
    requiring that classes and / or methods that can be overridden be
    marked explicitly in code (with something like `inheritable` /
    `overridable`).

    The motivation to have all classes final by default is to make
    inheritability of classes explicit, so that we're forced to think
    about that aspect when designing the class. The motivation for
    "sealed by default" is to enable not having to think about
    inheritability while working on one's own codebase, but forcing us
    to think about it when having to ship a framework, so that API
    contracts can be kept in future versions of the framework. There
    will also be performance benefits with both ideas because the
    compiler can optimize finalized method calls better.

    The arguments against the "final by default" idea included:

      - It's annoying to have to annotate every class / method in order
        to inherit / override it
      - It might make it harder for beginners to learn Swift
      - We can't use testing methods that relied on overriding

    The arguments against the "sealed by default" idea included:

      - We've had to monkey-patch Apple frameworks to workaround bugs or
        other behaviours, which might be affected by this change.
        (Personally, [I
        think](https://twitter.com/roopeshchander/status/679177591752757249)
        this point is valid but not relevant).

        Michael Tsai has a [good roundup of this
        argument](http://mjtsai.com/blog/2015/12/21/swift-proposal-for-default-final/)
        from the mailing list, twitter and blog posts.

 3. <a name="naming-functions"></a>
    Generalized naming for any function
    ([last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151221/004555.html))

    Swift functions can be assigned to variables, but when the same base
    name is shared by multiple functions (like
    `insertSubview(view:UIView, aboveSubview: UIView)` and
    `insertSubview(view: UIView, belowSubview: UIView)`), Swift doesn't
    have a way for us to specifically name a particular function. Doug
    Gregor has a proposal to name functions like this, and extends it to
    support setters, getters, initializers and subscripting. The syntax
    for this is still being debated.

**Update (Dec 30):** You might also want to take a look at Erica Sadun's
[roundup](http://ericasadun.com/2015/12/28/we-are-devo-swift-evolution-continues/)
of last week's Swift Evolution discussions.
