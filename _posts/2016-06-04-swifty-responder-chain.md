---
layout: post
title: "Swifty Responder Chain"
description: "Trying to create a responder chain implementation in pure Swift"
tagline: ""
category: posts
---

From Jordan Rose's tweets in response to [my previous post][prev_post]
(his responses are inlined in the post), it's clear there's no
guarantee that Swift will get an Objective-C-ish runtime (there's no
guarantee it won't either). So I'm trying to approach the problem from the
other side: What do we need in Swift to create UIKit-like "magic
interfaces" without swinging all the way to the [dynamic
side][brent_what_is_dynamism], à la Objective-C.
For a start, let's see if we can create a responder chain for a
hypothetical pure-Swift equivalent for UIKit.

[prev_post]: /posts/2016/a-dynamic-foundation/
[brent_what_is_dynamism]: http://inessential.com/2016/05/26/a_definition_of_dynamic_programming_in_t

Brent Simmons' [first go][brent_take_1] at a pure-Swift responder chain
requires us to handle all commands in one `respondsToCommand:` method
with a switch statement to select between commands, which is
[no][brent_switch] [good][gte_switch].  Joe Groff
[suggested][joe_groff_suggestion] we use protocol conformance, but that
still [requires][brent_take_2] a `performSelector:`, which the Swift
runtime _might_ not give us. So what can we do with what we have, and
with what we're more likely to get?

[brent_take_1]: http://inessential.com/2016/05/15/a_hypothetical_responder_chain_written_i
[brent_take_2]: http://inessential.com/2016/05/17/responder_chain_followup

[joe_groff_suggestion]: https://twitter.com/jckarter/status/731991024206123008
[brent_switch]: http://inessential.com/2016/05/25/oldie_complains_about_the_old_old_ways
[gte_switch]: https://twitter.com/gte/status/735370823507312640

### A better interface

Building on that [suggestion][joe_groff_suggestion] by Joe Groff, if we
can't invoke selectors directly, we can introduce a new `Command` type
to abstract that out.  The `Command` type gives us a uniform interface
to perform a command on a corresponding `Responder` object. (This
interface is conceptually similar to what Matthew Johnson [came up
with][matthew_johnson_tweet], but is simpler and therefore hopefully
easier to wrap our heads around.)

[matthew_johnson_tweet]: https://twitter.com/anandabits/status/736689306798981120
[matthew_johnson_github]: https://gist.github.com/anandabits/ec26f67f682093cf18b170c21bcf433e

With this interface, whenever we introduce a new command, we have to
define two new types:

 1. A responder: A sub-protocol of `Responder` declaring the methods
    used to respond to that command
 2. A command: A concrete subtype of `Command` that provides a uniform
    interface using the methods declared in the responder. This is what
    will get associated with a menu item or an equivalent thing that
    would "launch" the command.

~~~ Swift
protocol CopyResponder: Responder {
    func canCopy() -> Bool
    func copy()
}

struct CopyCommand: Command {
    func canPerformOnResponder(responder: CopyResponder) -> Bool {
        return responder.canCopy()
    }
    func performOnResponder(responder: CopyResponder) {
        responder.copy()
    }
}
~~~

To enable an object to respond to the `CopyCommand`, all we have to do
is to extend `CopyResponder` and implement its methods.

~~~ Swift
extension MyView: CopyResponder {
    func canCopy() -> Bool {
        return true
    }
    func copy() {
        print("Copying from MyView")
    }
}
~~~

Query functions like `canCopy()` make it possible for menus to be
enabled / disabled based on the state of the app at runtime (like what
Gus Mueller [does with Acorn][gus_dynamic]), by having responder objects
change their answer at runtime.

[gus_dynamic]: http://shapeof.com/archives/2016/5/dynamic_swift.html

With this interface, the app developer doesn't have to write any switch
statements to route commands. There's no `performSelector:`-like runtime
magic either.

### Behind the scenes

There's a bunch of framework code we need for making it possible for the
app developer to define commands and responders like above.

To start simple, a responder is something that has an optional
next responder, so that we can have a chain of responders.

~~~ Swift
protocol Responder {
    var nextResponder: Responder? { get }
}
~~~

A command needs to be able to define methods that take in the instances
of the corresponding responder sub-type as an argument, so we need to
set the corresponding responder type as an associated type.

~~~ Swift
protocol Command {
    associatedtype AssociatedResponder
    func canPerformOnResponder(responder: AssociatedResponder) -> Bool
    func performOnResponder(responder: AssociatedResponder)
}
~~~

To enable views to be included in the responder chain, you can extend
`Responder` and return whatever is appropriate as the next responder.

~~~ Swift
class View {
    var superview: View? = nil
}

extension View: Responder {
    var nextResponder: Responder? { return self.superview }
}
~~~

The application knows what the first responder is, and it can traverse
the responder chain querying each responder in the process.

~~~ Swift
class Application {
    var firstResponder: Responder? = nil
    func performCommand<C: Command>(command: C) {
        var r: Responder? = self.firstResponder
        while r != nil {
            if let r = r {
                if command.canPerformOnResponder(r) {
                    command.performOnResponder(r)
                    return
                }
            }
            r = r?.nextResponder
        }
    }
}
~~~

