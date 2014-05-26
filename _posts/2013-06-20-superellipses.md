---
layout: post
title: "Superellipses?"
description: "No, of course not. It's <del>still rounded rects</del> <ins>something else</ins> in iOS 7."
tagline: "No, of course not."
category: posts
tags: [ios7]
---

[Mark Edwards][marcedwards superellipse tweet] and 
[Rene Ritchie][imore superellipse article] think
that iOS 7 icons are shaped as superellipses. Apparently, when you plot
the function for a superellipse in Grapher and tweak the values, you get
a shape that looks a lot like the iOS 7 icon's shape. The problem with
this methodology is that superellipses do tend to look a lot like
rounded rects, so it's hard to say for sure without zooming in.

[marcedwards superellipse tweet]: https://twitter.com/marcedwards/status/347451374214213633
[imore superellipse article]: http://www.imore.com/cracking-ios-7-icon-superellipse-formula

The bounding rect of any iOS 7 icon is a square (and that's true of the
app icon template of any respectable mobile OS), so we need to consider
only supercircles as candidates. (A supercircle is just a superellipse
whose bounding rect is a square.)

We are only concerned with the _shape_ of the supercircle, so let's forget
about position and size. Let's consider only
supercircles centered at the origin and having a unit radius (i.e. width
is 2 units). The general equation for such supercircles is:

<img src="/images/superellipses/general_supercircle_equation.png"
title="General supercircle equation" />

At `n=2`, it's just a plain old circle. As `n` increases, the shape
becomes more and more like a square (hence the name squircle at `n=4`).

<figure>
<img
src="/images/superellipses/supercircles_at_different_n.png"
title="General supercircle equation" />
<figcaption>
Supercircles at n=2, n=3, n=4, n=5, n=6 and n=7. As n increases, the shape expands.
</figcaption>
</figure>

At higher values of `n`, it looks remarkably like a rounded rect. I
suspect that is the reason for the confusion about iOS 7 icons being
shaped as superellipses.

If we look at the iOS 7 Mail icon close enough and compare it with a
plot of the superellipse formula that Mark Edwards 
[came up with][marcedwards superellipse tweet], we see
that it doesn't quite match up exactly.

<figure>
<a href="/images/superellipses/ios7_icon_vs_squircle.png">
    <img
    src="/images/superellipses/ios7_icon_vs_squircle.png"
    title="iOS 7 Mail icon vs Supercircle" />
</a>
<figcaption>
The iOS 7 Mail icon vs Supercircle
</figcaption>
</figure>

In fact, it seems to match up better with a rounded rectangle, but one
with a greater rounding radius than an iOS 6 icon.

(**Update 22/Jun/2013**: Yes, it matches a little better with a
roundrect, but it still doesn't match _exactly_.)

<figure>
<a href="/images/superellipses/ios7_icon_vs_roundrect.png">
    <img
    src="/images/superellipses/ios7_icon_vs_roundrect.png"
    title="iOS 7 Mail icon vs Rounded Rect" />
</a>
<figcaption>
The iOS 7 Mail icon vs Rounded rect
</figcaption>
</figure>

Here's a close-up of both the rounded rect and the supercircle overlaid
on the icon. That's how close the squircle is to a rounded rect.

<figure>
<a
href="/images/superellipses/ios7_icon_vs_roundrect_vs_squircle_closeup.png">
    <img
    src="/images/superellipses/ios7_icon_vs_roundrect_vs_squircle_closeup.png"
    title="iOS 7 Mail icon vs Rounded Rect vs Squircle" />
</a>
<figcaption>
Icon vs Rounded Rect (purple) Vs Supercircle
</figcaption>
</figure>

So, it looks like it's <del>still a rounded rect in iOS 7</del> 
(**See update below**). Apple is not going
to give up on it's iconic icon shape so soon. And anyway, it's very
unlikely that Apple would take to a shape that is really [owned by the
Nokia brand][own a shape].

[own a shape]: http://interuserface.net/2011/06/own-a-shape/

**Update 22/Jun/2013**: Rene Ritchie [pointed me][reneritchie wwdc tweet] to
the WWDC session that made it pretty clear that it was not a round
rect. Nevertheless, it doesn't look like a superellipse to me still. The
hunt [is on][marcedwards not-there-yet tweet].

[reneritchie wwdc tweet]: https://twitter.com/reneritchie/status/347755614866386944
[marcedwards not-there-yet tweet]: https://twitter.com/marcedwards/status/348303855152410625

