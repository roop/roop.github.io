---
layout: post
title: "Imagining developing for a split-screen iPad"
description: "What all would Apple have to keep in mind
              if it allows third-party apps to go
              split-screen on the iPad?"
category: posts
tags: [ios, ios7, multitasking, split-screen]
postscript: "This post was discussed in [Daring Fireball](http://daringfireball.net/linked/2014/05/28/split-screen-ipad),&nbsp; [iOS Dev Weekly](http://iosdevweekly.com/issues/148#code)&nbsp; and [Reddit](http://www.reddit.com/r/iOSProgramming/comments/26rv5b/imagining_developing_for_a_splitscreen_ipad/)."
---

Given [the rumour] that Apple is considering split-screen multitasking
for the iPad in iOS 8, I tried to imagine how that would work for
third-party apps.

[the rumour]: http://9to5mac.com/2014/05/13/apple-plans-to-match-microsoft-surface-with-split-screen-ipad-multitasking-in-ios-8/

### A new layout-design

First of all, a split screen would mean a new layout to design for.

[As Jared Sinclair explains], AutoLayout will help, but will not
automagically make an app support a split-screen layout. For an app to
remain good-looking and snappy, "split-screen" has to be a separate mode
(like Landscape and Portrait) that apps have to be specially
layout-designed for.

[As Jared Sinclair explains]: http://blog.jaredsinclair.com/post/85635304505/

### A new API

Therefore, the only way split-screen multitasking would be possible for
third-party apps is by an opt-in API. Only apps that know how to layout
their screen in a split-screen mode shall use the API and let iOS know
about the app's willingness to participate in a split-screen mode. When
the user wants to add an app to a split screen, iOS can list only the
apps that have opted-in.

### Design considerations

Apps that opt-in for the split-screen mode would have more than just a
new layout design to worry about. They would at least have to consider
three additional stuff: the status bar, the keyboard and gestures from the
edge.

#### Status bar

Before iOS 7, the status bar had a fixed look and was visually separated
from the content.

<figure>
<a href="{{ site.url }}/images/imagining-split-screen-ipad/ios5statusbar.png"
     title="Status bar before iOS 7"><img
     src="{{ site.url }}/images/imagining-split-screen-ipad/ios5statusbar.png" /></a>
<figcaption>
Status bar for iPad Safari before iOS 7.
</figcaption>
</figure>

Starting with iOS 7, we have a transparent status bar that blends in
with the app's toolbar or content. An app can dynamically choose the
visibility and style for the status bar, based on the look of what lies
next to the status bar.

<figure>
<a href="{{ site.url }}/images/imagining-split-screen-ipad/ios7statusbar.png"
     title="Light status bar in iOS 7"><img
     src="{{ site.url }}/images/imagining-split-screen-ipad/ios7statusbar.png"
     style="margin-top: 1.5em" /></a>
<a href="{{ site.url }}/images/imagining-split-screen-ipad/ios7statusbar_dark.png"
     title="Dark status bar in iOS 7"><img
     src="{{ site.url }}/images/imagining-split-screen-ipad/ios7statusbar_dark.png"
     style="margin-top: 1.5em" /></a>
<figcaption>
Status bar for iPad Safari in iOS 7. Default (top) and in private mode
(bottom).
</figcaption>
</figure>

In the split-screen mode, the two apps on either half of the screen
_might_ give conflicting requests to iOS for status bar properties. There
are two ways I can think of in which iOS could handle that scenario:

When in split screen mode:

  1. Force apps to stick to a single status bar style (probably dark
     text on light background) _or_
  2. Revert to the pre-iOS-7 look of a visually separate status bar

The mockups I have seen all seem to assume the second method and feature an
un-iOS-7-ish visually separated status bar.

<figure>
<a href="{{ site.url }}/images/imagining-split-screen-ipad/ios8_mockup_9to5mac.png"
     title="iOS 8 mockup by Michael Steeber (9to5Mac)"><img
     src="{{ site.url }}/images/imagining-split-screen-ipad/ios8_mockup_9to5mac.png"
     style="margin-top: 1em" /></a>
