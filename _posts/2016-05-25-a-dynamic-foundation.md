---
layout: post
title: "A dynamic foundation"
description: "In which I argue that Swift has a solid base on which to
build dynamic features like Objective-C's."
tagline: ""
category: posts
---

Brent Simmons, in a series of posts, argued the case for making Swift
more dynamic like Objective-C.

[brent_summary]: http://inessential.com/2016/05/18/what_im_doing_with_these_articles
[brent_optimism]: http://inessential.com/2016/05/15/the_case_for_dynamic-swift_optimism

In [one of those posts][brent_summary], he summarizes his case as:

> I’m documenting problems that Mac and iOS developers solve using the
> dynamic features of the Objective-C runtime. [...]
>
> The point is that these problems will need solving in a possible
> future world without the Objective-C runtime. These kinds of problems
> should be considered as that world is being designed. The answers
> don’t have to be the same answers as Objective-C &ndash; but they need to be
> good answers, better than (for instance) writing a script to generate
> some code.

Though we app developers rarely have to deal directly with the
Objective-C runtime, many frameworks we use do that in order to give us
powerful features in deceptively simple interfaces (like KVO and Core
Data). That, in turn, makes it easier for us to develop apps.

So the reduced dynamism in Swift doesn't affect us now, but it might hit
us many years (or decades) later, when eventually, Swift completely
supersedes Objective-C and Apple starts making Swift _frameworks_ that
are meant to be used only from Swift, and therefore don't use the
Objective-C runtime at all.

But I'm not too worried about it because I think Swift has a good
foundation to build its dynamism on. I share Brent Simmons' optimism in
[his earlier post][brent_optimism]:

> I strongly suspect that Swift is designed with dynamic programming in
> mind — and is potentially better for dynamic programming than
> Objective-C. (Since, after all, it drops the C, is a fresh start that
> learns from the past, doesn’t have to worry about breakage so much —
> and, hey, look at that team.)
>
> If this is true — and I don’t have an easy way to evaluate it, so I’ll
> assume it — then we can assume that the reason it hasn’t been an issue
> so far is that we already have the Objective-C runtime. (Even if all
> your code is Swift, your app is using that runtime.)

I believe there's a way to evaluate how true Brent's optimistic
suspicion is: To look at Swift's internals at present and see if the
dynamism we desire can be built on top of it. (**Update:** This does
not constitute evidence. See updates below.)

Internally, Swift class instances, even of pure Swift classes [^1], have
a memory layout that looks exactly like an Objective-C class instance -
with an `isa` pointer that points to a class object with a 
dispatch table (as Mike Ash [found out][mike_ash] early on). However,
while in Objective-C, all methods are listed in the dispatch table, in
Swift, only those methods marked `dynamic` are listed [^2]. Since Swift
guarantees that `dynamic` methods are never devirtualized, and since
`dynamic` is part of [Swift's library ABI goals][library_evolution], <del>it's
highly likely that the dispatch table would remain in future Swift versions
as well</del> (See update below).

The presence of this dispatch table would enable Objective-C-like
dynamic features to be built on top.  I'm reasonably hopeful that well
before it's time for Objective-C to be phased out, Swift would have
gained equivalent dynamic features, though maybe not with the same
interface. However, unlike Objective-C, the dynamism in Swift would
almost definitely remain disabled by default and would need to be
enabled for specific methods and properties using the `dynamic`
modifier.

---
**Update:**

[Jordan Rose](https://twitter.com/UINT_MIN/status/735925069545754625) from the Swift team:

> Without commenting on anything else, I wouldn't count this as evidence; it's not present on Linux.
>
> `dynamic` is currently limited to `@objc` methods, which is fine because there's no other way to replace them.

This is indeed true. Even the doc for `dynamic` starts with “When Swift APIs are imported by the Objective-C runtime”. My mistake in having missed noting that when I wrote this post.

Since the `dynamic` modifier exists only for interoperability with
Objective-C, this isn't sufficient evidence to say that Swift classes
will gain dynamism similar to Objective-C classes.

---

A related fun observation: All pure Swift root classes have an `isa`
saying that they inherit from a `SwiftObject` class, which is at present
actually [written in Objective-C][SwiftObject.mm] [^3], conforming to the
`NSObject` protocol (<del>and the code is designed to work even when
Swift-Objective-C-interoperability is not required i.e. when
`SWIFT_OBJC_INTEROP` is false</del> (**Update:** `SwiftObject` is [not even available](https://twitter.com/UINT_MIN/status/735928797745188864)
when Swift-Objective-C-interoperability is off).
Because of this, you can actually send `respondsToSelector:` to an
instance of a _pure Swift class_ to check for methods marked `dynamic`
(after casting to `AnyObject` to keep the compiler happy):

~~~
class C {
    func staticfn() { }
    dynamic func dynamicfn() { }
}
let c = C()
(c as! AnyObject).respondsToSelector("nonexistingfn") // false
(c as! AnyObject).respondsToSelector("staticfn") // false
(c as! AnyObject).respondsToSelector("dynamicfn") // true
~~~

---
**Update:**

[Jordan Rose](https://twitter.com/UINT_MIN/status/735925203654430720):

> A good amount of `SwiftObject` comes down to "there should be no problems putting a Swift object in an NSArray".

That makes sense as well. Swift classes pose as Objective-C classes when
interoperating with Objective-C so that the Objective-C code can talk to
Swift without having to resort to Swift-specific hacks.

---

Given how pure Swift classes closely resemble Objective-C classes
internally, I think Swift will eventually gain dynamic features
comparable to Objective-C, and I think that will happen well before
it's time for the Objective-C runtime to get sunsetted.

---
**Update:**

[Jordan Rose](https://twitter.com/UINT_MIN/status/735936491742887936):

> All of this doesn't mean we won't do it! It just means we won't do it just because Objective-C does it.

So there's still hope. :)

[library_evolution]: https://github.com/apple/swift/blob/master/docs/LibraryEvolution.rst
[mike_ash]: https://www.mikeash.com/pyblog/friday-qa-2014-07-18-exploring-swift-memory-layout.html
[SwiftObject.mm]: https://github.com/apple/swift/blob/85fde8b/stdlib/public/runtime/SwiftObject.mm

---

[^1]: What I call pure Swift classes are classes that are neither marked `@objc`, nor have `NSObject` as an ancestor

[^2]: I deduced this by using Mike Ash's code [here](https://github.com/mikeash/memorydumper)

[^3]: Because it's written in Objective-C, the runtime dynamism probably doesn't work on Linux (I don't know for sure because I haven't yet tried it on Linux), but that's not a problem for me as an app developer

