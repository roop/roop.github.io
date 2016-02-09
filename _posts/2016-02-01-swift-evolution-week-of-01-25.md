---
layout: weekly
weekly-title: "Last week in Swift Evolution"
title: "Week of Jan 25, 2016"
description: "Review of Swift API guidelines continues. <code>#selector(foo:)</code>
is accepted. Chris Lattner explains what Swift 3.0 is really about."
category: last-week-in-swift-evolution
rss: /last-week-in-swift-evolution/rss.xml
---

Here's a summary of selected updates from last
week from the Swift Evolution
[repo](https://github.com/apple/swift-evolution) and [mailing
list](https://lists.swift.org/pipermail/swift-evolution/):

### Under review this week

  - <a name="api-design-guidelines"></a>
    **API Design guidelines**

    _Previously:_ Weeks of
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#api-design-guidelines),
    [Jan 18](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-18/#api-design-guidelines).

    Given that the API redesign is one of the most important changes
    coming to Swift in 3.0, Apple wrote about this effort [in the Swift
    blog](https://swift.org/blog/swift-api-transformation/).
    
    The [API Design Guidelines](https://github.com/apple/swift-evolution/blob/master/proposals/0023-api-guidelines.md) is the umbrella proposal.
    The changes proposed in the other two proposals are motivated by these API design guidelines. The guidelines themselves are here: [proposed version](https://swift.org/documentation/api-design-guidelines/), [in-development version](http://apple.github.io/swift-internals/api-design-guidelines/).

    Discussion on the guidelines included these points:

      - On the guideline that says:

        > When a mutating method is described by a verb, name its non-mutating counterpart according to the “ed/ing” rule, e.g. the non-mutating versions of `x.sort()` and `x.append(y)` are `x.sorted()` and `x.appending(y)`.

          - Chris Lattner seconded the `InPlace` suffix to name
            mutating methods (like `sortInPlace()`,
            `reverseInPlace()`)
          - Erica Sadun shared her thoughts in a [Github
            document](https://github.com/erica/SwiftStyle/blob/master/Grammatical.md).

      - As per the guidelines, redundant type names should be removed
        from function names (like `removeElement()` becomes
        `remove()`).

        So, how would we want the API to look at the call site for
        inserting into an list of items? These are the possibile
        options:

          1. `items.insert(12, atIndex: 2)` (current API)
          2. `items.insert(12, at: 2)`
          3. `items.insert(12, index: 2)`

        Radosław Pietruszewski [prefers](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/007664.html) (3), because it conveys more
        meaning, even if (2) reads more like English. He says:

        > I think there’s an agreement that making Swift sound like
        > English is a non-goal. In fact, trying to do so is often
        > harmful as we end up putting unnecessary words, like
        > “with”, “and”, “by”, etc. (I’m not saying they’re always
        > unnecessary; they often help convey the semantics or
        > intent, or are needed for the method name to make sense.
        > But too often they’re just glue words used to make code
        > sound like English without any actual readability
        > benefits, and merely adding noise.)

      - Chris Lattner
        [supported](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/007636.html)
        the idea of changing enums so that the enum cases are
        defined with lowerCamelCase names.

      - The guidelines doesn't currently talk about how
        [initialisms](http://www.dailywritingtips.com/initialisms-and-acronyms/)
        (like HTML, UTF) should be handled in the API. The
        predominant opinion is that they should either be
        all-uppercase (like in function names e.g. `HTMLoutput()`),
        or all-lowercase (like in variable names e.g. `let
        htmlString = "<u>\(string)</u>"`)
    
    The deadline for this review has been extended to Feb 5, 2016.

  - <a name="modern-debug-identifiers"></a>
    [Modernizing \_\_LINE\_\_, etc.](https://github.com/apple/swift-evolution/blob/master/proposals/0028-modernizing-debug-identifiers.md)

    While discussing how Obj-C selectors can be referenced cleanly from
    Swift (_previously:_ Weeks of [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#objc-selectors), [Jan 18](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-18/#objc-selectors)), Chris Lattner [suggested](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160118/007297.html) that "we could consider renaming `__LINE__` and friends to `#LINE`".

    Erica Sadun has developed that small suggestion into a full-fledged
    proposal to rename `__LINE__` to `#line`, `__FILE__` to `#file`,
    `__COLUMN__` to `#column` and `__DSO_HANDLE__` to `#dso_handle`.

    The review of this proposal runs till tomorrow (Feb 2, 2016).

### Accepted proposals

  - [Referencing Obj-C selectors cleanly](https://github.com/apple/swift-evolution/blob/master/proposals/0022-objc-selectors.md)

    _Previously:_ Weeks of
    [Jan 11](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-11/#objc-selectors),
    [Jan 18](/last-week-in-swift-evolution/2016/swift-evolution-week-of-01-18/#objc-selectors).

    This proposal was [accepted](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/007797.html) for Swift 2.2 with the syntax changed to `#selector(Foo.bar)`.

### Other stuff

  - Chris Lattner 
    [commented](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/007737.html)
    on what Swift 3.0 is "really about":

    > Overall, this comes back to a higher order philosophy about what
    > Swift 3 is really about: it is about driving the next wave of
    > adoption of Swift by even more people.  This will hopefully come
    > from new audiences coming on-board as Corelibs + Swift on Linux
    > (and other platforms) become real, SwiftPM being great and growing
    > into its own, and the Swift language/stdlib maturing even more.

    In the same mail, he also talked about the goal of stabilizing the
    syntax in Swift 3.0:

    > While our community has generally been very kind and understanding
    > about Swift evolving under their feet, we cannot keep doing this
    > for long.  While I don’t think we’ll want to guarantee 100% source
    > compatibility from Swift 3 to Swift 4, I’m hopeful that it will
    > be much simpler than the upgrade to Swift 2 was or Swift 3 will
    > be.

### Other weeklies

  - Jesse Squires' [Swift Weekly Brief Issue #7](http://swiftweekly.github.io/issue-7/)
    covers the week till last Thursday