We cannot take an instance of `Command` as an argument directly, because
`Command` has an associated type, so we use generics to define
`performCommand()` &ndash; `C` is a placeholder type that conforms to
the `Command` protocol.

The calls to `canPerformOnResponder()` and `performOnResponder()` above pass
a `Responder` argument, but the `Command` protocol only declares methods
that take in `AssociatedResponder` instances. So to make the above code
work, we need to generalize those methods to take in any `Responder`.

~~~ Swift
extension Command {
    func canPerformOnResponder(responder: Responder) -> Bool {
        if let associatedResponder = responder as? AssociatedResponder {
            return self.canPerformOnResponder(associatedResponder)
        }
        return false
    }
    func performOnResponder(responder: Responder) {
        if let associatedResponder = responder as? AssociatedResponder {
            self.performOnResponder(associatedResponder)
        }
    }
}
~~~

And that's it. Now we have all the pieces in place to enable us to
define commands, declare corresponding responders and make arbitrary
types behave as those responders.

This pattern can be extended for framework events as well:

~~~ Swift
protocol HandleTouchResponder: Responder {
    func canHandleTouch(touches: [Touch]) -> Bool
    func handleTouch(touches: [Touch])
}

struct HandleTouchCommand: Command {
    let touches: [Touch] = []
    func canPerformOnResponder(responder: HandleTouchResponder) -> Bool {
        return responder.canHandleTouch(self.touches)
    }
    func performOnResponder(responder: HandleTouchResponder) {
        responder.handleTouch(self.touches)
    }
}
~~~

### Where's the switch?

So how does the command dispatch happen in the above architecture?

Since we always start command propagation with a command whose concrete
command type is known, we can always get to its associated type. From
the associated type, we know how to perform the command on a responder
object that conforms to the associated type. That's how we're able to
get away without a switch &ndash; we're using the commands as a sort-of
distributed dispatch table.

### Furthermore

I'm suprised we can get this far towards a usable responder chain with
the current state of Swift, but with improvements to the language, we
can do better.

#### Constraining the associated responder

We should be able to tell Swift that the associated type has to conform
to `Responder`, like this:

~~~ Swift
protocol Command {
    associatedtype AssociatedResponder: Responder
    func canPerformOnResponder(responder: AssociatedResponder) -> Bool
    func performOnResponder(responder: AssociatedResponder)
}
~~~

but it looks like [we can't do that at present][swift_bug].

[swift_bug]: https://bugs.swift.org/browse/SR-1581

#### Protocols with associated types

Protocols with associated types have a lot of restrictions on how they
can be used. One of those is that you can't instantiate or cast to those
types.

If the Swift protocol system gets cleverer, I can imagine simplifying
the interface by moving the `associatedtype` to the `Responder`.

~~~ Swift
protocol Command {
}

protocol Responder {
    associatedtype AssociatedCommand: Command
    func canPerformCommand(command: AssociatedCommand)
    func performCommand(command: AssociatedCommand)
}
~~~

If `Command` and `Responder` could be declared like this, it would
reduce the boilerplate required for declaring new commands. But as of
Swift 2.2, with the above declaration of `Responder`, we can't have
variables of `Responder` or any of its sub-protocols like
`CopyResponder`, nor can we downcast to those types, so we can't get far
with this.

**Update 6/Jul/2015:** As Matthew Johnson [explained][mj_twitter] to me
over Twitter, this solution is requires that a type be able to conform
to a root protocol in multiple ways (i.e. `MyView` needs to conform to
`Responder` through `CopyResponder` as well as through
`GoFishingResponder`), which the Swift team frowns upon. Therefore this
solution for a responder chain might not become feasible even in the
future.

[mj_twitter]: https://twitter.com/anandabits/status/741245072004374528

Moreoever, this way of modelling a responder chain can be done using
protocol generics:

~~~ Swift
protocol Responder<C: Command> {
    func canPerformCommand(command: C)
    func performCommand(command: C)
}

protocol Responder<CopyCommand> {
    func canPerformCommand(command: CopyCommand)
    func performCommand(command: CopyCommand)
}

extend MyView: Responder<CopyCommand> {
    ...
}
~~~

And Doug Gregor's [Generics Manifesto mail][generics_manifesto_mail]
says generic protocols is unlikely to become part of Swift because:

> [These use cases] seem too few to justify the potential for
> confusion between associated types and generic parameters of
> protocols; we’re better off not having the latter.

I think the responder chain is a good use case for generic protocols
&ndash; there's no confusion in a type conforming to `Responder` in
multiple ways. However, I think there is going to be still a problem
casting an object to `Responder`, so it might not be a sufficiently good
use case. **&lt;/End of update&gt;**

[generics_manifesto_mail]: https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160229/011666.html

### So far so good

It's amazing that Swift protocols can get us this far towards a
responder chain implementation (the full working code is
[here][gist]). And we can already see how it can get better as Swift
gets its wrinkles ironed out.

That said, the responder chain was probably an easy start. KVO and Core
Data will get progressively harder to create without dynamic language
features.

[gist]: https://gist.github.com/roop/397e4cc5e7deeab6a896ef4034a06ae5

