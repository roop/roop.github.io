---
layout: post
title: "The Symbian Experience"
tagline: "As a user and as a developer"
description: "As Symbian phones are about to stop shipping this summer, a look back at the last few years of the OS, and some opinions on why it couldn't make it."
category: posts
tags: [symbian, nokia]
postscript: "This post is based on my inputs to TechCrunch for their [article on Symbian](http://techcrunch.com/2013/06/13/rip-symbian/)."
---

My first go at developing on Symbian was with Symbian C++, and
that wasn't a nice experience for me, or my 
[co-developer] [Girish]. Even doing simple things required one to think
about [low level stuff] [SymbianUtopia], since the whole
system was designed for those really memory-constrained devices of the
previous generation.  Classes and methods were named arbitrarily, and the
convention of adding prefixes and suffixes to the names to impart
information only made it worse. The net effect was that it was hard to
read and understand code, sometimes even our own code.

[Girish]: http://blog.forwardbias.in/category/girish
[SymbianUtopia]: http://www.theregister.co.uk/2010/11/03/symbian_utopia_lost/

Around that time, I had just started learning Qt, and compared to Qt,
Symbian C++ was a nightmare.  While having a [clean API] [QtAPIDesign]
was always a focus for Qt, it appears to me that Symbian C++ [ignored it
completely] [SymbianDevsDevsDevs].

[QtAPIDesign]: http://doc.qt.digia.com/qq/qq13-apis.html
[SymbianDevsDevsDevs]: http://www.theregister.co.uk/2010/11/09/symbian_developers_mailbag/

The game shifted gears when Nokia bought Trolltech in Jan 2008, and 
[within a few months] [EspenOnQtForS60] of the acquisition, started an
effort to port Qt to Symbian. At that time, the first-gen iPhone was
shipping, and was doing great. The second-gen iPhone launched that July,
with the App Store. Later that year, the first Android phone launched.
The era of touch-centric phones had begun.

[EspenOnQtForS60]: http://web.archive.org/web/20090716233252/http://labs.trolltech.com/blogs/2008/10/20/were-porting-qt-to-s60

Presumably, around that time, Nokia too were looking at going big on
touchscreen phones. As of 2009, the Qt guys at Nokia were 
[experimenting with QML] [QtDeclarativeUIIntro], a new language to
create fluid user-interface elements. That later evolved into 
[Qt Quick][], a UI technology that made it really easy to
make touch-centric apps for the new touchscreen phones running
Symbian^3.  Once Qt Quick and it's UI components became available in
Symbian in mid-2011, the number of modern-looking apps on Symbian
increased significantly.

[QtDeclarativeUIIntro]: http://blog.qt.digia.com/blog/2009/05/13/qt-declarative-ui/
[Qt Quick]: http://qt-project.org/wiki/Qt_Quick

Though the apps started looking nice and modern, the UI of the OS itself
(Symbian^3 and Symbian Anna) was still looking dated. For example,
though running on a touchscreen, they had two text buttons in the bottom
taskbar, a legacy from the buttoned phones. The OS UI now supported
different orientations, but when the phone orientation
changed, weird things happened on the screen before the screen settled
in the new orientation. Till Symbian Anna, if you wanted to type in
portrait mode, you had to make do with a telephone keypad (that has 2
and ABC on the same button). Stuff like these were unacceptable
at a time when iOS and Android were offering much better
touch-centric user interfaces, and were made for touch from the
ground-up.

The Symbian user experience did have it's own unique pluses compared to
iOS / Android. The alarm in Symbian worked even when switched off. 
The always-on low-power notifications screen worked brilliantly with
AMOLED displays. And the power consumption was consistently frugal. But
the main problem, that the user interface still looked like it was from
a pre-iPhone era, remained.

Nokia fixed most of those issues with Nokia Belle, introduced in late
2011, but it was too late. And, Belle merely brought the smartphone user
experience on par with the competition, didn't significantly elevate it
to a higher level. The best Symbian experience ever was on the flagship
Nokia 808, and I think the UX in the 808 can just about compete with the
UX in iOS/Android. And that's forgetting about apps and ecosystem for a
moment.

In my opinion, Symbian's downfall was that it couldn't
adapt itself to a touch-centric user experience quick enough. If Nokia
had been able to ship Belle two years earlier than it actually did, it
would've atleast had a fair chance at competing with iOS and Android.

