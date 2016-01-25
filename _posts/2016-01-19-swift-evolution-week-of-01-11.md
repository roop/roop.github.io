---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Jan 11, 2016"
description: "<code>memberwise init</code> was rejected. Swift API
guidelines review starts this week. Discussion started on property behaviors."
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
---

Here's a summary of selected updates from last
week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Up for review this week

  - <a name="api-design-guidelines"></a>
    [API Design guidelines](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160111/006609.html)

    This is an effort to define guidelines for how a Swift API should
    look like, and apply that to the Swift stdlib and to how Objective-C
    functions are accessed from Swift.

    This involves three interlinked sub-proposals:

      1. [API design guidelines](https://swift.org/documentation/api-design-guidelines/)
      2. [stdlib API changes](https://github.com/apple/swift-evolution/blob/master/proposals/0006-apply-api-guidelines-to-the-standard-library.md)
      3. [Objective-C name translation](https://github.com/apple/swift-evolution/blob/master/proposals/0005-objective-c-name-translation.md)

    This review is going to run from tomorrow till the end of the month.

  - <a name="swift-pm-testing"></a>
    [Testing for Swift packages][swift_tests]

    The Swift Package Manager team had proposed to include tests in
    Swift packages.
    
    This went under review last week ([discussed previously here
    ](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-04/#swift-pm-testing)), but many concerns were expressed (Rick Ballard posted
    an [overview of all concerns][rick_ballard_test_concerns_summary]).
    The proposal has been revised based on the feedback got last week
    and is now up for re-review.

    These are the major changes that have been made to the proposal:

      * The previous version supported looking for tests under a
        `FooTests` folder but also said this usage was not recommended.
        This revision removes this unrecommended-but-supported option,
        so tests will always be looked for under the `Tests` folder.
      * The command is now `swift test`, rather than `swift build --test`
      * To build the package without building the tests, we will be
        able to say `swift build --without-tests`

    One of the concerns expressed on the previous proposal was that the
    `swift test` shouldn't be dependant on XCTest and should be
    test-framework-agnostic. This issue has not been resolved in the
    revised proposal &ndash; it's still based on XCTest.

    The revised proposal is now up for review, ending today (Jan 19th).
    I'd say this proposal is now in a good shape to be accepted. Once
    this proposal is implemented, generic support for test frameworks
    can be addressed in a separate proposal.
 
  - <a name="objc-selectors"></a>
    [Referencing selectors cleanly](https://github.com/apple/swift-evolution/blob/master/proposals/0022-objc-selectors.md)

    Currently, we reference Objective-C selectors in Swift using
    "stringly typing":

        button.addTarget(self, action:"foo:", ...)

    where the `"foo:"` string gets implicitly converted to a `Selector`
    instance.

    This proposal would make this type-safe by initializing `Selector`s
    by directly accessing the method name, building on the just-reviewed
    proposal for [naming functions](#naming-functions):

        button.addTarget(self, action:Selector(foo:), ...)

    The review of this proposal runs from today till Jan 23.

[swift_tests]: https://github.com/apple/swift-evolution/blob/master/proposals/0019-package-manager-testing.md
[rick_ballard_test_concerns_summary]: https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160111/006466.html

### Rejected proposals

  - [Flexible memberwise initialization][flexible_init]

    [As expected](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-04/#memberwise-init), this proposal was not accepted as it stood.

    Chris Lattner has shared [detailed notes](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160111/006469.html)
    from the core team meeting discussion on this proposal. He says:

    > The core team really doesn’t want to discuss this right now, given
    > that this is purely a sugar proposal and we need to stay focused
    > on the primary Swift 3 goals.

[flexible_init]: https://github.com/apple/swift-evolution/blob/master/proposals/0018-flexible-memberwise-initialization.md

### Review discussions

  - <a name="naming-functions"></a>
    [Naming functions with argument labels][naming_functions]

    Swift functions can be assigned to variables, but when the same base
    name is shared by multiple functions (like
    `insertSubview(view:UIView, aboveSubview: UIView)` and
    `insertSubview(view: UIView, belowSubview: UIView)`), Swift doesn’t
    have a way for us to specifically name a particular function. Doug
    Gregor's proposal addresses this issue. The syntax looks like this:

        let someView = UIView()
        let fn1 = someView.insertSubview(_:aboveSubview:)

    While at draft stage, the discussion also considered extending this
    to support assigning setters, getters, initializers and
    subscripting (as discussed [previously here](/last-week-in-swift-evolution/2016/swift-evolution-week-of-12-28/#naming-functions)). However, the proposal submitted
    for review does not cover those cases. Given that this is a simpler and
    quite straightforward proposal, this is likely to be accepted.

    Doug Gregor already has a [working implementation](https://github.com/DougGregor/swift/compare/se-0021-generalized-naming) of this proposal.

[naming_functions]: https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md

  - [Language version build config](https://github.com/apple/swift-evolution/blob/master/proposals/0020-if-swift-version.md)

    Once this proposal is implemented, we can write code for multiple
    Swift versions in the same file, like this:

        #if swift(>=2.2)
            for i in (0 ..< 10) {
                print(i)
            }
        #else
            for (var i = 0; i < 10; i++) {
                print(i)
            }
        #endif

    Only `>=` is proposed for now, so you can't say `#if swift(==2.2)`,
    for example. This again, I think, is likely to be accepted.

### Discussions

  - Behaviors ([last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160111/006523.html))

    The current `lazy` annotation on a property declares the "behavior"
    of the propoerty. Joe Groff proposes to generalize this to allow us
    to define custom "behaviors" that can be used to annotate
    properties.

    This is really cool, and the draft proposal shows multiple use
    cases, including lazy, memoization and a version of `didSet` that
    fires only when there's an actual change of value.

  - Creating a Unicode string from code units ([last week](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160111/006678.html))

    Currently, creating a Uncode `String` from a sequence of bytes works
    cleanly for just one case: null-terminated UTF-8 bytes.

    Zachary Waldowski has created a [draft proposal](https://github.com/zwaldowski/swift-evolution/blob/string-from-code-units/proposals/0000-string-from-code-units.md) to create `String` instances from an arbitrary sequence of code units with an arbitrary encoding.

### Merged proposal implementations

  - The implementation for [constraining AnySequence.init](https://github.com/apple/swift-evolution/blob/master/proposals/0014-constrained-AnySequence.md) was [merged](https://github.com/apple/swift/pull/895) into Swift 2.2

  - The [associatedtype](https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md) implementation was [attempted to be merged](https://github.com/apple/swift/pull/964)

### Other weeklies

  - Jesse Squires' [Swift Weekly Brief Issue #5](http://swiftweekly.github.io/issue-5/)
    covers the week till last Thursday

