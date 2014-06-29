---
layout: post
title: "Arrays in Swift Beta 2"
tagline: "Problem, solution and workaround"
description: "What's the problem with Arrays in Swift (Beta 2), and how do we
              get around it?"
category: posts
postscript: "Updated 29/Jun/2014: Appending doesn't always trigger
             a realloc - it can only potentially do so. Added direct links
             to Github Gists (to be more RSS-reader-friendly)."
---

We know that Arrays in Swift [behave][weird1] [a little][weird2]
[weirdly][weird3], and also that [Apple will fix this going
forward][fix1].

[weird1]: https://github.com/andrewsardone/swift-playground/issues/2
[weird2]: http://blog.human-friendly.com/swift-arrays-the-bugs-the-bad-and-the-ugly-incomplete
[weird3]: https://medium.com/@owensd/swift-arrays-a8f1f91bed78

[fix1]: https://twitter.com/brentdax/status/479690847932256256

In this post, we'll see:

 - **The problem:** What makes Arrays behave weirdly?

 - **The solution:** How is the behaviour is going to be, once Apple
   fixes it?
 
 - **A workaround:** Is it possible to make Arrays behave sanely in Beta
   2 itself?

### The problem

To understand the problem with Arrays, we need to know how Swift handles
copying of Arrays. The [Swift book] offers a helpful hint in the form of
a note under ["Assignment and Copy Behavior for Collection
Types"][copy-behavior] that says:

> The descriptions below refer to the “copying” of arrays, dictionaries,
> strings, and other values. Where copying is mentioned, the behavior
> you see in your code will always be as if a copy took place. However,
> Swift only performs an actual copy behind the scenes when it is
> absolutely necessary to do so. Swift manages all value copying to
> ensure optimal performance, and you should not avoid assignment to try
> to preempt this optimization.

[Swift book]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/
[copy-behavior]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-XID_109

We'll first see the nitty-gritties of what that note really means, and
validate that against how Arrays behave.

#### Structs are value types

We know that in Swift, structs and enums are value types (as opposed to
classes which are reference types).

So, when we do `var dstInstance: MyStruct = srcInstance`, the contents
of `srcInstance` are copied over to `dstInstance`. After this, the data
in `srcInstance` and `dstInstance` are independent and can be
independently modified without affecting each other.

Assigning structs is a more expensive operation (compared to assigning
classes) because the contents need to be copied to a new memory
location. Assigning a class will generally be quicker because only a
pointer to the contents needs to be copied.

#### Arrays try to look like value types

Strings, Arrays and Dictionaries in Swift are implemented as structs
because it is desirable to pass them around as values and not worry
about the contents of the Array or Dictionary getting modified behind
our back.

But instances of these structs can potentially hold a large amount of
data, and it's going to affect the performance of our code if Swift has
to copy all the bytes of a String or all the elements of an Array every
time we assigned it to a variable or passed it to a function.

To make copy performance of large value types manageable, Swift does
some magic under the hood.

When we create an Array, Swift allocates a buffer to hold the elements,
and stores a _pointer_ to this buffer in the Array structure (actually a
sub-struct of the Array struct, but what matters here is that only the
pointer is part of the data stored in the Array struct).

    var a1: Array<Int> = [10, 11]

<figure>
<a href="/images/swift-arrays/01.png"><img
   src="/images/swift-arrays/01.png" /></a>
</figure>

When we append an element, Swift reallocs the buffer to accomodate the
additional elements (typically rounded upwards to the nearest power of
2, in anticipation of future appends) and then adds the element to the
buffer.

    a1.append(20)

<figure>
<a href="/images/swift-arrays/02.png"><img
   src="/images/swift-arrays/02.png" /></a>
</figure>

Note that the realloc can either grow the allocated memory block, or can
result in a new block of memory to be allocated (followed by a memcopy
to the new location). In the latter case, the buffer pointer would have
to be updated to point to the new location.

Now the interesting part - assigning to another variable:

    var a2 = a1

<figure>
<a href="/images/swift-arrays/03.png"><img
   src="/images/swift-arrays/03.png" /></a>
</figure>

So, instead of copying the contents to the new Array instance (as a
normal value type would have done), Swift internally keeps the content
of both the Array instances in a single shared buffer.

The same also holds good for passing it to a function or closure.

    func arrayLength(arg: Array<Int>) -> Int {
        // How does it look here?
        return countElements(arg)
    }
    var l1 = arrayLength(a1)

Within the execution of the function, it looks like this:

<figure>
<a href="/images/swift-arrays/04.png"><img
   src="/images/swift-arrays/04.png" /></a>
</figure>

This way, assigning an Array to a variable and passing an Array to
functions will be very fast because it does not involve copying of the
contents of the Array.

However, we still want assign-by-value and pass-by-value semantics,
which is to say that when we modify `a2`, we want only `a2` to change,
and `a1` to remain intact.

To achieve this, just before modifying an Array whose data is in a
shared buffer, Swift `unshare()`s the buffer. That is, if the buffer
is shared with other Array instances, it creates a new buffer and copies
the contents to the new buffer; if the buffer is not shared, there's
nothing to do.

    a2.append(30)

<figure>
<a href="/images/swift-arrays/05.png"><img
   src="/images/swift-arrays/05.png" /></a>
</figure>

So, effectively, a copy is done only when shared data is about to be
modified.

This is the classic copy-on-write pattern (also called lazy copying, or
implicit sharing). Though not tied to any language, this is a popular
pattern in C++ (more so in Qt). In Cocoa, this pattern is used in the
internal implementations of some parts of [Core Data][core-data-cow] and
[Core Graphics][core-graphics-cow].

[core-data-cow]: https://developer.apple.com/library/mac/documentation/cocoa/conceptual/coredata/Articles/cdTechnologyOverview.html
[core-graphics-cow]: https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CGBitmapContext/Reference/reference.html#jumpTo_4

Aside: Note that to perform an `unshare()`, Swift needs to know whether
the data is in a shared buffer or an exclusive non-shared buffer. The
obvious way would be to use reference counting to track how many Array
instances point to a buffer, which is probably what Swift is using too.
The mechanics would be similar to ARC, but this is not ARC (Swift
structs are not ARC-ed). The reference count would also be handy to
determine when to deallocate the buffer.

#### Verifying the theory

According to the Swift runtime headers (that you get to see when you
Cmd-click on a keyword), Array has a `withUnsafePointerToElements`
function, which can call a passed closure with a pointer to the
underlying storage. This gives us a way to check whether two Array
instances use the same underlying storage or not.

Diving into code:

<script src="https://gist.github.com/roop/4f1af9b557d4ff39aabf.js"></script>

[(GitHub Gist)](https://gist.github.com/roop/4f1af9b557d4ff39aabf)

_Note: This seems to work in standalone Swift scripts and
XCode projects, but not in the Playground._

#### Revisiting the note

Now, let's parse [that note](#the-problem) again:

> Where copying is mentioned, the behavior
> you see in your code will always be as if a copy took place.

Externally, it will behave like a value type.

> However, Swift only performs an actual copy behind the scenes when it is
> absolutely necessary to do so.

Swift performs an actual copy only if the code requires modification of
the contents and if the contents are shared internally.

> Swift manages all value copying to
> ensure optimal performance

Swift will figure out when to copy (i.e. if you attempt to modify the
contents and if the contents are shared internally).

> and you should not avoid assignment to try
> to preempt this optimization.

Don't be scared of doing `var tmp = arrayWithThousandsOfItems` because it's
not as expensive as it looks.

#### The bug in Array copy-semantics

In a normal implementation of the copy-on-write pattern, the `unshare()`
would be triggered on any modification of the array contents, including
those that do not change the size of the array.

In Swift Arrays (as of XCode 6 Beta 2), only operations that change the
length of the Array seems to trigger an `unshare()`. This is the core
problem with Swift Arrays.

Since changing the value at a particular index does not trigger an
`unshare()`, the modification is done on the shared buffer, and hence
the original Array instance is also modified.

Chris Lattner has confirmed in multiple places that this will be fixed
in a future beta release.

#### Immutability of Arrays

A struct declared with `let` cannot have it's contained fields modified.
For an Array, this means that the buffer pointer cannot be modified.

Therefore, the following operations are disabled on `let` Arrays:

 1. Appending elements can potentially require a realloc, which can
    change the buffer pointer
 2. `unshare()`, which can change the buffer pointer
 3. Replacing a range of items with some items (either using
    `replaceRange` or like `a[1..5] = [42, 43]`), which can potentially
    require a realloc, and therefore can change the buffer pointer

Notably, changing the value at an index is still possible on `let`
Arrays, because at present, this does not change the buffer pointer.
So, the problem of immutable Arrays not being really immutable is a
direct result of the bug in Array copy-semantics.

### The solution

Chris Lattner has [confirmed][lattner-on-mike-ash] that "array semantics
are going to change significantly in later seeds, to be more similar to
dictionary and strings."

[lattner-on-mike-ash]: https://mikeash.com/pyblog/friday-qa-2014-06-20-interesting-swift-features.html#comment-2ba71511053a8017a556bc4ef9c091fa

So the solution Apple is going to give us in a later seed will ensure
that changing the value of an index will also trigger an `unshare()`.
That will make it really behave like a value type in all situations,
like in the case of Dictionary and String.

### A workaround

With a little tweak to the trick we used to examine the Array storage,
we can also override `subscript()` to call `unshare()` whenever we're
about to change the value at an index.

<script src="https://gist.github.com/roop/92bac4f644d2259d91fd.js"></script>

[(GitHub Gist)](https://gist.github.com/roop/92bac4f644d2259d91fd)

_Note: This workaround seems to work in standalone Swift scripts and
XCode projects, but not in the Playground._


And we have Arrays using copy-on-write correctly and behaving more
consistently with Dictionarys and Strings, just like how it is expected
to be once this is fixed in a later beta. This also prevents us from
being able to modify the value at an index of a `let` Array.

However, this workaround doesn't handle `sort()` modifying the Array
without `unshare()`-ing it. For that, we'll have to wait for the fix
from Apple.

If you have any feedback on my conclusions about Swift Arrays, I'd love
to discuss on Twitter ([@roopeshchander]).

[@roopeshchander]: http://www.twitter.com/roopeshchander