<figcaption>
Mockup for iOS 8 split-screen multitasking by Michael Steeber
<a href="http://9to5mac.com/2014/05/13/apple-plans-to-match-microsoft-surface-with-split-screen-ipad-multitasking-in-ios-8/">(9to5Mac)</a>
</figcaption>
</figure>

<figure>
<a href="{{ site.url }}/images/imagining-split-screen-ipad/ios8_mockup_sambeckett.png"
     title="iOS 8 mockup by Sam Beckett"><img
     src="{{ site.url }}/images/imagining-split-screen-ipad/ios8_mockup_sambeckett.png"
     style="margin-top: 1em" /></a>
<figcaption>
From “iOS 8 Concept - Split Screen multitasking” by Sam Beckett
<a href="https://www.youtube.com/watch?v=_H6g-UpsSi8">(Youtube)</a>
</figcaption>
</figure>

But then, it would be unlike Apple to go back on a design feature they just
introduced last year.

Either way, apps will probably not be given a choice of status bar style
in the split mode, and developers will have to design around this
limitation.

#### On-screen keyboard

Say we're typing a document in Pages and have Safari opened on the
left-half of the screen for reference, like in this mockup by
9to5Mac:

<figure>
<a href="{{ site.url }}/images/imagining-split-screen-ipad/safari_pages_9to5mac_1.png">
<img src="{{ site.url }}/images/imagining-split-screen-ipad/safari_pages_9to5mac_1.png"
     title="Safari + Pages"
     />
</a>
</figure>

But when we're typing, it's going to look more like this:

<figure>
<a href="{{ site.url }}/images/imagining-split-screen-ipad/safari_pages_9to5mac_2.png">
<img src="{{ site.url }}/images/imagining-split-screen-ipad/safari_pages_9to5mac_2.png"
     title="Safari + Pages"
     />
</a>
</figure>

For split-screen multitasking to be really useful, the webpage on
the left should be scrollable so that the end of the page is visible
above the on-screen keyboard. (Otherwise, we will have to keep
dismissing the keyboard to see the content.) To do that, Safari
would have to adjust its views when the keyboard is opened, even if the
keyboard is triggered by _another app_.

iOS already notifies apps when the keyboard opens and closes, so the
same API can be used for this purpose. An app that currently resizes
its content views to allow for the on-screen keyboard (like Safari)
_might_ work as intended without having to make any changes with respect
to this.

But there are apps with screens that are designed to be scrolled
horizontally, not vertically. Like iBooks and Flipboard. Those apps can
afford to do that because they don't have to handle the opening of a
keyboard in those screens. For apps like that to participate in the
split-screen mode, they might have to freshly implement vertical
scrolling. Only then will split-screen multitasking be useful for the
scenario of refer-in-one-half and write-in-another-half.

So, if an app has a screen that doesn't scroll vertically, the UX for
those screens might have to be redesigned.

Of course, with an external keyboard, this problem almost disappears.
(Only almost, because App1 still has to deal with any input accessory
views of App2 being shown at the bottom of the screen.)

#### Swiping from the edge

This is not that big a deal, but still worth mentioning.

Some apps do something specific when the user swipes from the right
or left edge of the screen, like [Paper]. In the split-screen mode,
the swipe might originate in the "other-half" app, and so the
swipe-from-the-edge gesture might never be delivered to our app.

An app looking for the swipe-from-edge gesture would have to expose
the same functionality using other interactions (which is the right
thing to do anyway, irrespective of the "split-screen" mode).

[Paper]: https://www.fiftythree.com/paper "Paper by FiftyThree"

### My bet

The case of the status bar might look like a small thing, but I take
that as a significant hint. The iOS 7 transparent status bar works best
with one app on the screen at a time. Apple being the company that
thinks through well before shipping a design, it's unlikely that they
set the status bar transparent in iOS 7 and change course in the next
major version.

My bet is that we are not going to see split-screen mutitasking being
allowed for third-party apps in iOS 8, at least initially. If at all
we're going to get this feature, it would probably be restricted to some
of the Apple apps.

