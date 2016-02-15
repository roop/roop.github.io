---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Feb 8, 2016"
description: "Review of property behaviors starts. <code>inout</code> placement, <code>??=</code> operator and creating strings from code units reviewed. RFC on Swift binary compat."
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
---

Here's a summary of selected updates from last
week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Review discussions

  - <a name="behaviors"></a>
    [Property behaviors](https://github.com/apple/swift-evolution/blob/master/proposals/0030-property-behavior-decls.md)

    _Previously:_ Week of
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#behaviors).

    Currently, `lazy` is a feature that's baked into Swift compiler.
    Joe Groff is trying to generalize it out of the compiler, thereby
    enabling us to write our own `lazy` or create other `lazy`-like
    annotations on properties.

    Per the proposal, you can say `var [lazy] foo = 42`, and specify the
    implementation of how lazy properties should behave as code.
    Specifically for `lazy`, I would expect this implementation to be in
    the Swift Standard Library. We can also implement our own behaviors
    like `mycustomlazy` or `cached`.

    This is a huge change to how Swift properties work, and so the
    review period extends to the whole of February.

  - <a name="optional-value-setter"></a>
    [Optional Value Setter](https://github.com/apple/swift-evolution/blob/master/proposals/0024-optional-value-setter.md)

    James Campbell proposes to add a new operator `??=` for assigning a
    value to an optional variable only if the variable is nil.

        var i: Int = nil
        i ??= 42 // Assigns the value 42 because it was nil earlier

    This is the same as doing `i = i ?? 42`, but uses the variable just
    once, making it more typable, less error prone and easier to
    read (especially when it's a long variable with sub-paths).

  - <a name="string-from-code-units"></a>
    [String from Code Units](https://github.com/apple/swift-evolution/blob/master/proposals/0027-string-from-code-units.md)

    _Previously:_ Week of
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#string-from-code-units).

    Currently, creating a Uncode `String` from a sequence of bytes works
    cleanly for just one case: null-terminated UTF-8 bytes.

    Zachary Waldowski proposes to create `String` instances from an
    arbitrary sequence of code units with an arbitrary encoding.

    This addresses a gap in the current string API and is very likely to
    be accepted. There's a working implementation
    [in the PR](https://github.com/apple/swift/pull/1109).

  - <a name="inout-decoration"></a>
    [Adjusting inout Declarations for Type Decoration](https://github.com/apple/swift-evolution/blob/master/proposals/0031-adjusting-inout-declarations.md)

    _Previously:_ Week of
    [Dec 21](/last-week-in-swift-evolution/2015/swift-evolution-week-of-12-21/#inout).

    Joe Groff and Erica Sadun propose that the inout decoration in
    function declarations be moved from the label side to the type side:

        // Current syntax
        func foo(inout x: T)

        // Proposed syntax
        func foo(x: inout T)

    This has been discussed in depth before, and this syntax was the
    most favourite, so this proposal too is likely to be accepted.

### Accepted proposals

  - <a name="remove-implicit-tuple-splat"></a>
    [Remove passing function arguments as tuples](https://github.com/apple/swift-evolution/blob/master/proposals/0029-remove-implicit-tuple-splat.md)

    _Previously_: Week of
    [Feb 1](/last-week-in-swift-evolution/2016/swift-evolution-week-of-02-01/#remove-implicit-tuple-splat).

    I had thought this was an odd and hardly used feature, but Brent
    Royal-Gordon uses this feature "reasonably often". He 
    [said](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160201/009340.html):

    > Tuple splat allows you to write generic functions that work with a
    > function of any arity. ... Even something so simple as a function
    > composition operator is impossible to write without it.

    Nevertheless, since this proposal is accepted, tuple splatting is
    going away in Swift 3.0 without an immediate replacement. Joe Groff
    [writes](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009582.html):

    > ... maintaining this behavior in the type checker is a severe
    > source of implementation complexity, and actively interferes with
    > our plans to solidify the type system. We feel that removing the
    > existing behavior is a necessary step toward stabilizing the
    > language and toward building a well-designed alternative feature
    > for explicit argument forwarding in the future. 

    Consequently, Brent has started a discussion on alternative ways to
    pass tupled arguments to functions. The alternatives
    [he came up with](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009596.html)
    are:

     1. A special `parameters` label: `concatenate(parameters: tuple)`
     2. An `apply` method on functions: `concatenate(_:to:).apply(to: tuple)`
     3. A splat operator with `*` or `!`: `concatenate(_:to: *tuple)`

### Discussions

  - <a name="lib-binary-compat"></a>
    **Binary compatibility of Swift libraries**

    _Previously:_ Week of
    [Jan 4](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-04/#lib-binary-compat) (See: Other).

    Jordan Rose is [inviting
    comments](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160208/009451.html)
    on the Swift [Library
    Evolution document](http://jrose-apple.github.io/swift-library-evolution/):

    > Our current design in Swift is to provide opt-out load-time
    > abstraction of implementation for all language features.

    This implies that all language features will be part of the ABI,
    and will be available for use across module boundaries.

    > [We will design] the language and its implementation to minimize
    > unnecessary and unintended abstraction:
    > 
    >  1. Avoiding unnecessary language guarantees and taking advantage
    >     of that flexibility to limit load-time costs.
    >  2. Within the domain that defines an entity, all the details of
    >     its implementation are available.
    >  3. When entities are not exposed outside their defining module,
    >     their implementation is not constrained.
    >  4. By default, entities are not exposed outside their defining
    >     modules. This is independently desirable to reduce accidental
    >     API surface area, but happens to also interact well with the
    >     performance design.

    The earlier discussion on making classes final by default for
    libraries (previously: Week of [Dec 21](/last-week-in-swift-evolution/2015/swift-evolution-week-of-12-21/#final-by-default))
    is a reflection of the first point above.

    Erica Sadun wrote about this discussion [in her blog](http://ericasadun.com/2016/02/08/swift-resilience-conversation-commences/).

### Other weeklies

  - Jesse Squires' [Swift Weekly Brief Issue #9](http://swiftweekly.github.io/issue-9/)
    talks about open-source Swift happenings last week.

    In particular, Jesse talks about the discussion on garbage
    collection in the mailing lists that I haven't followed at all.

