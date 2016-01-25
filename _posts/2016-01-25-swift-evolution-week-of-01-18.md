---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Jan 18, 2016"
description: "Review of Swift API guidelines is in progress. Testing for
Swift packages is accepted."
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
---

Here's a summary of selected updates from last
week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Under review this week

  - **API Design guidelines**

    _Previously:_ Week of
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#api-design-guidelines).

    In Swift 3, the look of the language is getting a redesign, which is
    governed by three interlinked proposals:

      - [Objective-C name translation](https://github.com/apple/swift-evolution/blob/master/proposals/0005-objective-c-name-translation.md)

        These are some of the changes planned for how Obj-C
        APIs are accessed from Swift 3:

          - Prune redundant type names from function arguments, so it's
            `remove(member: Element)` rather than `removeElement(member:
            Element)`
          - Prepend `is` to Boolean properties, so it's
            `application.isStatusBarHidden` rather than
            `application.statusBarHidden` ([See all changes like this](https://github.com/apple/swift-3-api-guidelines-review/commit/a6ce38eec58e8c2da901d0049a04e4b875c403b2))
          - Strip the `NS` prefix from Foundation APIs, so it's `Rect`
            rather than `NSRect` ([See all changes like this](https://github.com/apple/swift-3-api-guidelines-review/commit/45e9023fc0f448ede91e34f37187fc54d3974074))

      - [Swift stdlib API changes](https://github.com/apple/swift-evolution/blob/master/proposals/0006-apply-api-guidelines-to-the-standard-library.md)

        These are some of the changes planned for the Swift Standard
        Library:

          - Remove `Type` suffix from protocol names, so it's
            `Collection`, not `CollectionType`
          - Generators are now called iterators. For example, `SequenceType.generate() -> Generator` becomes `SequenceType.iterator() -> Iterator`.
          - Make non-mutating methods read as non-phrases. So `sort()`
            that currently returns a sorted list becomes `sorted()`.

      - [API Design Guidelines](https://github.com/apple/swift-evolution/blob/master/proposals/0023-api-guidelines.md)

        The changes proposed in the other two proposals are motivated by these API design guidelines. The guidelines themselves are here: [proposed version](https://swift.org/documentation/api-design-guidelines/), [in-development version](http://apple.github.io/swift-internals/api-design-guidelines/).

        Discussion on the guidelines included these points:

          - There's a guideline that says:

            > When a mutating method is described by a verb, name its non-mutating counterpart according to the “ed/ing” rule, e.g. the non-mutating versions of `x.sort()` and `x.append(y)` are `x.sorted()` and `x.appending(y)`.

            But as Joe Groff 
            [pointed out](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160118/007368.html),
            English is a weird language, so while it works for sort,
            it doesn't work as unambiguously for split, cut, etc.

          - As per the guidelines, redundant type names should be removed
            from function names (like `removeElement()` becomes
            `remove()`), except when the type of the
            argument doesn't sufficiently describe what it is (like
            `addObserver()`, in which case the function name should
            remain as is).

            As per the proposal, the function should continue to be
            invoked as `addObserver(obj, forKeyPath: path)`. David Owens 
            [suggested](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160118/007401.html)
            that invoking it as `add(observer: obj, forKeyPath: path)`
            would read better.

          - While we're here, why should enums remain UpperCamelCase?
            Like other non-types, shouldn't they be lowerCamelCase?
    
### Accepted proposals

  - <a name="swift-pm-testing"></a>
    [Testing for Swift packages][swift_tests]

    _Previously:_ Weeks of
    [Dec 28](/last-week-in-swift-evolution/2016/swift-evolution-week-of-12-28/#swift-pm-testing),
    [Jan 4](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-04/#swift-pm-testing),
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#swift-pm-testing).

    The Swift Package Manager team's proposal to include tests in
    Swift packages was accepted.

    This proposal doesn't support arbitrary test frameworks. Rick
    Ballard, the review manager, [says](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160118/007278.html):

    > This initial proposal includes support for using XCTest; we expect
    > that adding generic support for other testing frameworks will be
    > discussed in an upcoming proposal.
    
[swift_tests]: https://github.com/apple/swift-evolution/blob/master/proposals/0019-package-manager-testing.md

  - <a name="naming-functions"></a>
    [Naming functions with argument labels][naming_functions]

    _Previously:_ Weeks of
    [Dec 28](/last-week-in-swift-evolution/2016/swift-evolution-week-of-12-28/#naming-functions),
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#naming-functions).

    Doug Gregor's proposal to identify functions with argument labels
    was accepted. The code would look something like this:

        let someView = UIView()
        // Assign function to variable
        let fn1 = someView.insertSubview(_:aboveSubview:)
        // Call assigned function
        fn1(subview1, aboveSubview: subview2)

    Doug Gregor already has a [working implementation](https://github.com/DougGregor/swift/compare/se-0021-generalized-naming) of this proposal.

[naming_functions]: https://github.com/apple/swift-evolution/blob/master/proposals/0021-generalized-naming.md

### Review discussions

  - [Referencing Obj-C selectors cleanly](https://github.com/apple/swift-evolution/blob/master/proposals/0022-objc-selectors.md)

    _Previously:_ Week of
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#objc-selectors).

    The syntax originally proposed to refer to Obj-C selectors was
    `Selector(foo:)`. Last week, the Swift team felt that this
    particular syntax was hard to implement. Joe Groff [says](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160118/007046.html):

    > We discussed this proposal during our core team review meeting,
    > and concerns were raised about the implementability of the
    > proposed 'Selector(Type.method)' syntax. Producing the selector is
    > something that can't quite be modeled as a real initializer, so
    > the type checker would have to perform heroics to disambiguate a
    > selector literal reference from a proper initializer call. We
    > recommend reconsidering alternative syntactic forms for
    > referencing the selector.

    The current favorite alternate syntax for this is `#selector(foo:)`.

### Merged proposal implementations

  - The [associatedtype](https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md) implementation was [merged](https://github.com/apple/swift/commit/38c1de69e4b4c27ac1916d1e6fe601beb5d3a5f4)

### Other weeklies

  - Jesse Squires' [Swift Weekly Brief Issue #6](http://swiftweekly.github.io/issue-6/)
    covers the week till last Thursday

