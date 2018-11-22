---
layout: page
permalink: /work/qt-creator/index.html
---

<section markdown="1">

  <aside class="roop-intro">
  <p>{% include roop-intro.html %}</p>
  <p>Here are some of the things I've worked on.</p>
  </aside>

## Contribution to Qt Creator

While [consulting on Qt](/work/qt-consulting/), I was working on
multiple projects at once, and one of them had tab expansion turned off
(my preference is to always expand tabs). At that time, Qt Creator did
not have the ability to maintain per-project tab expansion settings.

I made an enhancement to Qt Creator to help me deal with this -- I [added
an option][commits] to automatically infer tab expansion based on the
surrounding code.

My contribution was enhanced by other contributors and eventually became
the ‘Mixed’ option under Settings > Text Editor > Behavior > Tab policy.

[commits]: <https://github.com/qt-creator/qt-creator/commits?author=roop>
