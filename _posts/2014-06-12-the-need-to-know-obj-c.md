---
layout: post
title: "The need to know Objective-C"
description: "If you want to be an iOS dev, should you start with Objective-C
              or Swift?"
category: posts
---

Aaron Hillegass, of Big Nerd Ranch, [says][AH]:

> When Apple announced Swift, I heard a few people say “Hurray! Now I
> can be an iOS developer without learning Objective-C!” I have three
> messages for these people:
>
>  - If you want to be an iOS developer, you will still need to know
>    Objective-C.
>  - Objective-C is easier to learn than Swift.
>  - Once you know Objective-C, it will be easy to learn Swift.

[AH]: http://www.bignerdranch.com/blog/ios-developers-need-to-know-objective-c/

I disagree.

### Rockstars vs. Newbies

While a rockstar iOS developer would know both Swift and Objective-C,
the same wouldn't apply to someone who says to himself "Now I can be an
iOS developer without learning Objective-C", who is most likely just
starting on iOS development.

For someone new to the iOS dev world, I would recommend starting out
with Swift rather than Objective-C.

Looking at Aaron Hillegass' arguments one by one:

  - _"If you want to be an iOS developer, you will still need to know
    Objective-C."_

    Agreed, but only if you want to be an iOS developer of a
    super-advanced app right away. As someone starting out with iOS
    development, you most likely don't need to talk to C/C++ code or
    swizzle methods or meddle with the Objective-C runtime. You can get
    to that when you need to.

    I certainly agree that "the community talks in Objective-C" in blogs
    and in Stack Overflow, but that's going to change faster than we
    think, especially with [Apple being okay about discussing beta stuff
    publicly][oleb].

  - _"Objective-C is easier to learn than Swift."_

    This is subjective. To me, Swift code looks more readable than
    Objective-C, and therefore, easier to pick up for a newbie.

    Swift does have some seemingly odd behaviours (like copy semantics
    of Arrays), but I expect those quirks to either go away or get
    explained in the near future.

  - _"Once you know Objective-C, it will be easy to learn Swift."_

    I totally agree with this. However, this cannot be a reason to start
    off learning iOS development with Objective-C rather than Swift.

[oleb]: http://oleb.net/blog/2014/06/apple-lifted-beta-nda/

### Swift is the future

I would argue in favour of Swift because it's the future of iOS and Mac
development.

Almost anyone starting a new project is going to use Swift. If you
intend to be an independent developer once you learn iOS development,
it's a no-brainer to start off with Swift.

While most iOS dev blogs talk in Objective-C at present, a couple of
months later, most of the new snippets that get published there are
going to be in Swift.

Most importantly, I believe Swift is going to give us completely new
design patterns that wouldn't be possible in Objective-C. For one, I
think KVO is going to look quite different in Swift, thanks to closures
and property observers. The Objective-C-way will start to look dated in
comparison.

It's a good idea to skate to where the puck is going to be.
