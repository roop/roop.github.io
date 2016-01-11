---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Jan 4, 2016"
description: "<code>associatedtype</code> was accepted. <code>memberwise init</code> and package testing were hotly debated."
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
---

Here's a summary of selected updates from last
week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Accepted proposals

  - [New keyword for associate type declaration][associated]

    We declare associated types currently using the `typealias` keyword,
    and the same keyword is used to create an alias for a type. Loïc
    Lecrenier's proposal to use a new keyword `associatedtype` to
    declare associated types was reviewed last week and
    accepted for Swift 2.2 ([as expected](/last-week-in-swift-evolution/2016/swift-evolution-week-of-12-28#associatedtype)).

    Chris Lattner [shared](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160104/006071.html) the thought process behind
    choosing this particular name for the keyword.

### Review discussions

  - [Testing for Swift packages][swift_tests]

    The Swift Package Manager team had proposed to include tests in
    Swift packages. The package manager looks for tests within a `Tests`
    subdirectory, and the tests can be run with a `swift build --tests`
    command.

    These are the concerns expressed on this proposal:

      * Tests are built whenever the package is built. We don't want to
        build the tests whenever we make a small change and recompile.
      * `swift build --tests` is weird because building is different
        from testing
      * It's XCTest-only, while there are already many alternative
        testing tools. Ideally, Swift packages should handle testing
        generically, without caring about what framework is being used
        for testing.

    Drew Crawford posted an excellent [summary of the arguments](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160111/006197.html)
    against this proposal earlier today.

  - [Flexible memberwise initialization][flexible_init]

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

    There were many supporters to this proposal. Most reviewers agreed 
    that this proposal addresses a problem worth solving, and addresses
    it thoroughly, considering all possibile scenarios of usage.
    
    There was some discussion on whether the `...` in the syntax is
    really necessary given that the `memberwise` keyword is present.
    (The `...` syntax was proposed by Chris Lattner while this was at
    draft stage.) The argument made in favour of `...` was that it's a
    placeholder for the compiler-generated arguments, and serves as a
    visual reminder that the compiler is going to fill that up.

    There were some arguments posed against this proposal:

      * There are too many things to consider here that it makes this
        feature too complicated when weighed against what we get in
        return. For example, when we use this feature, we have to think
        about `let` vs `var` members, overwriting declared defaults and
        how this interacts with access modifiers. (To illustrate the
        complexity, Jordan Rose [tried to imagine](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160104/005960.html)
        how the Swift book would explain this feature.)

      * Instead of addressing this problem as a specific feature, this
        can be covered by (or revisited after) more general future Swift
        features. In particular, Dave Abrahams and Joe Groff felt that
        this could be addressed better when Swift has [variadic
        generics](https://en.wikipedia.org/wiki/Variadic_template), by
        using something like [Python's
        **args](https://docs.python.org/dev/tutorial/controlflow.html#arbitrary-argument-lists).

    These arguments against this proposal are valid. To me, it looks
    like this proposal is unlikely to get accepted in its present form. 

[associated]: https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md
[swift_tests]: https://github.com/apple/swift-evolution/blob/master/proposals/0019-package-manager-testing.md
[flexible_init]: https://github.com/apple/swift-evolution/blob/master/proposals/0018-flexible-memberwise-initialization.md
[memberwise_swift_book]: https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID214

  - [Add StaticString.UnicodeScalarView](https://github.com/apple/swift-evolution/blob/master/proposals/0010-add-staticstring-unicodescalarview.md)

    Kevin Ballard proposes a way for substrings of `StaticString`s to be
    created, and for them too to be `StaticString`s.

### Accepted proposals

  * [Constraining AnySequence.init](https://github.com/apple/swift-evolution/blob/master/proposals/0014-constrained-AnySequence.md)

### Rejected proposals

  * [Require self for accessing instance members](https://github.com/apple/swift-evolution/blob/master/proposals/0009-require-self-for-accessing-instance-members.md)

    Douglas Gregor has outlined the [rationale for rejecting](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160104/005478.html) this proposal.

### Discussions

I found the discussions reviewing the proposals the most interesting
this week, so I didn't really follow any of the other discussions,
sorry.

Michael Tsai has [blogged about](http://mjtsai.com/blog/2016/01/10/proposal-xctest-support-for-swift-error-handling/) one of the discussions last week on XCTest support.

### Other

 - Slava Pestov from the Swift team [tweeted](https://twitter.com/slava_pestov/status/685257918220898304) a link to a [work-in-progress draft](https://github.com/apple/swift/blob/master/docs/LibraryEvolution.rst) on the Swift team's goals
   for the Swift ABI (scheduled for Swift 3.0)

