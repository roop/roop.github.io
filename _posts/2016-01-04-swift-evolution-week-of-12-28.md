---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Dec 28, 2015"
description: "<code>associatedtype</code>, <code>memberwise init</code> and package testing
are up for review. Discussion has started on automatic protocol forwarding." 
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
---

Happy New Year, everyone. Here's a summary of selected updates from last
week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Up for review this week

  - [New keyword for associate type declaration][associated]

    We declare associated types currently using the `typealias` keyword,
    and the same keyword is used to create an alias for a type. Loïc
    Lecrenier proposed a new keyword to declare associated types. The
    keyword was initially `associated`, and has now been changed to
    `associatedtype`.
    
    This proposal is up for review now
    (Jan 4 &ndash; Jan 6). This is quite a straightforward proposal (the main
    debate earlier was about what the new keyword should be named) and looks
    likely to be accepted.

  - [Testing for Swift packages][swift_tests]

    The Swift Package Manager team had proposed to include tests in
    Swift packages. The package manager looks for tests within a `Tests`
    subdirectory, and the tests can be run with a `swift build --tests`
    command.

    This proposal is scheduled for review this week (Jan 5 &ndash; Jan 7).

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

    This proposal is scheduled for review this week (Jan 6 &ndash; Jan 10).

[associated]: https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md
[swift_tests]: https://github.com/apple/swift-evolution/blob/master/proposals/0019-package-manager-testing.md
[flexible_init]: https://github.com/apple/swift-evolution/blob/master/proposals/0018-flexible-memberwise-initialization.md
[memberwise_swift_book]: https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Initialization.html#//apple_ref/doc/uid/TP40014097-CH18-ID214

### Discussions

These are the discussions from last week that I found the most interesting:

 1. <a name="protocol-forwarding"></a>
    Automatic protocol forwarding ([last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/004737.html))

    Per this [draft proposal](https://github.com/anandabits/swift-evolution/blob/automatic-protocol-forwarding/proposals/NNNN-automatic-protocol-forwarding.md) from Matthew Johnson, we can have function calls on a type (the forwarding type) automatically forwarded to a member (of type forwardee) of the forwarding type.

 2. <a name="naming-functions"></a>
    Generalized naming for any function
    ([previously here](http://roopc.net/last-week-in-swift-evolution/2015/swift-evolution-week-of-12-21/#naming-functions),
    [last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/004631.html))

    Swift functions can be assigned to variables, but when the same base
    name is shared by multiple functions (like
    `insertSubview(view:UIView, aboveSubview: UIView)` and
    `insertSubview(view: UIView, belowSubview: UIView)`), Swift doesn't
    have a way for us to specifically name a particular function. Doug
    Gregor had proposed to name functions like this, and extended it to
    support setters, getters, initializers and subscripting.

    Last week, Joe Groff [suggested](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/004741.html) that the same idea can be extended to disambiguate function calls (whether to call `foo()` in `SuperProtocolA` or in `SuperProtocolB`).

 3. <a name="objc-selector"></a>
    Expression to retrieve the Objective-C selector of a method
    ([earlier](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151221/004559.html),
    [last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/004661.html))

    In a discussion related to the previous one, here's a pitch to fix
    the "stringly typing" when we access Obj-C selectors in Swift. So,
    instead of the brittle `btn.addTarget(self, action:"foo", ...)`,
    we can say something like:
    `btn.addTarget(self, action:Selector(MyClass.foo), ...)`.
 
 4. Kevin Ballard suggested a bunch of additions to the Swift
    standard library:

      - [CollectionType.cycle](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/004635.html): Returns an infinite sequence looping through a finite collection
      - [SequenceType.find()](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/004814.html): Returns the first element matching a predicate
      - [BufferedSequence, BufferedGenerator](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/005010.html): Wrapper around a
        sequence / generator that lets us peek into it without modifying its index
    
### Other

 - The Swift team published a list of ["commonly rejected" ideas][wont_fix]
 - Chris Lattner [explained](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151228/005046.html) his thoughts on what stuff should go into the [Swift standard library](https://github.com/apple/swift/tree/master/stdlib/public)
   and what into the [Swift foundation](https://github.com/apple/swift-corelibs-foundation)

[wont_fix]: https://github.com/apple/swift-evolution/blob/master/commonly_proposed.md

